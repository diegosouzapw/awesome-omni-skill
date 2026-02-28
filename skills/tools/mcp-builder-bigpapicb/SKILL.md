---
name: mcp-builder
description: Build MCP (Model Context Protocol) servers in TypeScript or Python. Use when creating custom tools, resources, or prompts that extend AI assistant capabilities via MCP.
compatibility: Requires Node.js 18+ or Python 3.10+
metadata:
  version: "1.0"
---

# MCP Builder

## Decision Tree

```
What are you building?
    ├─ Tool (action the AI can take) → Define tool with input schema
    ├─ Resource (data the AI can read) → Define resource with URI
    └─ Prompt (reusable template) → Define prompt with arguments

What language?
    ├─ TypeScript → See references/typescript-guide.md
    └─ Python → See references/python-guide.md
```

## Quick Start (TypeScript)

```bash
npx @anthropic-ai/create-mcp-server my-server
cd my-server && npm install
```

```typescript
import { McpServer } from '@anthropic-ai/mcp-server';

const server = new McpServer({ name: 'my-server', version: '1.0.0' });

server.tool('greet', { name: { type: 'string' } }, async ({ name }) => {
  return { content: [{ type: 'text', text: `Hello, ${name}!` }] };
});

server.run();
```

## Quick Start (Python)

```bash
pip install mcp
```

```python
from mcp.server import Server

server = Server("my-server")

@server.tool("greet")
async def greet(name: str) -> str:
    return f"Hello, {name}!"

server.run()
```

## Configuration

Register in `.claude/mcp.json`:

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["path/to/my-server/dist/index.js"]
    }
  }
}
```

## Best Practices

- Return structured data (JSON) not just text
- Include error handling with meaningful messages
- Add input validation on all tool parameters
- Keep tools focused (one action per tool)
- Use descriptive names and descriptions (shown to the AI)

## For detailed guides see:
- [TypeScript MCP guide](references/typescript-guide.md)
- [Python MCP guide](references/python-guide.md)
