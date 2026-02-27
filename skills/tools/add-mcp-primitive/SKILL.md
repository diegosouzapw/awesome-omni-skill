---
name: add-mcp-primitive
description: Add new MCP primitives (Tool, Resource, Prompt). Use when asked to add a new tool, resource, or prompt to the MCP server.
---

# Adding MCP Primitives

## Tool

1. Create feature in `src/features/<name>/<name>.ts`
2. Create definition in `src/definitions/tools/<name>.ts`
3. Register in `src/definitions/tools/index.ts`

```typescript
// src/definitions/tools/my-tool.ts
import { defineTool } from "@/definitions/define.ts";
import { z } from "zod";

export const myTool = defineTool({
  name: "my-tool",
  title: "My Tool",
  description: "What this tool does",
  inputSchema: {
    param: z.string().describe("Parameter description"),
  },
  handler: async ({ param }) => ({
    content: [{ type: "text", text: "Result" }],
  }),
});
```

## Resource

1. Create definition in `src/definitions/resources/<name>.ts`
2. Register in `src/definitions/resources/index.ts`

```typescript
// src/definitions/resources/my-resource.ts
import { defineResource } from "@/definitions/define.ts";

export const myResource = defineResource({
  name: "my-resource",
  uri: "my-resource://path",
  title: "My Resource",
  description: "What this resource provides",
  mimeType: "text/plain",
  handler: async (uri) => ({
    contents: [{ uri: uri.href, mimeType: "text/plain", text: "Content" }],
  }),
});
```

## Prompt

1. Create definition in `src/definitions/prompts/<name>.ts`
2. Register in `src/definitions/prompts/index.ts`

```typescript
// src/definitions/prompts/my-prompt.ts
import { definePrompt } from "@/definitions/define.ts";
import { z } from "zod";

export const myPrompt = definePrompt({
  name: "my-prompt",
  title: "My Prompt",
  description: "What this prompt does",
  argsSchema: {
    topic: z.string().describe("Topic to discuss"),
  },
  handler: ({ topic }) => ({
    messages: [
      { role: "user", content: { type: "text", text: `Discuss ${topic}` } },
    ],
  }),
});
```
