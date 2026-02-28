---
name: infra-guardian
description: "OpenClaw Agent Infrastructure Guardian — keep your agent's infrastructure alive. Process lifecycle management with detached execution, auto-restart on failure. Cron scheduler health monitoring (per-job detection, auto-recovery). Direct Telegram/messaging alerts independent of OpenClaw. System-level watchdog that runs from crontab, not OpenClaw cron. Use when launching background processes, monitoring cron job health, or when things keep dying silently."
---

# Infra Guardian

Keep your OpenClaw agent's infrastructure alive. Processes, cron jobs, the works.

## What It Does

| Layer | What | How |
|-------|------|-----|
| **Process Management** | Launch, track, auto-restart background processes | `setsid` + `nohup` + registry + healthcheck |
| **Cron Health** | Detect stalled OpenClaw cron jobs per-job | Reads `jobs.json` from disk, checks `nextRunAtMs` vs interval |
| **Auto-Recovery** | Restart gateway when cron scheduler stalls | SIGUSR1 when >50% jobs overdue |
| **Alerting** | Telegram alerts independent of OpenClaw | Direct Bot API calls — works even if OpenClaw is down |

**Core principle:** Monitoring components must not depend on the system they monitor.

## Quick Start

```bash
# Process management
bash scripts/managed-process.sh register my-bot "python3 /path/to/bot.py" 480
bash scripts/managed-process.sh start my-bot
bash scripts/managed-process.sh status

# Infrastructure watchdog (add to system crontab, NOT OpenClaw cron)
*/10 * * * * /path/to/scripts/managed-process.sh watchdog
```

## Commands

| Command | Usage | Description |
|---------|-------|-------------|
| `register` | `register <name> <command> [duration_min]` | Define a managed process. Duration 0 = indefinite. |
| `start` | `start <name>` | Launch registered process (fully detached). |
| `stop` | `stop <name>` | Graceful shutdown via SIGTERM. |
| `restart` | `restart <name>` | Stop then start. |
| `status` | `status [name]` | Show all processes or one specific. |
| `healthcheck` | `healthcheck` | Check all registered processes, restart dead ones. |
| `watchdog` | `watchdog` | **Unified check:** cron health + process health (for system crontab). |
| `cron-health` | `cron-health` | Check OpenClaw cron scheduler per-job health. |
| `proc-health` | `proc-health` | Check key process liveness (configurable patterns). |
| `deregister` | `deregister <name>` | Remove process from registry + clean up files. |

## Infrastructure Watchdog

The `watchdog` command is the unified health check. Run it from **system crontab** — never from OpenClaw cron (you can't monitor the scheduler using the scheduler).

```bash
# Unified (recommended)
bash scripts/managed-process.sh watchdog

# Individual checks
bash scripts/managed-process.sh cron-health
bash scripts/managed-process.sh proc-health
```

### Cron Health (`cron-health`)

Reads OpenClaw's cron state directly from disk (`~/.openclaw/cron/jobs.json`). No API dependency.

**Per-job detection:**
- Each enabled job checked independently
- `nextRunAtMs` overdue by >2× its interval → STALE
- Jobs with `kind: "every"` use their `everyMs` as interval
- Jobs with `kind: "cron"` use 24h as max expected interval

**Auto-recovery:**
- If >50% of jobs are stale → sends SIGUSR1 to restart gateway
- Alerts via Telegram Bot API directly (reads bot token from OpenClaw config)
- 30-minute cooldown between alerts (no spam)

**Why this matters:** OpenClaw has a known cron bug ([#8424](https://github.com/openclaw/openclaw/issues/8424)) where `kind: "cron"` jobs permanently stall after missing a run. This watchdog catches it within 20 minutes instead of discovering it 8 hours later.

### Process Health (`proc-health`)

Checks known background processes via `pgrep`. Configurable patterns in the script.

- Alerts via Telegram if any monitored process is down
- 30-minute cooldown

### Setup

```bash
# Add to system crontab (not OpenClaw cron!)
crontab -e
# Add this line:
*/10 * * * * /home/clawdbot/clawd/scripts/managed-process.sh watchdog
```

## Process Management

### The Problem

When AI agents launch background processes via `exec &` or `nohup`, those processes are tied to the parent session. Session ends → child processes get SIGTERM → die silently → nobody knows.

### The Rule

**ALL long-running processes MUST go through this framework.** No exceptions.

- ❌ `python script.py &`
- ❌ `nohup python script.py &`
- ❌ `exec` with `background: true`
- ✅ `bash scripts/managed-process.sh register <name> <cmd>` then `start <name>`

### Detached Execution

Processes launch via `setsid` + `nohup` + `disown`, giving them:
- Own session ID (SID) — not tied to any terminal
- PPID=1 (init) — survives parent death
- Immune to SIGHUP/session cleanup

### Health Monitoring

The `healthcheck` command (can run from cron every 5 min):
1. Checks each registered process is alive (PID exists)
2. If dead: checks if it completed normally (ran ≥80% of expected duration)
3. If premature death: auto-restarts and writes alert flag
4. Alert cooldown: 15 min between alerts (no spam)

### Alert Integration

When a process dies, the healthcheck writes:
- `/tmp/process_monitor_alert.flag` — trigger file
- `/tmp/process_monitor_alert.txt` — alert message

Configure your agent's heartbeat to check these files and forward alerts.

### Process Registry

All processes are tracked in `.process-registry.json`:

```json
{
  "my-bot": {
    "command": "python3 /path/to/bot.py",
    "duration_min": 480,
    "auto_restart": true,
    "max_restarts": 5,
    "restart_cooldown": 30
  }
}
```

## Signal Handling Best Practice

Your scripts should log signals, not silently exit:

```python
import signal
from datetime import datetime

def handler(signum, frame):
    sig_name = signal.Signals(signum).name
    print(f"⚠️ SIGNAL: {sig_name} at {datetime.now()}", flush=True)
    global shutdown
    shutdown = True

signal.signal(signal.SIGTERM, handler)
signal.signal(signal.SIGINT, handler)
```

Never call `sys.exit(0)` in signal handlers — it makes crashes look like clean exits, preventing watchdogs from restarting.

## Architecture

```
System Crontab (every 10 min)
  └─ managed-process.sh watchdog
       ├─ cron-health: reads ~/.openclaw/cron/jobs.json
       │    ├─ per-job: nextRunAtMs overdue > 2× interval?
       │    ├─ if >50% stale → SIGUSR1 gateway restart
       │    └─ alert → Telegram Bot API (direct, no OpenClaw)
       └─ proc-health: pgrep known patterns
            └─ alert → Telegram Bot API (direct, no OpenClaw)
```

**Independence chain:** System crontab → bash script → disk read → direct Telegram API. Zero OpenClaw dependencies in the monitoring path.
