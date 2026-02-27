---
name: mnemonic
description: Unified memory system - aggregates communications and AI sessions across all channels into searchable, analyzable memory
homepage: https://github.com/Napageneral/mnemonic
metadata: {"nexus":{"emoji":"🧠","os":["darwin","linux"],"requires":{"bins":["mnemonic"]},"install":[{"id":"brew","kind":"brew","formula":"Napageneral/tap/mnemonic","bins":["mnemonic"],"label":"Install via Homebrew"},{"id":"go","kind":"shell","script":"go install github.com/Napageneral/mnemonic/cmd/mnemonic@latest","bins":["mnemonic"],"label":"Install via Go"}]}}
---

# Mnemonic — Unified Memory System

Mnemonic aggregates your communications and AI sessions across all channels (iMessage, Gmail, Cursor, Codex, etc.) into a unified searchable memory with identity resolution, semantic search, and analysis.

## Why Mnemonic?

Your communications and AI interactions are fragmented:
- iMessage threads with some people
- Email conversations with others
- AI chat sessions in Cursor
- More AI sessions in Codex

Mnemonic unifies them into one memory layer, so you can:
- Search across ALL channels semantically
- Extract memories and entities automatically
- Enable smart forking for AI sessions
- Query your complete communication history

## Architecture

Mnemonic uses a **ledger-based** architecture:

- **Core Ledger** — Shared infrastructure: episodes, analysis runs, facets, embeddings
- **Events Ledger** — Human communications: iMessage, Gmail, calendar, trimmed AI turns
- **Agents Ledger** — Full fidelity AI sessions: messages, turns, tool calls (for smart forking)

## Quick Start

```bash
# Initialize
mnemonic init

# Configure your identity
mnemonic me set --name "Your Name" --phone "+1234567890" --email "you@example.com"

# Connect adapters
mnemonic connect imessage
mnemonic connect gmail --account you@gmail.com
mnemonic connect cursor  # AI sessions via AIX

# Sync all channels
mnemonic sync

# Query
mnemonic events --person "Dad" --since "2025-01-01"
mnemonic search "authentication flow"
mnemonic people --top 20
```

## Commands

### Setup

| Command | Description |
|---------|-------------|
| `mnemonic init` | Initialize config and database |
| `mnemonic me set --name "..." --phone "..." --email "..."` | Configure your identity |
| `mnemonic connect <adapter>` | Configure a channel adapter |
| `mnemonic adapters` | List configured adapters |

### Sync

| Command | Description |
|---------|-------------|
| `mnemonic sync` | Sync all enabled adapters |
| `mnemonic sync imessage` | Sync specific adapter (positional) |
| `mnemonic sync --adapter imessage` | Sync specific adapter |
| `mnemonic sync --full` | Force full re-sync |

### Query

| Command | Description |
|---------|-------------|
| `mnemonic events` | List events with filters |
| `mnemonic events --person "Dad"` | Filter by person |
| `mnemonic events --channel imessage` | Filter by channel |
| `mnemonic events --since 2025-01-01` | Filter by date |
| `mnemonic search "query"` | Semantic search across all content |
| `mnemonic people` | List all people |
| `mnemonic people --top 20` | Top contacts by event count |
| `mnemonic timeline 2026-01` | Events in time period |
| `mnemonic db query <sql>` | Raw SQL access |

### Identity Management

| Command | Description |
|---------|-------------|
| `mnemonic identify` | List all people + identities |
| `mnemonic identify --merge "Person A" "Person B"` | Merge two people |
| `mnemonic identify --add "Dad" --email "dad@example.com"` | Add identity |

## Adapters

### iMessage (via Eve)

Prerequisites:
```bash
brew install Napageneral/tap/eve
eve init && eve sync
```

Connect:
```bash
mnemonic connect imessage
```

### Gmail (via gogcli)

Prerequisites:
```bash
brew install steipete/tap/gogcli
gog auth add your@gmail.com
```

Connect:
```bash
mnemonic connect gmail --account your@gmail.com
```

### AI Sessions (via AIX)

Connect:
```bash
mnemonic connect cursor
```

This imports AI sessions from Cursor via AIX:
- **Events Ledger**: Trimmed turns (user message + response)
- **Agents Ledger**: Full fidelity (all messages, tool calls, turns)

## Output Formats

All commands support `--json` / `-j`:

```bash
mnemonic events --json | jq '.events[] | select(.channel == "imessage")'
mnemonic people --top 10 --json
```

## Configuration

Config: `~/.config/mnemonic/config.yaml`

```yaml
me:
  canonical_name: "Your Name"
  identities:
    - channel: imessage
      identifier: "+1234567890"
    - channel: email
      identifier: "you@example.com"

adapters:
  imessage:
    type: eve
    enabled: true
  gmail:
    type: gogcli
    enabled: true
    account: you@gmail.com
  cursor:
    type: aix
    enabled: true
```

Data: `~/Library/Application Support/Mnemonic/mnemonic.db`

## Bootstrap (for AI agents)

```bash
# Check if installed
which mnemonic && mnemonic version

# Install
brew install Napageneral/tap/mnemonic
# OR: go install github.com/Napageneral/mnemonic/cmd/mnemonic@latest

# Setup
mnemonic init

# Configure identity
mnemonic me set --name "User Name" --email "user@example.com"

# Connect adapters (assumes Eve/gogcli/AIX already set up)
mnemonic connect imessage
mnemonic connect gmail --account user@gmail.com
mnemonic connect cursor

# Sync
mnemonic sync

# Verify
mnemonic db query "SELECT COUNT(*) as count FROM events"
mnemonic people --top 5
```

## Tips for Agents

1. Use `mnemonic people --top 10` to understand who the user communicates with most
2. Use `mnemonic events --person "Name"` to get context on a relationship
3. Use `mnemonic search "topic"` for semantic search across all content
4. Use `mnemonic timeline --today` for recent activity
5. Filter by channel to focus on specific contexts
6. Use `--json` output for programmatic access
7. Raw SQL via `mnemonic db query` for complex queries
