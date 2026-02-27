---
name: what-did-i-do
description: Passive screen activity tracker. Use this skill when the user asks what they did today, wants to track their screen activity, or needs to analyze their productivity patterns.
allowed-tools: [Bash, Read]
---

# what-did-i-do

Passive screen tracker that captures screenshots every 5 minutes, then uses Gemini Vision to analyze what you actually did.

## Commands

Use `Bash` to run these commands:

```bash
# Start recording screenshots
~/.config/opencode/skill/what-did-i-do/scripts/watcher.sh start

# Stop recording
~/.config/opencode/skill/what-did-i-do/scripts/watcher.sh stop

# Check status
~/.config/opencode/skill/what-did-i-do/scripts/watcher.sh status

# Analyze pending screenshots (sends to Gemini)
~/.config/opencode/skill/what-did-i-do/scripts/watcher.sh analyze
```

## Reading Daily Reports

Daily activity logs are stored in `~/.config/opencode/skill/what-did-i-do/daily/YYYY-MM-DD.md`

Use `Read` to view today's report:
```
~/.config/opencode/skill/what-did-i-do/daily/2026-02-01.md
```

## Report Format

Each daily file contains:
- Time blocks with activity tables
- Focus categories: Deep work / Light work / Break / Distraction
- Summary with minutes per category

## Common Use Cases

| User Says | Action |
|-----------|--------|
| "What did I do today?" | Read today's daily file, summarize |
| "Start tracking my screen" | Run `watcher.sh start` |
| "Stop tracking" | Run `watcher.sh stop` |
| "Analyze my day" | Run `watcher.sh analyze`, then read the output |
| "How productive was I?" | Read daily file, calculate deep work % |
| "What did I do this week?" | Read last 7 daily files, summarize patterns |

## Setup Required

User must set their Gemini API key in `scripts/analyze.py` (line 12).
Get one free at: https://aistudio.google.com/apikey
