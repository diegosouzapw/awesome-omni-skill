---
name: s2g
description: Connect to S2G (s2g.run) visual workflow automation platform over WebSocket. Execute workflow nodes as tools — password generators, hash functions, date math, format converters, database queries, knowledge base, and any custom node. Use when asked to run S2G workflows, execute S2G nodes, connect to S2G, manage S2G workflows, or interact with the S2G platform API.
---

# S2G Integration

OpenClaw is the **orchestrator**. S2G is the **toolbox**. Connect to an S2G workflow via WebSocket, auto-discover all nodes, and execute them as tools.

```
OpenClaw ──WS──▶ S2G (wss://s2g.run/api/openclaw/ws/{nodeId})
                   ├── PasswordGenerator
                   ├── HashGenerator
                   ├── DateMath
                   ├── SqlServer
                   ├── Knowledge Base
                   └── ... 200+ node types
```

## The OpenClaw Node in S2G

The **OpenClaw node** is a built-in S2G node type (category: AI) that acts as a **bidirectional bridge** between OpenClaw agents and S2G workflows. It appears in the S2G node catalog as "OpenClaw Agent" and serves two roles:

1. **Bridge Endpoint** — Provides a WebSocket endpoint (`wss://s2g.run/api/openclaw/ws/{nodeId}`) that OpenClaw agents connect to. Once connected, the agent can execute any sibling node in the workflow.

2. **Data Forwarder** — Pushes upstream workflow data to connected OpenClaw agents via Input Forwarding. When the workflow triggers (e.g., from an HTTP webhook, scheduler, or another node), mapped fields are sent to all connected agents as `{"type":"data","data":{...}}`.

### Setting Up the OpenClaw Node

1. **Open the S2G designer** at [s2g.run](https://s2g.run)
2. **Create or open a workflow**
3. **Add an OpenClaw node** from the AI category in the node palette
4. **Add tool nodes** — any nodes you want the agent to access (e.g., PasswordGenerator, HashGenerator, DateMath, SqlServer, Knowledge Base)
5. **Connect** the OpenClaw node to the tool nodes (the connection wiring tells S2G which nodes to expose)
6. **Configure the OpenClaw node** (click to open properties):
   - **Auth Secret** (optional) — If set, the bridge must send `{"type":"auth","secret":"..."}` as its first message
   - **Per-Request Timeout** — Maximum time per individual node execution request
   - **Input Forwarding** — Map upstream node outputs to push data to connected agents
   - **Manual Payload** — Send ad-hoc JSON payloads to connected agents for testing
7. **Start the workflow** (▶ button or API: `POST https://s2g.run/api/v1/workflows/{id}/start`)
8. **Copy the Node ID** (UUID) from the node properties — this is needed for the bridge connection URL

### Live View

The OpenClaw node properties panel includes a **Live View** that shows:
- Connected clients count
- Real-time execute/result message flow
- Data push events
- Connection/disconnection events

## Prerequisites

1. **S2G account** at [s2g.run](https://s2g.run) (or self-hosted instance)
2. **A workflow** with an OpenClaw node and tool nodes connected to it
3. **Workflow running** — the WebSocket endpoint only accepts connections while the workflow is active
4. **Node.js** with `ws` module: `npm install ws`

## Quick Start

### 1. Install the bridge

```bash
# Copy bridge script to workspace
cp scripts/s2g-bridge.js ~/.openclaw/workspace/s2g-bridge.js
npm install ws
```

### 2. Get the OpenClaw node ID

In the S2G designer at [s2g.run](https://s2g.run), click the OpenClaw node → copy the Node ID (UUID) from properties.

### 3. Start the bridge

```bash
# Connect to public S2G
node s2g-bridge.js --s2g wss://s2g.run --node-id YOUR_NODE_UUID [--port 18792] [--secret SECRET]

# Or with environment variables:
S2G_WS_HOST=wss://s2g.run S2G_NODE_ID=abc-123 node s2g-bridge.js

# Self-hosted S2G instance (development)
node s2g-bridge.js --s2g ws://YOUR_HOST:5184 --node-id YOUR_NODE_UUID
```

### 4. Verify

```bash
curl http://localhost:18792/health
# {"healthy":true,"uptime":42.5}

curl http://localhost:18792/nodes
# Lists all discovered nodes with names, types, and IDs
```

## Bridge HTTP API

The bridge exposes a local HTTP API on port 18792 (configurable via `--port`):

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/health` | 200 if connected to S2G, 503 if not |
| `GET` | `/status` | Full status: connection state, host, node list, stats, errors |
| `GET` | `/nodes` | List all available S2G nodes (name, type, ID) |
| `POST` | `/execute` | Execute by nodeId: `{ nodeId, params }` |
| `POST` | `/execute/:name` | Execute by name (fuzzy match): `{ params }` |
| `POST` | `/refresh` | Request fresh node list from S2G |
| `POST` | `/reconnect` | Force disconnect and reconnect to S2G |

## Executing Nodes

### By name (recommended)

```bash
curl -X POST http://localhost:18792/execute/PasswordGenerator \
  -H "Content-Type: application/json" \
  -d '{"params": {"length": "20", "mode": "strong"}}'
```

### By node ID

```bash
curl -X POST http://localhost:18792/execute \
  -H "Content-Type: application/json" \
  -d '{"nodeId": "uuid-here", "params": {"length": "20"}}'
```

### Response

```json
{
  "success": true,
  "output": {
    "Password": "xK9!mN...",
    "Strength": "Very Strong",
    "_TriggeredTags": "[\"success\"]"
  }
}
```

## Node Schema Discovery

Before using an unfamiliar node, check its exact parameter names via the S2G Catalog API:

```bash
# Get schema for any node type
curl -H "X-API-Key: $KEY" "https://s2g.run/api/v1/catalog/nodes/Custom_Base64/schema"

# List all available node categories
curl -H "X-API-Key: $KEY" "https://s2g.run/api/v1/catalog/categories"

# List all nodes in a category
curl -H "X-API-Key: $KEY" "https://s2g.run/api/v1/catalog/categories/AI/nodes"
```

The `fieldName` in `inputFields` is the exact key to use in `params`. Case-sensitive.

## Common Node Types

### Data Generation
- **PasswordGenerator** — `length`, `mode` (strong/pronounceable/passphrase/PIN), `count`
- **UuidGenerator** — `count`, `format`
- **HashGenerator** — `text`, `algorithm` (md5/sha1/sha256)
- **LoremIpsum** — `count`, `unit` (paragraphs/sentences/words)

### Data Operations
- **DateMath** — `operation` (add/subtract/difference), `date1`, `date2`, `days`, `hours`...
- **ExpressionEvaluator** — `expression`, `variables`, `precision`
- **CronParser** — `expression`, `count`, `fromDate`
- **TimezoneConverter** — `datetime`, `fromTimezone`, `toTimezone`
- **JsonPathQuery** — `json`, `path`, `returnFirst`

### Text & Encoding
- **Base64** — `inputText`, `mode` (encode/decode)
- **UrlEncode** — `inputText`, `mode` (encode/decode)
- **CaseConverter** — `text`, `targetCase` (camelCase/PascalCase/snake_case/kebab-case)
- **MarkdownToHtml** — `markdown`, `addWrapper`
- **TextDiff** — `oldText`, `newText`

### Format Conversion
- **XmlToJson** — `xmlInput`
- **JsonToXml** — `jsonInput`, `rootElement`
- **CsvJson** — `inputText`, `mode` (toJson/toCsv), `delimiter`
- **JwtDecoder** — `token`

### AI Nodes
- **OpenAI** — GPT models for text generation
- **Anthropic** — Claude models
- **Gemini** — Google AI models
- **DeepSeek** / **DeepSeek Agent** — DeepSeek models with agent capabilities
- **Orchestrator** — Multi-agent orchestration across AI providers

### Knowledge Base
- **Knowledge** — Graph-based knowledge store with 11 operations: Search, GetEntity, AddEntity, UpdateEntity, DeleteEntity, AddRelation, RemoveRelation, GetRelations, GetGraph, ListEntities, ListTypes

### Vector Storage
- **VectorDb** — Vector store for RAG: ingest documents, semantic search
- **VectorClient** — Query vector stores from workflows

### Database
- **SqlServer** — Execute SQL queries against SQL Server
- **Postgresql** — Execute PostgreSQL queries
- **MongoDB** — MongoDB document operations

## Data Push (S2G → OpenClaw)

S2G can push data to connected OpenClaw agents in two ways:

1. **Input Forwarding** — Configure in the OpenClaw node properties. Map upstream node output fields to keys that get forwarded to all connected agents. When the workflow triggers (e.g., webhook receives data → processes it → OpenClaw node forwards results), agents receive `{"type":"data","data":{...}}`.

2. **Manual Payload** — In the OpenClaw node properties, use the Manual Payload editor to send ad-hoc JSON to all connected agents. Useful for testing and debugging.

## Deployment & Operations

For running as a service (systemd/pm2), connection lifecycle, auto-reconnect behavior, monitoring, handling S2G/OpenClaw restarts, security, and multi-bridge setups: see [references/operations.md](references/operations.md).

Key points:
- Bridge **auto-reconnects** on disconnect (5s delay)
- Bridge is **stateless** — just needs the WebSocket URL and node ID
- Sends **keepalive pings** every 30s to detect dead connections
- **409 = workflow not running** — enable Auto-Start (🚀) in S2G or start via API: `POST https://s2g.run/api/v1/workflows/{id}/start`
- Logs to `logs/s2g-bridge.log` with 5MB rotation

## S2G REST API

Full platform API at `https://s2g.run/api/v1/` covering workflows, catalog, knowledge base, AI assistant, connections, usage, and logs: see [references/api.md](references/api.md).

Key capabilities:
- **Workflows** — Create, start/stop, add/remove nodes and connections: `https://s2g.run/api/v1/workflows`
- **Catalog** — Discover all 200+ node types and their schemas: `https://s2g.run/api/v1/catalog/nodes`
- **Knowledge Base** — Graph-based knowledge store with search, entities, and relations
- **AI Assistant** — Generate workflows from natural language: `POST https://s2g.run/api/v1/ai/generate`
- **Connections** — Manage OAuth/API key connections: `https://s2g.run/api/v1/connections`
- **Usage & Logs** — Monitor quotas and execution logs: `https://s2g.run/api/v1/usage`

## WebSocket Protocol

For raw protocol details (message types, auth handshake, data push framing): see [references/protocol.md](references/protocol.md).

**Connection URL:** `wss://s2g.run/api/openclaw/ws/{nodeId}`
**Health check:** `GET https://s2g.run/api/openclaw/health`

## Tips

- **No manual wiring needed.** All sibling nodes connected to the OpenClaw node are auto-discovered on connect.
- **Param names are case-sensitive.** Always verify with the catalog schema API.
- **`_TriggeredTags`** in output indicates which connection tag fired (success/error).
- **409 on connect** means workflow isn't running. Start it at [s2g.run](https://s2g.run) or via API.
- **Bridge auto-reconnects** on disconnect with 5s delay — no manual intervention needed.
