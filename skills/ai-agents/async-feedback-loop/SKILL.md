---
name: Async Feedback Loop
description: Enables mid-stream course correction by monitoring a FEEDBACK.md file for user interventions. Allows the agent to incorporate new instructions without restarting the task.
version: 1.0.0
author: Antigravity Skills Library
created: 2026-01-16
leverage_score: 5/5
---

# SKILL-010: Async Feedback Loop

## Overview

Long-running tasks often drift. This skill provides a mechanism for the user to inject guidance via a `FEEDBACK.md` file. The agent checks this file and pivots immediately if new instructions are found.

## Trigger Phrases

- `check feedback`
- `read instructions`
- `get user input`

## Inputs

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `--file` | string | No | `FEEDBACK.md` | The feedback channel file |
| `--acknowledge` | switch | No | False | Mark feedback as read (append timestamp) |

## Outputs

1. Console: "New feedback found: ..." or "No new feedback."
2. File Update: Appends "Read by Agent at <time>" if acknowledged.

## Implementation

See `check_feedback.ps1`.

## Integration

```powershell
# In a loop:
.\skills\10_async_feedback\check_feedback.ps1 -Acknowledge
if ($LASTEXITCODE -eq 0) {
    # Pivot strategy
}
```
