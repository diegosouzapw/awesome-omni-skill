---
name: engram
description: Persistent semantic memory layer for AI agents. Local-first storage (SQLite+LanceDB) with Ollama embeddings. Store and recall facts, decisions, preferences, events, relationships across sessions. Supports memory decay, deduplication, typed memories (5 types), memory relationships (7 graph relation types), agent/user scoping, semantic search, context-aware recall, auto-extraction from text (rules/LLM/hybrid), import/export, REST API, MCP protocol. Solves context window and compaction amnesia. Server at localhost:3400, dashboard at /dashboard. Install via npm (engram-memory), requires Ollama with nomic-embed-text model.
requires:
  bins:
    - engram
install:
  npm: engram-memory
  setup: |
    # Install Ollama and embedding model
    brew install ollama  # macOS
    ollama pull nomic-embed-text
    
    # Start server (run as daemon or manually)
    engram serve
---

# Engram: Persistent Memory for AI Agents

Engram gives you **durable semantic memory** that survives sessions, compaction, crashes. All local, no cloud, no token cost.

## Boot Sequence (MANDATORY)

**On every session start**, run:
```bash
engram search "<current task context>" --limit 10
```

Example: `engram search "client onboarding status churn risk" --limit 10`

This recalls relevant memories from previous sessions before you start work.

## Storing Memories

**5 memory types:** `fact` | `decision` | `preference` | `event` | `relationship`

```bash
# Facts — objective information
engram add "API rate limit is 100 req/min" --type fact --tags api,limits

# Decisions — choices made
engram add "We chose PostgreSQL over MongoDB for better ACID" --type decision --tags database

# Preferences — user/client likes/dislikes
engram add "Dr. Steph prefers text over calls" --type preference --tags dr-steph,communication

# Events — milestones, dates
engram add "Launched v2.0 on January 15, 2026" --type event --tags launch,milestone

# Relationships — people, roles, connections  
engram add "Mia is client manager, reports to Danny" --type relationship --tags team,roles
```

**When to store:**
- Client status changes (churn risk, upsell opportunity, complaints)
- Important decisions made about projects/clients
- Facts learned during work (credentials, preferences, dates)
- Milestones completed (onboarding steps, launches)

## Searching

**Semantic search** (finds meaning, not just keywords):
```bash
# Basic search
engram search "database choice" --limit 5

# Filter by type
engram search "user preferences" --type preference --limit 10

# Filter by agent (see only your memories + global)
engram search "project status" --agent theo --limit 10
```

## Context-Aware Recall

**Recall** ranks by: semantic similarity × recency × salience × access frequency

```bash
engram recall "Setting up new client deployment" --limit 10
```

Better than search when you need **the most relevant memories for a specific context**.

## Memory Relationships

**7 relation types:** `related_to` | `supports` | `contradicts` | `caused_by` | `supersedes` | `part_of` | `references`

```bash
# Manual relation
engram relate <memory-id-1> <memory-id-2> --type supports

# Auto-detect relations via semantic similarity
engram auto-relate <memory-id>

# List relations for a memory
engram relations <memory-id>
```

Relations boost recall scoring — well-connected memories rank higher.

## Auto-Extract from Text

**Ingest** extracts memories from raw text (rules-based by default, optionally LLM):

```bash
# From stdin
echo "Mia confirmed client is happy. We decided to upsell SEO." | engram ingest

# From command
engram extract "Sarah joined as CTO last Tuesday. Prefers async communication."
```

Uses memory types, tags, confidence scoring automatically.

## Management

```bash
# Stats (memory count, types, storage size)
engram stats

# Export backup
engram export -o backup.json

# Import backup
engram import backup.json

# View specific memory
engram get <memory-id>

# Soft delete (preserves for audit)
engram forget <memory-id> --reason "outdated"

# Apply decay manually (usually runs daily automatically)
engram decay
```

## Memory Decay

Inspired by biological memory:
- Every memory has **salience** (0.0 → 1.0)
- Daily decay: `salience *= 0.99` (configurable)
- Accessing a memory boosts salience
- Low-salience memories fade from search results
- Nothing deleted — archived memories can be recovered

## Agent Scoping

**4 scope levels:** `global` → `agent` → `user` → `session`

By default:
- Agents see their own memories + global memories
- `--agent <agentId>` filters to specific agent
- Scope isolation prevents memory bleed between agents

## REST API

Server runs at `http://localhost:3400` (start with `engram serve`).

```bash
# Add memory
curl -X POST http://localhost:3400/api/memories \
  -H "Content-Type: application/json" \
  -d '{"content": "...", "type": "fact", "tags": ["x","y"]}'

# Search
curl "http://localhost:3400/api/memories/search?q=query&limit=5"

# Recall with context
curl -X POST http://localhost:3400/api/recall \
  -H "Content-Type: application/json" \
  -d '{"context": "...", "limit": 10}'

# Stats
curl http://localhost:3400/api/stats
```

**Dashboard:** `http://localhost:3400/dashboard` (visual search, browse, delete, export)

## MCP Integration

Engram works as an MCP server. Add to your MCP client config:

```json
{
  "mcpServers": {
    "engram": {
      "command": "engram-mcp"
    }
  }
}
```

**MCP tools:** `engram_add`, `engram_search`, `engram_recall`, `engram_forget`

## Configuration

`~/.engram/config.yaml`:

```yaml
storage:
  path: ~/.engram

embeddings:
  provider: ollama           # or "openai"
  model: nomic-embed-text
  ollama_url: http://localhost:11434

server:
  port: 3400
  host: localhost

decay:
  enabled: true
  rate: 0.99                 # 1% decay per day
  archive_threshold: 0.1

dedup:
  enabled: true
  threshold: 0.95            # cosine similarity for dedup
```

## Best Practices

1. **Boot with recall** — Always `engram search "<context>" --limit 10` at session start
2. **Type everything** — Use correct memory types for better recall ranking
3. **Tag generously** — Tags enable filtering and cross-referencing
4. **Ingest conversations** — Use `engram ingest` after important exchanges
5. **Let decay work** — Don't store trivial facts; let important memories naturally stay salient
6. **Use relations** — `auto-relate` after adding interconnected memories
7. **Scope by agent** — Keep agent memories separate for clean context

## Troubleshooting

**Server not running?**
```bash
engram serve &
# or install as daemon: see ~/.engram/daemon/install.sh
```

**Embeddings failing?**
```bash
ollama pull nomic-embed-text
curl http://localhost:11434/api/tags  # verify Ollama running
```

**Want to reset?**
```bash
rm -rf ~/.engram/memories.db ~/.engram/vectors.lance
engram serve  # rebuilds from scratch
```

---

**Created by:** Danny Veiga ([@dannyveigatx](https://x.com/dannyveigatx))  
**Source:** https://github.com/Dannydvm/engram-memory  
**Docs:** https://github.com/Dannydvm/engram-memory/blob/main/README.md
