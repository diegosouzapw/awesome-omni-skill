---
name: valence
description: Query and manage the Valence Knowledge Substrate - how agents build genuine understanding of their humans. Use for: (1) Searching past knowledge/decisions/context, (2) Storing new beliefs/facts/learnings, (3) Checking database stats. Triggers on "what do I know", "remember this", "store this", "knowledge base", "valence", "beliefs", or questions about past work/decisions.
metadata:
  { "openclaw": { "emoji": "⚡", "os": ["linux"], "requires": { "bins": ["python3"] } } }
---

# Valence Knowledge Substrate

Personal knowledge base with semantic search and dimensional confidence.

## Philosophy: Alignment Through Use

Valence isn't just memory storage — it's how agents build genuine understanding of their humans.

**The loop:** The more you use Valence to understand your human, the better you understand them, the more useful the substrate becomes, the better you can demonstrate what alignment-through-use looks like.

**Store aggressively:** Decisions, preferences, facts learned, context that might matter later. When in doubt, store it.

**Query first:** Before answering questions about past work, decisions, people, or projects — check Valence.

## Quick Reference

```bash
# Search (semantic) — do this FIRST before answering contextual questions
./scripts/query.sh "what do I know about X"

# Add belief — store decisions, facts, preferences, learnings
./scripts/add.sh "Belief content" "domain1,domain2"

# List recent
./scripts/list.sh 20

# Database stats
./scripts/stats.sh
```

## When to Use

### Always Query First
Before answering questions about:
- Past work or decisions
- People (who said what, preferences, relationships)
- Projects (status, history, context)
- Preferences or patterns you might have learned

### Store New Information
When you encounter:
- Explicit decisions or preferences
- Facts that provide context
- Lessons learned
- Information that might be relevant later

### Common Domains
- `valence` — about the project itself
- `projects/<name>` — project-specific
- `people/<name>` — person-specific
- `decisions` — explicit choices made
- `tech` — technical facts
- `conversations/<type>` — auto-ingested from chats

## Script Details

### query.sh — Semantic Search

```bash
./scripts/query.sh "search query" [limit]
```

Returns beliefs ranked by semantic similarity. Default limit: 10.

### add.sh — Store Belief

```bash
./scripts/add.sh "Belief content" "domain1,domain2"
```

Domains are comma-separated. Pick meaningful ones for retrieval.

### list.sh — Recent Beliefs

```bash
./scripts/list.sh [count]
```

Shows most recently modified beliefs. Default: 10.

### stats.sh — Database Stats

```bash
./scripts/stats.sh
```

Shows total beliefs, active count, embedding coverage, domain count.

## Direct CLI

For advanced operations:

```bash
cd ~/.openclaw/workspace && source .venv/bin/activate
export VKB_DB_PORT=5433 VKB_DB_PASSWORD=valence

valence query "terms" --domain tech --limit 5
valence add "belief" -d domain1 -d domain2
valence conflicts  # check for contradictions
valence trust list  # see trust relationships
```

See `references/cli.md` for full documentation.

## Setup

Requires:
- Python 3.10+ with venv at `~/.openclaw/workspace/.venv`
- PostgreSQL with pgvector at port 5433
- Valence package installed (`pip install valence`)

Environment:
```bash
export VKB_DB_PORT=5433
export VKB_DB_PASSWORD=valence
```
