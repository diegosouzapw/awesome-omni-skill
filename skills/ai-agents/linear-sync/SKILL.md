---
name: linear-sync
description: This skill should be used when the session-start hook injects Linear context (e.g., "[Linear/..." or "[LINEAR-SETUP]" or "[LINEAR-DIGEST]"), when the prompt-check hook injects "[Linear/<workspace>] Issue(s) referenced:", when the commit guard hook blocks a command for missing an issue ID or injects "[CROSS-ISSUE-COMMITS]", when the user mentions Linear issues (e.g., "ENG-123", "OPL-456"), when creating commits/branches/PRs in a Linear-linked repo, when the user asks about Linear workflow or issue tracking, or when working in any repository that has a .claude/linear-sync.json config file. Provides behavioral rules for Linear-GitHub sync workflow orchestration.
version: 0.1.5
---

# Linear Sync Workflow

This system keeps Linear and GitHub in sync automatically. Linear API queries go through the `linear-sync` subagent (foreground), while simple mutations use `linear-api.sh` directly via Bash.

## Session Start Behavior

At the start of every session, the `linear-session-start` hook fires and checks the repo:

- **Linked repo**: The hook injects config context (workspace, project, team, label, formats, last_issue if any, stale branch warnings, and pre-fetched digest if due). Follow the session kickoff directive below.
- **Opted-out repo** (`workspace: "none"`): The hook is silent. Do nothing related to Linear.
- **Unregistered repo**: The hook injects a `[LINEAR-SETUP]` directive. Follow the setup wizard rules below. If a `.linear-sync-template.json` is found, the hook also injects `[LINEAR-TEMPLATE]` with defaults to pre-fill.
- **Broken workspace reference**: The hook injects a `[LINEAR-SETUP]` directive to reconfigure or opt out.

## Session Kickoff (Linked Repos)

When the hook injects config context for a linked repo:

**Step 1: Digest and stale branches**

- **Notification digest** (if `[LINEAR-DIGEST]` data is present in hook context): The session-start hook pre-fetches your active issues inline. Show the digest briefly (1 line) before asking what to work on.
- **GitHub issue sync**: Runs automatically via PostToolUse hook after `git push` or `gh pr create` — no action needed at session start.
- **Stale branches** (if `[STALE-BRANCHES]` is present): Show a brief warning. Example: "Heads up: branch `dan/ENG-100-old-feature` has had no commits in 7 days."

**Step 2: Resume or ask**

If the hook context includes `last_issue: <ISSUE_ID>`, offer to resume that issue first:

Use AskUserQuestion: "What are you working on today in <repo>?"
1. **"Resume <ISSUE_ID>"** — Delegate to `linear-sync` subagent (foreground) to fetch the issue summary. After fetching, save `last_issue` for this repo (direct Bash — see Execution Model). Check for blockers in the response and warn if any.
2. **"Work on a different issue"** — Delegate to `linear-sync` subagent (foreground) to fetch the dev's assigned in-progress issues ("Fetch My Issues" task). Present the returned list as AskUserQuestion choices, plus an option to "Enter an issue ID manually". After selection, save `last_issue` for this repo (direct Bash). Check for blockers in the response and warn if any.
3. **"Start something new"** — Ask for a one-line description via AskUserQuestion. Check for duplicates first (see Duplicate Detection below). Then delegate to `linear-sync` subagent (foreground) to create a ticket in the correct project with the repo label. After creation, offer to assign to current cycle (see Cycle Assignment below). Use the returned issue ID going forward.
4. **"Just exploring / no ticket needed"** — Proceed normally. The commit guard hook will catch untagged commits later and you can offer to create a ticket at that point.

If no `last_issue` is present, skip option 1 and start with options 2-4 (renumber accordingly).

**When the dev picks any issue (resume, existing, or new):** If the issue is unassigned, auto-assign it directly via `linear-api.sh` (see Execution Model — Direct Bash). No need to ask — if they're working on it, they should own it. **Parallel optimization:** Run `query { viewer { id } }` in the same message as the subagent fetch so the viewer ID is ready when the issue details come back.

Keep the kickoff brief and natural: "What are you working on today in <repo>?"

## Setup Wizard Rules

When a `[LINEAR-SETUP]` directive is injected:

1. **Always use AskUserQuestion** for every choice. The dev never types workspace names, project IDs, or labels manually.
2. **Always offer "This repo doesn't use Linear"** as an option. If chosen, opt the repo out via Read/Write tools (read state file, set `workspace: "none"`, write back) and move on.
3. Use the `linear-sync` subagent in **foreground** mode to fetch real data from Linear MCP (workspaces, teams, projects, labels).
4. Present fetched data as AskUserQuestion choices.
5. For labels, suggest `repo:<repo-name>` as the default.
6. Detect the GitHub org from the repo's remote URL. Include it as `github_org` in the committed config.
7. Write `.claude/linear-sync.json` to the repo root with the shared config.
8. Write only the `workspace` reference to the local state file for credential routing. Project, team, and label live in the repo-level config.
9. The subagent will commit and push `.claude/linear-sync.json` as part of the Link Repo task.
10. After setup, confirm in one line and proceed with the session kickoff.

### Project Rules

- Multiple repos can share the same project. Include "Create a new project" as the last option.
- If the dev picks "Create a new project", ask for a name, delegate to the subagent to create it.

## Enforcement Rules (Linked Repos Only)

These conventions are mechanically enforced by hooks. They apply ONLY to repos with a valid workspace link.

| Action | Requirement | Enforcement |
|--------|------------|-------------|
| `git commit -m "..."` | Issue ID in message (e.g., `ENG-123: fix bug`) | Hook blocks with exit 2 |
| `git checkout -b` / `git switch -c` | Issue ID in branch name (e.g., `ENG-123-fix-bug`) | Hook blocks with exit 2 |
| `gh pr create --title "..."` | Issue ID in PR title (e.g., `ENG-123: Fix the bug`) | Hook blocks with exit 2 |
| `git push` | Cross-issue commit check | Hook allows with advisory context |
| Issue creation | Repo label auto-applied and auto-created if missing | Subagent handles |
| Issue creation | Check for duplicates first | Subagent search + AskUserQuestion |
| Issue creation | Infer priority from keywords | AskUserQuestion if detected |
| Issue creation | Offer cycle/sprint assignment | AskUserQuestion if active cycle |
| Issue picked / resumed | Auto-assign if unassigned | Direct Bash (`linear-api.sh`) |
| `gh pr create` (after success) | **Offer** a progress comment on the Linear issue | PostToolUse hook reminder + AskUserQuestion |
| `git push` (final push) | **Offer** a progress comment on the Linear issue | PostToolUse hook reminder + AskUserQuestion |
| Session ending / major milestone | **Offer** a progress comment on the Linear issue | AskUserQuestion |

## Linear Comments

**IMPORTANT: You MUST offer a comment at every natural stopping point.** Never post automatically, but always ask.

The PostToolUse hook automatically injects a `[Linear Comment Reminder]` after `git push` and `gh pr create` in linked repos. When you see this reminder, follow the comment flow below.

Natural stopping points (offer every time):
- **After creating a PR** — this is the most important one
- **After a significant push** when wrapping up a chunk of work
- **When the dev says they're done** or is ending the session

**Smart comment drafts:** Before drafting a comment, run `git log main..HEAD --oneline` (or the appropriate base branch) to gather the commit history for this branch. Use the commit messages to auto-draft a concise progress summary.

Use AskUserQuestion with a draft comment:

    I can add a quick update to ENG-423:

    "Added sliding window rate limiter (3 commits):
     - Redis-backed per-user rate limiting
     - Configurable window size and limits
     - 94% test coverage
     PR: #142"

    1. Post this comment
    2. Let me edit it
    3. Skip

If they pick "edit", ask them what they'd like it to say. Never post without showing them exactly what will be posted first. If they skip, don't ask again until the next stopping point.

**Posting comments:** Use direct Bash with `linear-api.sh` (see Execution Model — no subagent needed for simple mutations).

## Issue Status

**If the dev explicitly asks to change a status, do it.** No pushback, no warnings. Use direct Bash with `linear-api.sh` and confirm.

For **automatic** status changes, only act at these moments:
- **"Start something new"** — Set to **In Progress** on creation.
- **Blocked commit recovery** — Set to **In Progress** on creation.

Don't automatically change status beyond those two cases. The Linear <-> GitHub integration handles the rest.

## Commit Message Formatting

Commits in linked repos must include an issue ID (e.g., `ENG-123: description`). Do NOT append `Co-Authored-By` lines to commit messages. Keep commit messages clean: just the issue ID prefix and a concise description.

## Branch Naming

Auto-generate branch names from the issue ID and title. Slugify and truncate the description portion to keep the total branch name under 50 characters.

Examples:
- `alice/ENG-456-add-sliding-window-rate`
- `alice/CP-89-fix-auth-timeout-mobile`

Keep the meaningful words, drop filler. The dev never manually names branches in a linked repo.

## Blocked Commit Recovery

When a commit, branch, or PR is blocked by the hook for missing an issue ID:

1. **Proactively offer to create a Linear ticket.** Do not just report the error.
2. Use AskUserQuestion to ask for a one-line description of the work.
3. Delegate to `linear-sync` subagent (foreground) to create the ticket.
4. Retry the original command with the new issue ID inserted.
5. Do not make the dev start over or re-type their commit message.

## Duplicate Issue Detection

Before creating any new issue:

1. Delegate to `linear-sync` subagent (foreground) with the "Search Issues" task.
2. If duplicates found, present them via AskUserQuestion with option to use existing or create anyway.
3. If no duplicates, proceed with creation normally.

## Blocking Issue Warnings

When an issue is fetched, check for blocker warnings. If blockers are present:
1. Show the warning: "Heads up: this issue is blocked by ENG-100."
2. Do NOT prevent the dev from working on it. It's advisory.
3. Use AskUserQuestion: "Work on it anyway? / Switch to the blocking issue instead? / Pick a different issue?"

## PR Auto-Description

When `gh pr create` is about to run and you have issue context:
1. Draft the PR body using issue title, description, and acceptance criteria.
2. Structure the PR body as:
   ```
   ## Summary
   <1-2 sentence summary from issue context>

   ## Linear Issue
   <ISSUE_ID>: <title>

   ## Changes
   <bullet points from `git log main..HEAD --oneline`>

   ## Acceptance Criteria
   <from issue description, or "See Linear issue" if none>
   ```
3. The issue ID must still appear in the `--title`.

## Cycle/Sprint Auto-Assignment

When creating a new issue:
1. After creation, delegate to `linear-sync` subagent (foreground) to fetch the active cycle for the team.
2. If an active cycle exists, use AskUserQuestion to offer adding the issue to it.
3. If no active cycle, skip silently.
4. To add: use direct Bash with `linear-api.sh` (no subagent needed).

## Priority Inference

Scan descriptions for urgency signals:

| Keywords | Suggested Priority |
|----------|-------------------|
| `urgent`, `asap`, `critical`, `emergency`, `p0`, `hotfix`, `production down`, `outage` | 1 (Urgent) |
| `important`, `high priority`, `p1`, `blocker`, `blocking`, `regression` | 2 (High) |
| `bug`, `fix`, `broken`, `error`, `p2` | 3 (Medium) |
| `nice to have`, `low priority`, `p3`, `p4`, `when possible`, `minor`, `cleanup`, `chore`, `refactor` | 4 (Low) |

If keywords detected, use AskUserQuestion to confirm. If no keywords match, create without priority.

## Cross-Issue Commit Warning

If the commit guard hook injects a `[CROSS-ISSUE-COMMITS]` advisory on push, show it: "Advisory: This branch has commits referencing different issues (ENG-123, ENG-456). This is usually fine for related work, but you may want to split into separate branches if the work is unrelated." This is a warning, not a block.

## Closed Issues Without PRs

If the session-start digest mentions closed issues without linked PRs, surface it briefly: "Note: ENG-123 was marked Done but has no linked PR." This is informational only.

## Team Config Templates

If a `.linear-sync-template.json` exists in the repo root and the hook injects `[LINEAR-TEMPLATE]`, use its values as defaults during the setup wizard. Present them for confirmation: "Found a team config template. Use these defaults? Workspace: X, Project: Y, Team: Z, Label: W". The dev can override any value.

## Execution Model

Operations use three tiers to avoid context bloat and work around background subagent permission limitations:

### Hooks (zero context cost)
- **GitHub sync** — PostToolUse hook runs `sync-github-issues.sh` after `git push` / `gh pr create` in linked repos. Injects `[Linear Comment Reminder]` with the result.
- **Notification digest** — SessionStart hook pre-fetches active issues inline via `linear-api.sh`. Result appears in `[LINEAR-DIGEST]` context.

### Foreground subagent (`linear-sync`)
Use for operations that return data the main agent needs to present:
- **Fetch Issue Summary** — issue details + blocker warnings
- **Fetch My Issues** — assigned in-progress issues list
- **Create Issue** — creates ticket, returns issue ID
- **Search Issues** — duplicate detection before creation
- **Fetch Active Cycle** — cycle info for assignment offer
- **Link Repo** — setup wizard (one-time)

**When delegating to the subagent**, always include the `scripts_dir` from the session-start hook context in your prompt so the subagent can find `linear-api.sh`. Example: "Search for duplicates in OPL project. scripts_dir: /path/to/scripts"

### Direct Bash (auto-approved by PreToolUse hook)
Use `linear-api.sh` at the path from the session-start hook's `scripts_dir` field for Linear API mutations. A PreToolUse hook auto-approves single-line `bash */linear-api.sh` commands (and multiline with variable assignments), so these run without permission prompts:
- **Auto-assign**: `bash linear-api.sh 'query { viewer { id } }'` then `bash linear-api.sh 'mutation { issueUpdate(id: "...", input: { assigneeId: "..." }) { issue { id } } }'`
- **Add to cycle**: `bash linear-api.sh 'mutation { issueUpdate(id: "...", input: { cycleId: "..." }) { issue { id } } }'`
- **Post comment**: See the `!` escaping workaround below, then: `bash linear-api.sh "$QUERY" '{"input": {"issueId": "...", "body": "..."}}'`
- **Status change**: `bash linear-api.sh 'mutation { issueUpdate(id: "...", input: { stateId: "..." }) { issue { id } } }'`

### CRITICAL: `!` escaping in GraphQL queries
The Bash tool escapes `!` to `\!` even inside single quotes, breaking GraphQL non-null types like `CommentCreateInput!`. **Always use printf on a separate line to construct queries containing `!`:**
```bash
QUERY=$(printf 'mutation($input: CommentCreateInput%s) { commentCreate(input: $input) { comment { id } } }' '!')
bash linear-api.sh "$QUERY" '{"input": {"issueId": "...", "body": "..."}}'
```
**Important**: Put `QUERY=` and `bash linear-api.sh` on **separate lines** (not chained with `&&`). The auto-approve hook validates each line independently and rejects `&&` chains for security.

This applies to **any** query with `!` (e.g., `IssueCreateInput!`, `CommentCreateInput!`, `String!`). Queries without `!` (simple queries, `issueUpdate`, `viewer`) can use the normal single-line syntax.

### Parallel execution
**Always batch independent API calls in the same message.** Claude can make multiple tool calls simultaneously when they don't depend on each other. Key patterns:
- **Issue resume + auto-assign**: Run the subagent fetch AND `query { viewer { id } }` in the same message. When both return, you have the issue UUID and viewer ID — fire the assign mutation immediately.
- **Multiple independent mutations**: If you need to assign + add to cycle + change status, batch all three mutations in one message.
- **State update + API call**: Read/Write to the state file can happen in parallel with an API mutation since they're independent operations.

**Sequential only when dependent**: A mutation that needs data from a query (e.g., assign needs viewer ID) must wait for the query to complete first. But two independent queries should always run in parallel.

### Read/Write tools (no Bash needed)
Use the Read and Write tools for state file updates — no permission prompts, no subagent overhead:
- **Save last_issue**: Read `~/.claude/linear-sync/state.json`, update `repos.<repo>.last_issue`, Write back
- **Opt repo out**: Read state file, set `repos.<repo>.workspace` to `"none"`, Write back

## Companion Files

This skill orchestrates components at the plugin root (path injected by session-start hook as `scripts_dir`):
- **Subagent**: `agents/linear-sync.md` — handles Linear API queries (foreground only)
- **Hooks**: `hooks/scripts/linear-session-start.sh` (session init + inline digest), `linear-commit-guard.sh` (commit/branch/PR enforcement), `linear-prompt-check.sh` (issue reference injection), `linear-post-push-sync.sh` (GitHub sync + comment reminder)
- **Scripts**: `scripts/linear-api.sh` (API wrapper), `scripts/sync-github-issues.sh` (GH sync)
- **State**: `~/.claude/linear-sync/state.json` (credential routing, session state)
