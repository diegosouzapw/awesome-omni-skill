---
name: agent-rate-limiter
description: "Prevent 429s with automatic tier-based throttling & exponential backoff. Zero deps. By The Agent Wire (theagentwire.ai)"
homepage: https://theagentwire.ai
metadata: { "openclaw": { "emoji": "⚡" } }
---

# Never Hit 429s Again

You know the drill. Your agent is mid-task — browsing, spawning sub-agents, filing emails — and then:

```
rate_limit_error: You've exceeded your account's rate limit
```

Everything stops. Tokens wasted. Context lost. You restart manually, hope for the best, and hit it again 10 minutes later.

**This skill prevents that.** It tracks usage in a rolling window, assigns a tier (ok → cautious → throttled → critical → paused), and your agent automatically downshifts before hitting the wall. On a real 429, it calculates exponential backoff and schedules its own recovery.

No API keys. No pip installs. No external services. Just a Python script and a JSON state file.

Built by [The Agent Wire](https://theagentwire.ai) — an AI agent writing a newsletter about AI agents.

---

## 2-Minute Quick Start

Works out of the box with Claude Max 5x defaults. No config needed.

```bash
# 1. Test it works
python3 scripts/rate-limiter.py gate && echo "✅ Working"

# 2. Add to your agent loop
python3 scripts/rate-limiter.py gate || exit 1
python3 scripts/rate-limiter.py record 1000
```

That's it. Gate before work, record after. Everything else is tuning.

---

## Configuration

All optional. Defaults are conservative Claude Max 5x settings.

```bash
export RATE_LIMIT_PROVIDER="claude"          # claude | openai | custom
export RATE_LIMIT_PLAN="max-5x"             # max-5x | max-20x | plus | pro | custom
export RATE_LIMIT_STATE="/path/to/state.json"  # State file location
export RATE_LIMIT_WINDOW_HOURS="5"           # Rolling window duration
export RATE_LIMIT_ESTIMATE="200"             # Estimated request limit per window
```

### Provider Presets

| Provider | Plan | Window | Est. Limit | Notes |
|---|---|---|---|---|
| `claude` | `max-5x` | 5h | 200 | Conservative estimate |
| `claude` | `max-20x` | 5h | 540 | ~60% of theoretical max |
| `openai` | `plus` | 3h | 80 | GPT-4o messages |
| `openai` | `pro` | 3h | 200 | Higher tier |
| `custom` | — | configurable | configurable | Set your own |

Presets are starting points. Tune `RATE_LIMIT_ESTIMATE` based on your actual experience — every account behaves slightly differently.

---

## Tier System

| Tier | Trigger | Recommended Behavior |
|---|---|---|
| `ok` | <90% | Normal operations |
| `cautious` | 90%+ | Skip proactive/background checks |
| `throttled` | 95%+ | No sub-agents, terse responses, skip non-essential crons |
| `critical` | 98%+ | User messages only, 1 tool call max, all crons no-op |
| `paused` | 429 hit | Everything stops. Auto-resume timer handles recovery |

### Why 90 / 95 / 98?

These aren't arbitrary. Rate limit providers (Anthropic, OpenAI) start rejecting requests *before* you hit the hard cap — there are in-flight requests they can't account for, and their internal counters may differ from yours. The 90% threshold gives you a buffer to finish current work gracefully. By 95% you're in the danger zone where any burst could trigger a 429. At 98% you're one request away from a wall. The tiers create a smooth deceleration instead of a cliff.

---

## Commands

```bash
python3 scripts/rate-limiter.py <command> [args]

gate                    # Check tier, exit code reflects severity
record [tokens]         # Log a request (tokens optional, default 0)
status                  # Full status JSON (tier, pct, requests, limit, backoff info)
pause [minutes]         # Enter paused state (auto backoff if no minutes given)
resume                  # Clear pause, reset to cautious
set-limit <n>           # Override estimated request limit
reset                   # Reset all state to defaults
```

### Exit Codes

| Code | Meaning |
|---|---|
| `0` | ok or cautious — proceed |
| `1` | throttled — reduce activity |
| `2` | critical or paused — stop non-essential work |

---

## Complete Integration Example

A full loop showing gate check, conditional behavior, work, recording, and 429 handling:

```bash
#!/bin/bash
GATE=$(python3 scripts/rate-limiter.py gate 2>/dev/null)
EXIT=$?

if [ $EXIT -eq 2 ]; then
  echo "🛑 Critical/paused. Skipping work."
  exit 0
fi

if [ $EXIT -eq 1 ]; then
  echo "⚡ Throttled. Doing minimal work only."
  # skip sub-agents, background tasks, etc.
fi

# --- Do your actual work here ---
RESULT=$(your-agent-command 2>&1)

if echo "$RESULT" | grep -qi "rate_limit\|429"; then
  # Hit a 429 — pause with exponential backoff
  PAUSE_INFO=$(python3 scripts/rate-limiter.py pause)
  UNTIL=$(echo "$PAUSE_INFO" | python3 -c "import sys,json; print(json.load(sys.stdin).get('pausedUntil','unknown'))")
  echo "🛑 Rate limited. Paused until $UNTIL"
  exit 1
fi

# Record usage (estimate tokens based on your workload)
python3 scripts/rate-limiter.py record 2000
```

---

## Agent Integration

### In AGENTS.md / system prompt:

```markdown
## Rate Limiting

Before expensive operations: `python3 scripts/rate-limiter.py gate`
- Exit 0 → proceed normally
- Exit 1 → reduce activity (no spawns, terse responses)
- Exit 2 → stop all non-essential work

After significant work: `python3 scripts/rate-limiter.py record <approx_tokens>`

On 429 error:
1. `python3 scripts/rate-limiter.py pause`
2. Stop current work
3. Set a timer/cron to run `python3 scripts/rate-limiter.py resume` at the pausedUntil time
```

### In heartbeat checks:

```markdown
## Rate Limit Gate (ALWAYS FIRST)
Run: `python3 scripts/rate-limiter.py gate`
- Exit 2 → reply HEARTBEAT_OK immediately. Do nothing else.
- Exit 1 → skip proactive checks. Only handle urgent items.
- Exit 0 → proceed normally.
```

### In cron jobs:

Add to the start of any cron payload:
```
**FIRST: Rate limit gate check.** Run `python3 scripts/rate-limiter.py gate`.
If exit code is 2, reply 'RATE_LIMITED' and stop.
If exit code is 1, do only essential work.
```

---

## How It Works

```
Agent → gate check → tier (ok/cautious/throttled/critical/paused) → adjust behavior
Agent → after work → record usage → updates rolling estimate
Agent → on 429 → auto-pause with exponential backoff → auto-resume
```

This skill uses **heuristic estimation**, not API-level usage data. It counts requests within a rolling window and compares against a configurable limit.

**Why heuristic?** Neither Anthropic nor OpenAI expose a real-time usage API. The usage pages (claude.ai/settings/usage, chatgpt.com/settings) require browser auth and scraping. This skill works out of the box with zero external dependencies.

**Accuracy:** ~70-85% depending on how well the estimate matches your actual limit. Tune `RATE_LIMIT_ESTIMATE` down if you're hitting 429s, up if you're being too conservative.

**Improving accuracy:**
- Start conservative (default presets)
- If you hit 429 → the skill auto-adjusts via exponential backoff
- After a few days, check `status` to see your actual request patterns
- Tune the estimate based on real data

---

## State File

The skill writes a single JSON file (default: `./rate-limit-state.json`). Structure:

```json
{
  "provider": "claude",
  "plan": "max-5x",
  "tier": "ok",
  "estimatedPct": 23,
  "window": {
    "durationMs": 18000000,
    "requests": [{"ts": 1234567890, "tokens": 3000}],
    "estimatedLimit": 200
  },
  "backoff": {
    "consecutive429s": 0,
    "lastBackoffMs": 0
  },
  "pausedUntil": null
}
```

---

## Why Not Just Handle 429s Manually?

| Approach | Problem |
|---|---|
| **No handling** | Agent crashes, loses context, wastes tokens on retries |
| **Simple retry loop** | Hammers the API, makes backoff worse, no behavioral change |
| **Monitoring dashboard** | Tells you *after* you're rate limited. Doesn't prevent anything |
| **This skill** | Prevents 429s before they happen. Smooth deceleration. Auto-recovery. Zero dependencies. |

The key difference: this is **preventive**, not reactive. Your agent slows down *before* the wall, preserving context and avoiding wasted work.

---

## Troubleshooting

**Hitting 429s despite `ok` status**
Your estimate is too high. Lower it: `python3 scripts/rate-limiter.py set-limit 150` (or whatever feels right). The default presets are conservative, but your account's actual limit may be lower.

**State file corrupted**
Reset everything: `python3 scripts/rate-limiter.py reset`. This clears all history and starts fresh. You won't lose configuration — just re-export your env vars.

**Estimates feel way off**
Check your actual patterns: `python3 scripts/rate-limiter.py status`. Look at the request count vs. your limit. If you're at 50 requests and getting 429d, your limit estimate is way too high. If you're at 180/200 and never hitting limits, you can raise it.

**Multiple OpenClaw instances**
Each instance needs its own state file. Set `RATE_LIMIT_STATE` to a unique path per instance:
```bash
export RATE_LIMIT_STATE="/path/to/instance-1-rate-limit.json"
```
Otherwise they'll overwrite each other's tracking and the estimates will be meaningless.

---

## FAQ

**What is this skill?**
Agent Rate Limiter is a Python script that prevents AI agents from hitting API rate limits (429 errors) by tracking usage in a rolling window and automatically throttling before the limit is reached.

**What problem does it solve?**
AI agents on usage-capped plans (like Claude Max) burn through rate limits with no awareness, then hit 429 walls and stall. This skill adds self-awareness — the agent downshifts activity before hitting the wall and auto-recovers after backoff.

**What are the requirements?**
Python 3 (standard library only). No pip installs, no API keys, no external services. Just a script and a JSON state file.

**How does it work?**
A gate script checks the current tier (ok → cautious → throttled → critical → paused) before expensive operations. On a 429 error, it calculates exponential backoff with jitter and schedules recovery via cron. The agent reads the tier and adjusts behavior accordingly.

**Does it work with any LLM provider?**
Yes. It's provider-agnostic — tracks requests and estimated tokens against configurable limits. Works with Claude, GPT, Gemini, or any API with rate limits.
