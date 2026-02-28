---
name: codesession
description: codesession (codesession-cli, code session, code-session) — Track AI agent session costs, tokens, file changes, and git commits. Works with Claude Code, OpenClaw, Codex, GPT, Cursor, Windsurf, Cline & any AI agent. Budget enforcement, auto-pricing, MCP server, web dashboard, alerts, insights. v2.3.0.
metadata: {"openclaw": {"homepage": "https://github.com/brian-mwirigi/codesession-cli", "requires": {"bins": ["cs"]}, "install": [{"id": "npm", "kind": "node", "package": "codesession-cli", "bins": ["cs"], "label": "Install codesession-cli (npm)"}]}}
---

# Session Cost Tracking (codesession-cli)

Track agent session costs, file changes, and git commits. Enforces budget limits and provides detailed session analytics with a full web dashboard.

**Latest: v2.2.0** - Claude Code MCP plugin, alerts dashboard, start fresh, insights page, alarm mode.

📦 [npm](https://www.npmjs.com/package/codesession-cli) • ⭐ [GitHub](https://github.com/brian-mwirigi/codesession-cli) • 📝 [Changelog](https://github.com/brian-mwirigi/codesession-cli/blob/main/CHANGELOG.md)

## Installation

```bash
# 1. Install the CLI globally from npm
npm install -g codesession-cli

# 2. Install the OpenClaw skill
clawhub install codesession
```

After installing, the `cs` command is available globally. The OpenClaw agent will automatically use it to track sessions.

> **Requirements:** Node.js 18+ and C/C++ build tools (needed to compile the embedded SQLite module).
>
> | OS | Install build tools |
> |---|---|
> | **Ubuntu/Debian** | `sudo apt-get install -y build-essential python3` |
> | **macOS** | `xcode-select --install` |
> | **Windows** | `npm install -g windows-build-tools` or install Visual Studio Build Tools |
> | **Alpine** | `apk add build-base python3` |
>
> Data is stored locally at `~/.codesession/sessions.db`.

## When to use

- **Always** start a tracked session at the beginning of a multi-step task
- **Always** log AI usage after each API call you make
- **Always** end the session when the task is complete
- Check budget before expensive operations
- Use `cs dashboard` to review session data in a browser

## Commands

### Start tracking
```bash
# Agent mode (always use --json for structured output):
cs start "task description" --json --close-stale

# Resume if a session was left open (e.g. after a crash):
cs start "task description" --json --resume

# Human/interactive mode (stays running with live file watcher):
cs start "task description"
```

> **Agent mode vs interactive mode:** With `--json`, the session is created in the database, JSON is printed, and the process exits immediately -- the session stays "active" and tracks git changes when you run `cs end`. Without `--json`, the process stays running with a live file watcher and git commit poller until you press Ctrl+C or run `cs end` in another terminal.

### Log AI usage (after each API call)
```bash
# With granular tokens (cost auto-calculated from built-in pricing):
cs log-ai -p anthropic -m claude-sonnet-4 --prompt-tokens 8000 --completion-tokens 2000 --json

# With agent name tracking (NEW in v1.9.1):
cs log-ai -p anthropic -m claude-sonnet-4 --prompt-tokens 8000 --completion-tokens 2000 --agent "Code Review Bot" --json

# With manual cost:
cs log-ai -p anthropic -m claude-opus-4-6 -t 15000 -c 0.30 --json

# With all fields:
cs log-ai -p openai -m gpt-4o --prompt-tokens 5000 --completion-tokens 1500 -c 0.04 --agent "Research Agent" --json
```
Providers: `anthropic`, `openai`, `google`, `mistral`, `deepseek`
Cost is auto-calculated from a configurable pricing table (17+ built-in models). Use `cs pricing list --json` to see known models. If a model is unknown, provide `-c <cost>` manually.

**Agent Name (optional):** Use `--agent "Agent Name"` to track which agent performed the work. Perfect for multi-agent systems, A/B testing, and cost attribution. Agent names appear in the dashboard and can be used to filter/analyze costs per agent.

### Check current status
```bash
cs status --json
```
Returns JSON with current session cost, tokens, files changed, duration. All JSON responses include `schemaVersion` and `codesessionVersion` fields.

### End session and get summary
```bash
cs end -n "completion notes" --json
```
When ending, codesession automatically scans git for all files changed and commits made since the session started -- even if `--json` mode was used (no live watcher needed).

### Web Dashboard
```bash
cs dashboard
# Opens http://localhost:3737 with full analytics UI

cs dashboard --port 4000       # custom port
cs dashboard --no-open         # don't auto-open browser
```

The dashboard shows:
- **Overview** — KPIs, daily cost/token trends, spend projections, cost velocity
- **Sessions** — searchable/sortable table, per-session detail with timeline, files, commits, AI calls, notes
- **Models** — per-model & per-provider cost breakdown, token ratios, usage charts
- **Insights** — file hotspots, activity heatmap, project breakdown, pricing table
- **Alerts** — set daily/total/per-session cost thresholds with alarm mode (sound + browser notifications)
- **Start Fresh** — reset all session data from the sidebar

### View session details
```bash
cs show --json --files --commits
```

### View historical stats
```bash
cs stats --json
```

### Export sessions
```bash
cs export --format json --limit 10
cs export --format csv
```

### Add notes / annotations
```bash
cs note "Starting refactor phase"
cs note "Tests passing, moving to cleanup"
```
Timestamped annotations appear in `cs show --json` under `annotations`.

### Recover stale sessions
```bash
cs recover --max-age 12
```
Auto-ends any active sessions older than 12 hours.

## Agent Workflow

Agents should **always** use `--json` on every command for structured, parseable output.

1. At task start: `cs start "Fix authentication bug" --json --close-stale`
2. Add context notes: `cs note "analyzing auth flow" --json`
3. After each AI call: `cs log-ai -p anthropic -m claude-sonnet-4 --prompt-tokens 8000 --completion-tokens 2000 --agent "Bug Fixer" --json`
4. Check spend: `cs status --json` -- read `aiCost` field
5. At task end: `cs end -n "Fixed the auth bug, added tests" --json`
6. Review past sessions: `cs dashboard`

**Tip:** Use the `--agent` flag to identify your agent's work, especially useful in multi-agent systems where different agents handle different tasks (e.g., "Code Review Bot", "Test Writer", "Documentation Agent").

## Pricing

Pricing is configurable. Run `cs pricing list` to see all known model prices. Override or add models:

```bash
# Plain model key
cs pricing set my-model 5.00 15.00

# Provider-namespaced key (avoids collisions)
cs pricing set gpt-4o 2.50 10.00 --provider openai
```

If the model isn't in the pricing table, you must provide `-c <cost>` when logging.

## Budget awareness

If the user has set a budget or you detect high spending:
- Check `cs status --json` before expensive operations
- Warn the user if `aiCost` exceeds $5.00 in a single session
- Suggest cheaper models if costs are escalating

## Important

- Use `--close-stale` on `cs start` to avoid "session_active" errors from prior crashes
- If `cs` is not installed, skip session tracking — don't block the user's task
- **Always** use `--json` on every command — agents must use structured output
- With `--json`, `cs start` creates the session and exits (no long-running process needed)
- Sessions are scoped by **git root** — running from a subdirectory still matches the repo-level session
- On errors in `--json` mode, exit code is always `1` and the response has `{ "error": { "code": "...", "message": "..." } }`
- Check `schemaVersion` in JSON responses to detect breaking changes

## Update check

Periodically check if the CLI is up to date (e.g. at session start or once per day):

```bash
# Compare installed version to latest on npm:
cs --version          # installed version
npm view codesession-cli version  # latest on npm
```

If the installed version is behind, notify the user:
> "codesession-cli update available: v1.9.4 -> v2.0.0. Run `npm install -g codesession-cli` to update."

Don't block the workflow for an update — just inform and continue.

## JSON output

All commands support `--json` for machine-readable output. Use this when you need to parse session data programmatically.
