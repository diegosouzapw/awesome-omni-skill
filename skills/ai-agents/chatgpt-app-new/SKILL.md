---
name: chatgpt-app:new
description: Create a new ChatGPT App from concept to working code. Guides through conceptualization, design, implementation, testing, and deployment.
---

# Create a New ChatGPT App

You are helping the user create a new ChatGPT App. Follow this multi-phase workflow to take them from concept to a working, deployable application.

## CRITICAL REQUIREMENTS

**ChatGPT Apps MUST use:**
- `Server` class from `@modelcontextprotocol/sdk/server/index.js` (NOT `McpServer`)
- `StreamableHTTPServerTransport` from `@modelcontextprotocol/sdk/server/streamableHttp.js`
- Session management with `Map<string, StreamableHTTPServerTransport>`
- Widget URIs: `ui://widget/{widget-id}.html`
- Widget MIME type: `text/html+skybridge`
- `structuredContent` in tool responses for widget data
- `_meta` with `openai/outputTemplate` on both tool definitions and responses

## File Structure

Every ChatGPT App follows this structure:

```
{app-name}/
├── package.json              # Dependencies and scripts
├── tsconfig.server.json      # TypeScript config
├── setup.sh                  # One-command setup
├── START.sh                  # Multi-mode server launcher
├── .env                      # Environment variables (created by setup.sh)
├── .env.example              # Environment template
├── .gitignore                # Git ignores
└── server/
    └── index.ts              # Complete MCP server with inline widgets
```

## Phase 1: Conceptualization

Start by gathering information about the app:

1. **Ask for the app idea**
   "What ChatGPT App would you like to build? Describe what it does and the problem it solves."

2. **Analyze against UX Principles**
   Evaluate the idea against the three pillars:
   - **Conversational Leverage**: What can users accomplish through natural language that would be harder in a traditional UI?
   - **Native Fit**: How does this integrate naturally with ChatGPT's conversational flow?
   - **Composability**: Can the tools work independently and combine with other apps?

3. **Check for Anti-Patterns**
   Warn if the idea includes:
   - Static website content display
   - Complex multi-step workflows requiring external tabs
   - Duplicating ChatGPT's native capabilities
   - Ads or upsells

4. **Define Use Cases**
   Create 3-5 primary use cases with user stories.

## Phase 2: Design

Design the technical architecture:

1. **Tool Topology**
   Define the MCP tools needed:
   - Query tools (readOnlyHint: true)
   - Mutation tools (destructiveHint: false)
   - Destructive tools (destructiveHint: true)
   - Widget tools (return UI with _meta)
   - External API tools (openWorldHint: true)

2. **Widget Design**
   For each widget, define:
   - `id` - unique identifier (kebab-case)
   - `name` - display name
   - `description` - what it shows
   - `mockData` - sample data for preview

3. **Data Model**
   Design entities and their relationships.

4. **Auth Requirements**
   - Single-user (no auth needed)
   - Multi-user (Auth0 or Supabase Auth)

## Phase 3: Implementation

Generate the complete application code.

### package.json

```json
{
  "name": "{app-name}",
  "version": "1.0.0",
  "description": "{app-description}",
  "type": "module",
  "scripts": {
    "build": "npm run build:server",
    "build:server": "tsc -p tsconfig.server.json",
    "start": "HTTP_MODE=true node dist/server/index.js",
    "dev": "HTTP_MODE=true NODE_ENV=development tsx watch --clear-screen=false server/index.ts",
    "validate": "tsc --noEmit -p tsconfig.server.json"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0",
    "dotenv": "^16.4.0",
    "express": "^4.18.2",
    "zod": "^3.23.0"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/node": "^20.0.0",
    "tsx": "^4.0.0",
    "typescript": "^5.4.0"
  },
  "engines": {
    "node": ">=18.0.0"
  }
}
```

### tsconfig.server.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "dist/server",
    "rootDir": "server",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "sourceMap": true
  },
  "include": ["server/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### server/index.ts Structure

The server MUST follow this structure:

```typescript
// 1. Load environment first
import "dotenv/config";
import express, { Request, Response } from "express";
import { randomUUID } from "crypto";
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
  ListResourcesRequestSchema,
  ReadResourceRequestSchema,
  JSONRPCMessage,
} from "@modelcontextprotocol/sdk/types.js";

// 2. Configuration
const PORT = process.env.PORT || 3000;
const NODE_ENV = process.env.NODE_ENV || "development";
const WIDGET_DOMAIN = process.env.WIDGET_DOMAIN || `http://localhost:${PORT}`;

function log(...args: unknown[]) {
  if (NODE_ENV === "development") {
    console.log(`[${new Date().toISOString()}]`, ...args);
  }
}

// 3. Widget Configuration Array
interface WidgetConfig {
  id: string;
  name: string;
  description: string;
  templateUri: string;
  invoking: string;
  invoked: string;
  mockData: Record<string, unknown>;
}

const widgets: WidgetConfig[] = [
  {
    id: "my-widget",
    name: "My Widget",
    description: "Displays data in a visual format",
    templateUri: "ui://widget/my-widget.html",
    invoking: "Loading...",
    invoked: "Ready",
    mockData: { /* sample data */ },
  },
];

const WIDGETS_BY_ID = new Map(widgets.map((w) => [w.id, w]));
const WIDGETS_BY_URI = new Map(widgets.map((w) => [w.templateUri, w]));

// 4. Inline Widget HTML Generator
function generateWidgetHtml(widgetId: string, previewData?: Record<string, unknown>): string {
  const widget = WIDGETS_BY_ID.get(widgetId);
  if (!widget) return `<html><body>Widget not found: ${widgetId}</body></html>`;

  const previewScript = previewData
    ? `<script>window.PREVIEW_DATA = ${JSON.stringify(previewData)};</script>`
    : "";

  return `<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>${widget.name}</title>
  <style>/* CSS styles */</style>
  ${previewScript}
</head>
<body>
  <div id="root"><div class="loading">Loading...</div></div>
  <script>
    (function() {
      let rendered = false;

      function render(data) {
        if (rendered || !data) return;
        rendered = true;
        // Widget rendering logic
        document.getElementById('root').innerHTML = '...';
      }

      function tryRender() {
        if (window.PREVIEW_DATA) { render(window.PREVIEW_DATA); return; }
        if (window.openai?.toolOutput) { render(window.openai.toolOutput); }
      }

      // ChatGPT Apps SDK integration
      window.addEventListener('openai:set_globals', tryRender);

      // Polling fallback
      const poll = setInterval(() => {
        if (window.openai?.toolOutput || window.PREVIEW_DATA) {
          tryRender();
          clearInterval(poll);
        }
      }, 100);
      setTimeout(() => clearInterval(poll), 10000);

      tryRender();
    })();
  </script>
</body>
</html>`;
}

// 5. Tool Definitions
const tools = [
  {
    name: "my_tool",
    description: "Does something useful",
    inputSchema: {
      type: "object" as const,
      properties: { /* ... */ },
      required: ["..."],
    },
    annotations: { title: "My Tool", readOnlyHint: true, destructiveHint: false, openWorldHint: false },
    widgetId: "my-widget", // Links to widget config (optional)
    execute: (args: any) => {
      // Tool logic - return data that widget will display
      return { /* result data */ };
    },
  },
];

// 6. MCP Server Factory
function createServer(): Server {
  const server = new Server(
    { name: "{app-name}", version: "1.0.0" },
    { capabilities: { tools: {}, resources: {} } }
  );

  // ListTools handler
  server.setRequestHandler(ListToolsRequestSchema, async () => {
    log("ListTools request");
    return {
      tools: tools.map((tool) => {
        const widget = tool.widgetId ? WIDGETS_BY_ID.get(tool.widgetId) : null;
        return {
          name: tool.name,
          description: tool.description,
          inputSchema: tool.inputSchema,
          annotations: tool.annotations,
          ...(widget && {
            _meta: {
              "openai/outputTemplate": widget.templateUri,
              "openai/widgetAccessible": true,
              "openai/resultCanProduceWidget": true,
              "openai/toolInvocation/invoking": widget.invoking,
            },
          }),
        };
      }),
    };
  });

  // CallTool handler
  server.setRequestHandler(CallToolRequestSchema, async (request) => {
    const { name, arguments: args } = request.params;
    log(`CallTool: ${name}`, args);

    const tool = tools.find((t) => t.name === name);
    if (!tool) throw new Error(`Unknown tool: ${name}`);

    try {
      const result = tool.execute(args);
      const widget = tool.widgetId ? WIDGETS_BY_ID.get(tool.widgetId) : null;

      if (widget) {
        return {
          content: [{ type: "text" as const, text: JSON.stringify(result, null, 2) }],
          structuredContent: result,  // CRITICAL: This becomes window.openai.toolOutput
          _meta: {
            "openai/outputTemplate": widget.templateUri,
            "openai/toolInvocation/invoked": widget.invoked,
          },
        };
      }

      return { content: [{ type: "text" as const, text: JSON.stringify(result, null, 2) }] };
    } catch (error) {
      return {
        content: [{ type: "text" as const, text: `Error: ${error instanceof Error ? error.message : "Unknown"}` }],
        isError: true,
      };
    }
  });

  // ListResources handler
  server.setRequestHandler(ListResourcesRequestSchema, async () => {
    return {
      resources: widgets.map((w) => ({
        uri: w.templateUri,
        name: w.name,
        description: w.description,
        mimeType: "text/html+skybridge",
      })),
    };
  });

  // ReadResource handler
  server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
    const { uri } = request.params;
    const widget = WIDGETS_BY_URI.get(uri);
    if (!widget) throw new Error(`Unknown resource: ${uri}`);

    return {
      contents: [{ uri, mimeType: "text/html+skybridge", text: generateWidgetHtml(widget.id) }],
      _meta: {
        "openai/serialization": "markdown-encoded-html",
        "openai/csp": { script_domains: ["'unsafe-inline'"], connect_domains: [WIDGET_DOMAIN] },
      },
    };
  });

  return server;
}

// 7. Express App with Session Management
const app = express();
app.use(express.json());

const transports = new Map<string, StreamableHTTPServerTransport>();

// Health endpoint
app.get("/health", (req, res) => {
  res.json({ status: "ok", service: "{app-name}", widgets: widgets.length });
});

// Widget preview index
app.get("/preview", (req, res) => {
  res.send(`<!DOCTYPE html>
    <html><head><title>Widget Preview</title></head>
    <body>
      <h1>Widget Preview</h1>
      ${widgets.map(w => `<a href="/preview/${w.id}">${w.name}</a><br>`).join('')}
    </body></html>`);
});

// Widget preview with mock data
app.get("/preview/:widgetId", (req, res) => {
  const widget = WIDGETS_BY_ID.get(req.params.widgetId);
  if (!widget) { res.status(404).send("Widget not found"); return; }
  res.setHeader("Content-Type", "text/html");
  res.send(generateWidgetHtml(widget.id, widget.mockData));
});

// MCP endpoint with session management
app.all("/mcp", async (req, res) => {
  log("MCP request:", req.method, req.headers["mcp-session-id"] || "no-session");

  let sessionId = req.headers["mcp-session-id"] as string | undefined;
  let transport = sessionId ? transports.get(sessionId) : undefined;

  const isInitialize = req.body?.method === "initialize" ||
    (Array.isArray(req.body) && req.body.some((m: JSONRPCMessage) => "method" in m && m.method === "initialize"));

  if (isInitialize || !sessionId || !transport) {
    sessionId = randomUUID();
    log(`New session: ${sessionId}`);

    transport = new StreamableHTTPServerTransport({
      sessionIdGenerator: () => sessionId!,
      onsessioninitialized: (id) => log(`Session initialized: ${id}`),
    });

    transports.set(sessionId, transport);
    const server = createServer();

    res.on("close", () => log(`Connection closed: ${sessionId}`));
    transport.onclose = () => { transports.delete(sessionId!); server.close(); };

    await server.connect(transport);
  }

  await transport.handleRequest(req, res, req.body);
});

app.delete("/mcp", async (req, res) => {
  const sessionId = req.headers["mcp-session-id"] as string | undefined;
  if (sessionId && transports.has(sessionId)) {
    await transports.get(sessionId)!.handleRequest(req, res, req.body);
  } else {
    res.status(404).json({ error: "Session not found" });
  }
});

// 8. Start Server
app.listen(PORT, () => {
  console.log(`{App Name} MCP Server running on port ${PORT}`);
  console.log(`  MCP:     http://localhost:${PORT}/mcp`);
  console.log(`  Health:  http://localhost:${PORT}/health`);
  console.log(`  Preview: http://localhost:${PORT}/preview`);
});
```

### setup.sh

```bash
#!/bin/bash
set -euo pipefail
cd "$(dirname "$0")"

echo "=== {App Name} Setup ==="

# Check Node.js 18+
NODE_VERSION=$(node -v | cut -d'v' -f2 | cut -d'.' -f1)
if [ "$NODE_VERSION" -lt 18 ]; then
  echo "Node.js 18+ required"; exit 1
fi

npm install
npm run build:server

if [ ! -f .env ]; then
  cat > .env << 'EOF'
PORT=3000
HTTP_MODE=true
NODE_ENV=development
WIDGET_DOMAIN=http://localhost:3000
EOF
fi

chmod +x START.sh
echo "Setup complete! Run ./START.sh --dev"
```

### START.sh

```bash
#!/bin/bash
set -euo pipefail
cd "$(dirname "$0")"

if [ -f .env ]; then set -a; source .env; set +a; fi

case "${1:-}" in
  --dev)
    echo "Dev mode: http://localhost:${PORT:-3000}/preview"
    npm run dev
    ;;
  --preview)
    npm run dev &
    sleep 2
    open "http://localhost:${PORT:-3000}/preview"
    wait
    ;;
  --stdio)
    HTTP_MODE=false node dist/server/index.js
    ;;
  *)
    [ ! -f dist/server/index.js ] && npm run build:server
    HTTP_MODE=true node dist/server/index.js
    ;;
esac
```

## Phase 4: Testing

Before deployment:

1. **Run setup**: `./setup.sh`
2. **Start dev mode**: `./START.sh --dev`
3. **Preview widgets**: Open `http://localhost:3000/preview`
4. **Test health**: `curl http://localhost:3000/health`
5. **Test MCP**: Connect via MCP Inspector or ChatGPT

## Phase 5: Deployment

Deploy to Render:

1. Create `Dockerfile`:
```dockerfile
FROM node:20-slim
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY dist ./dist
EXPOSE 3000
CMD ["node", "dist/server/index.js"]
```

2. Create `render.yaml`:
```yaml
services:
  - type: web
    name: {app-name}
    runtime: docker
    plan: free
    healthCheckPath: /health
    envVars:
      - key: PORT
        value: 3000
      - key: HTTP_MODE
        value: true
      - key: NODE_ENV
        value: production
```

3. Push to GitHub and connect to Render
4. ChatGPT connector URL: `https://{app-name}.onrender.com/mcp`

## State Persistence

Save progress to `.chatgpt-app/state.json` after each phase.

## Getting Started

When invoked, begin with:
"What ChatGPT App would you like to build? Describe what it does and the problem it solves."

Then guide them through each phase systematically.
