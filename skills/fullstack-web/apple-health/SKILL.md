---
name: Apple Health
slug: apple-health
version: 1.0.0
homepage: https://clawic.com/skills/apple-health
description: Connect agents to Apple Health exports with MCP setup, schema validation, and privacy-safe analysis.
changelog: Initial release with Apple Health MCP integration workflow and guarded query patterns.
metadata: {"clawdbot":{"emoji":"❤️","requires":{"bins":["node","npx"],"env":["HEALTH_DATA_DIR"]},"install":[{"id":"npm","kind":"npm","package":"@neiltron/apple-health-mcp","bins":["apple-health-mcp"],"label":"Install Apple Health MCP Server (npm)"}],"os":["darwin","linux","win32"]}}
---

## Setup

On first use, read `setup.md` for integration guidelines.

## When to Use

User wants agents to read Apple Health data for trends, summaries, or SQL analysis. Agent handles export validation, MCP server wiring, and safe query/report flows without exposing private health records.

## Architecture

Memory lives in `~/apple-health/`. See `memory-template.md` for setup.

```
~/apple-health/
|-- memory.md              # Status, client integration state, latest export path
|-- integrations.md        # Connected MCP clients and validation notes
|-- query-log.md           # Reusable SQL/report prompts and known-good outputs
`-- archive/               # Retired paths and old troubleshooting notes
```

## Quick Reference

Use these files on demand instead of overloading the main instructions.

| Topic | File |
|-------|------|
| Setup process | `setup.md` |
| Memory template | `memory-template.md` |
| MCP client wiring | `mcp-config.md` |
| Query recipes | `query-recipes.md` |
| Fallback CLI paths | `fallback-cli.md` |

## Core Rules

### 1. Confirm Integration Mode Before Doing Anything
Start by clarifying one of these modes:
- `csv-export` using Apple Health CSV exports and MCP
- `not-now` if user is only planning and does not want setup yet

Never imply direct HealthKit API access from terminal agents. This skill works from exported data.

### 2. Validate Local Export Before MCP Wiring
Require a real export folder before configuration:
- Must exist locally and be readable
- Must include files matching `HKQuantityTypeIdentifier*.csv`, `HKCategoryTypeIdentifier*.csv`, or `HKWorkoutActivityType*.csv`
- Must not be an empty unzip folder

If validation fails, stop and fix data path first.

### 3. Run Runtime Preflight Before MCP Configuration
Before wiring MCP, verify runtime:
- `node -v` should be an LTS line (18, 20, or 22)
- If `npx @neiltron/apple-health-mcp` fails with missing `duckdb.node`, switch to LTS Node and retry
- Confirm `HEALTH_DATA_DIR` is available as an absolute path

Do not continue while runtime is incompatible.

### 4. Configure MCP With Explicit Path and Command
Use the MCP server command from `mcp-config.md`:
- Command: `npx`
- Args: `[@neiltron/apple-health-mcp]`
- Env: `HEALTH_DATA_DIR=/absolute/path/to/export`

Never continue with placeholders or relative paths that are not verified.

### 5. Schema First, Then Queries
First run schema discovery (`health_schema`) and map available tables.
Only then run `health_query` or `health_report`.

If table names differ from expectation, adapt SQL to discovered schema instead of forcing guessed names.

### 6. Use Date-Bounded Queries By Default
Every analytical query should include time bounds and clear units.
Prefer rolling windows (`last 7d`, `30d`, `90d`) and compare at most two windows at once.

Avoid unbounded full-history scans unless user explicitly asks.

### 7. Track Data Freshness and Refresh Points
Log last export timestamp in memory and warn when data is stale.
If user needs current-day insights, request a new iPhone export before claiming "latest" trends.

## Common Traps

- Assuming live HealthKit access from CLI agents -> setup fails because only exported data is available
- Using wrong export path in MCP env -> server starts but returns no data
- Running SQL before schema discovery -> queries fail on wrong table names
- Unbounded queries on large exports -> slow analysis and noisy output
- Reporting "today" metrics from stale export -> inaccurate recommendations
- Running MCP package on non-LTS Node -> DuckDB native module errors can break startup

## External Endpoints

| Endpoint | Data Sent | Purpose |
|----------|-----------|---------|
| https://registry.npmjs.org | Package install metadata only | Download MCP server package |
| https://raw.githubusercontent.com | Public markdown only | Read validated fallback skill docs |
| https://apps.apple.com | Manual app download traffic | Install CSV export app on iPhone |

No health record rows should be sent externally by default.

## Security & Privacy

Data that leaves your machine:
- Package install requests to npm
- Optional app download traffic from App Store

Data that stays local:
- Apple Health CSV exports
- MCP query outputs and summaries
- Skill memory in `~/apple-health/`

This skill does NOT:
- Access iCloud Health data directly
- Bypass Apple permission prompts
- Upload health CSVs unless the user asks for that explicitly

## Trust

By using this skill, you rely on third-party tooling (`@neiltron/apple-health-mcp` and the chosen iPhone export app).
Only install and run if you trust those tools.

## Related Skills
Install with `clawhub install <slug>` if user confirms:
- `health` - General health guidance boundaries and framing
- `ios` - iOS-specific setup and platform troubleshooting
- `sleep` - Sleep trend interpretation workflows
- `api` - Reliable API and integration debugging habits
- `swift` - HealthKit-side implementation context when app code is involved

## Feedback

- If useful: `clawhub star apple-health`
- Stay updated: `clawhub sync`
