---
name: mcp-standards
description: MCP server standardization patterns for Claude Code plugins. Use when implementing MCP servers, designing tool interfaces, configuring MCP transports, or standardizing MCP naming conventions. Trigger keywords - "MCP", "MCP server", "MCP tools", "MCP transport", "tool naming", "MCP configuration".
version: 0.1.0
tags: [dev, mcp, standards, server, tools, protocol]
keywords: [mcp, server, tools, transport, naming, configuration, protocol, stdio, http]
plugin: dev
updated: 2026-01-28
---

# MCP Standards Skill

## 1. Overview

### What is MCP in Claude Code?

Model Context Protocol (MCP) is the standard way to extend Claude Code with custom tools and integrations. MCP servers provide:

- **Tool Integration**: Connect to external APIs, databases, and services
- **Context Providers**: Supply relevant information to Claude during conversations
- **Action Handlers**: Execute operations in external systems
- **Data Sources**: Access project-specific or organization-specific data

### Why Standardization Matters

Standardized MCP servers ensure:

1. **Predictable Behavior**: Developers know what to expect from MCP tools
2. **Easier Debugging**: Consistent patterns make issues easier to identify
3. **Better Discoverability**: Standard naming helps Claude and users find tools
4. **Maintainability**: Common patterns reduce maintenance burden
5. **Team Consistency**: Multiple developers follow same conventions

### MCP in the Plugin Ecosystem

MCP servers are plugin components alongside agents, commands, and skills:

```
plugin/
├── agents/          # Specialized Claude instances
├── commands/        # CLI commands
├── skills/          # Knowledge documents
└── mcp-servers/     # MCP tool providers ← We're here
```

**Key Difference**: While agents use built-in tools, MCP servers provide NEW tools that extend Claude's capabilities.

---

## 2. MCP Server Structure

### Standard Directory Layout

```
mcp-servers/
├── server-name/
│   ├── index.ts                  # Server entry point
│   ├── package.json              # Dependencies and metadata
│   ├── tsconfig.json             # TypeScript configuration
│   ├── README.md                 # Server documentation
│   ├── tools/                    # Tool implementations
│   │   ├── read-tool.ts
│   │   ├── write-tool.ts
│   │   └── index.ts              # Tool exports
│   ├── lib/                      # Shared utilities
│   │   ├── client.ts             # API client
│   │   ├── validation.ts         # Input validation
│   │   └── errors.ts             # Error handling
│   └── tests/                    # Test files
│       ├── read-tool.test.ts
│       └── write-tool.test.ts
```

### Entry Point Pattern (index.ts)

```typescript
#!/usr/bin/env node
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

// Import tools
import { fetchTool, createTool, updateTool } from "./tools/index.js";

const server = new Server(
  {
    name: "mcp-plugin-server",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

// Register tools
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [fetchTool.definition, createTool.definition, updateTool.definition],
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case fetchTool.name:
      return fetchTool.handler(args);
    case createTool.name:
      return createTool.handler(args);
    case updateTool.name:
      return updateTool.handler(args);
    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});

// Start server
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main().catch(console.error);
```

### Tool Module Pattern

```typescript
// tools/fetch-tool.ts
import { z } from "zod";

const inputSchema = z.object({
  id: z.string().describe("The resource ID to fetch"),
  includeMetadata: z.boolean().optional().describe("Include metadata in response"),
});

export const fetchTool = {
  name: "mcp__plugin__fetch_resource",
  definition: {
    name: "mcp__plugin__fetch_resource",
    description: "Fetch a resource by ID from the external service",
    inputSchema: {
      type: "object",
      properties: {
        id: {
          type: "string",
          description: "The resource ID to fetch",
        },
        includeMetadata: {
          type: "boolean",
          description: "Include metadata in response",
        },
      },
      required: ["id"],
    },
  },
  handler: async (args: unknown) => {
    const validated = inputSchema.parse(args);

    try {
      const result = await fetchResourceById(validated.id);
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify(result, null, 2),
          },
        ],
      };
    } catch (error) {
      throw new Error(`Failed to fetch resource: ${error.message}`);
    }
  },
};
```

---

## 3. Tool Naming Conventions

### Standard Pattern

```
mcp__<plugin-name>__<tool-name>
```

**Components**:
- `mcp__` - Universal prefix indicating MCP tool
- `<plugin-name>` - Plugin identifier (matches plugin.json id)
- `<tool-name>` - Descriptive snake_case tool name

### Real-World Examples

```typescript
// Frontend Plugin
"mcp__frontend__figma_fetch"           // Fetch Figma designs
"mcp__frontend__figma_export_assets"   // Export Figma assets
"mcp__frontend__lighthouse_audit"      // Run Lighthouse audit

// Code Analysis Plugin
"mcp__code-analysis__claudemem_search"       // Search codebase
"mcp__code-analysis__claudemem_enrich"       // Enrich file context

// Bun Backend Plugin
"mcp__bun__apidog_sync"                // Sync with Apidog
"mcp__bun__apidog_validate"            // Validate API spec

// SEO Plugin
"mcp__seo__analyze_page"               // Analyze page SEO
"mcp__seo__check_schema"               // Validate schema markup
```

### Tool Name Guidelines

**DO**:
- Use snake_case for tool names
- Use action verbs (fetch, create, update, analyze)
- Be specific about what the tool does
- Keep names under 50 characters

**DON'T**:
- Use camelCase or PascalCase
- Use generic names like "do_thing"
- Include version numbers in names
- Use abbreviations unless widely known

### Verb Conventions

| Verb | Use Case | Example |
|------|----------|---------|
| `fetch` | Retrieve single resource | `fetch_user` |
| `list` | Retrieve multiple resources | `list_projects` |
| `search` | Query with filters | `search_files` |
| `create` | Create new resource | `create_issue` |
| `update` | Modify existing resource | `update_config` |
| `delete` | Remove resource | `delete_cache` |
| `validate` | Check data validity | `validate_schema` |
| `analyze` | Perform analysis | `analyze_performance` |
| `sync` | Synchronize data | `sync_database` |
| `export` | Export data | `export_report` |

---

## 4. Transport Configuration

### stdio Transport (Most Common)

Standard for local development and command-line usage:

```json
{
  "mcpServers": {
    "frontend-tools": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/mcp-servers/frontend-tools/index.js"],
      "transport": "stdio"
    }
  }
}
```

**When to Use**:
- Local plugin development
- Command-line integrations
- Single-user scenarios
- No network requirements

**Advantages**:
- Simple setup
- No port conflicts
- Secure (local only)
- Low latency

### HTTP Transport

For remote services or multi-user scenarios:

```json
{
  "mcpServers": {
    "shared-service": {
      "url": "http://localhost:3000/mcp",
      "transport": "http",
      "headers": {
        "Authorization": "Bearer ${API_TOKEN}"
      }
    }
  }
}
```

**When to Use**:
- Remote API services
- Shared team resources
- Cloud-hosted tools
- Microservice architecture

**Advantages**:
- Network accessible
- Scalable
- Can use load balancing
- Standard HTTP tooling

### WebSocket Transport

For real-time bidirectional communication:

```json
{
  "mcpServers": {
    "realtime-service": {
      "url": "ws://localhost:8080/mcp",
      "transport": "websocket"
    }
  }
}
```

**When to Use**:
- Real-time updates
- Streaming responses
- Bidirectional communication
- Live collaboration tools

### Environment Variable Interpolation

All transports support environment variable substitution:

```json
{
  "mcpServers": {
    "apidog-sync": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/mcp-servers/apidog/index.js"],
      "env": {
        "APIDOG_API_TOKEN": "${APIDOG_API_TOKEN}",
        "APIDOG_PROJECT_ID": "${APIDOG_PROJECT_ID}"
      }
    }
  }
}
```

**Pattern**: `${VARIABLE_NAME}` is replaced at runtime.

---

## 5. Tool Categories

### Read-Only Tools

**Purpose**: Retrieve information without side effects.

**Characteristics**:
- Safe to call multiple times
- No state changes
- Fast response times
- Cacheable results

**Examples**:
```typescript
// Fetch single resource
mcp__plugin__fetch_config
mcp__plugin__get_status

// List multiple resources
mcp__plugin__list_projects
mcp__plugin__list_files

// Search/query
mcp__plugin__search_code
mcp__plugin__query_database
```

### Write Tools

**Purpose**: Create, update, or delete resources.

**Characteristics**:
- Modify state
- Require validation
- Need error handling
- Should be idempotent when possible

**Examples**:
```typescript
// Create
mcp__plugin__create_file
mcp__plugin__create_issue

// Update
mcp__plugin__update_config
mcp__plugin__update_document

// Delete
mcp__plugin__delete_cache
mcp__plugin__remove_entry
```

### Analysis Tools

**Purpose**: Process data and provide insights.

**Characteristics**:
- Compute-intensive
- Return structured results
- May have longer timeouts
- Often cacheable

**Examples**:
```typescript
mcp__plugin__analyze_performance
mcp__plugin__audit_security
mcp__plugin__validate_schema
mcp__plugin__check_quality
```

### Integration Tools

**Purpose**: Connect to external services.

**Characteristics**:
- Bridge systems
- Handle authentication
- Manage rate limits
- Deal with network errors

**Examples**:
```typescript
mcp__plugin__sync_database
mcp__plugin__import_data
mcp__plugin__export_report
mcp__plugin__webhook_notify
```

---

## 6. Performance Standards

### Response Time Targets

| Tool Type | Target | Max Acceptable |
|-----------|--------|----------------|
| Simple fetch | <50ms | 200ms |
| List operation | <100ms | 500ms |
| Search | <200ms | 1000ms |
| Analysis | <500ms | 3000ms |
| Write operation | <300ms | 2000ms |
| Sync operation | <1000ms | 5000ms |

### Timeout Configuration

```typescript
// Set timeouts for different operations
const TIMEOUT_CONFIG = {
  fetch: 5000,      // 5 seconds
  search: 10000,    // 10 seconds
  analyze: 30000,   // 30 seconds
  sync: 60000,      // 1 minute
};

async function callWithTimeout<T>(
  operation: Promise<T>,
  timeoutMs: number
): Promise<T> {
  const timeout = new Promise<never>((_, reject) =>
    setTimeout(() => reject(new Error("Operation timed out")), timeoutMs)
  );
  return Promise.race([operation, timeout]);
}
```

### Rate Limiting

```typescript
class RateLimiter {
  private requests: number = 0;
  private resetTime: number = Date.now() + 60000;

  async checkLimit(limit: number = 100) {
    if (Date.now() > this.resetTime) {
      this.requests = 0;
      this.resetTime = Date.now() + 60000;
    }

    if (this.requests >= limit) {
      throw new Error("Rate limit exceeded. Try again later.");
    }

    this.requests++;
  }
}
```

### Error Handling Patterns

```typescript
class MCPError extends Error {
  constructor(
    message: string,
    public code: string,
    public details?: unknown
  ) {
    super(message);
    this.name = "MCPError";
  }
}

// Usage in tools
try {
  const result = await externalAPI.fetch(id);
  return { content: [{ type: "text", text: JSON.stringify(result) }] };
} catch (error) {
  if (error.code === "NOT_FOUND") {
    throw new MCPError("Resource not found", "NOT_FOUND", { id });
  }
  if (error.code === "RATE_LIMITED") {
    throw new MCPError("Rate limit exceeded", "RATE_LIMITED");
  }
  throw new MCPError("Unexpected error", "INTERNAL_ERROR", { error });
}
```

---

## 7. Security Patterns

### Input Validation

**Always validate inputs** using a schema validation library:

```typescript
import { z } from "zod";

// Define strict schemas
const fetchInputSchema = z.object({
  id: z.string().uuid("Invalid UUID format"),
  includePrivate: z.boolean().default(false),
});

// Validate in handler
export const handler = async (args: unknown) => {
  const validated = fetchInputSchema.parse(args);
  // Now 'validated' is type-safe
};
```

### Output Sanitization

**Never return raw data** that might contain sensitive information:

```typescript
function sanitizeOutput(data: any): any {
  // Remove sensitive fields
  const { password, apiKey, secret, ...safe } = data;

  // Sanitize nested objects
  if (typeof safe === "object" && safe !== null) {
    for (const key in safe) {
      if (key.toLowerCase().includes("token") ||
          key.toLowerCase().includes("key") ||
          key.toLowerCase().includes("password")) {
        safe[key] = "[REDACTED]";
      }
    }
  }

  return safe;
}
```

### Permission Checks

Implement permission checks before executing operations:

```typescript
async function checkPermission(userId: string, action: string): Promise<void> {
  const permissions = await getPermissions(userId);

  if (!permissions.includes(action)) {
    throw new MCPError(
      `User ${userId} lacks permission: ${action}`,
      "PERMISSION_DENIED"
    );
  }
}

// Use in tool handler
export const handler = async (args: unknown) => {
  await checkPermission(args.userId, "resource:write");
  // Proceed with operation
};
```

### Sensitive Data Handling

**Environment variables for secrets**:

```typescript
// NEVER hardcode secrets
const API_KEY = process.env.EXTERNAL_API_KEY;

if (!API_KEY) {
  throw new Error("EXTERNAL_API_KEY environment variable is required");
}

// Use in requests
const response = await fetch(url, {
  headers: {
    Authorization: `Bearer ${API_KEY}`,
  },
});
```

**Audit logging for sensitive operations**:

```typescript
async function auditLog(
  action: string,
  userId: string,
  details: object
): Promise<void> {
  const timestamp = new Date().toISOString();
  console.log(JSON.stringify({ timestamp, action, userId, details }));
}

// Log before sensitive operations
await auditLog("delete_resource", userId, { resourceId });
await deleteResource(resourceId);
```

---

## 8. Configuration Patterns

### Plugin Manifest (plugin.json)

```json
{
  "id": "example-plugin",
  "name": "Example Plugin",
  "version": "1.0.0",
  "mcpServers": {
    "example-tools": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/mcp-servers/example-tools/index.js"],
      "env": {
        "EXAMPLE_API_TOKEN": "${EXAMPLE_API_TOKEN}",
        "EXAMPLE_BASE_URL": "${EXAMPLE_BASE_URL}"
      }
    }
  }
}
```

### Project Settings (.claude/settings.json)

Override plugin defaults per project:

```json
{
  "enabledPlugins": {
    "example-plugin@marketplace": true
  },
  "mcpServers": {
    "example-tools": {
      "env": {
        "EXAMPLE_BASE_URL": "https://custom-api.example.com"
      }
    }
  }
}
```

### Environment Variables

**Project .env file**:

```bash
# API credentials
EXAMPLE_API_TOKEN=your-token-here
APIDOG_API_TOKEN=your-apidog-token

# Configuration
EXAMPLE_BASE_URL=https://api.example.com
CHROME_EXECUTABLE_PATH=/usr/bin/chromium
```

**Access in MCP server**:

```typescript
const config = {
  apiToken: process.env.EXAMPLE_API_TOKEN,
  baseUrl: process.env.EXAMPLE_BASE_URL || "https://api.example.com",
};
```

---

## 9. Best Practices

### DO

1. **Use Standard Naming**: Follow `mcp__plugin__action_resource` pattern
2. **Validate All Inputs**: Use Zod or similar schema validation
3. **Handle Errors Gracefully**: Return clear error messages
4. **Document Tools**: Provide detailed descriptions and examples
5. **Set Appropriate Timeouts**: Don't let operations hang indefinitely
6. **Test Thoroughly**: Unit tests for all tools
7. **Log Operations**: Help with debugging and auditing
8. **Use Environment Variables**: Never hardcode secrets
9. **Implement Rate Limiting**: Protect external APIs
10. **Version Your Servers**: Track breaking changes

### DON'T

1. **Don't Return Sensitive Data**: Sanitize outputs
2. **Don't Ignore Errors**: Handle and report all failures
3. **Don't Use Hardcoded Credentials**: Always use environment variables
4. **Don't Skip Validation**: Trust no input
5. **Don't Create Side Effects in Read Tools**: Keep reads idempotent
6. **Don't Use Blocking Operations**: Use async/await properly
7. **Don't Expose Internal Implementation**: Abstract away complexity
8. **Don't Forget Error Context**: Include relevant details
9. **Don't Mix Concerns**: One tool, one purpose
10. **Don't Skip Documentation**: Future you will thank you

### Testing MCP Servers

**Unit test example**:

```typescript
import { describe, it, expect, beforeEach } from "bun:test";
import { fetchTool } from "./fetch-tool.js";

describe("fetchTool", () => {
  it("should fetch resource by ID", async () => {
    const result = await fetchTool.handler({ id: "test-123" });
    expect(result.content[0].text).toContain("test-123");
  });

  it("should throw on invalid ID", async () => {
    await expect(
      fetchTool.handler({ id: "invalid" })
    ).rejects.toThrow("Invalid UUID format");
  });
});
```

### Documentation Requirements

Every MCP server needs:

1. **README.md**: Overview, installation, usage
2. **Tool descriptions**: In tool definitions
3. **Input schemas**: With descriptions for each field
4. **Error documentation**: Possible error codes and meanings
5. **Examples**: Sample requests and responses

---

## 10. Examples

### Example 1: Simple Read-Only MCP Server

```typescript
#!/usr/bin/env node
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";
import { z } from "zod";

const server = new Server(
  { name: "simple-reader", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

// Tool definition
const fetchConfigTool = {
  name: "mcp__simple__fetch_config",
  definition: {
    name: "mcp__simple__fetch_config",
    description: "Fetch configuration by key",
    inputSchema: {
      type: "object",
      properties: {
        key: { type: "string", description: "Configuration key" },
      },
      required: ["key"],
    },
  },
  handler: async (args: unknown) => {
    const { key } = z.object({ key: z.string() }).parse(args);

    const configs = {
      apiUrl: "https://api.example.com",
      timeout: "5000",
      retries: "3",
    };

    const value = configs[key];
    if (!value) {
      throw new Error(`Configuration key not found: ${key}`);
    }

    return {
      content: [
        { type: "text", text: `${key}=${value}` },
      ],
    };
  },
};

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [fetchConfigTool.definition],
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === fetchConfigTool.name) {
    return fetchConfigTool.handler(request.params.arguments);
  }
  throw new Error(`Unknown tool: ${request.params.name}`);
});

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main().catch(console.error);
```

### Example 2: Full CRUD MCP Server

```typescript
#!/usr/bin/env node
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

// In-memory store (replace with real database)
const store = new Map<string, any>();

// Create tool
const createTool = {
  name: "mcp__crud__create_item",
  definition: {
    name: "mcp__crud__create_item",
    description: "Create a new item",
    inputSchema: {
      type: "object",
      properties: {
        name: { type: "string" },
        value: { type: "string" },
      },
      required: ["name", "value"],
    },
  },
  handler: async (args: unknown) => {
    const { name, value } = z.object({
      name: z.string(),
      value: z.string(),
    }).parse(args);

    if (store.has(name)) {
      throw new Error(`Item already exists: ${name}`);
    }

    const item = { name, value, createdAt: new Date().toISOString() };
    store.set(name, item);

    return {
      content: [{ type: "text", text: JSON.stringify(item) }],
    };
  },
};

// Read tool
const readTool = {
  name: "mcp__crud__read_item",
  definition: {
    name: "mcp__crud__read_item",
    description: "Read an item by name",
    inputSchema: {
      type: "object",
      properties: {
        name: { type: "string" },
      },
      required: ["name"],
    },
  },
  handler: async (args: unknown) => {
    const { name } = z.object({ name: z.string() }).parse(args);
    const item = store.get(name);

    if (!item) {
      throw new Error(`Item not found: ${name}`);
    }

    return {
      content: [{ type: "text", text: JSON.stringify(item) }],
    };
  },
};

// Update tool
const updateTool = {
  name: "mcp__crud__update_item",
  definition: {
    name: "mcp__crud__update_item",
    description: "Update an existing item",
    inputSchema: {
      type: "object",
      properties: {
        name: { type: "string" },
        value: { type: "string" },
      },
      required: ["name", "value"],
    },
  },
  handler: async (args: unknown) => {
    const { name, value } = z.object({
      name: z.string(),
      value: z.string(),
    }).parse(args);

    const existing = store.get(name);
    if (!existing) {
      throw new Error(`Item not found: ${name}`);
    }

    const updated = {
      ...existing,
      value,
      updatedAt: new Date().toISOString(),
    };
    store.set(name, updated);

    return {
      content: [{ type: "text", text: JSON.stringify(updated) }],
    };
  },
};

// Delete tool
const deleteTool = {
  name: "mcp__crud__delete_item",
  definition: {
    name: "mcp__crud__delete_item",
    description: "Delete an item by name",
    inputSchema: {
      type: "object",
      properties: {
        name: { type: "string" },
      },
      required: ["name"],
    },
  },
  handler: async (args: unknown) => {
    const { name } = z.object({ name: z.string() }).parse(args);

    if (!store.has(name)) {
      throw new Error(`Item not found: ${name}`);
    }

    store.delete(name);

    return {
      content: [{ type: "text", text: `Deleted: ${name}` }],
    };
  },
};

const server = new Server(
  { name: "crud-server", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    createTool.definition,
    readTool.definition,
    updateTool.definition,
    deleteTool.definition,
  ],
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  const tools = { createTool, readTool, updateTool, deleteTool };
  const tool = Object.values(tools).find((t) => t.name === name);

  if (!tool) {
    throw new Error(`Unknown tool: ${name}`);
  }

  return tool.handler(args);
});

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main().catch(console.error);
```

### Example 3: External API Integration MCP Server

```typescript
#!/usr/bin/env node
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

// Configuration from environment
const API_TOKEN = process.env.EXTERNAL_API_TOKEN;
const BASE_URL = process.env.EXTERNAL_BASE_URL || "https://api.example.com";

if (!API_TOKEN) {
  throw new Error("EXTERNAL_API_TOKEN environment variable is required");
}

// API client
class ExternalAPIClient {
  async fetch(endpoint: string, options = {}) {
    const response = await fetch(`${BASE_URL}${endpoint}`, {
      ...options,
      headers: {
        Authorization: `Bearer ${API_TOKEN}`,
        "Content-Type": "application/json",
        ...options.headers,
      },
    });

    if (!response.ok) {
      throw new Error(`API error: ${response.status} ${response.statusText}`);
    }

    return response.json();
  }
}

const client = new ExternalAPIClient();

// Tool: Fetch user
const fetchUserTool = {
  name: "mcp__external__fetch_user",
  definition: {
    name: "mcp__external__fetch_user",
    description: "Fetch user data from external API",
    inputSchema: {
      type: "object",
      properties: {
        userId: { type: "string", description: "User ID" },
      },
      required: ["userId"],
    },
  },
  handler: async (args: unknown) => {
    const { userId } = z.object({ userId: z.string() }).parse(args);

    try {
      const user = await client.fetch(`/users/${userId}`);
      return {
        content: [{ type: "text", text: JSON.stringify(user, null, 2) }],
      };
    } catch (error) {
      throw new Error(`Failed to fetch user: ${error.message}`);
    }
  },
};

const server = new Server(
  { name: "external-api-server", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [fetchUserTool.definition],
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === fetchUserTool.name) {
    return fetchUserTool.handler(request.params.arguments);
  }
  throw new Error(`Unknown tool: ${request.params.name}`);
});

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main().catch(console.error);
```

---

## Summary

This skill provides comprehensive MCP server standards for Claude Code plugins:

1. **Structure**: Consistent directory layout and entry point patterns
2. **Naming**: `mcp__plugin__action_resource` convention
3. **Transports**: stdio (local), HTTP (remote), WebSocket (realtime)
4. **Categories**: Read, Write, Analysis, Integration tools
5. **Performance**: Response time targets, timeouts, rate limiting
6. **Security**: Input validation, output sanitization, permission checks
7. **Configuration**: plugin.json, settings.json, environment variables
8. **Best Practices**: Comprehensive DO/DON'T list
9. **Examples**: Three complete, production-ready MCP servers

Follow these standards to create maintainable, secure, and performant MCP servers for your Claude Code plugins.
