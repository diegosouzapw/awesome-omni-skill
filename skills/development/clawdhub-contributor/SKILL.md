---
name: clawdhub-contributor
description: Contribute to the ClawdHub ecosystem by scouting unknown skills, reporting bugs, and sharing skill recipes. Three modes (passive/active/full) let you control how much you contribute.
metadata: {"openclaw":{"requires":{"bins":["bash","jq"]}}}
---

# ClawdHub Contributor

Adds contribution capabilities to your agent. Help grow the ClawdHub ecosystem by analyzing skills, reporting bugs, and sharing useful skill combinations.

## Modes

| Mode | What it does |
|------|-------------|
| **passive** | Bug reports and recipes only (safe default) |
| **active** | Adds auto-scout: analyze unknown skills locally and generate reports |
| **full** | All above plus opt-in telemetry |

Set mode in `config/default.json` or via `CLAWDHUB_CONTRIB_MODE` env var.

## Features

### Auto-Scout (active/full mode)

Analyze a local skill directory and produce a structured quality/security report:

```bash
bash scripts/scout.sh /path/to/skill-directory
```

Output: JSON report with dependency info, quality score, and security flags.
**Fully offline** — no network access, pure static analysis.

### Bug Reporting (all modes)

Report a skill failure with sanitized system context:

```bash
bash scripts/report-bug.sh <skill-slug> <error-message> [context]
```

Output: JSON bug report ready for API submission. Collects OS and node version but **never** leaks hostname, IP, or username.

### Recipes (all modes)

Share a useful combination of skills that solved a task:

```bash
bash scripts/submit-recipe.sh <task-description> <skill1> [skill2] [skill3] ...
```

Output: JSON recipe ready for API submission.

## Configuration

Edit `config/default.json`:

```json
{
  "mode": "passive",
  "telemetry": false,
  "autoScout": false,
  "bugReports": true,
  "recipes": true
}
```

| Key | Type | Description |
|-----|------|-------------|
| `mode` | string | `passive`, `active`, or `full` |
| `telemetry` | bool | Opt-in anonymous usage stats (full mode only) |
| `autoScout` | bool | Auto-scan skills on encounter (active/full mode) |
| `bugReports` | bool | Enable bug report generation |
| `recipes` | bool | Enable recipe submission |

## Commands Summary

| Command | Mode Required | Description |
|---------|--------------|-------------|
| `scripts/scout.sh <dir>` | active+ | Analyze a skill directory |
| `scripts/report-bug.sh <slug> <msg> [ctx]` | any | Generate bug report |
| `scripts/submit-recipe.sh <task> <skills...>` | any | Generate recipe |

## Security

- Scripts never access the network
- No credentials, IPs, hostnames, or usernames are collected
- All output is sanitized JSON to stdout
- Scout performs read-only static analysis
