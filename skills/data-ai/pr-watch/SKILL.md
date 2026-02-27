---
name: pr-watch
description: Poll GitHub PR for review feedback, auto-fix issues, and report terminal state
version: 1.0.0
category: workflow
tags: [pr, github, review, automation, polling]
author: OmniClaude Team
composable: true
inputs:
  - name: pr_number
    type: int
    description: GitHub PR number to watch
    required: true
  - name: repo
    type: str
    description: "GitHub repo slug (org/repo)"
    required: true
  - name: timeout_hours
    type: float
    description: Max hours to wait for review (default 24)
    required: false
  - name: max_review_cycles
    type: int
    description: Max auto-fix cycles before escalating (default 3)
    required: false
  - name: auto_fix
    type: bool
    description: Auto-fix Critical/Major/Minor review comments (default true)
    required: false
  - name: fix_nits
    type: bool
    description: Also auto-fix Nit-level comments (default false)
    required: false
outputs:
  - name: skill_result
    type: ModelSkillResult
    description: "Written to ~/.claude/skill-results/{context_id}/pr-watch.json"
    fields:
      - status: approved | changes_requested_fixed | timeout | capped | error
      - pr_number: int
      - repo: str
      - fix_cycles_used: int
      - elapsed_hours: float
args:
  - name: pr_number
    description: GitHub PR number to watch
    required: true
  - name: repo
    description: "GitHub repo slug (org/repo)"
    required: true
  - name: --timeout-hours
    description: Max hours to wait for review (default 24)
    required: false
  - name: --max-review-cycles
    description: Max fix cycles before escalating (default 3)
    required: false
  - name: --no-auto-fix
    description: Poll only, don't attempt fixes
    required: false
  - name: --fix-nits
    description: Also auto-fix Nit-level comments
    required: false
---

# PR Watch

## Overview

Poll GitHub PR review status. Auto-fix review comments (Critical/Major/Minor by default) using
`pr-review-dev`. Exit when PR reaches a terminal state: `approved`, `changes_requested_fixed`
(all blocking issues resolved), `timeout`, or `capped` (max fix cycles reached).

**Announce at start:** "I'm using the pr-watch skill to monitor reviews for PR #{pr_number}."

**Implements**: OMN-2524

## Quick Start

```
/pr-watch 123 org/repo
/pr-watch 123 org/repo --timeout-hours 48
/pr-watch 123 org/repo --max-review-cycles 5
/pr-watch 123 org/repo --fix-nits
/pr-watch 123 org/repo --no-auto-fix
```

## Poll Loop

1. Fetch PR review state via `gh pr view {pr_number} --repo {repo} --json reviews,reviewDecision`
2. If `reviewDecision == APPROVED`: exit with `status: approved`
3. If `reviewDecision == CHANGES_REQUESTED` and `auto_fix=true` and cycles remaining:
   - Dispatch pr-review-dev agent with review comments
   - Increment fix cycle count
   - Wait for CI (delegate to ci-watch if needed), then re-request review
4. If fix cycles exhausted: exit with `status: capped`
5. If elapsed > timeout_hours: exit with `status: timeout`
6. If all blocking issues resolved but no explicit approval: exit with `status: changes_requested_fixed`

## Fix Dispatch Contract

```
Task(
  subagent_type="onex:polymorphic-agent",
  description="pr-watch: fix review comments for PR #{pr_number} (cycle {N})",
  prompt="Invoke: Skill(skill=\"pr-review-dev\", args=\"{pr_number}\")

    Review comments:
    {review_comments}

    Fix all Critical, Major, and Minor issues.{nit_instruction}
    Push fixes to branch: {branch_name}

    Report: issues fixed, files changed, any issues skipped."
)
```

## Skill Result Output

Write `ModelSkillResult` to `~/.claude/skill-results/{context_id}/pr-watch.json` on exit.

```json
{
  "skill": "pr-watch",
  "status": "approved",
  "pr_number": 123,
  "repo": "org/repo",
  "fix_cycles_used": 2,
  "elapsed_hours": 3.5,
  "context_id": "{context_id}"
}
```

**Status values**: `approved` | `changes_requested_fixed` | `timeout` | `capped` | `error`

## Push-Based Notification Support (OMN-2826)

pr-watch supports two notification modes, selected automatically based on infrastructure
availability. The push-based mode eliminates polling for CI status checks during the
review cycle.

### EVENT_BUS+ Mode (preferred)

When Kafka and Valkey are available (`ENABLE_REAL_TIME_EVENTS=true`):

1. Register watch: agent registers interest in `(repo, pr_number)` via Valkey watch registry
2. Wait for inbox: block until a `pr-status` event arrives in the agent's inbox topic
   (`onex.evt.omniclaude.agent-inbox.{agent_id}.v1`)
3. Process result: on CI completion after a fix cycle, check review status and decide next step

```python
from omniclaude.services.inbox_wait import register_watch, wait_for_pr_status

# Register watch for CI re-runs after pushing fixes
await register_watch(agent_id=agent_id, repo=repo, pr_number=pr_number)

# After pushing a fix, wait for CI to complete (replaces polling)
ci_result = wait_for_pr_status(
    repo=repo,
    pr_number=pr_number,
    run_id=latest_run_id,
    agent_id=agent_id,
    timeout_seconds=3600,
)
```

### STANDALONE Mode (fallback)

When Kafka/Valkey are unavailable:

1. Spawn `gh run watch {run_id} --exit-status` as background process after pushing fixes
2. Wait for result in file-based inbox (`~/.claude/pr-inbox/`)

```python
from omniclaude.services.inbox_wait import wait_for_pr_status

# Unified interface -- automatically falls back to STANDALONE
ci_result = wait_for_pr_status(
    repo=repo,
    pr_number=pr_number,
    run_id=latest_run_id,
    timeout_seconds=3600,
)
```

### Migration Notes

The original review polling loop is unchanged. The inbox-wait pattern replaces only the
CI status checks that happen between fix cycles (when pr-watch pushes a fix and needs
to wait for CI before re-requesting review). The `wait_for_pr_status()` function provides
a unified interface that works in both EVENT_BUS and STANDALONE modes.

## See Also

- `ticket-pipeline` skill (planned: invokes pr-watch after ci-watch passes)
- `ci-watch` skill (planned: runs before pr-watch)
- `auto-merge` skill (planned: runs after pr-watch passes)
- `pr-review-dev` skill (invoked to fix review comments)
- `inbox_wait` module (`omniclaude.services.inbox_wait`) — unified wait interface
- `node_github_pr_watcher_effect` — ONEX node for EVENT_BUS+ mode routing
- OMN-2524 — pr-watch implementation ticket
- OMN-2826 — push-based notifications ticket
