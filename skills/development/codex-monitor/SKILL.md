---
name: codex-monitor
description: Monitor active Codex CLI sessions - shows status, working directory, and last messages
user_invocable: true
---

# Codex Session Monitor

When the user invokes this skill (e.g., "/sessions"), run the session monitoring script and report the results.

## What This Skill Does

Monitors all active Codex CLI sessions by parsing the JSONL rollout files in `~/.codex/sessions/`. Shows:

- Thread ID and working directory for each session
- Session status: WORKING, WAITING, or IDLE
- File size (indicates session length/activity)
- Last modification time
- Last 2-3 lines of the most recent user message
- Last 2-3 lines of the most recent Codex response

## How to Run

Execute the Python script:

```bash
python3 {{skill_dir}}/scripts/sessions.py
```

Report the full output to the user in a code block.

## Interpreting Results

- **WORKING**: Session has recent reasoning or function_call events - actively processing
- **WAITING**: Session has an input_wait event - blocked waiting for user input
- **IDLE**: No recent activity indicators - may be finished or stalled

Sessions are considered "active" if modified within the last 5 minutes.
