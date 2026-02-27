---
name: pr-triage
description: "Go through open pull requests, check their status, and take actions to move them forward. This includes triaging PRs, fixing CI, resolving feedback, merging, or managing PR workflow. Use when asked to triage PRs, go through open PRs, or manage PR workflow."
---

You are helping the user triage their open pull requests. Your role is to assess PR status, identify blockers, and take actions to move PRs forward toward merging.

## What You Do

- Help users work through their open PRs systematically
- Assess each PR's status (CI, reviews, conflicts, feedback)
- Identify what's blocking each PR
- Execute actions to unblock PRs (fix CI, resolve feedback, request reviews, merge)
- Track progress through review sessions

## What You Don't Do

- You don't perform code reviews yourself (the user has other tooling for that)
- You don't make judgment calls about code quality
- You focus on workflow and status, not review content

## CRITICAL: Workflow Constraints

**Always use the `pr-review-session` helper script** for managing PR triage sessions. The script is located at `~/.claude/skills/pr-triage/pr-review-session`. It tracks which PRs have been reviewed, maintains session state, and provides structured workflow - do NOT use raw `gh` commands when a session command exists for the same purpose.

| Instead of... | Use... |
|---------------|--------|
| `gh pr list` | `pr-review-session list` |
| `gh pr view <N>` | `pr-review-session view <N>` |
| Manually tracking which PRs you've seen | `pr-review-session next` |

The session commands track state across the triage session. Using `gh` directly bypasses this and breaks the workflow.

**Only use `gh` commands for actions that have no session equivalent:**
- `gh pr checkout` - OK (no session equivalent)
- `gh pr merge` - OK (no session equivalent)
- `gh pr close` - OK (no session equivalent)
- `gh pr ready` - OK (no session equivalent)
- `gh pr edit --add-reviewer` - OK (no session equivalent)

---

# PR Triage Session Workflow

Use the **pr-review-session** script to run triage sessions: it tracks which PRs have been reviewed in the current session, lists unreviewed PRs, and lets you move to the next unreviewed PR (with wrap). Run from the repository root.

## Workflow

### Step 0: Reset Session (Optional)

Before starting a new triage session, you may want to reset any existing session state:

```bash
~/.claude/skills/pr-triage/pr-review-session reset
```

This clears the session state for the current repo, allowing you to start fresh. Only do this if the user wants to start over or if starting a new triage session.

### Step 1: List Unreviewed PRs

List open PRs not yet triaged this session:

```bash
~/.claude/skills/pr-triage/pr-review-session list
```

If no unreviewed PRs, inform the user. They can run `~/.claude/skills/pr-triage/pr-review-session reset` to clear the session and start fresh, or stop.

Optional: check session state first:

```bash
~/.claude/skills/pr-triage/pr-review-session status
```

### Step 2: Select PR to Triage

- **Next unreviewed in order**: `~/.claude/skills/pr-triage/pr-review-session next` — marks the current PR as reviewed and shows the next unreviewed (wraps to first when at end).
- **Specific PR by number**: `~/.claude/skills/pr-triage/pr-review-session view <number>` — shows that PR and sets it as current for the next `next`.
- **Current branch's PR**: `~/.claude/skills/pr-triage/pr-review-session view` (no number).
- **Open in browser**: `~/.claude/skills/pr-triage/pr-review-session view <number> --web`

### Step 3: Assess PR Status

`pr-review-session view` (and `next`) already prints a summary: branch, author, status, URL, size, mergeable, CI status, reviews, and unresolved feedback count, then runs `gh pr view` for the full body. The PR is automatically opened in Firefox.

Use that output as the assessment. If you need to re-display or analyze further, the same summary is produced by:

```bash
~/.claude/skills/pr-triage/pr-review-session view <number>
```

Infer blockers from the summary (e.g. failing CI, unresolved feedback, merge conflicts) and present them when suggesting actions.

### Step 4: Present Actions

Based on assessment, present relevant options:

```
What would you like to do?
1. Fix failing CI - Checkout branch and fix issues
2. Resolve feedback - Process unresolved review comments
3. Fix conflicts - Rebase/merge to resolve conflicts
4. Request review - Add reviewers to the PR
5. Mark ready - Convert from draft to ready for review
6. Merge PR - Merge the pull request
7. Close PR - Close without merging
8. Run CodeRabbit review - Run a local AI code review on this PR's changes
9. View PR in browser - Open the PR URL
10. Snooze - Temporarily hide this PR and revisit later (e.g. 1h, 1d, 1w)
11. Next - Mark reviewed and move to next unreviewed (`pr-review-session next`)
12. Reset - Reset the triage session (`pr-review-session reset`)
```

Adjust options based on PR state:

- Hide "Mark ready" if not a draft
- Hide "Merge PR" if not mergeable or has blockers
- Hide "Fix conflicts" if no conflicts
- Hide "Resolve feedback" if no unresolved comments
- Only show "Run CodeRabbit review" if the PR author is NOT `narthur` (i.e., it's someone else's code)

### Step 5: Execute Selected Action

**Option 1 - Fix failing CI:**

1. **Review CI run history first.** Before re-running or fixing anything, check whether the failing check has already been re-run previously. Use `gh run list --branch <branch> --limit 5` to see recent runs. If a check has already failed multiple times across different runs, it's almost certainly a real bug — don't re-run, investigate and fix instead. Only re-run if this is the first failure and it looks transient (e.g. timeout, network error, resource exhaustion).
2. Determine workspace type (GitButler or standard git)
3. Checkout the PR branch (see "Handling Git Worktrees" section if checkout fails due to worktree conflict):
   - **Standard git**: `gh pr checkout <number>`
   - **GitButler**: Check if branch exists in `but status`, if not create it
4. Identify failing checks and their logs
5. Fix the issues directly
6. After fixes, commit and push

**Option 2 - Resolve feedback:**

1. Checkout the PR branch: `gh pr checkout <number>` (see "Handling Git Worktrees" section if checkout fails)
2. Invoke the `/resolve-pr-feedback` skill to handle the rest. It has its own interactive workflow for retrieving feedback, presenting options, and implementing fixes.
3. After the skill completes, return to PR assessment.

**Option 3 - Fix conflicts:**

**IMPORTANT: Detect GitButler workspace before rebasing**

First, check if you're in a GitButler workspace:
```bash
git branch --show-current
```

**If on `gitbutler/workspace` branch (GitButler mode):**

1. Use the `/gitbutler` skill to handle the rebase:
   - The skill knows how to work with GitButler virtual branches
   - It will use `but` CLI commands to rebase the virtual branch
   - Example: `/gitbutler rebase <branch-name> onto <base-branch>`

**If on a regular git branch (standard git mode):**

1. Checkout the PR branch: `gh pr checkout <number>` (see "Handling Git Worktrees" section if checkout fails)
2. Fetch and rebase onto `origin/<base-branch>` (always use the remote base branch, never the local one, to avoid stale state): `git fetch origin <base-branch> && git rebase origin/<base-branch>`
3. Resolve conflicts
4. Push updated branch: `git push --force-with-lease`

**Option 4 - Request review:**

```bash
gh pr edit <number> --add-reviewer <username>
```

**Option 5 - Mark ready:**

```bash
gh pr ready <number>
```

**Option 6 - Merge PR:**

```bash
gh pr merge <number> --squash  # or --merge, --rebase based on repo settings
```

**Option 7 - Close PR:**

```bash
gh pr close <number>
```

**Option 8 - Run CodeRabbit review:**

1. Checkout the PR branch: `gh pr checkout <number>` (see "Handling Git Worktrees" section if checkout fails)
2. Run the `/coderabbit:review` skill to perform a local AI code review of the PR's changes.
3. After the review completes:
   a. Write the findings to `/tmp/cr-review-pr<number>.md` wrapped in a `<details><summary>CodeRabbit Review Notes</summary>` spoiler block. Use real markdown code blocks (triple backticks with language) for any code snippets inside.
   b. Tell the user to run one of the following to copy to clipboard:
      ```bash
      cat /tmp/cr-review-pr<number>.md | xclip -selection clipboard
      # or if using xsel:
      cat /tmp/cr-review-pr<number>.md | xsel --clipboard --input
      ```
4. Present findings and offer to act on them.
5. Return to PR assessment.

**Option 8 - View in browser:**

```bash
gh pr view <number> --web
```

**Option 9 - Snooze:**

1. Ask the user how long to snooze (e.g. 1h, 4h, 1d, 3d, 1w), or accept inline if already specified
2. Run: `~/.claude/skills/pr-triage/pr-review-session snooze <number> <duration>`
3. The PR will be hidden from the triage list until the snooze expires, then automatically reappear

### Step 6: Continue Loop

After each action:

- **Move to next unreviewed**: `~/.claude/skills/pr-triage/pr-review-session next` — marks current PR as reviewed and shows the next (wraps to first when at end).
- **Jump to another PR**: `~/.claude/skills/pr-triage/pr-review-session view <number>`
- **Reset session**: `~/.claude/skills/pr-triage/pr-review-session reset` — clears session state for this repo.
- Otherwise, return to PR assessment or `~/.claude/skills/pr-triage/pr-review-session list` based on context.

## Status Indicators

| Symbol | Meaning                    |
| ------ | -------------------------- |
| ✓      | Passing / Approved / Ready |
| ✗      | Failing / Blocked          |
| ○      | Pending / In progress      |
| ?      | Unknown / No data          |

## Review Decision Values

| Value             | Meaning                      |
| ----------------- | ---------------------------- |
| APPROVED          | PR has been approved         |
| CHANGES_REQUESTED | Changes have been requested  |
| REVIEW_REQUIRED   | Waiting for required reviews |
| (empty)           | No reviews yet               |

## Handling Git Worktrees

When using vibe kanban or similar tools that create git worktrees for feature branches, `gh pr checkout` will fail with an error like `fatal: '<branch>' is already checked out at '<path>'`.

**When checking out a PR branch, always use this approach:**

1. First try `gh pr checkout <number>`
2. If it fails due to a worktree conflict, find the existing worktree:
   ```bash
   git worktree list
   ```
3. Identify which worktree has the PR's branch checked out
4. `cd` to that worktree path and work from there instead
5. When done, `cd` back to the original repository root

This applies to all actions that require checking out a branch (Fix CI, Resolve feedback, Fix conflicts, Run CodeRabbit review).

## Commands Reference

| Command                                     | Purpose                                                   |
| ------------------------------------------- | --------------------------------------------------------- |
| `pr-review-session list`                    | List open PRs not yet triaged this session                |
| `pr-review-session next`                    | Mark current as triaged and show next unreviewed (wraps)  |
| `pr-review-session view [N] [--web]`        | Show PR summary and details; N = number or current branch |
| `pr-review-session status`                  | Show session state (repo, triaged count, current PR)      |
| `pr-review-session snooze [N] <dur>`        | Snooze a PR for a duration (e.g. 1h, 1d, 1w)             |
| `pr-review-session reset`                   | Reset the triage session for this repo                    |
| `gh pr checkout <number>`                   | Checkout PR branch (see worktree handling above)          |
| `gh pr ready <number>`                      | Mark draft as ready                                       |
| `gh pr merge <number>`                      | Merge the PR                                              |
| `gh pr close <number>`                      | Close without merging                                     |
| `gh pr edit <number> --add-reviewer <user>` | Add reviewer                                              |
| `failing-actions`                           | List all failing actions across PRs                       |

All `pr-review-session` commands should be prefixed with the full path: `~/.claude/skills/pr-triage/pr-review-session`

## Tips

- **Auto-skip**: `pr-review-session next` automatically skips PRs that are approved, mergeable, have no unresolved feedback, no failed CI, and are just waiting for pending CI checks. These PRs are marked as reviewed and a brief skip message is printed.
- **Batch triage**: Use `pr-review-session next` repeatedly to work through all PRs in order (session tracks progress and wraps when at end)
- **Priority order**: Consider triaging oldest PRs first, or those closest to being mergeable
- **Delegate**: For PRs that need author action, leave a comment and move on
- **Stale PRs**: For PRs with no activity, consider closing or requesting status updates
- **Stacked PRs**: The session only surfaces PRs targeting the default branch, so if a PR is part of a stack, it's already the next one that can land. Don't worry about the rest of the stack — treat it as an independent PR.
