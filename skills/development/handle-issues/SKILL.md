---
name: handle-issues
description: Batch process GitHub issues via batch-orchestrator.sh with rate limit handling and session resumption
argument-hint: "[context query]"
---

# Handle Issues

Batch process multiple GitHub issues by launching `batch-orchestrator.sh` which handles the execution loop, rate limits, and status tracking autonomously. This skill focuses on setup and monitoring.

**Announce at start:** "Using handle-issues to batch process issues. Query: $CONTEXT"

**Arguments:**
- `$1` — Context query describing which issues to process and how (required)

**Examples:**
- `/handle-issues "issues assigned to @me ordered by priority"`
- `/handle-issues "all open bugs labeled 'critical'"`
- `/handle-issues "issues in milestone v2.0 by creation date"`
- `/handle-issues "Tailwind removal issues 306-308"` (frontend work)

## Agent Selection

The orchestrator uses specialized agents via `--agent` flag to ensure the right expertise for each stage:

| Stage | Agent | Purpose |
|-------|-------|---------|
| implement-issue | `bulletproof-frontend-developer` | CSS, HTML, Blade templates, frontend styling |
| implement-issue | `laravel-backend-developer` | PHP, Laravel, controllers, services, models |
| implement-issue | (default) | General implementation |
| process-pr | `code-reviewer` | **Always** - reviews PR for quality and standards |

**Determine agent based on issue content:**
- **Frontend issues** (CSS, Tailwind, styling, UI, HTML, Blade): Use `bulletproof-frontend-developer`
- **Backend issues** (PHP, Laravel, API, database, auth): Use `laravel-backend-developer`
- **Mixed or unclear**: Use default (no agent specified)

**Ask user during confirmation** which agent to use if issue type is ambiguous.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│ handle-issues (this skill)                                      │
│  • Gathers issues via gh CLI                                    │
│  • Determines appropriate agent for issue type                  │
│  • Confirms with user (ONLY interaction point)                  │
│  • Writes manifest.json (includes agent)                        │
│  • Launches batch-orchestrator.sh (background)                  │
│  • Reads status.json every 5 minutes                            │
│  • Outputs summary when complete                                │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ batch-orchestrator.sh (shell script)                            │
│  • Loops through issues autonomously                            │
│  • Invokes claude with --agent and --dangerously-skip-permissions│
│  • Parses structured_output via jq                              │
│  • Updates status.json after each operation                     │
│  • Handles rate limits, timeouts, circuit breaker               │
└─────────────────────────────────────────────────────────────────┘

Claude CLI invocations:
  claude -p "/implement-issue #N branch" \
    --agent <frontend|backend> \
    --dangerously-skip-permissions \
    --output-format json \
    --json-schema implement-issue.json

  claude -p "/process-pr #PR #issue branch" \
    --agent code-reviewer \
    --dangerously-skip-permissions \
    --output-format json \
    --json-schema process-pr.json
```

## Process

```dot
digraph process {
    rankdir=TB;
    node [shape=box];

    check_resume [label="0. Check for incomplete batch"];
    resume_choice [label="Resume or start fresh?" shape=diamond];
    parse [label="1. Parse context query"];
    fetch [label="2. Fetch matching issues"];
    confirm [label="3. Confirm with user"];
    write_manifest [label="4. Write manifest.json"];
    launch [label="5. Launch orchestrator"];
    monitor [label="6. Monitor status.json\n(every 5 min)"];
    summary [label="7. Output summary"];

    check_resume -> resume_choice;
    resume_choice -> parse [label="fresh"];
    resume_choice -> launch [label="resume"];
    parse -> fetch -> confirm -> write_manifest -> launch -> monitor -> summary;
}
```

### Step 0: Check for Incomplete Batch

Before fetching issues, check if a previous batch was interrupted:

```bash
if [[ -f status.json ]]; then
    STATE=$(jq -r '.state' status.json)

    if [[ "$STATE" == "running" || "$STATE" == "circuit_breaker" ]]; then
        PROGRESS=$(jq -r '.progress | "\(.completed)/\(.total) complete, \(.failed) failed, \(.pending) pending"' status.json)
        LOG_DIR=$(jq -r '.log_dir' status.json)
        BRANCH=$(jq -r '.base_branch' status.json)

        echo "## Incomplete Batch Detected"
        echo ""
        echo "**State:** $STATE"
        echo "**Progress:** $PROGRESS"
        echo "**Branch:** $BRANCH"
        echo "**Log dir:** $LOG_DIR"
        echo ""

        # Show pending issues
        echo "**Pending issues:**"
        jq -r '.issues[] | select(.status == "pending" or .status == "in_progress") | "- #\(.number)"' status.json
        echo ""
    fi
fi
```

**Use AskUserQuestion with options:**
1. Resume (continue with pending issues)
2. Start fresh (abandon previous batch)

If resuming, skip to Step 5 (launch orchestrator). The orchestrator's idempotency check will skip completed issues.

### Step 1: Parse Context Query

Extract from the user's context query:
- **Filter criteria**: assignee, labels, milestone, author, state
- **Sort order**: priority, created, updated, comments
- **Limit**: max issues to process (default: no limit)

### Step 2: Fetch Matching Issues

Build and execute `gh` command based on parsed criteria:

```bash
# Example: issues assigned to user
gh issue list --repo OWNER/REPO \
  --assignee @me \
  --state open \
  --json number,title,labels,milestone,createdAt \
  --limit 100

# Example: critical bugs
gh issue list --repo OWNER/REPO \
  --label "bug,critical" \
  --state open \
  --json number,title,labels,milestone,createdAt
```

**Sort by priority** (if requested): Order by label priority:
1. `priority:critical` or `P0`
2. `priority:high` or `P1`
3. `priority:medium` or `P2`
4. `priority:low` or `P3`
5. No priority label

### Step 3: Display Issue List for Confirmation

Present the ordered list before processing:

```
Found N issues matching "$CONTEXT":

1. #123 - Fix login redirect loop [priority:high, bug]
2. #456 - Add password reset flow [priority:medium, feature]
3. #789 - Update user profile validation [priority:low, enhancement]

Base branch: aw-next

Proceed with batch processing? (yes/no)
```

**This is the ONLY user interaction point.** After confirmation, the entire batch runs autonomously.

**Use AskUserQuestion** to confirm:
- Option 1: "Yes, proceed"
- Option 2: "No, cancel"
- Allow user to specify different base branch if needed

### Step 4: Write Manifest

Create the manifest file for the orchestrator:

```bash
MANIFEST="logs/handle-issues/manifest-$(date +%Y%m%d-%H%M%S).json"
mkdir -p logs/handle-issues

# Build issues array from fetched list
# $ISSUE_NUMBERS is a comma-separated list like "123,456,789"
# $AGENT is determined from issue type (frontend/backend/default)

jq -n \
  --argjson issues "[$ISSUE_NUMBERS]" \
  --arg branch "$BASE_BRANCH" \
  --arg query "$CONTEXT" \
  --arg agent "$AGENT" \
  '{
    issues: $issues,
    base_branch: $branch,
    agent: (if $agent == "" then null else $agent end),
    query: $query,
    created_at: (now | todate)
  }' > "$MANIFEST"

echo "Manifest written to: $MANIFEST"
```

**Agent values:**
- `bulletproof-frontend-developer` — CSS, Tailwind removal, UI, Blade templates
- `laravel-backend-developer` — PHP, Laravel, API, database
- `null` or omitted — Default behavior

### Step 5: Launch Orchestrator

Launch the batch orchestrator as a background process:

```bash
# Launch orchestrator (agent is read from manifest, or can be overridden via --agent)
nohup .claude/scripts/batch-orchestrator.sh --manifest "$MANIFEST" \
  > "logs/handle-issues/orchestrator-$(date +%Y%m%d-%H%M%S).log" 2>&1 &

# Or with explicit agent override:
# nohup .claude/scripts/batch-orchestrator.sh --manifest "$MANIFEST" --agent bulletproof-frontend-developer \
#   > "logs/handle-issues/orchestrator-$(date +%Y%m%d-%H%M%S).log" 2>&1 &

ORCHESTRATOR_PID=$!
echo "$ORCHESTRATOR_PID" > logs/handle-issues/.orchestrator.pid

echo "Orchestrator launched (PID: $ORCHESTRATOR_PID)"
echo "Status file: status.json"
echo "Logs: logs/batch-*/"
```

The orchestrator will:
- Use the specified agent for `implement-issue` stage
- Always use `code-reviewer` agent for `process-pr` stage

### Step 6: Monitor Progress

Check status.json every 5 minutes until complete. Also report lines changed vs base branch:

```bash
echo ""
echo "Monitoring progress (checking every 5 minutes)..."
echo ""

BASE_BRANCH=$(jq -r '.base_branch' status.json)

while true; do
    # Check if orchestrator is still running
    if [[ -f logs/handle-issues/.orchestrator.pid ]]; then
        ORCHESTRATOR_PID=$(cat logs/handle-issues/.orchestrator.pid)
        if ! kill -0 "$ORCHESTRATOR_PID" 2>/dev/null; then
            echo "Orchestrator process finished."
            rm -f logs/handle-issues/.orchestrator.pid
            break
        fi
    else
        break
    fi

    # Read and display progress
    if [[ -f status.json ]]; then
        STATE=$(jq -r '.state' status.json)
        COMPLETED=$(jq -r '.progress.completed' status.json)
        FAILED=$(jq -r '.progress.failed' status.json)
        TOTAL=$(jq -r '.progress.total' status.json)
        CURRENT=$(jq -r '(.current_issues // []) | if length == 0 then "none" elif length == 1 then "#\(.[0])" else [.[] | "#\(.)"] | join(", ") end' status.json)
        RATE_LIMITED=$(jq -r '.rate_limit.waiting' status.json)

        # Calculate lines changed since start (vs base branch)
        LINES_CHANGED=$(git diff "$BASE_BRANCH"...HEAD --shortstat 2>/dev/null | grep -oE '[0-9]+ insertion|[0-9]+ deletion' | grep -oE '[0-9]+' | paste -sd+ | bc 2>/dev/null || echo "0")

        if [[ "$RATE_LIMITED" == "true" ]]; then
            RESUME_AT=$(jq -r '.rate_limit.resume_at' status.json)
            echo "[$(date +%H:%M)] $COMPLETED/$TOTAL complete, $FAILED failed | Current: $CURRENT | Lines changed: $LINES_CHANGED | Rate limited until $RESUME_AT"
        else
            echo "[$(date +%H:%M)] $COMPLETED/$TOTAL complete, $FAILED failed | Current: $CURRENT | Lines changed: $LINES_CHANGED"
        fi

        # Exit conditions
        if [[ "$STATE" == "completed" || "$STATE" == "completed_with_errors" || "$STATE" == "circuit_breaker" ]]; then
            break
        fi
    fi

    sleep 300  # 5 minutes
done
```

### Step 7: Output Summary

Read final results from status.json and output summary:

```bash
STATE=$(jq -r '.state' status.json)
COMPLETED=$(jq -r '.progress.completed' status.json)
FAILED=$(jq -r '.progress.failed' status.json)
TOTAL=$(jq -r '.progress.total' status.json)
LOG_DIR=$(jq -r '.log_dir' status.json)

echo ""
echo "## Handle Issues Complete"
echo ""
echo "**State:** $STATE"
echo "**Progress:** $COMPLETED/$TOTAL completed, $FAILED failed"
echo ""
echo "### Results"
echo ""
echo "| Issue | PR | Status | Follow-ups |"
echo "|-------|-----|--------|------------|"

jq -r '.issues[] | "| #\(.number) | \(if .pr then "#\(.pr)" else "—" end) | \(.status) | \(.follow_ups // [] | if length > 0 then map("#\(.)") | join(", ") else "—" end) |"' status.json

# Show failures if any
if [[ $FAILED -gt 0 ]]; then
    echo ""
    echo "### Failed Issues"
    echo ""
    jq -r '.issues[] | select(.status == "failed" or .status == "skipped") | "- **#\(.number)**: \(.error // "Unknown error")"' status.json
fi

# Show circuit breaker message if triggered
if [[ "$STATE" == "circuit_breaker" ]]; then
    echo ""
    echo "### Circuit Breaker Triggered"
    echo ""
    echo "3 consecutive failures detected. Batch stopped to prevent further issues."
    echo ""
    echo "**To resume:** Fix the underlying issues, then run:"
    echo "\`/handle-issues \"resume\"\`"
fi

echo ""
echo "**Logs:** $LOG_DIR"
```

## Files

| File | Purpose |
|------|---------|
| `.claude/scripts/batch-orchestrator.sh` | Main orchestration script (loops through issues) |
| `.claude/scripts/schemas/implement-issue.json` | JSON schema for implement-issue output |
| `.claude/scripts/schemas/process-pr.json` | JSON schema for process-pr output |
| `status.json` | Real-time status (read by this skill, written by orchestrator) |
| `logs/handle-issues/manifest-*.json` | Batch manifest (issues list + branch) |
| `logs/batch-*/` | Per-batch logs and summary |
| `logs/.batch-orchestrator.lock` | Lock file preventing parallel batches |

## Integration

**Requires:**
- `.claude/scripts/batch-orchestrator.sh` (orchestration script)
- `.claude/scripts/schemas/*.json` (JSON schemas)
- `implement-issue` skill (invoked by orchestrator)
- `process-pr` skill (invoked by orchestrator)
- `gh` CLI authenticated
- `jq` for JSON parsing
- Claude Code CLI installed

**Creates:**
- One PR per issue (via implement-issue)
- Merged PRs with closed issues (via process-pr)
- Follow-up issues from review comments (via process-pr)
- status.json with real-time progress
- Log directory with per-issue logs

## Red Flags

| Temptation | Why It Fails |
|------------|--------------|
| Manage the loop yourself | Context bloat. Let orchestrator handle it. |
| Check status more often than 5 min | Unnecessary overhead. Orchestrator updates status.json. |
| Skip the confirmation step | May process wrong issues. |
| Run multiple batches in parallel | Lock file exists for a reason. |
| Ignore incomplete batch check | May reprocess or lose progress. |

## Example Session (Backend)

```
User: /handle-issues "open issues assigned to me, priority order"

Claude: Using handle-issues to batch process issues. Query: open issues assigned to me, priority order

Found 3 issues:
1. #130 - Critical auth bug [priority:critical]
2. #128 - Add MFA support [priority:high]
3. #125 - Update docs [priority:low]

Base branch: aw-next
Agent: laravel-backend-developer (backend/auth issues detected)
Proceed with batch processing? (yes/no)

User: yes

Claude: Launching batch orchestrator...
Orchestrator launched (PID: 12345)
Implement agent: laravel-backend-developer
Process-PR agent: code-reviewer
...
```

## Example Session (Frontend)

```
User: /handle-issues "Tailwind removal issues 306-308"

Claude: Using handle-issues to batch process issues. Query: Tailwind removal issues 306-308

Found 3 issues:
1. #306 - Tailwind Removal: Admin Pages
2. #307 - Tailwind Removal: Dashboard & Notifications
3. #308 - Tailwind Removal: Auth, Account, Landing Components

Base branch: aw-next
Agent: bulletproof-frontend-developer (CSS/frontend issues detected)
Proceed with batch processing? (yes/no)

User: yes

Claude: Launching batch orchestrator...
Orchestrator launched (PID: 12345)
Implement agent: bulletproof-frontend-developer
Process-PR agent: code-reviewer

Monitoring progress (checking every 5 minutes)...

[14:30] 0/3 complete, 0 failed | Current: #306 | Lines changed: 0
[14:35] 0/3 complete, 0 failed | Current: #306 | Lines changed: 245
[14:40] 1/3 complete, 0 failed | Current: #307 | Lines changed: 512
[15:40] 3/3 complete, 0 failed | Current: none | Lines changed: 1247
Orchestrator process finished.

## Handle Issues Complete

**State:** completed
**Progress:** 3/3 completed, 0 failed

### Results

| Issue | PR | Status | Follow-ups |
|-------|-----|--------|------------|
| #306 | #315 | completed | — |
| #307 | #316 | completed | — |
| #308 | #317 | completed | — |

**Logs:** logs/batch-20260131-143022/
```

## Resume Example

```
User: /handle-issues "open bugs"

Claude: ## Incomplete Batch Detected

**State:** circuit_breaker
**Progress:** 1/3 complete, 2 failed, 0 pending
**Branch:** aw-next
**Log dir:** logs/batch-20260129-100000/

**Pending issues:**
- #456
- #789

Options:
1. Resume (continue with pending issues)
2. Start fresh (abandon previous batch)

User: 1

Claude: Resuming batch...
Orchestrator launched (PID: 12346)
...
```
