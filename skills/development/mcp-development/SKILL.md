---
name: mcp-development
description: Model Context Protocol (MCP) server development and AI/ML integration patterns. Covers MCP server implementation, tool design, resource handling, and LLM integration best practices. Use when developing MCP servers, creating AI tools, integrating with LLMs, or when asking about MCP protocol, prompt engineering, or AI system architecture.
---

# MCP Development

## What is MCP?

The Model Context Protocol (MCP) is an open protocol that enables AI assistants to interact with external tools, data sources, and services in a standardized way.

## Core Concepts

| Concept | Description |
|---------|-------------|
| **Tools** | Functions the AI can call |
| **Resources** | Data the AI can read |
| **Prompts** | Pre-defined prompt templates |
| **Transports** | Communication methods (stdio, HTTP) |

## MCP Server Structure

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
  name: "my-mcp-server",
  version: "1.0.0",
}, {
  capabilities: {
    tools: {},
    resources: {},
  }
});

// Define tools
server.setRequestHandler("tools/list", async () => ({
  tools: [
    {
      name: "get_data",
      description: "Fetch data from the API",
      inputSchema: {
        type: "object",
        properties: {
          id: { type: "string", description: "Item ID" }
        },
        required: ["id"]
      }
    }
  ]
}));

// Handle tool calls
server.setRequestHandler("tools/call", async (request) => {
  const { name, arguments: args } = request.params;
  
  if (name === "get_data") {
    const result = await fetchData(args.id);
    return { content: [{ type: "text", text: JSON.stringify(result) }] };
  }
  
  throw new Error(`Unknown tool: ${name}`);
});

// Start server
const transport = new StdioServerTransport();
await server.connect(transport);
```

## Tool Design Best Practices

### Clear Descriptions
```typescript
{
  name: "search_documents",
  description: "Search documents by keyword. Returns matching documents with relevance scores. Use when the user asks to find or search for specific content.",
  inputSchema: {
    type: "object",
    properties: {
      query: {
        type: "string",
        description: "Search query - can include multiple keywords"
      },
      limit: {
        type: "integer",
        description: "Maximum number of results (default: 10)",
        default: 10
      }
    },
    required: ["query"]
  }
}
```

### Error Handling
```typescript
server.setRequestHandler("tools/call", async (request) => {
  try {
    const result = await executeTool(request.params);
    return { content: [{ type: "text", text: result }] };
  } catch (error) {
    return {
      content: [{
        type: "text",
        text: `Error: ${error.message}`
      }],
      isError: true
    };
  }
});
```

## Resources

```typescript
server.setRequestHandler("resources/list", async () => ({
  resources: [
    {
      uri: "file:///docs/readme.md",
      name: "README",
      description: "Project documentation",
      mimeType: "text/markdown"
    }
  ]
}));

server.setRequestHandler("resources/read", async (request) => {
  const { uri } = request.params;
  const content = await readResource(uri);
  return {
    contents: [{
      uri,
      mimeType: "text/markdown",
      text: content
    }]
  };
});
```

## Transport Options

| Transport | Use Case |
|-----------|----------|
| stdio | Local CLI tools |
| HTTP/SSE | Web services, remote servers |

## Security Considerations

- [ ] Validate all input parameters
- [ ] Sanitize file paths
- [ ] Rate limit API calls
- [ ] Don't expose secrets
- [ ] Log all tool invocations
- [ ] Handle timeouts gracefully

## Detailed References

- **MCP Patterns**: See [references/mcp-patterns.md](references/mcp-patterns.md)
- **AI/ML Integration**: See [references/ai-ml-integration.md](references/ai-ml-integration.md)

