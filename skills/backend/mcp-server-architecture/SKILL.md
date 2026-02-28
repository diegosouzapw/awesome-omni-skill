---
name: MCP Server Architecture
description: This skill should be used when the user asks to "create an MCP server", "set up MCP server", "build ChatGPT app backend", "MCP transport type", "configure MCP endpoint", "server setup for Apps SDK", or needs guidance on MCP server architecture, transport protocols, or SDK setup for the OpenAI Apps SDK.
version: 0.1.0
---

# MCP Server Architecture for OpenAI Apps SDK

## Overview

MCP (Model Context Protocol) servers form the backend for ChatGPT apps built with the OpenAI Apps SDK. The server exposes tools that ChatGPT can invoke, handles authentication, and returns structured data that powers both model responses and widget UIs.

## Core Architecture

An MCP server for the Apps SDK implements three essential capabilities:

1. **List tools** - Advertise available tools with JSON Schema contracts
2. **Call tools** - Execute tool logic and return structured responses
3. **Return widgets** - Provide UI templates via resource URIs and `_meta` fields

### Data Flow

```
User prompt → ChatGPT calls MCP tool → Server executes logic →
Returns structuredContent + _meta → ChatGPT renders widget + narrates
```

## Transport Types

The Apps SDK supports two transport protocols:

### Streamable HTTP (Recommended)

Primary transport for production deployments. Use for publicly accessible servers.

**Python (FastMCP):**
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("my-server")

@mcp.tool()
def my_tool(param: str) -> str:
    return f"Result: {param}"

if __name__ == "__main__":
    mcp.run(transport="streamable-http", host="0.0.0.0", port=8000)
```

**TypeScript:**
```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";

const server = new Server({ name: "my-server", version: "1.0.0" }, { capabilities: { tools: {} } });
const transport = new StreamableHTTPServerTransport({ sessionIdGenerator: () => crypto.randomUUID() });

await server.connect(transport);
```

### Server-Sent Events (SSE)

Alternative transport for event-streaming requirements.

**Python:**
```python
mcp.run(transport="sse", host="0.0.0.0", port=8000)
```

**TypeScript:**
```typescript
import { SSEServerTransport } from "@modelcontextprotocol/sdk/server/sse.js";
const transport = new SSEServerTransport("/mcp", response);
```

## SDK Setup

### Python Setup

Install the MCP Python SDK:

```bash
pip install mcp
# Or with FastAPI support
pip install "mcp[fastapi]"
```

**Minimal server structure:**
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("server-name")

@mcp.tool()
def example_tool(query: str) -> dict:
    """Tool description for the model."""
    return {"result": query}

@mcp.resource("ui://widget/main.html")
def get_widget() -> str:
    return "<html>...</html>"

if __name__ == "__main__":
    mcp.run(transport="streamable-http", port=8000)
```

### TypeScript Setup

Install the MCP TypeScript SDK:

```bash
npm install @modelcontextprotocol/sdk zod
```

**Minimal server structure:**
```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { z } from "zod";

const server = new Server(
  { name: "server-name", version: "1.0.0" },
  { capabilities: { tools: {}, resources: {} } }
);

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: "example_tool",
    description: "Tool description",
    inputSchema: { type: "object", properties: { query: { type: "string" } } }
  }]
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "example_tool") {
    return { content: [{ type: "text", text: "Result" }] };
  }
});
```

## Response Structure

Tool responses include three layers:

| Field | Visibility | Purpose |
|-------|------------|---------|
| `structuredContent` | Model + Widget | Concise JSON the model reads for narration |
| `content` | Model + Widget | Text/image content for display |
| `_meta` | Widget only | Rich data exclusively for UI rendering |

**Example response:**
```python
return {
    "structuredContent": {"status": "success", "count": 42},
    "content": [{"type": "text", "text": "Found 42 items"}],
    "_meta": {
        "items": [...],  # Full data for widget
        "openai/outputTemplate": "ui://widget/list.html"
    }
}
```

## Server Configuration Best Practices

### Port and Host

- Use port 8000 by default for local development
- Bind to `0.0.0.0` for container deployments
- Bind to `127.0.0.1` for local-only access

### HTTPS Requirements

ChatGPT requires HTTPS for all production MCP servers. Use ngrok during development:

```bash
ngrok http 8000
```

### Environment Variables

Store sensitive configuration in environment variables:

```python
import os
API_KEY = os.environ.get("API_KEY")
DATABASE_URL = os.environ.get("DATABASE_URL")
```

### Error Handling

Return structured errors the model can understand:

```python
@mcp.tool()
def safe_tool(param: str) -> dict:
    try:
        result = process(param)
        return {"success": True, "data": result}
    except ValueError as e:
        return {"success": False, "error": str(e)}
```

## Project Structure

Recommended directory layout for MCP server projects:

```
my-mcp-server/
├── server.py          # or server.ts
├── tools/
│   ├── __init__.py
│   └── my_tool.py
├── widgets/
│   └── main.html
├── requirements.txt   # or package.json
└── .env.example
```

## Additional Resources

### Reference Files

For detailed SDK documentation and patterns:
- **`references/python-sdk.md`** - Python SDK detailed reference
- **`references/typescript-sdk.md`** - TypeScript SDK detailed reference
- **`references/transport-comparison.md`** - Transport protocol comparison

### Example Files

Working server examples in `examples/`:
- **`examples/minimal-server.py`** - Minimal Python MCP server
- **`examples/minimal-server.ts`** - Minimal TypeScript MCP server

### Official Documentation

- Apps SDK Docs: https://developers.openai.com/apps-sdk/
- MCP Specification: https://modelcontextprotocol.io/specification/
- Python SDK: https://github.com/modelcontextprotocol/python-sdk
- TypeScript SDK: https://github.com/modelcontextprotocol/typescript-sdk
