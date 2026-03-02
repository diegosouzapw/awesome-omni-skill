---
name: ci-fix-pipeline
description: Self-healing CI pipeline -- 3-attempt retry budget with strategy rotation, inbox-wait for results, autonomous fix loop with escalation
version: 2.0.0
category: workflow
tags:
  - ci
  - github-actions
  - autonomous
  - pipeline
  - slack
  - self-healing
author: OmniClaude Team
args:
  - name: --pr
    description: PR number to fix CI failures for
    required: false
  - name: --branch
    description: Branch name to fix CI failures for (default: current branch)
    required: false
  - name: --skip-patterns
    description: "Comma-separated job/step name patterns to skip (e.g., 'test_*,lint')"
    required: false
  - name: --max-fix-files
    description: "Max files in scope before creating a sub-ticket instead of fixing (default: 10)"
    required: false
  - name: --no-slack
    description: Disable Slack notifications for this run
    required: false
  - name: --ticket-id
    description: Ticket ID context (included in Slack messages and sub-ticket descriptions)
    required: false
  - name: --max-attempts
    description: "Maximum repair attempts with strategy rotation (default: 3)"
    required: false
  - name: --self-heal
    description: "Enable self-healing mode: retry loop with strategy rotation and inbox-wait (default: false)"
    required: false
composable: true
---

# CI Fix Pipeline

## Overview

Autonomous pipeline that fetches GitHub Actions CI failures and fixes them -- ALL failures by
default. No selective mode. Failures beyond `max_fix_files` trigger sub-ticket creation and
continue with the remaining fixable failures.

**v2.0 Self-Healing Mode (OMN-2829)**: When `--self-heal` is enabled, the pipeline enters a
multi-attempt repair loop with strategy rotation. Each attempt uses a different fix strategy
(targeted -> broad lint -> regenerate). Between attempts, the pipeline uses inbox-wait (not
polling) to detect CI re-run results. The `node_ci_repair_effect` ONEX node orchestrates this
loop.

**Workflow (standard)**: Fetch CI failures -> Slack start -> Classify + Sub-ticket large-scope -> Fix ALL fixable -> Commit -> Slack complete -> ModelSkillResult

**Workflow (self-healing)**: `_bin/ci-status.sh` -> detect failures -> attempt 1 (targeted fix) -> push -> inbox-wait -> if still failing -> attempt 2 (broad lint fix) -> push -> inbox-wait -> if still failing -> attempt 3 (regenerate) -> push -> inbox-wait -> inbox notification on success/exhaustion

**Announce at start:** "I'm using the ci-fix-pipeline skill to fix CI failures."

## Policy Defaults

```yaml
policy:
  fix_all: true                     # always fix all -- no selective mode
  max_fix_files: 10                 # files in scope trigger sub-ticket (not skip)
  fix_preexisting_in_touched: true  # fix pre-existing issues in touched files
  slack_on_start: true              # notify Slack before fixing
  slack_on_complete: true           # notify Slack with fix summary
  max_attempts: 3                   # self-heal: retry budget (1-3)
  self_heal: false                  # self-heal: enable multi-attempt loop
```

## Quick Start

```
/ci-fix-pipeline                              # Fix all CI failures on current branch
/ci-fix-pipeline --pr 42                      # Fix failures for PR #42
/ci-fix-pipeline --ticket-id OMN-1234         # Include ticket context in Slack messages
/ci-fix-pipeline --no-slack                   # Suppress Slack notifications
/ci-fix-pipeline --skip-patterns "test_*"     # Skip jobs/steps matching pattern
/ci-fix-pipeline --max-fix-files 20           # Raise the sub-ticket threshold
/ci-fix-pipeline --self-heal --pr 42          # Self-healing mode with retry loop
/ci-fix-pipeline --self-heal --max-attempts 2 # Limit to 2 attempts
```

## Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `--pr <number>` | none | PR number for CI failure fetch |
| `--branch <ref>` | current | Branch name for CI failure fetch |
| `--skip-patterns <patterns>` | none | Comma-separated job/step name patterns to skip |
| `--max-fix-files <n>` | 10 | Files-in-scope threshold; above this -> sub-ticket |
| `--no-slack` | false | Disable Slack notifications |
| `--ticket-id <id>` | none | Context ticket ID for Slack messages |
| `--self-heal` | false | Enable self-healing retry loop with strategy rotation |
| `--max-attempts <n>` | 3 | Maximum repair attempts (only with --self-heal) |

## Execution Phases

### Phase 1: Fetch CI Failures

Dispatch to polymorphic agent:

```
Task(
  subagent_type="onex:polymorphic-agent",
  description="Fetch CI failures for ci-fix-pipeline",
  prompt="Fetch CI failures using the ci-failures skill.

    Run: ${CLAUDE_PLUGIN_ROOT}/skills/ci-failures/ci-quick-review {N | branch_name}

    Return the raw JSON from ci-quick-review (pass through unchanged).
    The response has structure:
    {\"repository\": str, \"pr_number\": int,
     \"summary\": {\"total\": N, \"critical\": N, \"major\": N, \"minor\": N},
     \"failures\": [{\"workflow\": str, \"job\": str, \"job_id\": str, \"step\": str,
                    \"severity\": str, \"workflow_id\": str, \"job_url\": str}],
     \"fetched_at\": str}"
)
```

**Branch resolution**: After Phase 1, the orchestrator must have a branch name for use in later
phases. Resolve as follows:
- If `--branch` was provided: use that value directly.
- If `--pr` was provided (no `--branch`): run `gh pr view {N} --json headRefName --jq '.headRefName'`
  to get the branch name.
- If neither was provided: use the current branch (already known from `ci-quick-review` invocation).

### Phase 2: Slack Start Notification

If `slack_on_start: true` and Slack is available, notify:

```
ci-fix-pipeline starting
  PR/Branch: {context}
  Failures found: {N} ({critical} critical, {major} major, {minor} minor)
  Ticket: {ticket_id if provided}
```

Skip silently if Slack unavailable (non-blocking).

### Phase 3: Classify and Route Failures

For each failure:

1. **Skip check**: Does the failure `job` or `step` name match any `--skip-patterns` pattern?
   - Yes → mark as `skipped`, record reason
   - No → continue

2. **Scope check**: Does the failure `job` touch more than `max_fix_files` files?
   Determine scope by inspecting the job logs (via `gh api repos/{repo}/actions/jobs/{job_id}/logs`)
   to count affected files. If log inspection is unavailable, treat scope as within threshold.
   - Scope > max_fix_files → mark as `capped`, create Linear sub-ticket inline (see Sub-Ticket Creation below), continue to next failure
   - Scope ≤ max_fix_files → add to fix queue

Result: failures split into `skipped`, `capped`, and `to_fix` buckets.

### Phase 4: Fix Failures

Dispatch one polymorphic agent per severity group (critical first, then major, then minor) for all `to_fix` failures:

```
Task(
  subagent_type="onex:polymorphic-agent",
  description="Fix {severity} CI failures",
  prompt="**AGENT REQUIREMENT**: You MUST be a polymorphic-agent.

    Fix the following {severity} CI failures:

    {failures_list}

    Instructions:
    1. Read each affected file
    2. Apply the fix
    3. If fix_preexisting_in_touched is true: also fix any pre-existing lint/mypy
       issues in those files (only files already in scope — not a full repo scan)
    4. Do NOT commit

    Return classification for each failure:
    {\"fixed\": [failure_ids], \"architectural\": [failure_ids], \"unfixable\": [failure_ids]}"
)
```

**Post-fix architectural check**: For each failure returned as `architectural` by the fix agent:
- Send a Slack message (via `HandlerSlackWebhook` in omnibase_infra) describing the architectural
  change and asking for human approval. Include the failure description, the proposed fix, and
  the files affected. Wait for a reply (poll or webhook callback).
- Approved (human replies "approve" or "yes") → apply fix; Declined (any other reply or timeout after 10 min) → mark `escalated`

### Phase 5: Commit Fixes

**Skip this phase if `fixed` count is 0** (no code was changed — all failures were skipped, capped, or unfixable). Proceed directly to Phase 6 with `commit: null` in ModelSkillResult.

Otherwise, orchestrator stages and commits inline (no dispatch needed):

```bash
git add <changed_files>
git commit -m "fix(ci): resolve {N} {severity} failures [{ticket_id}]"
```

Commit message format: `fix(ci): resolve {N} {severity} failures [{ticket_id}]`
where `{severity}` is the highest severity fixed (e.g., `critical`, `major`, `minor`) and
`{ticket_id}` is the value from `--ticket-id` (or omitted if not provided).

### Sub-Ticket Creation

For each `capped` failure (scope > max_fix_files), created inline during Phase 3:

```python
# current_team: resolved from --ticket-id parent team (via mcp__linear-server__get_issue),
# or from the first team returned by mcp__linear-server__list_teams if no ticket is provided.
mcp__linear-server__create_issue(
    title=f"CI: {failure.job} — {failure.step} (large scope)",
    team=current_team,
    description=f"""
## CI Failure Requiring Human Review

**Job**: {failure.job}
**Step**: {failure.step}
**Severity**: {failure.severity}
**Scope**: Exceeds max_fix_files={max_fix_files} threshold
**Triggered by**: ci-fix-pipeline run for {ticket_id or branch}
**Job URL**: {failure.job_url}

## Definition of Done

- [ ] All affected files reviewed and fixed
- [ ] CI passing on {branch}
    """,
    parentId=ticket_id if ticket_id else None,
    labels=["ci-failure", "needs-human"]
)
```

### Phase 6: Slack Complete Notification

If `slack_on_complete: true`, notify with diff summary:

```
ci-fix-pipeline complete
  Fixed: {N} failures
  Skipped: {M} failures (patterns: {patterns})
  Sub-tickets created: {K} (large-scope failures)
  Escalated: {L} (architectural — declined or timed out)
  Ticket: {ticket_id if provided}
  Branch: {branch}
```

## ModelSkillResult Output

Emits to `~/.claude/skill-results/{context_id}/ci-fix-pipeline.json`
where `{context_id}` is the Claude session ID (from `$CLAUDE_SESSION_ID` env var) or `default`
if the session ID is unavailable:

```json
{
  "status": "completed|capped|escalated|failed",
  "fixed_count": 5,
  "skipped_count": 1,
  "capped_count": 2,
  "escalated_count": 0,
  "unfixable_count": 0,
  "sub_tickets": ["OMN-XXXX", "OMN-XYYY"],
  "commit": "abc1234",
  "branch": "feature/my-branch",
  "ticket_id": "OMN-1234"
}
```

**Status values**:
- `completed` — All fixable failures resolved
- `capped` — Some failures deferred to sub-tickets; fixed what was in scope
- `escalated` — One or more architectural failures declined or timed out; human review required
- `failed` — CI fetch failed or commit failed; pipeline halted

## Failure Handling

| Error | Behavior |
|-------|----------|
| CI fetch fails | Hard exit with `status: failed`, reason in output |
| Fix agent fails | Log failure, mark as `unfixable`, continue with others |
| Sub-ticket creation fails | Log warning, continue (non-blocking) |
| Slack unavailable | Skip notification, continue (non-blocking) |
| Commit fails | Exit with `status: failed`, leave changes staged |

## Sub-Ticket Threshold Policy

The `max_fix_files` threshold is a **routing decision**, not a skip:

- Failures within threshold: fixed autonomously
- Failures above threshold: sub-ticket created, pipeline continues with remaining

This ensures large-scope failures are never silently dropped — they are tracked in Linear.

## Self-Healing Mode (OMN-2829)

When `--self-heal` is enabled, the pipeline wraps the standard fix flow in a multi-attempt
retry loop orchestrated by `node_ci_repair_effect`.

### Architecture

```
CI fails
  -> _bin/ci-status.sh --pr N --repo ORG/REPO
  -> parse failure JSON
  -> node_ci_repair_effect.execute_effect(event)
  -> for attempt in 1..max_attempts:
       strategy = RepairStrategy.for_attempt(attempt)
       -> dispatch fix agent with strategy-specific prompt
       -> git add + commit + push
       -> inbox-wait for CI re-run result (not polling)
       -> if CI passing: inbox notification "repaired" -> exit
       -> if CI failing: rotate strategy, continue loop
  -> if all attempts exhausted: inbox notification "exhausted" -> exit
```

### Strategy Rotation

Each attempt uses a progressively broader fix strategy:

| Attempt | Strategy | Description |
|---------|----------|-------------|
| 1 | `targeted_fix` | Parse error logs, fix only the specific failing lines/files |
| 2 | `broad_lint_fix` | Run ruff/mypy auto-fix on all files touched by the PR |
| 3 | `regenerate_and_fix` | Rewrite failing sections, apply all auto-fixers |

### CI Status Extraction

Self-healing mode uses `_bin/ci-status.sh` instead of the heavier `ci-failures/ci-quick-review`
for fast, structured CI status checks between attempts:

```bash
# Fetch CI status as structured JSON
${CLAUDE_PLUGIN_ROOT}/_bin/ci-status.sh --pr 42 --repo OmniNode-ai/omniclaude
```

Output:
```json
{
  "status": "failing",
  "pr_number": 42,
  "repo": "OmniNode-ai/omniclaude",
  "branch": "jonah/omn-2829-self-healing-ci",
  "run_id": "12345678",
  "failed_jobs": [
    {
      "job_id": "56174634733",
      "job_name": "lint / ruff",
      "step": "Run ruff check",
      "conclusion": "failure",
      "log_excerpt": "..."
    }
  ],
  "failure_summary": "1 job(s) failed: lint / ruff",
  "fetched_at": "2026-02-26T13:00:00Z"
}
```

### Inbox-Wait Pattern

After each fix attempt pushes code, the pipeline waits for CI results using inbox-wait
rather than fixed-interval polling:

1. Push fix commit to branch
2. Call `node_ci_repair_effect.wait_for_ci_rerun()` which:
   - Polls `_bin/ci-status.sh` at 30s intervals
   - Detects when a new `run_id` appears (different from pre-push run)
   - Waits for terminal state (`passing` or `failing`)
   - Times out after 5 minutes (configurable)
3. If `passing`: record success, send inbox notification, exit
4. If `failing`: rotate strategy, begin next attempt

### Self-Healing Fix Dispatch

```
Task(
  subagent_type="onex:polymorphic-agent",
  description="ci-fix-pipeline: self-heal attempt {N}/{max} for PR #{pr_number}",
  prompt="**SELF-HEALING CI REPAIR -- Attempt {N}/{max}**

    Strategy: {strategy_name}
    {strategy_prompt}

    CI Failure Details:
    {failure_json}

    Branch: {branch}
    Repo: {repo}

    Instructions:
    1. Read the failure logs carefully
    2. Apply the fix strategy described above
    3. Stage and commit with message:
       fix(ci): self-heal attempt {N} -- {strategy_name} [{ticket_id}]
    4. Push to the branch

    Report: files changed, fix description, confidence level."
)
```

### ONEX Node: node_ci_repair_effect

**Tier**: EVENT_BUS+
**Type**: Effect node (external I/O)
**Location**: `plugins/onex/hooks/lib/node_ci_repair_effect.py`

Provides:
- `execute_effect(event)` -- Initialize repair run state
- `record_attempt_result(run_state, attempt, ...)` -- Record attempt outcome
- `finalize_with_error(run_state, error)` -- Record error and notify
- `fetch_ci_status(pr, repo, branch)` -- Wrapper around `_bin/ci-status.sh`
- `wait_for_ci_rerun(pr, repo, branch, prev_run_id)` -- Inbox-wait for new CI run
- `send_inbox_notification(run_state, message)` -- Write to `~/.claude/inbox/`
- `save_repair_state(run_state)` / `load_repair_state(run_id)` -- State persistence

### Self-Healing ModelSkillResult

When `--self-heal` is active, the ModelSkillResult includes additional fields:

```json
{
  "status": "repaired|exhausted|failed",
  "repair_run_id": "ci-repair-42-1740000000",
  "attempts_used": 2,
  "max_attempts": 3,
  "strategy_used": "broad_lint_fix",
  "fixed_count": 5,
  "skipped_count": 0,
  "commit": "abc1234",
  "branch": "jonah/omn-2829-self-healing-ci",
  "ticket_id": "OMN-2829",
  "inbox_notification_sent": true
}
```

**Status values (self-healing)**:
- `repaired` -- CI fixed within the retry budget
- `exhausted` -- All attempts used, CI still failing
- `failed` -- Error during repair (fetch failed, commit failed, etc.)

### Verification

To verify self-healing works end-to-end:

1. Push a deliberately failing commit (e.g., syntax error, failing test)
2. Run: `/ci-fix-pipeline --self-heal --pr <N> --ticket-id OMN-2829`
3. Confirm:
   - Attempt 1 uses `targeted_fix` strategy
   - If still failing, attempt 2 uses `broad_lint_fix`
   - If still failing, attempt 3 uses `regenerate_and_fix`
   - Inbox notification sent on success or exhaustion
   - State persisted to `~/.claude/state/ci-repair/`

## See Also

- `ci-failures` skill -- fetch and analyze CI failures (read-only)
- `ci-watch` skill -- poll CI status and auto-fix (OMN-2523)
- `local-review` skill -- review and fix local code changes
- `ticket-pipeline` skill -- end-to-end ticket pipeline (Phase 4 invokes ci-watch)
- `_bin/ci-status.sh` -- lightweight CI status extraction script
- `node_ci_repair_effect` -- ONEX effect node for self-healing orchestration
- `HandlerSlackWebhook` in omnibase_infra -- Slack delivery infrastructure
