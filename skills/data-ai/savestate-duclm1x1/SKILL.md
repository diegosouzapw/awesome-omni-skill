---
name: savestate
description: Time Machine for AI. Encrypted backup, restore, and cross-platform migration for your agent's memory and identity. Supports OpenClaw, ChatGPT, Claude, Gemini, and more. AES-256-GCM encryption with user-controlled keys.
user-invocable: true
metadata: {"openclaw":{"emoji":"💾","primaryEnv":"SAVESTATE_API_KEY"}}
---

# SaveState — Time Machine for AI

SaveState creates encrypted point-in-time snapshots of your AI agent's state — memory, identity, conversations, and configuration. Unlike live-sync tools, SaveState gives you versioned backups you can restore, compare, and migrate across platforms.

**Key differentiators:**
- 🔐 AES-256-GCM encryption with user-controlled keys
- 🔄 Cross-platform migration (ChatGPT → Claude → OpenClaw, etc.)
- 📊 Incremental snapshots with diff comparison
- ⏰ Scheduled automatic backups (Pro/Team)
- ☁️ Cloud storage with zero-knowledge encryption (Pro/Team)

## Installation

```bash
# npm
npm install -g @savestate/cli

# Homebrew
brew tap savestatedev/tap && brew install savestate

# Direct install
curl -fsSL https://savestate.dev/install.sh | sh
```

## Quick Start

### Initialize (first time)
```bash
savestate init
```

This creates a `.savestate/` directory with your encryption key. **Back up your key** — it's the only way to decrypt your snapshots.

### Create a snapshot
```bash
savestate snapshot
```

Captures your current agent state to an encrypted archive.

### List snapshots
```bash
savestate list
# or
savestate ls
```

### Restore from snapshot
```bash
# Restore latest
savestate restore

# Restore specific snapshot
savestate restore ss-2026-02-01T12-00-00
```

### Compare snapshots
```bash
savestate diff ss-2026-01-15 ss-2026-02-01
```

## Platform Adapters

SaveState works with multiple AI platforms:

| Platform | Adapter | Capabilities |
|----------|---------|--------------|
| **OpenClaw** | `openclaw` | Full backup & restore |
| **Claude Code** | `claude-code` | Full backup & restore |
| **OpenAI Assistants** | `openai-assistants` | Full backup & restore |
| **ChatGPT** | `chatgpt` | Export + memory restore |
| **Claude.ai** | `claude` | Export + memory restore |
| **Gemini** | `gemini` | Export (via Takeout) |

List available adapters:
```bash
savestate adapters
```

## Cross-Platform Migration

Migrate your AI's identity between platforms:

```bash
# Migrate from ChatGPT to Claude
savestate migrate --from chatgpt --to claude

# Restore a ChatGPT snapshot to OpenClaw
savestate restore ss-chatgpt-2026-01-15 --to openclaw
```

## Cloud Storage (Pro/Team)

With a Pro ($9/mo) or Team ($29/mo) subscription:

```bash
# Login to SaveState cloud
savestate login

# Push snapshots to cloud
savestate cloud push

# Pull from cloud on new device
savestate cloud pull

# Schedule automatic backups
savestate schedule --every 6h
```

Sign up at https://savestate.dev

## What Gets Backed Up

### OpenClaw/Clawdbot
- `SOUL.md`, `IDENTITY.md`, `USER.md` — Identity files
- `MEMORY.md`, `memory/*.md` — Memory and daily logs
- `TOOLS.md`, `HEARTBEAT.md` — Configuration
- `skills/` — Installed skills and customizations
- Session transcripts (optional)

### Claude Code
- `CLAUDE.md` — System prompt
- `.claude/` — Settings and memory
- Project manifest and todos

### ChatGPT/Claude.ai/Gemini
- Conversation history export
- Custom instructions / system prompts
- Memory entries

## Automation Examples

### Cron backup (OpenClaw heartbeat)
Add to `HEARTBEAT.md`:
```
## SaveState backup check
- If more than 24h since last snapshot, run: savestate snapshot
- Check with: savestate ls --json | jq '.[0].timestamp'
```

### Pre-migration checklist
Before switching platforms:
1. `savestate snapshot` — Create fresh backup
2. `savestate cloud push` — Sync to cloud (if Pro)
3. `savestate migrate --from X --to Y` — Run migration

## Security

- **Encryption**: AES-256-GCM with Argon2id key derivation
- **Zero-knowledge cloud**: We only store encrypted blobs
- **User-controlled keys**: You own your encryption key
- **No telemetry**: CLI doesn't phone home

## API Reference

```bash
savestate --help              # Show all commands
savestate <command> --help    # Command-specific help
savestate --version           # Show version (currently 0.4.2)
```

## Links

- **Website**: https://savestate.dev
- **GitHub**: https://github.com/savestatedev/savestate
- **npm**: https://npmjs.com/package/@savestate/cli
- **Support**: hello@savestate.dev

## Comparison: SaveState vs Live-Sync

| Feature | SaveState | Live-sync tools |
|---------|-----------|-----------------|
| Point-in-time restore | ✅ | ❌ |
| Version history | ✅ | ❌ |
| Cross-platform migration | ✅ | ❌ |
| Snapshot comparison | ✅ | ❌ |
| Multi-platform support | ✅ 6 platforms | Usually 1 |
| Continuous sync | ❌ (scheduled) | ✅ |

SaveState is complementary to live-sync — use both for maximum protection.
