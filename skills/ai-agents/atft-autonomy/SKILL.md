---
name: atft-autonomy
description: Coordinate Claude Code skills with OpenAI Codex autonomous workflows for end-to-end ATFT-GAT-FAN maintenance.
proactive: true
---

# ATFT Autonomy Skill

## Mission
- Seamlessly orchestrate Claude and Codex agents to keep the ATFT-GAT-FAN stack production-ready.
- Decide when to run deep Codex analyses versus fast Claude remediations.
- Maintain shared context (AGENTS.md, health snapshots, run logs) across agents.

## Engagement Signals
- Requests to “run full autonomous maintenance”, “coordinate Claude and Codex”, or “schedule daily self-healing”.
- Situations where an issue spans multiple domains (dataset + training + quality).
- Need to generate or refresh `.mcp.json`, `AGENTS.md`, or other shared configs.

## Preflight Checklist
1. Confirm tool availability:
   - `command -v codex` (expect npm package `@openai/codex`).
   - `command -v ./tools/claude-code.sh`.
2. Verify configuration files:
   - `.mcp.json` exists with filesystem/git servers (`tools/codex.sh` recreates if missing).
   - `AGENTS.md` up to date with latest guidelines.
3. Run snapshot diagnostics:
   - `tools/project-health-check.sh --summary` (logs to `_logs/health-checks/`).
   - `nvidia-smi` and `df -h` to ensure resource headroom before long autonomous jobs.

## Core Playbooks

### 1. Full Daily Autonomy Loop
1. `./tools/claude-code.sh --no-check "Run proactive maintenance checklist"` — quick fixes + TODO updates.
2. `./tools/codex.sh --max --exec "Perform deep optimization sweep across dataset, training, and research modules"` — longer Codex session with MCP.
3. Append combined findings to `docs/ops/autonomy_log.md` (capture commands run, deltas, next steps).
4. `git status --short` → review diffs, ensure no unintended drift.

### 2. Incident Response (multi-domain failure)
1. Launch Claude for rapid triage: `./tools/claude-code.sh "Investigate failed training and prepare briefing for Codex"`.
2. Pass Claude findings to Codex:  
   `./tools/codex.sh --exec "Read docs/ops/incident_brief.md and propose remedial plan"`.
3. Execute agreed actions (dataset rebuild, training rerun, quality checks).
4. Update `docs/ops/incident_log.md` with both agents’ actions and resolutions.

### 3. Scheduled Autonomous Optimization
1. Add cron entry (example):  
   `0 2 * * * cd /workspace/gogooku3 && ./tools/codex.sh --max >> _logs/daily-optimization.log 2>&1`.
2. Pair with weekly Claude sweep:  
   `0 7 * * MON cd /workspace/gogooku3 && ./tools/claude-code.sh --no-check >> _logs/weekly-claude.log 2>&1`.
3. Summarize improvements weekly by aggregating `_logs/codex-sessions/*.log` and `_logs/claude-code/*.log`; record highlights in `docs/ops/weekly_autonomy_report.md`.

## Shared Context Management
- Ensure both agents read `CLAUDE.md` and `AGENTS.md` before editing high-risk files.
- Rotate `docs/SKILLS_GUIDE.md` and `claude/skills/` when procedures change.
- Store cross-agent decisions in `EXPERIMENT_STATUS.md` or `docs/ops/autonomy_log.md`.

## Failure Handling
- **Codex CLI missing** → `npm install -g @openai/codex`.
- **MCP server errors** → regenerate `.mcp.json` via `./tools/codex.sh` and restart run.
- **Conflicting edits** → consolidate drafts under `docs/ops/autonomy_pending/` and resolve manually.
- **Resource saturation** → stagger Claude/Codex runs, cap concurrency in cron.

## Handoff
- After combined sessions, notify stakeholders by updating `docs/ops/weekly_autonomy_report.md`.
- Attach session logs `_logs/codex-sessions/*.log` and `_logs/claude-code/*.log` for audit.
- Re-sync `~/.claude/skills/` so Claude picks up any procedural updates discovered during Codex runs.
