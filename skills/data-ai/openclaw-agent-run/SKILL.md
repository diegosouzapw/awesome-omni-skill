---
name: openclaw-agent-run
description: Direct agent CLI invocations in OpenClaw. Use when running the `openclaw agent` command to trigger agent turns, targeting sessions or agents programmatically, delivering replies to channels, using --json output for automation, overriding thinking/verbose levels, or understanding session selection logic. Triggers for: "openclaw agent", "direct agent run", "send a message to agent", "--deliver", "--session-id", "agent turn", "CLI agent", "thinking level", "verbose mode", "--json output".
---

# Direct Agent Runs (`openclaw agent`)

## Basic Usage

```bash
# Target by agent ID
openclaw agent --agent main --message "Summarize the latest logs"

# Target by phone/channel destination (session key derived)
openclaw agent --to +15555550123 --message "Status update please"

# Reuse an existing session
openclaw agent --session-id {sessionId} --message "Continue from where we left off"
```

## Session Selection Rules

1. `--agent {id}` → targets that agent's `main` session key
2. `--to {dest}` → derives session from destination (group chats keep isolation; direct → `main`)
3. `--session-id {id}` → reuses exact session by ID (stable, OpenClaw-chosen IDs)
4. No flags → defaults to `main` agent on default session

## Output Modes

```bash
# Default: prints reply text (+ MEDIA:<url> lines for media)
openclaw agent --agent main --message "Hello"

# JSON: structured payload + metadata
openclaw agent --agent main --message "Hello" --json
```

JSON output shape:
```json
{
  "text": "...",
  "media": [],
  "sessionId": "...",
  "runId": "...",
  "agentId": "main",
  "model": "anthropic/claude-sonnet-4-5"
}
```

## Delivery to Channel

```bash
# Deliver reply to WhatsApp (default channel)
openclaw agent --to +15555550123 --message "Check the server" --deliver

# Deliver to specific channel and target
openclaw agent --agent ops --message "Generate report" \
  --deliver \
  --reply-channel slack \
  --reply-to "#reports"
```

## Thinking & Verbose Flags

Both **persist into the session store** (affect future turns too):

```bash
# Thinking levels: off | minimal | low | medium | high | xhigh
openclaw agent --agent main --message "Debug this" --thinking high

# Verbose levels: off | on | full
openclaw agent --agent main --message "Trace logs" --verbose full
```

Note: `--thinking` levels only work with models that support extended thinking (GPT-5.2, Codex, Claude 3.7+).

## Timeout Override

```bash
openclaw agent --agent main --message "Long analysis" --timeout 900
```

Default: 600s (10 min). Override for tasks known to be slow.

## Local vs Gateway

```bash
# Force local run (bypasses gateway, uses shell env API keys)
openclaw agent --local --agent main --message "Hello"

# Default: goes through gateway (http://localhost:18789)
# Falls back to local if gateway unreachable
```

See [references/automation-recipes.md](references/automation-recipes.md) for common automation patterns using `openclaw agent` in scripts.
