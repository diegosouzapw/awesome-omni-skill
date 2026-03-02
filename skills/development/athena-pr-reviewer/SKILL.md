---
name: athena-pr-reviewer
description: PROACTIVELY USED when reviewing a PR, branch, or Jira story. Handles code review against requirements and provides actionable feedback.
allowed-tools:
  - Bash
  - Read
  - Write
  - Task
  - TaskOutput
  - AskUserQuestion
  - Glob
  - Grep
  - TodoWrite
---

# Athena PR Reviewer

## Instructions

### Progress Tracking

Use `TodoWrite` throughout the review to show progress. Initialize at start:

```
TodoWrite([
  {content: "Detect PR target", status: "in_progress", activeForm: "Detecting PR target"},
  {content: "Gather context", status: "pending", activeForm: "Gathering context"},
  {content: "Select reviewers", status: "pending", activeForm: "Selecting reviewers"},
  {content: "Run reviews", status: "pending", activeForm: "Running reviews"},
  {content: "Aggregate findings", status: "pending", activeForm: "Aggregating findings"},
  {content: "Verify findings", status: "pending", activeForm: "Verifying findings"},
  {content: "Present summary", status: "pending", activeForm: "Presenting summary"}
])
```

Update todos as you complete each step. Mark current step `in_progress`, previous step `completed`.

### 1. Detect PR Target

Parse user input to identify the PR:

- **Direct PR reference** (`PR 123`, `#123`): Extract number directly
- **Jira ticket** (`PROJ-123`): Run `gh pr list --search "PROJ-123" --json number --jq '.[0].number'`
- **Current branch**: Run `gh pr view --json number --jq '.number'`
- **No PR found**: Extract Jira from branch with `git branch --show-current | grep -oE '[A-Z]+-[0-9]+'`

### 2. Gather Data (Script)

Run the gather-context script which collects all data in parallel:

```bash
~/.claude/skills/athena-pr-reviewer/scripts/gather-context.sh ${PR_NUM} ${JIRA_TICKET}
```

This script:
- Creates work directory at `/tmp/athena-review-${PR_NUM}/`
- Fetches in parallel: PR metadata, diff, Jira ticket, epic, CLAUDE.md guidelines, git blame, prior PR comments
- Writes combined context to `${WORK_DIR}/context.md`
- Writes diff to `${WORK_DIR}/diff.patch`

Output files:
- `context.md` - Combined PR + Jira + guidelines + history data
- `diff.patch` - Full PR diff
- `pr.json` - Raw PR metadata
- `jira.json` - Raw Jira ticket data
- `epic.json` - Epic context (if linked)
- `guidelines.md` - All CLAUDE.md files from repo
- `blame.md` - Git blame for changed files (who wrote what, when)
- `prior-comments.md` - Comments from past PRs touching same files

### 3. Detect and Select Reviewers

Before running reviews, detect available reviewer agents and let the user select which to use.

#### 3.1 Detect Available Reviewers

**Built-in reviewers (always available):**
- 6 Claude specialists: comment-analyzer, test-analyzer, error-hunter, type-reviewer, code-reviewer, simplifier
- External LLMs: Gemini, Codex (auto-detected by run-reviews.sh)

**Dynamic reviewers:**
Check available `subagent_type` values in your context for additional reviewers:
1. Pattern match - Find agents where name or description contains "reviewer" or "review"
2. Exclude: `athena-pr-reviewer` (this skill), data-gathering agents (hermes-pr-courier, heimdall-pr-guardian, etc.)

#### 3.2 Confirm or Customize Reviewers

First, show all detected reviewers and ask for confirmation:

```
I'll run the review with these agents:

**Claude specialists (6):** (built-in, always available)
- comment-analyzer, test-analyzer, error-hunter
- type-reviewer, code-reviewer, simplifier

**External LLMs (2):** (run outside Claude via CLI)
- Gemini, Codex

**Installed agents (N):** (detected in your Claude setup)
- {agent-1}, {agent-2}, ...
- (or "None detected" if empty)

Proceed with all {total} reviewers?
```

Use `AskUserQuestion` with:
- **Yes, run all** (Recommended) - Proceed with all detected reviewers
- **No, let me choose** - Show detailed selection UI

#### 3.3 Handle Response

**If "Yes, run all":** Proceed to step 4 with all detected reviewers.

**If "No, let me choose":** Show paginated multi-select UI:

- Max 4 options per question, max 4 questions per call
- Each batch shows "All in this batch" + 3 actual reviewers
- Use `multiSelect: true`
- Group by category (Built-in specialists, External agents, Dynamic agents)

Parse selection:
- If "All in this batch" selected → include all reviewers from that batch
- Otherwise include only individually selected reviewers

### 4. Run Reviews (Selected Reviewers in Parallel)

Execute all selected reviews simultaneously in a SINGLE message.

**First, update todos to show individual reviewers:**

Replace the "Run reviews" todo with one todo per selected reviewer:

```
TodoWrite([
  {content: "Detect PR target", status: "completed", activeForm: "Detecting PR target"},
  {content: "Gather context", status: "completed", activeForm: "Gathering context"},
  {content: "Select reviewers", status: "completed", activeForm: "Selecting reviewers"},
  // One todo per selected reviewer:
  {content: "Review: comment-analyzer", status: "pending", activeForm: "Running comment-analyzer"},
  {content: "Review: test-analyzer", status: "pending", activeForm: "Running test-analyzer"},
  {content: "Review: error-hunter", status: "pending", activeForm: "Running error-hunter"},
  // ... etc for all selected reviewers
  {content: "Aggregate findings", status: "pending", activeForm: "Aggregating findings"},
  {content: "Verify findings", status: "pending", activeForm: "Verifying findings"},
  {content: "Present summary", status: "pending", activeForm: "Presenting summary"}
])
```

**In ONE message, run all selected reviewers:**

#### 4.1 External LLMs (if selected)

If Gemini or Codex were selected, start with `run_in_background: true`:
```bash
~/.claude/skills/athena-pr-reviewer/scripts/run-reviews.sh ${WORK_DIR}
```

#### 4.2 Built-in Claude Specialists (if selected)

For each selected built-in specialist, spawn a Task agent:

| Specialist | Prompt File | Output File |
|------------|-------------|-------------|
| comment-analyzer | `prompts/comment-analyzer.md` | `claude-comments.md` |
| test-analyzer | `prompts/test-analyzer.md` | `claude-tests.md` |
| error-hunter | `prompts/error-hunter.md` | `claude-errors.md` |
| type-reviewer | `prompts/type-reviewer.md` | `claude-types.md` |
| code-reviewer | `prompts/code-reviewer.md` | `claude-general.md` |
| simplifier | `prompts/simplifier.md` | `claude-simplify.md` |

```
Task: general-purpose
Prompt: "Read ~/.claude/skills/athena-pr-reviewer/prompts/{SPECIALIST}.md for instructions.
Then read ${WORK_DIR}/context.md and ${WORK_DIR}/diff.patch.
Perform the review and use the Write tool to save your findings to: ${WORK_DIR}/reviews/{OUTPUT_FILE}
IMPORTANT: Use the Write tool directly, not Bash with cat/heredoc."
```

**Note: Each Task call returns a `task_id`. Save these - you will need them in step 4.4.**

#### 4.3 Dynamic Reviewer Agents (if selected)

For each selected dynamic agent, spawn using its own agent type:

```
Task: {agent-name}
Prompt: "Review the PR for code quality issues.

Context file: ${WORK_DIR}/context.md (contains requirements, PR metadata, guidelines)
Diff file: ${WORK_DIR}/diff.patch (annotated with line numbers)

Use your expertise to identify issues. For each finding include:
- File path and line number (use format from diff annotations)
- Severity: Critical/High/Medium/Low
- Confidence: 0-100
- Description and suggested fix

Use the Write tool to save your review to: ${WORK_DIR}/reviews/{agent-name}.md
IMPORTANT: Use the Write tool directly, not Bash with cat/heredoc."
```

**Note: Each Task call returns a `task_id`. Save these - you will need them in step 4.4.**

#### 4.4 Wait for ALL Agents to Complete

**STOP. Do NOT proceed until you complete this step.**

When you spawned Task agents in steps 4.1-4.3, each returned a `task_id`. You MUST now:

1. **List all task IDs** you received from spawning agents
2. **For EACH task_id, call TaskOutput:**
   ```
   TaskOutput(task_id: "abc123", block: true)
   TaskOutput(task_id: "def456", block: true)
   TaskOutput(task_id: "ghi789", block: true)
   ... one call per spawned agent
   ```
3. **Do NOT skip any** - even if you see some agent output appear automatically
4. **Update todos** - Mark each reviewer as `completed` when its TaskOutput returns

**FORBIDDEN:**
- ❌ Proceeding when "most" agents are done
- ❌ Relying on automatic output appearing
- ❌ Skipping TaskOutput for agents that seem fast
- ❌ Moving to Step 5 before ALL TaskOutput calls return

**REQUIRED:**
- ✓ Call TaskOutput for EVERY spawned agent
- ✓ Use `block: true` on each call
- ✓ Mark reviewer todo as `completed` when TaskOutput returns
- ✓ Wait for each call to return before proceeding

Only after ALL TaskOutput calls have returned, proceed to Step 5.

### 5. Aggregate Reviews

Read ALL review files from `${WORK_DIR}/reviews/` directory and combine findings.

**Possible reviewers (depending on selection):**
- External: gemini.md, codex.md
- Built-in: claude-comments.md, claude-tests.md, claude-errors.md, claude-types.md, claude-general.md, claude-simplify.md
- Dynamic: {agent-name}.md (any additional detected agents)

**Confidence Filtering:**
- Drop findings with confidence < 80
- Keep findings 50-79 only if flagged by 2+ reviewers

**Priority Boost Rule:** Items flagged by 2+ reviewers get bumped up one severity level.

| Reviewers | Original | Final Severity |
|-----------|----------|----------------|
| 3+        | High     | Critical       |
| 2         | High     | Critical       |
| 3+        | Medium   | High           |
| 2         | Medium   | High           |
| 1         | Any      | No boost       |

Deduplicate similar findings, noting which reviewer(s) flagged each and average confidence.

### 5.5 Verify Findings

For each aggregated finding, verify against actual code to filter hallucinations:

1. Read `${WORK_DIR}/diff.patch` to get the actual code
2. For each finding with file:line reference:
   - Extract the actual code at that location from the diff
   - Compare the finding's description to what the code actually does
3. Use the verifier prompt (`~/.claude/skills/athena-pr-reviewer/prompts/verifier.md`) to validate each finding
4. Filter based on verdict:
   - **✓ VERIFIED** → Keep in final output
   - **✗ REJECTED** → Write to `${WORK_DIR}/rejected.md` with reason
   - **⚠️ PARTIAL** → Keep but move to "Suggestions" section

Output verified findings to `${WORK_DIR}/verified-findings.md`

### 6. Synthesize Actionable Items

Present combined review to user:

```markdown
# PR Review: {PR_TITLE} (#{PR_NUM})

## Requirements Status
| Requirement | Status | Notes |
|-------------|--------|-------|

## Action Items (Verified)

### Critical (consensus, verified)
- [ ] file:line - issue - fix [reviewer1 + reviewer2 + reviewer3] (3+, avg 92%) ✓

### High Priority (verified)
- [ ] file:line - issue - fix [reviewer1 + reviewer2] ← boosted (2, avg 85%) ✓
- [ ] file:line - issue - fix [reviewer1] (95%) ✓

### Medium Priority (verified)
- [ ] file:line - issue - fix [reviewer1] (88%) ✓

### Suggestions
- improvements (including PARTIAL findings downgraded from higher severity)

## Rejected Findings
Findings that failed verification are saved to: `${WORK_DIR}/rejected.md`

## Review Sources
[List all .md files found in ${WORK_DIR}/reviews/]

## Recommendation: APPROVE / REQUEST_CHANGES
```

### 7. Offer Deep Dive

After presenting the summary, offer to iterate through action items:

```
Would you like me to walk through any of these issues in detail? I can:
- Explain each issue with more context
- Show the actual code and a potential fix
- Give my opinion on priority and approach

Reply with "yes" to go through them one by one, or pick specific items (e.g., "explain the first critical issue").
```

If the user accepts, **replace todos with action items**:

```
TodoWrite([
  {content: "Critical: {first issue summary}", status: "in_progress", activeForm: "Reviewing critical issue"},
  {content: "High: {second issue summary}", status: "pending", activeForm: "Reviewing high priority issue"},
  {content: "High: {third issue summary}", status: "pending", activeForm: "Reviewing high priority issue"},
  {content: "Medium: {fourth issue summary}", status: "pending", activeForm: "Reviewing medium priority issue"},
  // ... one per action item, ordered by severity
])
```

Go through items **ONE AT A TIME**:

For each item:
1. Mark current item as `in_progress`
2. Show the relevant code snippet from the diff
3. Explain why it's problematic with more context
4. Propose a concrete fix with code
5. Share your opinion on severity and whether it's a blocker

**CRITICAL: After presenting ONE item, STOP and wait for user input.**

When user says "next", "continue", or similar:
1. Mark current item as `completed`
2. Mark next item as `in_progress`
3. Present the next item

Ask: "Ready for the next issue? (N remaining)" or let them say "skip", "stop", or ask questions about the current item.

Do NOT present multiple items in a single response.

## Examples

**User:** "Review PR 456"
1. Detect PR 456, find linked Jira ticket
2. Gather context via script (parallel CLI calls)
3. Detect available reviewers (built-in + any dynamic agents)
4. Present selection UI - user picks which reviewers to run
5. Run selected reviews in parallel
6. Aggregate findings, boost items flagged by 2+ reviewers
7. Verify findings against actual diff (filter hallucinations)
8. Present verified actionable summary

**User:** "Review CSD-123"
1. Find PR linked to CSD-123
2. Gather context including acceptance criteria
3. Present reviewer selection (may include custom security-reviewer agent if installed)
4. Run selected reviews in parallel
5. Present findings with reviewer attribution

**User:** "Review this branch"
1. Get PR from current branch
2. Extract Jira from branch name if needed
3. Detect all available reviewers
4. User selects reviewers via paginated UI
5. Full review workflow with selected reviewers
