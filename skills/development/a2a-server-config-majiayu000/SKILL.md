---
name: a2a-server-config
description: Agent-to-Agent (A2A) server configuration patterns for HTTP, STDIO, SSE, and WebSocket transports. Use when building A2A servers, configuring MCP transports, setting up server endpoints, or when user mentions A2A configuration, server transport, MCP server setup, or agent communication protocols.
allowed-tools: Bash, Read, Write, Edit, Grep, Glob
---

# A2A Server Configuration

Provides complete patterns and templates for configuring Agent-to-Agent (A2A) servers with different transport mechanisms (HTTP, STDIO, SSE, WebSocket) following MCP (Model Context Protocol) standards.

## Security: API Key Handling

**CRITICAL:** When generating any configuration files or code:

- NEVER hardcode actual API keys or secrets
- NEVER include real credentials in examples
- NEVER commit sensitive values to git

- ALWAYS use placeholders: `your_service_key_here`
- ALWAYS create `.env.example` with placeholders only
- ALWAYS add `.env*` to `.gitignore` (except `.env.example`)
- ALWAYS read from environment variables in code
- ALWAYS document where to obtain keys

**Placeholder format:** `{service}_{env}_your_key_here`

## Instructions

### Phase 1: Analyze Requirements

Determine server configuration needs:

1. **Transport Type**
   - HTTP: Remote access, REST-like communication, CORS support
   - STDIO: Local process communication, pipe-based I/O
   - SSE (Server-Sent Events): Real-time streaming, one-way server push
   - WebSocket: Bidirectional real-time communication

2. **Framework Detection**
   - Python: FastAPI, Flask, Starlette
   - TypeScript: Express, Fastify, Node.js native http
   - Detect from package.json or requirements.txt

3. **Configuration Needs**
   - Port and host settings
   - CORS configuration
   - Authentication requirements
   - Environment variables

### Phase 2: Select and Load Templates

Based on requirements, use templates from `templates/`:

**Python Templates:**
- `templates/python-http-server.py` - FastAPI HTTP server
- `templates/python-stdio-server.py` - STDIO transport
- `templates/python-sse-server.py` - SSE streaming
- `templates/python-websocket-server.py` - WebSocket bidirectional

**TypeScript Templates:**
- `templates/typescript-http-server.ts` - Express HTTP server
- `templates/typescript-stdio-server.ts` - STDIO transport
- `templates/typescript-sse-server.ts` - SSE streaming
- `templates/typescript-websocket-server.ts` - WebSocket bidirectional

### Phase 3: Configure Transport

Apply configuration based on transport type:

**HTTP Configuration:**
```python
# Python (FastAPI)
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        "main:app",
        host="0.0.0.0",
        port=8000,
        reload=True
    )
```

```typescript
// TypeScript (Express)
const PORT = process.env.PORT || 8000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

**STDIO Configuration:**
```python
# Python
mcp.run(transport="stdio")
```

```typescript
// TypeScript
server.connect(new StdioServerTransport());
```

**SSE Configuration:**
```python
# Python
@app.get("/events")
async def events():
    return EventSourceResponse(event_generator())
```

**WebSocket Configuration:**
```python
# Python
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
```

### Phase 4: Add CORS and Security

For HTTP/SSE/WebSocket servers:

```python
# Python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Configure appropriately
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

```typescript
// TypeScript
import cors from 'cors';
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || '*',
  credentials: true
}));
```

### Phase 5: Environment Configuration

Create `.env.example` with placeholders:

```bash
# Server Configuration
PORT=8000
HOST=0.0.0.0
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:5173

# API Keys (NEVER commit real values)
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# Transport Settings
TRANSPORT_TYPE=http
ENABLE_CORS=true
```

### Phase 6: Validation

Run validation script:
```bash
bash scripts/validate-config.sh <server-file>
```

Checks:
- No hardcoded API keys
- Environment variable usage
- CORS configuration
- Transport setup validity
- .gitignore includes .env files

## Scripts

- `scripts/validate-config.sh` - Validate server configuration
- `scripts/generate-server.sh` - Generate server from template
- `scripts/test-transport.sh` - Test transport connectivity

## Templates

**Python:**
- `templates/python-http-server.py` - HTTP server with FastAPI
- `templates/python-stdio-server.py` - STDIO transport
- `templates/python-sse-server.py` - SSE streaming server
- `templates/python-websocket-server.py` - WebSocket server

**TypeScript:**
- `templates/typescript-http-server.ts` - HTTP server with Express
- `templates/typescript-stdio-server.ts` - STDIO transport
- `templates/typescript-sse-server.ts` - SSE streaming server
- `templates/typescript-websocket-server.ts` - WebSocket server

## Examples

- `examples/http-fastapi-example.md` - Complete HTTP server with FastAPI
- `examples/stdio-simple-example.md` - Basic STDIO server
- `examples/sse-streaming-example.md` - SSE streaming configuration
- `examples/websocket-bidirectional-example.md` - WebSocket bidirectional communication

## Requirements

- Framework-specific dependencies (FastAPI/Express/etc.)
- CORS middleware for HTTP/SSE/WebSocket
- Environment variable management (python-dotenv/dotenv)
- No hardcoded API keys or secrets
- .gitignore protection for sensitive files

## Use Cases

1. **Setting up HTTP server for remote A2A communication**
   - Load http template
   - Configure CORS
   - Set environment variables
   - Validate configuration

2. **Configuring STDIO for local agent communication**
   - Load stdio template
   - Configure process pipes
   - Test connectivity

3. **Implementing SSE for real-time agent updates**
   - Load sse template
   - Configure event streams
   - Set up CORS
   - Test streaming

4. **Setting up WebSocket for bidirectional agent chat**
   - Load websocket template
   - Configure connection handling
   - Set up authentication
   - Test bidirectional flow
