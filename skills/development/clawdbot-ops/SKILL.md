---
name: clawdbot-ops
description: Use when starting, stopping, diagnosing, or troubleshooting Clawdbot. Includes gateway management, health checks, and common fixes.
---

# Clawdbot Operations Guide

## Quick Reference

### Start Clawdbot

```bash
# Start gateway (foreground, see logs)
clawdbot gateway start

# Start gateway as background daemon
clawdbot gateway start --daemon

# Check if running
clawdbot gateway status
```

### Stop Clawdbot

```bash
# Stop gateway
clawdbot gateway stop

# Force stop if hung
clawdbot gateway stop --force
```

### Health Check & Diagnosis

```bash
# Full health check with auto-fix suggestions
clawdbot doctor

# View recent logs
clawdbot logs

# View logs in real-time
clawdbot logs --follow
```

## Common Issues & Fixes

### Issue: Gateway won't start

```bash
# 1. Check if already running
clawdbot gateway status

# 2. Kill zombie processes
pkill -f clawdbot

# 3. Check port conflicts (default: 18789)
lsof -i :18789

# 4. Reset and restart
clawdbot gateway stop --force
clawdbot gateway start
```

### Issue: RPC probe failed / Gateway connection error

Symptoms: `gateway closed (1006 abnormal closure)`, `Gateway agent failed; falling back to embedded`

```bash
# 1. Kill all clawdbot processes completely
pkill -9 -f clawdbot

# 2. Wait and reinstall gateway
sleep 2 && clawdbot gateway install

# 3. Wait for startup and verify
sleep 5 && clawdbot gateway status

# Should show: RPC probe: ok
```

### Issue: Telegram/Discord not connecting

```bash
# 1. Run doctor to diagnose
clawdbot doctor

# 2. Reconfigure credentials
clawdbot configure

# 3. Check device pairing
clawdbot devices list
```

### Issue: Config corruption

```bash
# Backup exists at ~/.clawdbot/clawdbot.json.bak
cp ~/.clawdbot/clawdbot.json.bak ~/.clawdbot/clawdbot.json

# Or reset config entirely
clawdbot reset --config
```

### Issue: Skills not loading

```bash
# Check skills status
clawdbot plugins list

# Reinstall a skill
npx clawhub@latest install <skill-name> --force
```

## Full Command Reference

| Command | Description |
|---------|-------------|
| `clawdbot setup` | Initialize config and workspace |
| `clawdbot onboard` | Interactive setup wizard |
| `clawdbot configure` | Configure credentials & devices |
| `clawdbot doctor` | Health checks + quick fixes |
| `clawdbot gateway start` | Start the gateway |
| `clawdbot gateway stop` | Stop the gateway |
| `clawdbot gateway status` | Check gateway status |
| `clawdbot logs` | View gateway logs |
| `clawdbot logs --follow` | Tail logs in real-time |
| `clawdbot dashboard` | Open Control UI |
| `clawdbot devices list` | List paired devices |
| `clawdbot reset` | Reset local config/state |
| `clawdbot uninstall` | Full uninstall |

## Config Locations

| Path | Purpose |
|------|---------|
| `~/.clawdbot/clawdbot.json` | Main config |
| `~/.clawdbot/clawdbot.json.bak` | Config backup |
| `~/.clawdbot/logs/` | Log files |
| `~/.clawdbot/credentials/` | API keys & tokens |
| `~/.clawdbot/telegram/` | Telegram session |
| `~/.clawdbot/memory/` | Agent memory |

## Dev Mode

For development/testing with isolated state:

```bash
# Use dev profile (state in ~/.clawdbot-dev, port 19001)
clawdbot --dev gateway start

# Or use named profile
clawdbot --profile test gateway start
```

## Startup Checklist

Before starting Clawdbot, verify:

1. **Config exists**: `ls ~/.clawdbot/clawdbot.json`
2. **No zombie processes**: `pgrep -f clawdbot`
3. **Port available**: `lsof -i :18789`
4. **Credentials set**: `clawdbot doctor`

## Emergency Recovery

If everything is broken:

```bash
# 1. Kill all processes
pkill -9 -f clawdbot

# 2. Backup current config
cp ~/.clawdbot/clawdbot.json ~/.clawdbot/clawdbot.json.emergency-backup

# 3. Reset (keeps CLI installed)
clawdbot reset

# 4. Re-run setup
clawdbot onboard
```
