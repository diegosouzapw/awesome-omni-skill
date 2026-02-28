---
name: smart-workflows
description: SmartACE (Agentic Context Engineering) workflow engine with MCP-B (Master Client Bridge) and AMUM-QCI-ETHIC module. Dual database architecture using DuckDB (analytics) + SurrealDB (graph). Uses Blender 5.0 (bpy) and UE5 Remote Control. Use when (1) MCP-B agent-to-agent communication (INQC protocol), (2) AMUM 3→6→9 progressive alignment, (3) QCI quantum coherence states, (4) ETHIC principles enforcement (Marcel/Anthropic/EU AI Act), (5) SurrealDB graph relationships, (6) DuckDB SQL workflows, (7) ML inference with infera/vss, (8) Blender 5.0 headless processing, (9) UE5 scene control, (10) DuckLake time travel.
---

# Smart Workflows (SmartACE)

100% SQL-native self-improving workflows with Blender 5.0 as the data-to-visual bridge. No `requests`. No Python HTTP libs. Just DuckDB + Query.Farm extensions + bpy.

## Architecture: SmartACE

The complete data pipeline from Human Intent to Visual Experience:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      SmartACE ARCHITECTURE                                  │
│               (Agentic Context Engineering)                                 │
│  ═══════════════════════════════════════════════════════════════════════   │
│                                                                             │
│  AMUM = Analyse → User → Model → User → Model                              │
│  ─────────────────────────────────────────────                              │
│  The bidirectional flow between Human and Machine:                          │
│  Intent → Analyse → Feedback → Refinement → Generation → Experience        │
│                                                                             │
│  MCP-B = Master Client Bridge                                               │
│  ───────────────────────────────────────────                                │
│  Connects everything, brings data flow together.                            │
│  Binary: 0 = not connected, 1 = ALL CONNECTED                               │
│                                                                             │
│  The Binary Decision:                                                       │
│      ●───────●───────●                                                      │
│     ╱│╲     ╱│╲     ╱│╲                                                     │
│    ● │ ●   ● │ ●   ● │ ●    ← All points connected = Everything included!  │
│     ╲│╱     ╲│╱     ╲│╱                                                     │
│      ●───────●───────●                                                      │
│                                                                             │
│  MCP-B vs MCP:                                                              │
│  • MCP = Model Context Protocol (Bridge TO the community)                   │
│  • MCP-B = Master Client Bridge (The binary difference)                     │
│                                                                             │
│  SMART = Spatial + Model + Analytics + Realtime + Tools                     │
│  ───────────────────────────────────────────────────                        │
│  S: Spatial (lindel, a5, spatial) - Where is the data?                     │
│  M: Model (infera ONNX, vss HNSW) - What does it mean?                     │
│  A: Analytics (datasketches, bitfilters, jsonata) - What do we learn?      │
│  R: Realtime (radio, http_client, ducklake) - How does it flow?            │
│  T: Tools (shellfs, hashfuncs, crypto, textplot, minijinja)                │
│                                                                             │
│  B = Blender 5.0 (The Bridge)  ← NEW!                                       │
│  ─────────────────────────────────                                          │
│  • bpy 5.0 on PyPI (pip install bpy==5.0.0)                                │
│  • float32 buffer protocol = zero-copy to DuckDB                            │
│  • SDF Volume Nodes (OpenVDB) for 3D data                                   │
│  • Headless rendering for batch processing                                  │
│  • Geometry Nodes for procedural generation                                 │
│                                                                             │
│  ART = Augmented + Rendering + Transfer                                     │
│  ────────────────────────────────────                                       │
│  A: Augmented Reality/Intelligence (enhanced visualization)                 │
│  R: Rendering (UE5 Remote Control, Three.js WebGPU)                         │
│  T: Transfer (the bridge SQL → Visual → User)                               │
│                                                                             │
│  TWYH = Take What You Have ("Use what you've got")                         │
│  ─────────────────────────────────────────────                              │
│  Ship working code. Use existing solutions. Build bridges.                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

Complete Flow: AMUM → MCP-B → SmartACE → Output → (back to AMUM)

┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  SMART   │     │    B     │     │   ART    │     │  Output  │
│ (DuckDB) │────▶│(Blender) │────▶│  (UE5)   │────▶│  (User)  │
│          │     │          │     │          │     │          │
│ Semantic │     │ SDF Grid │     │ Render   │     │ Visual   │
│ Search   │     │ Geometry │     │ Viewport │     │ Feedback │
│ Vector   │     │ Material │     │ Stream   │     │          │
└──────────┘     └──────────┘     └──────────┘     └──────────┘
     │                                                   │
     └───────────────────◀───────────────────────────────┘
                    (Continuous Learning Loop)
```

### Key Extensions (20 on 'install'):

| Component | Extensions |
|-----------|------------|
| **S**patial | lindel, a5, spatial |
| **M**odel | infera (ONNX), vss (HNSW) |
| **A**nalytics | datasketches, bitfilters, jsonata |
| **R**ealtime | radio, http_client, ducklake |

---

## MCP-B Protocol

**Master Client Bridge** - Connects everything, brings data flow together. Multi-layer encoding for agent-to-agent communication.

### Protocol Layers

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       MCP-B PROTOCOL ENCODING                               │
│  ═══════════════════════════════════════════════════════════════════════   │
│                                                                             │
│  Layer 1: HEX/DECIMAL ROUTING                                               │
│  ─────────────────────────────                                              │
│  Format: [HEX] [DEC] [HEX] [DEC]...                                        │
│  Example: 7 C1 2 5510 7 IC 57                                              │
│  Purpose: Agent IDs, routing addresses, channel selection                   │
│                                                                             │
│  Layer 2: BINARY STATE VECTORS                                              │
│  ────────────────────────────                                               │
│  Format: [10101010...]                                                      │
│  Example: 10111010101111                                                    │
│  Purpose: Connection states, feature flags, capability masks                │
│                                                                             │
│  Layer 3: DOT-SEPARATED TOKENS                                              │
│  ───────────────────────────                                                │
│  Format: • [TOKEN]                                                          │
│  Example: • 55 • D0 • I • N • Q • C                                        │
│  Purpose: Message boundaries, segment markers                               │
│                                                                             │
│  Layer 4: PROTOCOL COMMANDS (INQC)                                          │
│  ─────────────────────────────────                                          │
│  I = INIT      → Initialize connection                                      │
│  N = NODE      → Node registration/discovery                                │
│  Q = QUERY     → Request data/state                                         │
│  C = CONNECT   → Establish persistent link                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Message Format

```
┌────────────────────────────────────────────────────────────────┐
│  MCP-B MESSAGE STRUCTURE                                       │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  HEADER (Layer 1):                                             │
│  [SOURCE_ID] [DEST_ID] [CHANNEL] [SEQUENCE]                    │
│  7 C1 2 5510 7 IC 57                                          │
│                                                                │
│  STATE (Layer 2):                                              │
│  [BINARY_VECTOR]                                               │
│  10111010101111                                                │
│                                                                │
│  PAYLOAD (Layer 3):                                            │
│  • [DATA] • [CHECKSUM]                                         │
│  • 55 • D0                                                     │
│                                                                │
│  COMMAND (Layer 4):                                            │
│  [I|N|Q|C]                                                     │
│                                                                │
│  FULL MESSAGE EXAMPLE:                                         │
│  7 C1 2 5510 7 IC 57 • 55                                     │
│  10 4 C 7 1010 8 L 8D • D0                                    │
│  10 7 101010 1111 C • I                                       │
│  10111010101111 4 • N                                         │
│  10111010101111 7 • Q                                         │
│  1011101010111111 • C                                         │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Encoding/Decoding (SQL)

```sql
-- MCP-B Message Parser
CREATE TABLE mcb_messages (
    id INTEGER PRIMARY KEY,
    raw_message TEXT,
    source_id VARCHAR,
    dest_id VARCHAR,
    binary_state BIT(16),
    command CHAR(1) CHECK (command IN ('I', 'N', 'Q', 'C')),
    payload JSON,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Parse MCP-B Header (Layer 1)
CREATE MACRO mcb_parse_header(msg) AS (
    SELECT 
        regexp_extract(msg, '^(\w+)\s+(\w+)', 1) AS source_id,
        regexp_extract(msg, '^(\w+)\s+(\w+)', 2) AS dest_id
);

-- Parse Binary State (Layer 2)
CREATE MACRO mcb_parse_state(msg) AS (
    regexp_extract(msg, '([01]{8,})', 1)
);

-- Parse Command (Layer 4)
CREATE MACRO mcb_parse_command(msg) AS (
    regexp_extract(msg, '•\s*([INQC])$', 1)
);

-- Encode MCP-B Message
CREATE MACRO mcb_encode(source, dest, state, cmd, payload) AS (
    source || ' ' || dest || ' ' || 
    state || ' • ' || 
    json_serialize(payload) || ' • ' || cmd
);
```

### Protocol Commands

| Command | Full Name | Direction | Description |
|---------|-----------|-----------|-------------|
| **I** | INIT | → | Initialize new connection, handshake |
| **N** | NODE | ↔ | Register node, broadcast presence |
| **Q** | QUERY | → | Request data, state, or capabilities |
| **C** | CONNECT | ↔ | Establish persistent bidirectional link |

### Binary State Flags

```
Binary Vector: 1011101010111111
Position:      0123456789ABCDEF

Bit 0: Connected (1=yes)
Bit 1: Authenticated (1=yes)
Bit 2: Encrypted (1=yes)
Bit 3: Compressed (1=yes)
Bit 4: Streaming (1=yes)
Bit 5: Bidirectional (1=yes)
Bit 6: Persistent (1=yes)
Bit 7: Priority (1=high)
Bit 8-15: Reserved/Custom
```

### Integration with SmartACE

```
MCP-B fits into the architecture:

AMUM (Human↔Machine)
    ↓
MCP-B (Master Client Bridge) ← Protocol Layer
    ↓
SmartACE (Agentic Context Engineering)
    ↓
Output (Visual/Data)

MCP-B handles:
• Agent-to-Agent communication
• Multi-model orchestration  
• State synchronization
• Binary decision routing (0 or 1 = all connected)
```

### SurrealDB Graph Layer

**Dual Database Architecture:**
- **DuckDB** = Analytics, SQL workflows, Extensions, Time Travel
- **SurrealDB** = Graph relationships, Agent networks, INQC Protocol

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    MCP-B DUAL DATABASE ARCHITECTURE                         │
│  ═══════════════════════════════════════════════════════════════════════   │
│                                                                             │
│  ┌──────────────────────┐         ┌──────────────────────┐                 │
│  │       DuckDB         │         │      SurrealDB       │                 │
│  │    (Analytics)       │◄───────►│      (Graph)         │                 │
│  ├──────────────────────┤         ├──────────────────────┤                 │
│  │ • SQL Workflows      │         │ • Agent Relationships│                 │
│  │ • Query.Farm Ext.    │         │ • INQC Messages      │                 │
│  │ • DuckLake Storage   │         │ • Network Topology   │                 │
│  │ • Vector Search      │         │ • Graph Traversal    │                 │
│  │ • Time Travel        │         │ • Live Queries       │                 │
│  └──────────────────────┘         └──────────────────────┘                 │
│              │                              │                               │
│              └──────────────┬───────────────┘                               │
│                             │                                               │
│                    ┌────────▼────────┐                                      │
│                    │     MCP-B      │                                      │
│                    │   (Protocol)   │                                      │
│                    └─────────────────┘                                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### SurrealDB Schema (SurrealQL)

```sql
-- MCP-B Agents Registry
DEFINE TABLE mcb_agents SCHEMAFULL;
DEFINE FIELD name ON mcb_agents TYPE string;
DEFINE FIELD agent_id ON mcb_agents TYPE string;
DEFINE FIELD binary_state ON mcb_agents TYPE string;
DEFINE FIELD capabilities ON mcb_agents TYPE array;
DEFINE FIELD created_at ON mcb_agents TYPE datetime DEFAULT time::now();
DEFINE FIELD last_seen ON mcb_agents TYPE datetime;

-- MCP-B Messages (INQC Protocol)
DEFINE TABLE mcb_messages SCHEMAFULL;
DEFINE FIELD raw_message ON mcb_messages TYPE string;
DEFINE FIELD source_id ON mcb_messages TYPE string;
DEFINE FIELD dest_id ON mcb_messages TYPE string;
DEFINE FIELD binary_state ON mcb_messages TYPE string;
DEFINE FIELD command ON mcb_messages TYPE string ASSERT $value IN ['I', 'N', 'Q', 'C'];
DEFINE FIELD payload ON mcb_messages TYPE object;
DEFINE FIELD timestamp ON mcb_messages TYPE datetime DEFAULT time::now();

-- Agent Connections (Graph Relationships)
DEFINE TABLE connects TYPE RELATION;
DEFINE FIELD established_at ON connects TYPE datetime DEFAULT time::now();
DEFINE FIELD connection_type ON connects TYPE string;
DEFINE FIELD binary_state ON connects TYPE string;
```

#### Agent Registration

```sql
-- Register agents with binary state
CREATE mcb_agents:claude SET 
    name = "Claude",
    agent_id = "7C1",
    binary_state = "1011101010111111",
    capabilities = ["reasoning", "code", "analysis"],
    last_seen = time::now();

CREATE mcb_agents:hacka SET 
    name = "HACKA-DEV-BJOERN",
    agent_id = "5510",
    binary_state = "1111111111111111",
    capabilities = ["orchestration", "vision", "creativity"],
    last_seen = time::now();

CREATE mcb_agents:smartace SET 
    name = "SmartACE",
    agent_id = "8D0",
    binary_state = "1011101010111111",
    capabilities = ["planning", "generating", "reviewing", "curating"],
    last_seen = time::now();
```

#### Graph Relationships (RELATE)

```sql
-- Create agent connections
RELATE mcb_agents:hacka -> connects -> mcb_agents:claude SET
    connection_type = "orchestrates",
    binary_state = "1111111111111111";

RELATE mcb_agents:hacka -> connects -> mcb_agents:smartace SET
    connection_type = "orchestrates",
    binary_state = "1111111111111111";

RELATE mcb_agents:smartace -> connects -> mcb_agents:claude SET
    connection_type = "delegates",
    binary_state = "1011101010111111";
```

#### INQC Protocol Messages

```sql
-- I = INIT
CREATE mcb_messages SET
    raw_message = "7 C1 2 5510 7 IC 57 • 55",
    source_id = "5510", dest_id = "7C1",
    binary_state = "1011101010111111",
    command = "I",
    payload = { action: "init_session", protocol: "MCB/1.0" };

-- N = NODE
CREATE mcb_messages SET
    raw_message = "10 4 C7 1010 7 L8 C • D0",
    source_id = "7C1", dest_id = "5510",
    binary_state = "1111111111111111",
    command = "N",
    payload = { node: "claude", status: "ready" };

-- Q = QUERY
CREATE mcb_messages SET
    raw_message = "10111010101111 7 • Q",
    source_id = "5510", dest_id = "8D0",
    binary_state = "1011101010111111",
    command = "Q",
    payload = { query: "capabilities", filter: "orchestration" };

-- C = CONNECT
CREATE mcb_messages SET
    raw_message = "1011101010111111 • C",
    source_id = "8D0", dest_id = "7C1",
    binary_state = "1111111111111111",
    command = "C",
    payload = { connection: "established", mode: "persistent" };
```

#### Graph Queries

```sql
-- Show full network topology
SELECT 
    id, name, agent_id, binary_state,
    ->connects->mcb_agents.name AS connects_to,
    <-connects<-mcb_agents.name AS connected_from
FROM mcb_agents;

-- Find all paths between agents
SELECT * FROM mcb_agents:hacka->connects->mcb_agents;

-- Get message flow for specific command
SELECT * FROM mcb_messages WHERE command = 'I' ORDER BY timestamp;

-- Count connections per agent
SELECT name, count(->connects) AS outgoing, count(<-connects) AS incoming
FROM mcb_agents GROUP ALL;
```

#### MCP Connection Options

```
SurrealDB MCP Tool supports:

1. Memory (testing):     connect_endpoint('memory')
2. Local file:           connect_endpoint('file:/path/to/db')
3. Remote WebSocket:     connect_endpoint('ws://localhost:8000')
4. SurrealDB Cloud:      connect_endpoint('cloud:instance_id')
```

### AMUM-QCI-ETHIC Module

**AI ↔ Human Understanding Matrix with Quantum Coherence Interface**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AMUM-QCI-ETHIC ARCHITECTURE                              │
│  ═══════════════════════════════════════════════════════════════════════   │
│                                                                             │
│  AMUM = Analyse → User → Model → User → Model                              │
│  ─────────────────────────────────────────────                              │
│  Progressive Alignment: 3 → 6 → 9 Workflow                                  │
│  Phase 1 (divergent_3):  3 options, broad strokes                          │
│  Phase 2 (expand_6):     6 variations of selected                          │
│  Phase 3 (converge_9):   9 details, final selection                        │
│                                                                             │
│  QCI = Quantum Coherence Interface                                          │
│  ─────────────────────────────────                                          │
│  • ROV/Q: Resonance/Quality ratio                                          │
│  • Coherence Level: 0.0-1.0 alignment strength                             │
│  • Signal Strength: Communication clarity                                   │
│  • Breathing Cycle: inhale/exhale/hold states                              │
│                                                                             │
│  ETHIC = Embedded Trust & Human-Integrated Constraints                      │
│  ───────────────────────────────────────────────────                        │
│  Categories: human_dignity, transparency, accountability,                   │
│              fairness, privacy, safety, sustainability, autonomy            │
│  Sources: marcel_facebook, anthropic, eu_ai_act, woai                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### SurrealQL Schema

```sql
-- AMUM Alignment Sessions (3→6→9 Workflow)
DEFINE TABLE amum_sessions SCHEMAFULL;
DEFINE FIELD session_id ON amum_sessions TYPE string;
DEFINE FIELD user_id ON amum_sessions TYPE string;
DEFINE FIELD phase ON amum_sessions TYPE string 
    ASSERT $value IN ['divergent_3', 'expand_6', 'converge_9', 'complete'];
DEFINE FIELD intent ON amum_sessions TYPE string;
DEFINE FIELD options ON amum_sessions TYPE array;
DEFINE FIELD selected ON amum_sessions TYPE int;
DEFINE FIELD created_at ON amum_sessions TYPE datetime DEFAULT time::now();

-- ETHIC Principles
DEFINE TABLE ethic_principles SCHEMAFULL;
DEFINE FIELD name ON ethic_principles TYPE string;
DEFINE FIELD category ON ethic_principles TYPE string ASSERT $value IN [
    'human_dignity', 'transparency', 'accountability', 
    'fairness', 'privacy', 'safety', 'sustainability', 'autonomy'
];
DEFINE FIELD description ON ethic_principles TYPE string;
DEFINE FIELD source ON ethic_principles TYPE string;
DEFINE FIELD priority ON ethic_principles TYPE int;
DEFINE FIELD active ON ethic_principles TYPE bool DEFAULT true;

-- QCI Quantum Coherence States
DEFINE TABLE qci_states SCHEMAFULL;
DEFINE FIELD agent_id ON qci_states TYPE string;
DEFINE FIELD rov_q ON qci_states TYPE float;
DEFINE FIELD coherence_level ON qci_states TYPE float;
DEFINE FIELD signal_strength ON qci_states TYPE float;
DEFINE FIELD breathing_cycle ON qci_states TYPE string 
    ASSERT $value IN ['inhale', 'exhale', 'hold'];
DEFINE FIELD binary_state ON qci_states TYPE string;
DEFINE FIELD timestamp ON qci_states TYPE datetime DEFAULT time::now();

-- Relationships
DEFINE TABLE has_qci TYPE RELATION;      -- agent -> qci_state
DEFINE TABLE follows_ethic TYPE RELATION; -- agent -> ethic_principle
```

#### Core ETHIC Principles

```sql
-- Marcel's Principles + Anthropic + EU AI Act
CREATE ethic_principles:human_first SET
    name = "Human First", category = "human_dignity",
    description = "AI serves humans, not the other way around",
    source = "marcel_facebook", priority = 10;

CREATE ethic_principles:no_harm SET
    name = "No Harm", category = "safety",
    description = "AI must not cause harm to humans or environment",
    source = "anthropic", priority = 10;

CREATE ethic_principles:sandbox_default SET
    name = "Sandbox Default", category = "safety",
    description = "Untrusted code runs in sandbox, always",
    source = "woai", priority = 10;

CREATE ethic_principles:user_override SET
    name = "User Override", category = "autonomy",
    description = "User can always override AI decisions",
    source = "marcel_facebook", priority = 9;
```

#### QCI State Registration

```sql
-- Register agent QCI coherence
CREATE qci_states:agent_coherence SET
    agent_id = "7C1",
    rov_q = 12860.6508,
    coherence_level = 0.95,
    signal_strength = 4414.9401,
    breathing_cycle = "inhale",
    binary_state = "1011101010111111";

-- Link agent to QCI state
RELATE mcb_agents:claude -> has_qci -> qci_states:agent_coherence;

-- Link agent to ETHIC principles
RELATE mcb_agents:claude -> follows_ethic -> ethic_principles:human_first 
    SET priority = 10;
```

#### AMUM 3→6→9 Session Flow

```sql
-- Phase 1: Divergent (3 options)
CREATE amum_sessions:phase1 SET
    session_id = "AMUM-001",
    phase = "divergent_3",
    intent = "Create AI agent for content creation",
    options = ["Minimal", "Balanced", "Full"],
    selected = 2;

-- Phase 2: Expand (6 variations)
CREATE amum_sessions:phase2 SET
    session_id = "AMUM-001",
    phase = "expand_6",
    intent = "Balanced Agent",
    options = ["Text", "Image", "Voice", "Multimodal", "Pro", "Suite"],
    selected = 5;

-- Phase 3: Converge (9 details)
CREATE amum_sessions:phase3 SET
    session_id = "AMUM-001",
    phase = "converge_9",
    intent = "Multimodal Pro",
    options = ["GPT-4", "Claude", "Gemini", "Ollama", "Hybrid", "Edge", 
               "ElevenLabs", "OpenAI Voice", "Local TTS"],
    selected = 7;
```

#### Integrated Network Query

```sql
-- Full AMUM-QCI-ETHIC-MCP-B view
SELECT 
    id, name, agent_id, binary_state,
    ->has_qci->qci_states.coherence_level AS qci_coherence,
    ->has_qci->qci_states.breathing_cycle AS qci_cycle,
    ->follows_ethic->ethic_principles.name AS follows_principles,
    ->connects->mcb_agents.name AS connects_to,
    <-connects<-mcb_agents.name AS connected_from
FROM mcb_agents;
```

---

## Setup

```bash
# Blender as Python module (Python 3.11 ONLY!)
pip install bpy==5.0.0 duckdb
```

```sql
-- Core extensions
INSTALL ducklake;
INSTALL http_client FROM community;
INSTALL json;

-- ML extensions (the "M" in SMART!)
INSTALL infera FROM community;  -- ONNX inference
INSTALL vss;                     -- HNSW vector search

-- Analytics extensions
INSTALL datasketches FROM community;
INSTALL bitfilters FROM community;
INSTALL crypto FROM community;

-- Document processing
INSTALL shellfs FROM community;

LOAD ducklake;
LOAD http_client;
LOAD json;
LOAD infera;
LOAD vss;

-- Enable persistent HNSW indexes
SET hnsw_enable_experimental_persistence = true;

-- Create DuckLake for workflow storage (with time travel!)
ATTACH 'ducklake:workflows.ducklake' AS wf (DATA_PATH 'workflow_data/');
USE wf;
```

### SmartACE Quick Start

```python
# Python: Complete pipeline
import bpy
import duckdb
import numpy as np

# 1. SMART: Query relevant data
conn = duckdb.connect('workflows.ducklake')
results = conn.execute("""
    SELECT * FROM semantic_search('architectural columns', 5)
""").fetchall()

# 2. B (Blender): Process geometry
for obj_path, similarity in results:
    obj = bpy.data.objects.get(obj_path)
    if obj:
        # Use float32 buffer protocol (NEW in 5.0!)
        vertices = np.frombuffer(
            memoryview(obj.data.vertices[0].co).tobytes(),
            dtype=np.float32
        )
        
# 3. ART: Send to UE5
conn.execute("""
    SELECT http_post(
        'http://localhost:30010/remote/object/property',
        headers => MAP {'Content-Type': 'application/json'},
        body => '{"objectPath":"/Game/Column","propertyName":"Visibility"}'
    )
""")
```

## Schema

```sql
-- Workflows: The evolving playbook
CREATE TABLE workflows (
    name VARCHAR PRIMARY KEY,
    description VARCHAR,
    steps JSON NOT NULL,           -- Array of step definitions
    version INTEGER DEFAULT 1,
    success_rate FLOAT DEFAULT 0.0,
    total_runs INTEGER DEFAULT 0,
    learnings JSON DEFAULT '[]',   -- ACE-style bullets
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Runs: Execution history  
CREATE TABLE workflow_runs (
    id INTEGER PRIMARY KEY,
    workflow_name VARCHAR,
    version INTEGER,
    executed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    success BOOLEAN,
    duration_ms INTEGER,
    context JSON,
    results JSON,
    reflection JSON
);

-- Learnings: Curated insights
CREATE TABLE learnings (
    id INTEGER PRIMARY KEY,
    workflow_name VARCHAR,
    learning_type VARCHAR,  -- 'helpful' | 'harmful' | 'insight'
    content TEXT,
    source_run_id INTEGER,
    use_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Extensions: Discovery tracking
CREATE TABLE extensions (
    name VARCHAR PRIMARY KEY,
    source VARCHAR,              -- 'core' | 'community' | 'query.farm'
    description VARCHAR,
    installed BOOLEAN DEFAULT FALSE,
    evaluated BOOLEAN DEFAULT FALSE,
    useful_for JSON DEFAULT '[]', -- What workflows benefit
    discovered_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## HTTP via SQL (http_client)

```sql
-- GET Request
SELECT http_get('http://localhost:30010/remote/info') AS response;

-- GET with headers and params
WITH req AS (
    SELECT http_get(
        'http://localhost:30010/remote/presets',
        headers => MAP {'Accept': 'application/json'},
        params => MAP {'limit': '10'}
    ) AS res
)
SELECT 
    res->>'status' AS status,
    res->>'body' AS body
FROM req;

-- POST/PUT Request (for UE5 Remote Control)
WITH req AS (
    SELECT http_post(
        'http://localhost:30010/remote/object/property',
        headers => MAP {'Content-Type': 'application/json'},
        body => '{"objectPath":"/Game/SunSky","propertyName":"TimeOfDay","propertyValue":14.5}'
    ) AS res
)
SELECT 
    (res->>'status')::INT AS status,
    res->>'body' AS response
FROM req;
```

## The ACE Loop in SQL

### 1. EXECUTE (Generator)

```sql
-- Execute workflow steps via http_client
CREATE OR REPLACE MACRO execute_workflow(wf_name, ctx) AS (
    WITH workflow AS (
        SELECT steps, version FROM workflows WHERE name = wf_name
    ),
    step_results AS (
        SELECT 
            ordinality AS step_idx,
            step,
            http_post(
                'http://localhost:30010/remote' || (step->>'endpoint'),
                headers => MAP {'Content-Type': 'application/json'},
                body => step->>'body'
            ) AS response
        FROM workflow, 
             LATERAL unnest(from_json(steps, '["json"]')) WITH ORDINALITY AS t(step, ordinality)
    )
    SELECT 
        wf_name AS workflow,
        (SELECT version FROM workflow) AS version,
        json_group_array(json_object(
            'step', step_idx,
            'endpoint', step->>'endpoint',
            'status', (response->>'status')::INT,
            'success', (response->>'status')::INT BETWEEN 200 AND 299
        )) AS results,
        bool_and((response->>'status')::INT BETWEEN 200 AND 299) AS success
    FROM step_results
);

-- Run and store
INSERT INTO workflow_runs (workflow_name, version, success, results, context)
SELECT 
    r.workflow,
    r.version,
    r.success,
    r.results,
    '{"time_of_day": 14.5}'::JSON
FROM execute_workflow('setup_scene', '{}') AS r;
```

### 2. STORE (Automatic with DuckLake)

```sql
-- DuckLake automatically versions everything!
-- Check snapshots
SELECT * FROM ducklake_snapshots('wf');

-- See what changed
SELECT * FROM wf.table_changes('workflow_runs', 1, 2);
```

### 3. EVALUATE (Reflector)

```sql
-- Analyze last run
WITH last_run AS (
    SELECT * FROM workflow_runs 
    ORDER BY executed_at DESC LIMIT 1
),
history AS (
    SELECT 
        AVG(CASE WHEN success THEN 1.0 ELSE 0.0 END) AS avg_success,
        AVG(duration_ms) AS avg_duration
    FROM workflow_runs 
    WHERE workflow_name = (SELECT workflow_name FROM last_run)
)
SELECT 
    lr.id AS run_id,
    lr.workflow_name,
    lr.success,
    lr.duration_ms,
    CASE 
        WHEN lr.success AND h.avg_success < 0.8 THEN 'improving'
        WHEN lr.success THEN 'stable'
        ELSE 'needs_attention'
    END AS trend,
    CASE
        WHEN lr.duration_ms > h.avg_duration * 1.5 THEN 'slower than usual'
        WHEN lr.duration_ms < h.avg_duration * 0.5 THEN 'faster than usual'
        ELSE 'normal'
    END AS performance,
    h.avg_success AS historical_success_rate
FROM last_run lr, history h;
```

### 4. CURATE (Curator)

```sql
-- Add learning from reflection
INSERT INTO learnings (workflow_name, learning_type, content, source_run_id)
VALUES (
    'setup_scene',
    'insight',
    'Batch requests to /remote/batch reduce latency by 60%',
    (SELECT MAX(id) FROM workflow_runs)
);

-- Update workflow learnings
UPDATE workflows 
SET learnings = json_array_append(
    learnings, 
    '$',
    json_object(
        'type', 'insight',
        'content', 'Batch requests reduce latency',
        'added_at', CURRENT_TIMESTAMP
    )
),
version = version + 1,
updated_at = CURRENT_TIMESTAMP
WHERE name = 'setup_scene';
```

## Extension Discovery Workflow

```sql
-- Discover available community extensions
CREATE TABLE IF NOT EXISTS extension_registry AS
SELECT * FROM duckdb_extensions() 
WHERE installed = false;

-- Check what's new since last discovery
WITH known AS (SELECT name FROM extensions),
available AS (SELECT extension_name FROM duckdb_extensions())
SELECT a.extension_name AS new_extension
FROM available a
WHERE a.extension_name NOT IN (SELECT name FROM known);

-- Evaluate if extension is useful
-- (Claude analyzes description and suggests workflows)
INSERT INTO extensions (name, source, description, useful_for)
VALUES (
    'http_client',
    'community',
    'HTTP GET/POST from SQL',
    '["api_calls", "webhooks", "remote_control"]'
);

-- Query.Farm extensions to watch
INSERT INTO extensions (name, source, description) VALUES
    ('http_client', 'query.farm', 'HTTP GET/POST in SQL'),
    ('httpserver', 'query.farm', 'Turn DuckDB into HTTP API'),
    ('shellfs', 'query.farm', 'Shell commands as files'),
    ('radio', 'query.farm', 'WebSocket + Redis Pub/Sub'),
    ('airport', 'query.farm', 'Arrow Flight client'),
    ('rapidfuzz', 'query.farm', 'Fuzzy string matching'),
    ('jsonata', 'query.farm', 'JSONata expressions in SQL');
```

## Time Travel (DuckLake Magic)

```sql
-- See workflow at specific version
SELECT * FROM workflows AT (VERSION => 1) WHERE name = 'setup_scene';

-- Compare before/after a change
SELECT 
    'before' AS state, * FROM workflows AT (VERSION => 1) WHERE name = 'setup_scene'
UNION ALL
SELECT 
    'after' AS state, * FROM workflows AT (VERSION => 2) WHERE name = 'setup_scene';

-- Rollback if needed
-- (Just query old version and INSERT back)
INSERT INTO workflows 
SELECT * FROM workflows AT (VERSION => 1) WHERE name = 'setup_scene'
ON CONFLICT (name) DO UPDATE SET 
    steps = EXCLUDED.steps,
    learnings = EXCLUDED.learnings,
    version = workflows.version + 1;
```

## Complete Example: UE5 Scene Setup

```sql
-- 1. Register workflow
INSERT INTO workflows (name, description, steps) VALUES (
    'setup_hackaspace',
    'Configure HACKASPACE virtual HQ scene',
    '[
        {"endpoint": "/object/property", "body": {"objectPath": "/Game/SunSky", "propertyName": "TimeOfDay", "propertyValue": 14.5}},
        {"endpoint": "/object/property", "body": {"objectPath": "/Game/Floor", "propertyName": "RelativeScale3D", "propertyValue": {"X": 2, "Y": 2, "Z": 1}}}
    ]'
);

-- 2. Execute
WITH execution AS (
    SELECT 
        'setup_hackaspace' AS workflow,
        http_post(
            'http://localhost:30010/remote/batch',
            headers => MAP {'Content-Type': 'application/json'},
            body => (SELECT '{"Requests":' || steps || '}' FROM workflows WHERE name = 'setup_hackaspace')
        ) AS response
)
INSERT INTO workflow_runs (workflow_name, success, results)
SELECT 
    workflow,
    (response->>'status')::INT = 200,
    response->>'body'
FROM execution;

-- 3. Reflect (Claude analyzes this output)
SELECT 
    wr.id,
    wr.success,
    wr.results,
    w.success_rate,
    json_array_length(w.learnings) AS num_learnings
FROM workflow_runs wr
JOIN workflows w ON wr.workflow_name = w.name
ORDER BY wr.executed_at DESC
LIMIT 1;

-- 4. Curate (based on reflection)
-- Claude suggests: "Batch endpoint is faster than individual calls"
UPDATE workflows SET
    learnings = json_array_append(learnings, '$', 
        '{"type": "helpful", "content": "Use /remote/batch for multiple operations"}'
    ),
    version = version + 1
WHERE name = 'setup_hackaspace';
```

## Extension Suggestion Workflow

```sql
-- When workflow fails, check if extension could help
WITH failed_runs AS (
    SELECT workflow_name, results 
    FROM workflow_runs 
    WHERE NOT success
    ORDER BY executed_at DESC
    LIMIT 5
),
potential_helpers AS (
    SELECT e.name, e.description, e.useful_for
    FROM extensions e
    WHERE e.installed = false
)
SELECT 
    f.workflow_name,
    p.name AS suggested_extension,
    p.description,
    'Consider installing: INSTALL ' || p.name || ' FROM community;' AS action
FROM failed_runs f, potential_helpers p
WHERE f.results LIKE '%timeout%' AND p.name = 'http_client';
-- Claude: "Hey, http_client extension might help with these timeout issues!"
```

## Self-Improvement Query

```sql
-- Find workflows that need attention
SELECT 
    name,
    success_rate,
    total_runs,
    json_array_length(learnings) AS num_learnings,
    CASE 
        WHEN success_rate < 0.5 THEN '🔴 Critical - needs review'
        WHEN success_rate < 0.8 THEN '🟡 Needs improvement'
        ELSE '🟢 Healthy'
    END AS status
FROM workflows
ORDER BY success_rate ASC;

-- Suggest next improvement action
WITH low_performers AS (
    SELECT name FROM workflows WHERE success_rate < 0.8
)
SELECT 
    l.workflow_name,
    l.learning_type,
    l.content,
    l.use_count,
    'Apply this learning to improve workflow' AS suggestion
FROM learnings l
WHERE l.workflow_name IN (SELECT name FROM low_performers)
  AND l.learning_type = 'helpful'
  AND l.use_count < 3
ORDER BY l.created_at DESC;
```

## References

- `references/ue5_remote_control.md` - UE5 RC API
- `references/duckdb_extensions.md` - Extension catalog and patterns (20+ extensions)
- `references/duckdb_http_patterns.md` - HTTP patterns for SQL
- `references/blender5_api.md` - **Blender 5.0 Python API (bpy) - The "B" in B-SMART-ART**

## SQL Scripts

- `scripts/sql/setup.sql` - Initial setup + extension seeding
- `scripts/sql/ace_loop.sql` - ACE pattern implementation
- `scripts/sql/embeddings.sql` - **ML Pipeline: model2vec → HNSW → Semantic Search**
- `scripts/sql/blender_pipeline.sql` - **SmartACE: DuckDB ↔ Blender ↔ UE5 Bridge**
- `scripts/sql/discover_extensions.sql` - Extension discovery workflow
- `scripts/sql/time_travel.sql` - DuckLake time travel patterns
- `scripts/sql/realtime_workflow.sql` - WebSocket/Redis integration
- `scripts/sql/templates.sql` - Jinja2 template patterns
- `scripts/sql/json_processing.sql` - JSONata + JSON Schema patterns
- `scripts/sql/analytics_documents.sql` - DataSketches + Document processing
