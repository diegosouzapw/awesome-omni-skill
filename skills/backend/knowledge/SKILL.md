---
name: Knowledge
version: 1.9.0
description: Personal knowledge management using Graphiti knowledge graph with Neo4j/FalkorDB, supporting remote MCP access with connection profiles and TLS, OSINT/CTI ontology, and investigative search. USE WHEN 'store this', 'remember this', 'add to knowledge', 'search my knowledge', 'what do I know about', 'find in knowledge base', 'save to memory', 'graphiti', 'knowledge graph', 'entity extraction', 'relationship mapping', 'semantic search', 'episode', 'install knowledge', 'setup knowledge system', 'configure knowledge graph', 'remote knowledge server', 'connect to knowledge', 'knowledge profile', knowledge capture, retrieval, synthesis, memory decay, decay scoring, lifecycle state, importance classification, stability classification, health metrics, run maintenance, permanent memory, soft-delete, 'investigate entity', 'find connections', 'graph traversal', 'threat hunting', 'list ontology', 'custom entity types', 'CTI entities', 'OSINT entities', 'import STIX', 'STIX bundle', 'threat intel import'.
tools:
  # MCP Wrapper CLI (76%+ token savings vs direct MCP calls)
  - Bash(bun run */knowledge-cli.ts add_episode *)
  - Bash(bun run */knowledge-cli.ts search_nodes *)
  - Bash(bun run */knowledge-cli.ts search_facts *)
  - Bash(bun run */knowledge-cli.ts get_episodes *)
  - Bash(bun run */knowledge-cli.ts get_status)
  - Bash(bun run */knowledge-cli.ts clear_graph *)
  - Bash(bun run */knowledge-cli.ts health)
  # Feature 009: Memory decay and lifecycle
  - Bash(bun run */knowledge-cli.ts health_metrics *)
  - Bash(bun run */knowledge-cli.ts run_maintenance *)
  - Bash(bun run */knowledge-cli.ts classify_memory *)
  - Bash(bun run */knowledge-cli.ts recover_memory *)
  # Feature 010: Remote MCP access
  - Bash(bun run */knowledge-cli.ts status)
  - Bash(bun run */knowledge-cli.ts list_profiles)
  # Feature 018: OSINT/CTI Ontology
  - Bash(bun run */knowledge-cli.ts ontology:list)
  - Bash(bun run */knowledge-cli.ts ontology:validate)
  - Bash(bun run */knowledge-cli.ts ontology:reload)
  - Bash(bun run */knowledge-cli.ts stix:import *)
  - Bash(bun run */knowledge-cli.ts stix:status)
  # Feature 020: Investigative Search
  - Bash(bun run */knowledge-cli.ts investigate *)
  # Server management
  - Bash(bun run server-cli *)
---

# Knowledge

Persistent personal knowledge system powered by Graphiti knowledge graph with Neo4j (default) or FalkorDB backend. Automatically extracts entities, relationships, and temporal context from conversations, documents, and ideas.

## Workflow Routing

| Workflow | Trigger | File |
|----------|---------|------|
| **Install** | "install knowledge", "setup knowledge system", "configure knowledge graph", "install knowledge system" | `tools/Install.md` |
| **Capture Episode** | "remember this", "store this", "add to knowledge", "save this", "log this" | `workflows/CaptureEpisode.md` |
| **Search Knowledge** | "search my knowledge", "what do I know about", "find in my knowledge base", "recall" | `workflows/SearchKnowledge.md` |
| **Search Facts** | "what's the connection", "how are these related", "show relationships" | `workflows/SearchFacts.md` |
| **Search By Date** | "what did I learn today", "knowledge from last week", "show January entries", "yesterday's knowledge" | `workflows/SearchByDate.md` |
| **Get Recent Episodes** | "what did I learn", "recent additions", "latest knowledge" | `workflows/GetRecent.md` |
| **Get Status** | "knowledge status", "graph health", "knowledge stats", "system health", "memory health" | `workflows/GetStatus.md` |
| **Health Report** | "knowledge health report", "memory decay status", "lifecycle report", "decay metrics" | `workflows/HealthReport.md` |
| **Run Maintenance** | "run knowledge maintenance", "update decay scores", "recalculate memory decay" | `workflows/RunMaintenance.md` |
| **Clear Graph** | "clear knowledge", "reset graph", "delete all knowledge" | `workflows/ClearGraph.md` |
| **Bulk Import** | "import these documents", "bulk knowledge import" | `workflows/BulkImport.md` |
| **Investigate Entity** | "investigate entity", "find connections", "graph traversal", "threat hunting", "connected entities" | `workflows/InvestigateEntity.md` |
| **Ontology Management** | "list ontology", "custom entity types", "CTI entities", "OSINT entities", "ontology config" | `workflows/OntologyManagement.md` |
| **STIX Import** | "import STIX", "STIX bundle", "threat intel import", "CTI data" | `workflows/StixImport.md` |

## Core Capabilities

**Knowledge Graph Features:**

- **Automatic Entity Extraction** - Identifies people, organizations, locations, concepts, preferences, requirements
- **Relationship Mapping** - Tracks how entities connect with temporal context
- **Semantic Search** - Finds relevant knowledge using vector embeddings
- **Episode-Based Storage** - Preserves context and conversations over time
- **Multi-Source Input** - Accepts text, JSON, messages, and structured data

**Built-in Entity Types:**

**Standard Types:**
- **Preferences** - User choices, opinions, configurations
- **Requirements** - Features, needs, specifications
- **Procedures** - SOPs, workflows, how-to guides
- **Locations** - Physical or virtual places
- **Events** - Time-bound occurrences, experiences
- **Organizations** - Companies, institutions, groups
- **Documents** - Articles, reports, books, content

**OSINT/CTI Types (Feature 018):**
- **ThreatActor** - Malicious actors, APT groups, threat campaigns
- **Malware** - Malicious software, viruses, ransomware
- **Vulnerability** - CVE, security flaws, exploits
- **Campaign** - Coordinated threat activities
- **Indicator** - IoCs, hashes, IPs, domains (STIX Indicator)
- **Infrastructure** - Command & control, attack infrastructure
- **TTP** - Tactics, Techniques, and Procedures (MITRE ATT&CK)
- **Account** - Social media, email, service accounts
- **Domain** - Registered domains, DNS records
- **Email** - Email addresses, mailboxes
- **Phone** - Phone numbers, mobile devices
- **Image** - Photos, screenshots, media files
- **Investigation** - OSINT investigations, case files

## Prerequisites

**Required Setup:**

The skill is installed at `~/.claude/skills/Knowledge/` (or `$PAI_DIR/skills/Knowledge/`).

1. **Start the Graphiti MCP server:**

   ```bash
   cd ~/.claude/skills/Knowledge
   bun run server-cli start
   ```

2. **Verify server is running:**

   ```bash
   cd ~/.claude/skills/Knowledge && bun run server-cli status
   ```

3. **Other server commands:**

   ```bash
   bun run server-cli stop      # Stop containers
   bun run server-cli restart   # Restart containers
   bun run server-cli logs      # View logs
   bun run server-cli logs --mcp  # MCP server logs only
   bun run server-cli logs --db   # Database logs only
   ```

4. **Configure API key** (in PAI .env `~/.claude/.env`):

   ```bash
   MADEINOZ_KNOWLEDGE_OPENAI_API_KEY=sk-your-key-here
   ```

## Interface Priority: CLI-First, MCP-Fallback

**ALWAYS use this execution order:**

1. **PRIMARY: Knowledge CLI** (via Bash) - Reliable, token-efficient, human-readable
2. **FALLBACK: MCP Tools** - Only if CLI fails or for programmatic access

**Why CLI-first?**
- MCP tools may have session/connection issues in Claude Code
- CLI provides 25-35% token savings with compact output
- CLI has better error messages and troubleshooting
- CLI works reliably via direct Bash execution

### Knowledge CLI (Primary Interface)

**Run commands from the skill directory:**

```bash
cd ~/.claude/skills/Knowledge
```

**Commands:**

```bash
# Add knowledge (REQUIRES both title AND body as separate quoted strings)
bun run tools/knowledge-cli.ts add_episode "Short Title" "Full content body text here" "Source"

# Search entities (30%+ token savings)
bun run tools/knowledge-cli.ts search_nodes "query" 10

# Search relationships (30%+ token savings)
bun run tools/knowledge-cli.ts search_facts "query" 10

# Get recent episodes (25%+ token savings)
bun run tools/knowledge-cli.ts get_episodes 10

# Get system status
bun run tools/knowledge-cli.ts get_status

# Clear graph (destructive - requires --force)
bun run tools/knowledge-cli.ts clear_graph --force

# Check server health
bun run tools/knowledge-cli.ts health

# Feature 009: Memory decay and lifecycle
bun run tools/knowledge-cli.ts health_metrics
bun run tools/knowledge-cli.ts run_maintenance
bun run tools/knowledge-cli.ts run_maintenance --dry-run
bun run tools/knowledge-cli.ts classify_memory "content" --source "description"
bun run tools/knowledge-cli.ts recover_memory <uuid>

# Feature 018: OSINT/CTI Ontology
bun run tools/knowledge-cli.ts ontology:list
bun run tools/knowledge-cli.ts ontology:validate
bun run tools/knowledge-cli.ts ontology:reload
bun run tools/knowledge-cli.ts stix:import <file-or-url>
bun run tools/knowledge-cli.ts stix:status

# Feature 020: Investigative Search
bun run tools/knowledge-cli.ts investigate "entity-name" --depth 2
bun run tools/knowledge-cli.ts investigate "apt28" --relationship-type attributed_to --relationship-type uses
```

**Options:**

- `--raw` - Output raw JSON instead of compact format
- `--metrics` - Display token metrics after each operation
- `--metrics-file <path>` - Append metrics to JSONL file
- `--since <date>` - Filter results created after this date
- `--until <date>` - Filter results created before this date
- `--weighted` - Apply weighted scoring (60% semantic + 25% recency + 15% importance)
- `--depth <N>` - Investigation depth for graph traversal (1-3 hops, default: 1)
- `--relationship-type <TYPE>` - Filter by relationship type (can specify multiple)

**Temporal Search (date filtering):**

```bash
# Today's knowledge
bun run tools/knowledge-cli.ts search_nodes "query" --since today

# Last 7 days
bun run tools/knowledge-cli.ts search_facts "query" --since 7d

# Date range
bun run tools/knowledge-cli.ts search_nodes "query" --since 2026-01-01 --until 2026-01-15
```

**Date formats:** `today`, `yesterday`, `7d`, `1w`, `1m`, or ISO dates (`2026-01-26`)

**What Gets Captured:**

- Conversations and insights from work sessions
- Research findings and web content
- Code snippets and technical decisions
- Project documentation and notes
- Personal preferences and decisions
- Meeting notes and action items

## Examples

**Example 1: Capture a Learning**

User: "Remember that when using Podman volumes, you should always mount to /container/path not host/path"

→ Invokes CaptureEpisode workflow
→ **AI extracts title from content and calls CLI with TWO arguments:**

```bash
bun run tools/knowledge-cli.ts add_episode \
  "Podman Volume Mounting Syntax" \
  "When using Podman volumes, always mount to /container/path not host/path. The left side is host path, right side is container path." \
  "User learning"
```

→ Stores episode with extracted entities:

- Entity: "Podman volumes" (Topic)
- Entity: "volume mounting" (Procedure)
- Fact: "Podman volumes use /container/path syntax"
→ User receives: "✓ Captured: Podman volume mounting syntax"

**Example 2: Search Knowledge**

User: "What do I know about Graphiti?"

→ Invokes SearchKnowledge workflow
→ Searches knowledge graph for "Graphiti" entities
→ Returns related entities, facts, and summaries
→ User receives: "Based on your knowledge graph, Graphiti is..."

**Example 3: Find Relationships**

User: "How are FalkorDB and Graphiti connected?"

→ Invokes SearchFacts workflow
→ Searches for edges between FalkorDB and Graphiti entities
→ Returns facts showing relationship with temporal context
→ User receives: "FalkorDB is the graph database backend for Graphiti MCP server"

**Example 4: Get Recent Learning**

User: "What did I learn this week about PAI?"

→ Invokes GetRecent workflow
→ Retrieves recent episodes mentioning "PAI" or "Personal AI Infrastructure"
→ Returns chronological list with timestamps
→ User receives: "Recent additions: 1) PAI skills architecture... 2) Canonical skill structure..."

**Example 5: Clear and Reset**

User: "Clear my knowledge graph and start fresh"

→ Invokes ClearGraph workflow
→ Confirms destructive action
→ Deletes all entities and relationships
→ Rebuilds indices
→ User receives: "✓ Knowledge graph cleared. Ready for fresh knowledge capture."

## MCP Integration (Fallback Only)

**⚠️ Use MCP tools only when CLI fails or for programmatic TypeScript access.**

**MCP Server Endpoint:**

```
http://localhost:8000/mcp/
```

**Available MCP Tools (Fallback):**

| MCP Tool | Graphiti Concept | User-Friendly Action |
|----------|------------------|----------------------|
| `add_memory` | Episode | "Store this knowledge" |
| `search_nodes` | Nodes/Entities | "Search my knowledge" |
| `search_memory_facts` | Facts/Edges | "Find relationships" |
| `get_episodes` | Episodes | "Show recent additions" |
| `delete_episode` | Episode | "Remove this entry" |
| `delete_entity_edge` | Edge | "Remove relationship" |
| `get_entity_edge` | Edge | "Get relationship details" |
| `clear_graph` | Graph | "Clear all knowledge" |
| `get_status` | - | "Check knowledge status" |
| `get_knowledge_health` | - | "Get memory decay and lifecycle metrics" |
| `run_decay_maintenance` | - | "Run decay maintenance cycle" |
| `classify_memory` | - | "Classify memory importance and stability" |
| `recover_soft_deleted` | - | "Recover soft-deleted memory" |
| `investigate_entity` | - | "Investigate entity connections (graph traversal)" |
| `list_ontology_types` | - | "List custom entity and relationship types" |
| `validate_ontology` | - | "Validate ontology configuration" |
| `reload_ontology` | - | "Reload ontology configuration (hot-reload)" |
| `import_stix_bundle` | - | "Import STIX 2.1 bundle from file or URL" |

**Naming Convention (Hybrid Approach):**

- **User-facing (Skills/Workflows):** Knowledge-friendly language ("store knowledge", "search my knowledge")
- **Internal (TypeScript):** Graphiti-native methods (`addEpisode`, `searchNodes`, `searchFacts`)
- **MCP Layer:** Actual tool names (`add_memory`, `search_nodes`, `search_memory_facts`)

**Response Caching:**
Search operations (`search_nodes`, `search_memory_facts`) are cached to improve performance:

- **TTL:** 5 minutes (configurable via `cacheTtlMs`)
- **Max entries:** 100 (configurable via `cacheMaxSize`)
- **Scope:** Per-client instance (not shared across sessions)
- **Cache invalidation:** Automatic on TTL expiry, or manual via `clearCache()`

To disable caching, initialize the client with `enableCache: false`.

## Configuration Options

**Environment Variables** (set in PAI config: `$PAI_DIR/.env` or `~/.claude/.env`):

```bash
# LLM Configuration (OpenRouter recommended)
MADEINOZ_KNOWLEDGE_OPENAI_API_KEY=sk-or-v1-your-key-here
MADEINOZ_KNOWLEDGE_OPENAI_BASE_URL=https://openrouter.ai/api/v1
MADEINOZ_KNOWLEDGE_MODEL_NAME=openai/gpt-4o-mini
MADEINOZ_KNOWLEDGE_LLM_PROVIDER=openai

# Embedder Configuration (Ollama recommended - free & fast)
MADEINOZ_KNOWLEDGE_EMBEDDER_PROVIDER=ollama
MADEINOZ_KNOWLEDGE_EMBEDDER_PROVIDER_URL=http://host.containers.internal:11434
MADEINOZ_KNOWLEDGE_EMBEDDER_MODEL=mxbai-embed-large
MADEINOZ_KNOWLEDGE_EMBEDDER_DIMENSIONS=1024

# Concurrency (adjust based on API tier)
MADEINOZ_KNOWLEDGE_SEMAPHORE_LIMIT=10

# Group ID (for multiple knowledge graphs)
MADEINOZ_KNOWLEDGE_GROUP_ID=main

# Disable telemetry
MADEINOZ_KNOWLEDGE_GRAPHITI_TELEMETRY_ENABLED=false
```

**Model Recommendations:**

*Via OpenRouter (Recommended):*

- **openai/gpt-4o-mini** - Most reliable, $0.129/1K ops
- **google/gemini-2.0-flash-001** - Best value, $0.125/1K ops
- **openai/gpt-4o** - Fastest, $2.155/1K ops

*Direct OpenAI:*

- **gpt-4o-mini** - Fast, cost-effective for daily use
- **gpt-4o** - Better for complex reasoning

⚠️ **Known Failures:** Llama, Mistral, DeepSeek models fail Graphiti Pydantic validation

## Related Documentation

- `${PAI_DIR}/skills/CORE/SkillSystem.md` - Canonical skill structure guide
- `${PAI_DIR}/skills/CORE/SYSTEM/MEMORYSYSTEM.md` - PAI's memory documentation
- [Graphiti Documentation](https://help.getzep.com/graphiti)
- [Podman Configuration](../README.md)

**Last Updated:** 2026-02-04
