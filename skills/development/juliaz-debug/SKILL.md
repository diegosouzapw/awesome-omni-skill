---
name: juliaz-debug
description: "Cross-system diagnostics and troubleshooting for the juliaz_agents multi-agent system. Trigger when Raphael reports something is broken, not working, or behaving unexpectedly — messages not arriving, Julia not responding, bridge errors, orchestrator crashes, rate limits, silent failures, queue issues, or any 'why isn't X working' question. Also trigger for: 'debug', 'broken', 'not working', 'error', 'Julia isn't responding', 'messages stuck', 'bridge down', 'check logs', 'what went wrong', or any troubleshooting request. If something in the multi-agent system is misbehaving, this is the skill to reach for."
---

# juliaz-debug — Cross-System Diagnostics

> When something breaks in a multi-agent system, the problem is almost never where you first look. This skill provides systematic diagnostic procedures for the juliaz_agents ecosystem.

## First Response: Quick Health Scan

Before diving into any specific issue, always run this first:

```bash
# 1. What's running?
pm2 list 2>&1

# 2. Docker containers
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# 3. Who's on which port?
lsof -i :3000 -i :3001 -i :3002 -i :3003 -P -n 2>&1

# 4. Bridge health (the most common failure point)
curl -s http://localhost:3001/health 2>&1

# 5. Backend health
curl -s http://localhost:3000/health 2>&1

# 6. Cowork MCP health
curl -s http://localhost:3003/health 2>&1
```

This gives you the system state in 10 seconds. Read all results before forming a hypothesis.

## Diagnostic Decision Tree

```
"Julia isn't responding to Telegram messages"
├── Is bridge running? (pm2 list → check bridge status)
│   ├── NO → pm2 restart bridge → test again
│   └── YES → Are there pending messages?
│       ├── Check: curl http://localhost:3001/health (look at queue counts)
│       └── Messages stuck in "pending"?
│           ├── YES → Orchestrator not polling
│           │   ├── Is orchestrator running? (pm2 list)
│           │   │   ├── NO → pm2 restart orchestrator
│           │   │   └── YES → Check orchestrator logs: pm2 logs orchestrator --lines 50
│           │   │       ├── API key error → Check .env.secrets
│           │   │       ├── Rate limit (429) → Wait; check POLL_INTERVAL_MS
│           │   │       └── Connection error → Check bridge endpoint in orchestrator config
│           └── Messages stuck in "processing"?
│               └── Orchestrator picked up but didn't reply
│                   ├── Claude API timeout → Check pm2 logs orchestrator for timeout errors
│                   ├── Tool execution error → Check for failed tool calls in logs
│                   └── GPT-4o fallback also failed → Both APIs are down or keys expired

"Messages arrive but Julia's replies don't reach Telegram"
├── Is OpenClaw running? → openclaw health
│   ├── NO → openclaw gateway start --force
│   └── YES → Is it polling for replies?
│       ├── Check bridge: curl http://localhost:3001/health (look at openclaw heartbeat)
│       └── If stale heartbeat → OpenClaw relay skill may be broken
│           └── Check: openclaw/skills/julia-relay/ for issues

"Frontend chat not working"
├── Is frontend running? → curl http://localhost:3002
├── Is frontend chat endpoint OK? → curl -X POST http://localhost:3002/api/chat
├── Check: OPENAI_API_KEY in environment (frontend uses GPT-4o)
└── Check: frontend/app/api/chat/route.ts for errors

"Cowork MCP returning errors"
├── Is it running? → curl http://localhost:3003/health
├── Check API key: ANTHROPIC_API_KEY in .env.secrets
├── Check logs: pm2 logs cowork-mcp --lines 50
└── Common: 401 = bad key, 429 = rate limit, 500 = bug in tool handler
```

## Reading Logs Effectively

### PM2 Logs
```bash
# Last 50 lines for a service
pm2 logs orchestrator --lines 50
pm2 logs bridge --lines 50

# All logs, live tail
pm2 logs

# Error logs only (separate file)
# Located at: ~/.pm2/logs/<service>-error.log
```

### What to look for in orchestrator logs:
- `Polling bridge...` — confirms it's alive and checking
- `Processing message from chatId: ...` — message was picked up
- `Claude response:` — successful reply generated
- `Falling back to GPT-4o` — Claude API failed, trying backup
- `Error:` — any error with stack trace
- `Rate limited` or `429` — too many API calls
- `Timeout` — API call took >30s

### What to look for in bridge logs:
- `POST /incoming` — message received from OpenClaw
- `GET /pending-reply/:chatId` — OpenClaw polling for replies
- `MCP tool call:` — orchestrator calling bridge tools
- `Queue size:` — how many messages are buffered

### What to look for in cowork-mcp logs:
- `Tool call:` — which tool was invoked
- `Anthropic API error:` — what went wrong with Claude
- `Truncated response` — output was over 25K chars (content lost!)

## Common Failure Patterns

### 1. The Silent Death
**Symptom**: PM2 shows service as "online" but it's not actually working.
**Diagnosis**: Check restart count in `pm2 list`. If it's high (>5), the service is crash-looping.
**Fix**: `pm2 logs <service> --lines 100` to find the crash cause. Often: missing env var, port conflict, or syntax error after a code change.

### 2. The Rate Limit Spiral
**Symptom**: Orchestrator logs show repeated 429 errors, Julia stops responding.
**Diagnosis**: Orchestrator has exponential backoff (up to 55s), but if POLL_INTERVAL_MS is too low, it can hammer the API.
**Fix**: Increase `POLL_INTERVAL_MS` in ecosystem config. Wait for rate limit to clear (usually 60s). The orchestrator honors `Retry-After` headers.

### 3. The Queue Jam
**Symptom**: Messages pile up in bridge, none get processed.
**Diagnosis**: `curl http://localhost:3001/health` — check pending count. If messages are in "processing" state, the orchestrator grabbed them but never replied.
**Fix**: Check orchestrator logs. If it crashed mid-processing, messages stay in "processing" forever. Manual fix: edit `bridge/data/queue.json` and reset status to "pending".

### 4. The Memory Leak
**Symptom**: Service gradually slows down, then crashes after hours/days.
**Diagnosis**: Check PM2 memory usage: `pm2 monit`. If a service exceeds ~500MB, it's likely leaking.
**Fix**: PM2 has `max_memory_restart` — add to ecosystem config if not present. For bridge specifically, the queue.json file grows unbounded (no pruning of old replied messages).

### 5. The Docker Ghost
**Symptom**: Backend returns connection errors even though `pm2 list` shows everything "online".
**Diagnosis**: Docker containers may have stopped (Docker Desktop restart, system reboot).
**Fix**: `docker ps` to check. If containers are gone: `cd backend && docker compose up -d`.

### 6. The Fallback Loop
**Symptom**: Julia responds but much slower than usual, and responses feel different.
**Diagnosis**: Claude API is failing and orchestrator is falling back to GPT-4o every time.
**Fix**: Check orchestrator logs for the specific Claude error. Common: expired API key, billing issue, or Anthropic outage. Check https://status.anthropic.com.

### 7. The Hardcoded Path Break
**Symptom**: Email sending fails, or tools that call external scripts fail.
**Diagnosis**: The orchestrator has hardcoded paths like `/Users/raphael/Documents/Devs/juliaz_agents/openclaw/skills/email-aberer`.
**Fix**: If running on a different machine or path, update the paths in `orchestrator/src/tools.ts`.

## Queue Inspection

The bridge queue is a JSON file. Direct inspection when the API is misbehaving:

```bash
# View queue contents
cat bridge/data/queue.json | python3 -m json.tool

# Count messages by status
cat bridge/data/queue.json | python3 -c "
import json, sys
q = json.load(sys.stdin)
msgs = q.get('messages', [])
from collections import Counter
c = Counter(m.get('status') for m in msgs)
for status, count in c.items():
    print(f'{status}: {count}')
print(f'Total: {len(msgs)}')
"
```

### Manually unstick messages
If messages are stuck in "processing":
```bash
# Read, modify status, write back
python3 -c "
import json
with open('bridge/data/queue.json', 'r') as f:
    q = json.load(f)
for m in q.get('messages', []):
    if m.get('status') == 'processing':
        m['status'] = 'pending'
with open('bridge/data/queue.json', 'w') as f:
    json.dump(q, f, indent=2)
print('Done — stuck messages reset to pending')
"
```

## Database Diagnostics

```bash
# Check memory count
curl -s http://localhost:3000/memories | python3 -c "import sys,json; m=json.load(sys.stdin); print(f'{len(m)} memories')"

# Check task count
curl -s http://localhost:3000/tasks | python3 -c "import sys,json; t=json.load(sys.stdin); print(f'{len(t)} tasks')"

# Check usage/token consumption
curl -s http://localhost:3000/usage | python3 -c "
import sys, json
u = json.load(sys.stdin)
total = sum(r.get('totalTokens', 0) for r in u)
print(f'{len(u)} usage records, {total:,} total tokens')
"

# Check recent logs
curl -s http://localhost:3000/logs | python3 -c "
import sys, json
logs = json.load(sys.stdin)
for l in logs[-10:]:
    print(f'[{l.get(\"level\",\"?\")}] {l.get(\"source\",\"?\")} — {l.get(\"message\",\"?\")[:80]}')
"
```

## Network Connectivity Check

When services can't talk to each other:

```bash
# Test bridge from orchestrator's perspective
curl -s http://localhost:3001/health

# Test backend from bridge's perspective
curl -s http://localhost:3000/health

# Test cowork-mcp from orchestrator's perspective
curl -s http://localhost:3003/health

# Test MCP endpoint specifically
curl -s -X POST http://localhost:3001/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":1}'
```

## Escalation Path

If you can't fix it with these tools:

1. **Check recent git changes**: `git log --oneline -10` — was something deployed recently?
2. **Diff against working state**: `git diff HEAD~5` — what changed?
3. **Nuclear option**: Stop everything, rebuild, restart fresh:
   ```bash
   pm2 delete all
   cd backend && docker compose down
   cd backend && npm run build
   cd bridge && npm run build
   cd orchestrator && npm run build
   cd cowork-mcp && npm run build
   cd frontend && npx next build
   cd backend && docker compose up -d
   pm2 start ecosystem.dev.config.js
   openclaw gateway start --force
   ```
