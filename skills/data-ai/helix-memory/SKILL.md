---
name: helix-memory
description: Long-term memory system for Claude Code using HelixDB graph-vector database. Store and retrieve facts, preferences, context, and relationships across sessions using semantic search, reasoning chains, and time-window filtering.
domain: memory
type: system
frequency: daily
commands: [memory, recall]
---

# Helix Memory - Long-Term Memory for Claude Code

Store and retrieve persistent memory across sessions using HelixDB's graph-vector database. Features **semantic search** (via Ollama), **reasoning chains** (IMPLIES/CONTRADICTS/BECAUSE), **time-window filtering**, and **hybrid search**.

## IMPORTANT: Always Use the Bash CLI

**ALWAYS use the `memory` bash script** - never call Python scripts directly.

### Whitelisting

The memory CLI is **globally whitelisted** via symlink:
```
~/Tools/memory → ~/.claude/skills/helix-memory/memory
```

Whitelist pattern in settings.json:
```json
"Bash(~/Tools/memory:*)"
```

This means:
- All memory commands run **without permission prompts**
- Agents inherit this whitelist
- Use `~/Tools/memory` (shorter = fewer tokens)

### Usage

```bash
~/Tools/memory <command>
```

## Service Commands (Start/Stop)

```bash
# Start HelixDB (auto-starts Docker Desktop if needed)
memory start

# Stop HelixDB
memory stop

# Restart
memory restart

# Check status
memory status
```

## Memory Commands

```bash
# Search memories
memory search "topic"

# List all (sorted by importance)
memory list --limit 10

# Store (all aliases work identically - auto-categorize by default)
memory store "User prefers FastAPI over Flask"
memory add "User prefers FastAPI over Flask"
memory remember "User prefers FastAPI over Flask"
memorize "User prefers FastAPI over Flask"

# Store with explicit flags (skips auto-categorization)
memory store "content" -t preference -i 9 -g "tags"

# Store solution with link to problem
memory store "Fix: use async/await" -t solution --solves abc123

# Delete by ID (prefix OK)
memory delete abc123

# Find by tag
memory tag "wordpress"

# Show memory details with edges
memory show abc123

# Link memories (see Graph Relationships section)
memory link <from_id> <to_id> --type solves

# Help
memory help
```

## Python API (For hooks/advanced use only)

The `common.py` module provides high-level functions:

```python
import sys
sys.path.insert(0, '/path/to/helix-memory/hooks')
from common import (
    # Storage
    store_memory, store_memory_embedding, generate_embedding,
    # Retrieval
    get_all_memories, get_high_importance_memories,
    # Search
    search_by_similarity, search_by_text, hybrid_search,
    get_memories_by_time_window,
    # Reasoning chains
    create_implication, create_contradiction, create_causal_link, create_supersedes,
    get_implications, get_contradictions, get_reasoning_chain,
    # Utils
    check_helix_running, ensure_helix_running
)
```

## Key Features

### 1. Semantic Search (Ollama)
Real vector similarity using `nomic-embed-text` model:

```python
# Search finds semantically related content, not just keywords
results = search_by_similarity("verify code works", k=5)
# Finds: "test before completing" even without keyword match
```

### 2. Time-Window Search
Filter memories by recency:

```python
# Time windows: "recent" (4h), "contextual" (30d), "deep" (90d), "full" (all)
recent = get_memories_by_time_window("recent")      # Last 4 hours
contextual = get_memories_by_time_window("contextual")  # Last 30 days
all_time = get_memories_by_time_window("full")      # Everything
```

### 3. Hybrid Search
Combines vector similarity + text matching for best results:

```python
results = hybrid_search("python testing preferences", k=10, window="contextual")
```

### 4. Problem-Solution Linking

Link solutions to the problems they solve using the `--type solves` edge:

```bash
# Link existing memories
memory link <solution_id> <problem_id> --type solves

# Store solution with auto-link
memory store "Fix: use async/await for DB calls" -t solution --solves <problem_id>
```

**3-Step Workflow for Problem-Solution Linking:**
1. **Identify the problem** - Find/store the problem memory: `memory search "timeout error"`
2. **Store/find the solution** - `memory store "Fix: use connection pooling" -t solution`
3. **Link them** - `memory link <solution_id> <problem_id> --type solves`

**View linked solutions:** `memory show <problem_id>` displays `--SOLVED BY--` section.

### 5. Reasoning Chains (Graph Power!)
Create logical relationships between memories:

```python
# "prefers Python" IMPLIES "avoid Node.js suggestions"
create_implication(python_pref_id, avoid_node_id, confidence=9, reason="Language preference")

# "always use tabs" CONTRADICTS "always use spaces"
create_contradiction(tabs_id, spaces_id, severity=8, resolution="newer_wins")

# "migrated to FastAPI" BECAUSE "Flask too slow"
create_causal_link(fastapi_id, flask_slow_id, strength=9)

# New preference SUPERSEDES old one
create_supersedes(new_pref_id, old_pref_id)
```

Query reasoning chains:
```python
implications = get_implications(memory_id)    # What does this imply?
contradictions = get_contradictions(memory_id)  # What conflicts with this?
chain = get_reasoning_chain(memory_id)        # Full reasoning graph
```

## Memory Categories

| Category | Importance | Description |
|----------|------------|-------------|
| **preference** | 7-10 | User preferences that guide interactions |
| **fact** | 5-9 | Factual info about user/projects/environment |
| **context** | 4-8 | Project/domain background |
| **decision** | 6-10 | Architectural decisions with rationale |
| **task** | 3-9 | Ongoing/future tasks |
| **solution** | 6-9 | Bug fixes, problem solutions |

## Storing Memories

### Basic Storage
```python
memory_id = store_memory(
    content="User prefers Python over Node.js for backend",
    category="preference",
    importance=9,
    tags="python,nodejs,backend,language",
    source="session-abc123"  # or "manual"
)
```

### With Semantic Embedding
```python
# Generate real embedding via Ollama
vector, model = generate_embedding(content)

# Store embedding for semantic search
store_memory_embedding(memory_id, vector, content, model)
```

## Retrieving Memories

### Get All/Filtered
```python
all_mems = get_all_memories()
important = get_high_importance_memories(min_importance=8)
prefs = [m for m in all_mems if m.get('category') == 'preference']
```

### Search
```python
# Semantic (finds related meanings)
results = search_by_similarity("testing workflow", k=10)

# Text (exact substring match)
results = search_by_text("pytest")

# Hybrid (best of both)
results = hybrid_search("python testing", k=10, window="contextual")
```

## Schema Overview

### Nodes
- **Memory**: content, category, importance, tags, source, created_at
- **MemoryEmbedding**: vector (1536-dim), content, model
- **Context**: name, description, context_type
- **Concept**: name, concept_type, description

### Reasoning Edges
- **Implies**: Memory → Memory (confidence, reason)
- **Contradicts**: Memory → Memory (severity, resolution)
- **Because**: Memory → Memory (strength)
- **Supersedes**: Memory → Memory (superseded_at)

### Structural Edges
- **HasEmbedding**: Memory → MemoryEmbedding
- **BelongsTo**: Memory → Context
- **RelatedToConcept**: Memory → Concept
- **RelatesTo**: Memory → Memory (generic)

## REST API Endpoints

All endpoints: `POST http://localhost:6969/{endpoint}` with JSON body.

### Storage
```bash
# Store memory
curl -X POST http://localhost:6969/StoreMemory -H "Content-Type: application/json" \
  -d '{"content":"...", "category":"preference", "importance":9, "tags":"...", "source":"manual"}'

# Create implication
curl -X POST http://localhost:6969/CreateImplication -H "Content-Type: application/json" \
  -d '{"from_id":"...", "to_id":"...", "confidence":8, "reason":"..."}'
```

### Retrieval
```bash
# Get all memories
curl -X POST http://localhost:6969/GetAllMemories -H "Content-Type: application/json" -d '{}'

# Get implications
curl -X POST http://localhost:6969/GetImplications -H "Content-Type: application/json" \
  -d '{"memory_id":"..."}'

# Vector search
curl -X POST http://localhost:6969/SearchBySimilarity -H "Content-Type: application/json" \
  -d '{"query_vector":[...], "k":10}'
```

## Automatic Memory (Hooks)

Memory storage/retrieval happens automatically via Claude Code hooks:

- **UserPromptSubmit** (`load_memories.py`): Loads relevant memories before processing
- **Stop** (`reflect_and_store.py`): Analyzes conversation, stores important items (every 5 prompts)
- **SessionStart** (`session_start.py`): Initializes session context

### What Gets Auto-Stored
- Explicit: "remember this:", "store this:"
- Preferences: "I prefer...", "always use...", "never..."
- Decisions: "decided to...", "let's use..."
- Bug fixes: "the issue was...", "fixed by..."

## CLI Reference

```bash
# Service
memory start      # Start HelixDB (auto-starts Docker Desktop)
memory stop       # Stop HelixDB
memory restart    # Restart HelixDB
memory status     # Check status and memory count

# Memory operations
memory search "pytest"
memory list --limit 10
memory store/add/remember/rem "content"  # All auto-categorize
memory store "content" -t cat -i imp -g "tags"  # Explicit flags
memory store "solution" -t solution --solves <problem_id>  # Link solution to problem
memory delete <memory-id>
memory tag "tagname"
memory show <memory-id>    # Show details with edges
memory help

# Graph operations (linking memories)
memory link <from_id> <to_id> --type <edge_type>
```

### Link Command & Edge Types

The `memory link` command creates graph edges between memories:

```bash
memory link <from_id> <to_id> --type <edge_type>
```

**Available edge types:**

| Edge Type | Direction | Use Case |
|-----------|-----------|----------|
| `solves` | solution → problem | Link a fix to the bug it solves |
| `solved_by` | problem → solution | Link a bug to its fix |
| `supersedes` | new → old | New preference replaces old |
| `implies` | A → B | A logically implies B |
| `contradicts` | A ↔ B | A and B conflict |
| `leads_to` | cause → effect | Causal chain |
| `supports` | evidence → claim | Supporting evidence |
| `related` | A ↔ B | Generic relationship (default) |

**Examples:**
```bash
# Solution solves a problem
memory link sol_abc123 prob_def456 --type solves

# New preference supersedes old
memory link new_pref old_pref --type supersedes

# One decision implies another
memory link use_fastapi avoid_flask --type implies
```

### Show Command

`memory show <id>` displays memory details and linked edges:

```bash
memory show abc123
```

Output includes relationship sections:
- `--SOLVED BY--` - Solutions for problems
- `--SOLVES--` - Problems solved by solutions
- `--IMPLIES--` - Logical implications
- `--CONTRADICTS--` - Conflicts
- `--SUPERSEDES--` - Replaced memories
- `--RELATED--` - Generic relationships

## Project Tagging

Memories are **automatically tagged with project names** based on working directory.
Project detection uses directory name as fallback.

## Ollama Setup (For Real Semantic Search)

```bash
# Start Ollama service
brew services start ollama

# Pull embedding model (274MB)
ollama pull nomic-embed-text

# Verify
curl http://localhost:11434/api/tags
```

Without Ollama, falls back to Gemini API (if key set) or hash-based pseudo-embeddings.

## Best Practices

### DO:
- Store preferences immediately when expressed
- Use reasoning chains to link related memories
- Set appropriate importance (10=critical, 7-9=high, 4-6=medium, 1-3=low)
- Use hybrid_search for best recall
- Filter by time window to prioritize recent info

### DON'T:
- Store code snippets (use codebase)
- Store sensitive data (passwords, keys)
- Create duplicate memories (use find_similar_memories first)
- Forget embeddings (needed for semantic search)

## Troubleshooting

### DB Won't Start
```bash
# Use the memory script (handles Docker auto-start)
memory start

# Check container status
docker ps | grep helix
```

### Ollama Not Working
```bash
brew services restart ollama
ollama list  # Should show nomic-embed-text
```

### Vector Dimension Errors
HelixDB expects 1536-dim vectors. The code auto-pads smaller embeddings.

### Check Logs
```bash
docker logs $(docker ps -q --filter "name=helix-memory") 2>&1 | tail -20
```

## Resources

- **Helix CLI:** `~/.local/bin/helix`
- **HelixDB Docs:** https://docs.helix-db.com
- **Ollama:** https://ollama.ai
