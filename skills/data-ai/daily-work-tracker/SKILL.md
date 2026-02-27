---
name: daily-work-tracker
description: Use when the user wants to log work items (bugs, features, tasks), track time spent, or view a daily/weekly work report.
---

# Daily Work Tracker

## Overview
This skill allows you to record daily work activities and generate summary reports. It uses local scripts to maintain a JSON log of tasks.

## When to Use
- When the user says "I spent 2 hours fixing bug X".
- When the user says "Log a feature task: Login page, 4 hours".
- When the user asks "Show me my work report for today".
- When the user asks "What did I do yesterday?".

## Data Storage
Data is stored in `data/work-log.json` relative to the skill directory.

## Tools

### Log Work
Use the `run_command` tool to execute the logging script.

**Command Pattern:**
```bash
node .agent/skills/skills/daily-work-tracker/scripts/log-work.cjs --type "<TYPE>" --duration "<DURATION>" --description "<DESCRIPTION>"
```

- **TYPE**: `BUG`, `FEATURE`, `MEETING`, `OTHER` (Normalize to uppercase)
- **DURATION**: e.g., `2h`, `30m`, `1.5h`
- **DESCRIPTION**: A brief summary of the task.

### Generate Report
Use the `run_command` tool to generate a report.

**Command Pattern:**
```bash
node .agent/skills/skills/daily-work-tracker/scripts/generate-report.cjs --date "<DATE>"
```

- **DATE**: `today`, `yesterday`, or `YYYY-MM-DD`. Default is `today`.

### Git Hook Integration
You can set up a git hook to automatically prompt for work logging after every commit.
 
**Setup:**
```bash
node .agent/skills/skills/daily-work-tracker/scripts/install-hook.cjs
```
 
Once installed, after every `git commit`:
1. The tracker detects the commit. 
2. It asks for the time spent (e.g., `1h`).
3. It automatically logs the work item using the commit message as description.
 
**Note:** This works best in an interactive terminal.
 
## Examples

**User:** "I spent 2.5 hours fixing the navigation bug."
**Action:**
```bash
node .agent/skills/skills/daily-work-tracker/scripts/log-work.cjs --type "BUG" --duration "2.5h" --description "Fix navigation bug"
```

**User:** "Report for today."
**Action:**
```bash
node .agent/skills/skills/daily-work-tracker/scripts/generate-report.cjs --date "today"
```
