---
name: chatgpt-app:validate
description: Run validation suite on your ChatGPT App to check schemas, annotations, widgets, and UX compliance.
---

# Validate ChatGPT App

You are helping the user validate their ChatGPT App before testing and deployment.

## Validation Checks

### 1. Required Files Check (RUN FIRST)

Check that ALL mandatory files exist:

```bash
ls package.json tsconfig.server.json setup.sh START.sh .env.example server/index.ts
```

Expected file structure:
```
{app-name}/
├── package.json
├── tsconfig.server.json
├── setup.sh
├── START.sh
├── .env.example
├── .gitignore
└── server/
    └── index.ts
```

If ANY of the above are missing, report as **CRITICAL ERROR**.

### 2. Server Implementation Check

Verify server/index.ts uses correct patterns:

**MUST have:**
```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
```

**MUST NOT have:**
```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";  // WRONG
```

**Session management MUST exist:**
```typescript
const transports = new Map<string, StreamableHTTPServerTransport>();
```

### 3. Widget Configuration Check

Verify widgets array structure:
```typescript
const widgets: WidgetConfig[] = [
  {
    id: "widget-id",                           // kebab-case
    name: "Widget Name",
    description: "What it displays",
    templateUri: "ui://widget/widget-id.html", // MUST match this format
    invoking: "Loading...",
    invoked: "Ready",
    mockData: { /* sample data */ },
  },
];
```

### 4. Tool Response Check

Widget tools MUST return:
```typescript
return {
  content: [{ type: "text", text: JSON.stringify(result) }],
  structuredContent: result,  // CRITICAL - becomes window.openai.toolOutput
  _meta: {
    "openai/outputTemplate": widget.templateUri,
    "openai/toolInvocation/invoked": widget.invoked,
  },
};
```

### 5. Resource Handler Check

Verify ReadResource returns correct format:
```typescript
return {
  contents: [{
    uri,
    mimeType: "text/html+skybridge",  // MUST be this exact value
    text: generateWidgetHtml(widget.id),
  }],
  _meta: {
    "openai/serialization": "markdown-encoded-html",
    "openai/csp": { ... },
  },
};
```

### 6. Widget HTML Check

Verify generateWidgetHtml includes:
- `window.PREVIEW_DATA` support for local preview
- `window.openai.toolOutput` data access
- `openai:set_globals` event listener
- Polling fallback with setInterval
- `rendered` flag to prevent re-renders

### 7. Endpoint Check

Verify these endpoints exist:
- `GET /health` - Health check
- `GET /preview` - Widget preview index
- `GET /preview/:widgetId` - Individual widget preview
- `ALL /mcp` - MCP protocol handler
- `DELETE /mcp` - Session cleanup

### 8. Package.json Scripts Check

```json
{
  "scripts": {
    "build": "npm run build:server",
    "build:server": "tsc -p tsconfig.server.json",
    "start": "HTTP_MODE=true node dist/server/index.js",
    "dev": "HTTP_MODE=true NODE_ENV=development tsx watch --clear-screen=false server/index.ts"
  }
}
```

**MUST NOT have:** `dev:web`, `build:web`, `concurrently`

### 9. Annotation Validation
- Query tools have `readOnlyHint: true`
- Delete tools have `destructiveHint: true`
- External API tools have `openWorldHint: true`

### 10. Database Validation (if enabled)
- All migrations are valid SQL
- Tables have `user_subject` column
- Indexes exist for user queries

## Workflow

1. Run file existence checks
2. Read and analyze server/index.ts
3. Verify patterns match requirements
4. Collect errors and warnings
5. Display results

## Results Format

```
## Validation Results

### File Structure ✓
All required files present.

### Server Implementation ✓
Using correct Server class with session management.

### Widget Configuration ✓
2 widgets properly configured.

### Tool Responses ✓
All widget tools return structuredContent.

---
**Overall: PASS**
```

## Common Errors

| Error | Fix |
|-------|-----|
| Uses McpServer | Change to `Server` from `@modelcontextprotocol/sdk/server/index.js` |
| Missing session management | Add `Map<string, StreamableHTTPServerTransport>` |
| Wrong widget URI | Use `ui://widget/{id}.html` format |
| Wrong MIME type | Use `text/html+skybridge` |
| Missing structuredContent | Add to tool response for widget data |
| Has web/ folder | Remove - widgets are inline in server/index.ts |
