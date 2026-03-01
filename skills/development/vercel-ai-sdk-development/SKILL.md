---
name: vercel-ai-sdk-development
description: Use when building AI-powered applications with the Vercel AI SDK (V6+). Covers agents (ToolLoopAgent), tool design with execution approval and strict mode, MCP client integration, structured output with tool calling, streaming patterns, DevTools debugging, reranking, provider-specific tools, and UI integration with React/Next.js.
---

# Vercel AI SDK V6 Development

Guidance for building production-grade AI applications using the Vercel AI SDK V6+. This skill covers the agent abstraction, tool ecosystem, MCP integration, structured output, streaming, and UI patterns.

## When to Use

- Building any AI/LLM-powered feature in a TypeScript project
- Creating agents with multi-step tool calling loops
- Integrating LLM capabilities into Next.js, React, Svelte, Vue, or Node.js applications
- Designing tools with human-in-the-loop approval workflows
- Connecting to MCP servers from application code
- Generating structured output from LLM calls
- Implementing RAG pipelines with reranking
- Debugging AI call chains with DevTools

## Core Concepts

### Model Gateway

Use the AI SDK gateway for provider-agnostic model references. This decouples your code from specific providers and enables easy model switching:

```typescript
import { gateway } from 'ai';
const model = gateway('anthropic/claude-sonnet-4.5');
```

### Agents with ToolLoopAgent

Prefer `ToolLoopAgent` for reusable agent definitions. Define the agent once with its model, instructions, and tools, then use it across API routes, background jobs, and UI streams:

```typescript
import { ToolLoopAgent } from 'ai';

export const myAgent = new ToolLoopAgent({
  model: 'anthropic/claude-sonnet-4.5',
  instructions: 'You are a helpful assistant.',
  tools: { /* tools here */ },
});

const result = await myAgent.generate({ prompt: '...' });
```

Key agent patterns:
- Use `callOptionsSchema` with `prepareCall` to inject per-request context (user ID, feature flags, retrieved documents) in a type-safe way.
- Set `stopWhen: stepCountIs(N)` to cap the tool loop (default is 20 steps).
- Export `InferAgentUIMessage<typeof agent>` for end-to-end type safety from agent definition to UI components.
- For custom agent behavior, implement the `Agent` interface directly instead of extending `ToolLoopAgent`.

### Low-Level Functions

Use `generateText` / `streamText` when you need full control over a single LLM call without the agent loop, or when composing custom orchestration logic:

```typescript
import { generateText, streamText } from 'ai';

const { text } = await generateText({
  model: 'anthropic/claude-sonnet-4.5',
  prompt: '...',
});
```

## Tool Design Best Practices

### Structure

Define tools in dedicated files under a `tools/` directory. Each tool should have a clear `description`, a typed `inputSchema` using Zod, and an `execute` function:

```typescript
import { tool } from 'ai';
import { z } from 'zod';

export const weatherTool = tool({
  description: 'Get the current weather for a city',
  inputSchema: z.object({
    city: z.string().describe('City name'),
  }),
  execute: async ({ city }) => {
    // implementation
  },
});
```

### Execution Approval (Human-in-the-Loop)

Set `needsApproval: true` on tools that perform destructive or sensitive actions (deleting data, processing payments, modifying production resources). Use a function for conditional approval based on input:

```typescript
export const deleteRecord = tool({
  description: 'Delete a database record',
  inputSchema: z.object({ id: z.string() }),
  needsApproval: true,
  execute: async ({ id }) => { /* ... */ },
});
```

### Strict Mode

Enable `strict: true` on tools with simple, provider-compatible schemas to guarantee input validation. Leave it off for tools with complex schemas that may not be supported in strict mode by all providers:

```typescript
export const simpleTool = tool({
  description: '...',
  inputSchema: z.object({ query: z.string() }),
  strict: true,
  execute: async ({ query }) => { /* ... */ },
});
```

### Input Examples

Provide `inputExamples` for tools with complex or domain-specific input patterns. This helps models generate correctly structured inputs:

```typescript
export const searchTool = tool({
  description: 'Search documents with filters',
  inputSchema: z.object({
    query: z.string(),
    dateRange: z.string().describe('ISO date range, e.g. 2025-01-01..2025-12-31'),
  }),
  inputExamples: [
    { input: { query: 'quarterly report', dateRange: '2025-01-01..2025-03-31' } },
  ],
  execute: async (input) => { /* ... */ },
});
```

### Custom Model Output with toModelOutput

Use `toModelOutput` to control what tokens go back to the model, independent of what `execute` returns. This reduces token usage when tools return large payloads:

```typescript
export const docSearchTool = tool({
  description: 'Search documents',
  inputSchema: z.object({ query: z.string() }),
  execute: async ({ query }) => {
    return { fullDocument: '...very long content...' };
  },
  toModelOutput: async ({ input, output }) => ({
    type: 'text',
    value: `Found document matching "${input.query}": ${output.fullDocument.slice(0, 200)}...`,
  }),
});
```

## Structured Output with Tool Calling

Combine multi-step tool calling with structured output generation using the `output` option. Use `Output.object()`, `Output.array()`, `Output.choice()`, `Output.json()`, or `Output.text()`:

```typescript
import { Output, ToolLoopAgent } from 'ai';
import { z } from 'zod';

const agent = new ToolLoopAgent({
  model: 'anthropic/claude-sonnet-4.5',
  tools: { /* ... */ },
  output: Output.object({
    schema: z.object({
      summary: z.string(),
      confidence: z.number(),
      recommendations: z.array(z.string()),
    }),
  }),
});

const { output } = await agent.generate({ prompt: '...' });
```

## MCP Client Integration

Use `@ai-sdk/mcp` to connect to remote MCP servers and use their tools:

```typescript
import { createMCPClient } from '@ai-sdk/mcp';

const mcpClient = await createMCPClient({
  transport: {
    type: 'http',
    url: 'https://your-server.com/mcp',
    headers: { Authorization: 'Bearer ...' },
  },
});

const tools = await mcpClient.tools();
```

For servers requiring OAuth, use the `auth` helper with an `OAuthClientProvider` to handle PKCE, token refresh, and dynamic client registration automatically.

Access MCP resources and prompts:

```typescript
const resources = await mcpClient.listResources();
const data = await mcpClient.readResource({ uri: 'file:///path' });

const prompts = await mcpClient.experimental_listPrompts();
const prompt = await mcpClient.experimental_getPrompt({
  name: 'code_review',
  arguments: { code: '...' },
});
```

## Provider-Specific Tools

Leverage provider-native tools when the platform offers optimized capabilities:

- **Anthropic:** Memory tool, tool search (regex/BM25), code execution, programmatic tool calling via `allowedCallers`
- **OpenAI:** Shell tool, apply-patch tool, MCP tool
- **Google:** Google Maps grounding, Vertex RAG Store, file search
- **xAI:** Web search, X search, code execution, image/video analysis

Prefer provider tools over custom implementations when they offer platform-optimized model integration.

## Reranking for RAG

Use the `rerank` function to reorder retrieved documents by relevance before passing them to the model. This reduces noise and improves response quality:

```typescript
import { rerank } from 'ai';
import { cohere } from '@ai-sdk/cohere';

const { rerankedDocuments } = await rerank({
  model: cohere.reranking('rerank-v3.5'),
  documents,
  query: 'user question',
  topN: 5,
});
```

Supports both plain-text and structured document reranking. Currently available with Cohere, Amazon Bedrock, and Together.ai.

## UI Integration (React / Next.js)

### Streaming with useChat

Use `useChat` on the client with typed messages from your agent definition:

```typescript
import { useChat } from '@ai-sdk/react';
import type { MyAgentUIMessage } from '@/agents/my-agent';

const { messages, sendMessage } = useChat<MyAgentUIMessage>();
```

### API Route Pattern

Expose agents via API routes using `createAgentUIStreamResponse`:

```typescript
import { createAgentUIStreamResponse } from 'ai';
import { myAgent } from '@/agents/my-agent';

export async function POST(request: Request) {
  const { messages } = await request.json();
  return createAgentUIStreamResponse({
    agent: myAgent,
    uiMessages: messages,
  });
}
```

### Tool Approval in UI

Handle `needsApproval` tools by checking `invocation.state === 'approval-requested'` and calling `addToolApprovalResponse` with the approval decision.

## DevTools for Debugging

Wrap models with `devToolsMiddleware` to get full visibility into LLM calls during development:

```typescript
import { wrapLanguageModel } from 'ai';
import { devToolsMiddleware } from '@ai-sdk/devtools';

const debugModel = wrapLanguageModel({
  model: gateway('anthropic/claude-sonnet-4.5'),
  middleware: devToolsMiddleware(),
});
```

Run `npx @ai-sdk/devtools` and open `http://localhost:4983` to inspect input/output, token usage, timing, and raw provider payloads. Use this during development to trace multi-step agent behavior.

## Project Organization

Recommended file layout for AI SDK projects:

```
src/
├── agents/        # Agent definitions (one file per agent)
├── tools/         # Tool definitions (one file per tool)
├── api/           # API routes exposing agents
├── components/    # UI components for tool views
└── lib/           # Shared utilities, MCP clients, provider config
```

## Key Rules

- **Always use AI SDK V6+** — Do not use deprecated V4/V5 patterns. Use `ToolLoopAgent` instead of manual `while` loops around `generateText`.
- **Gateway for model references** — Use `gateway('provider/model')` for provider-agnostic code. Avoid hardcoding provider-specific client imports unless using provider tools.
- **Define agents, not inline configs** — Create reusable agent files rather than repeating model/tools/instructions at every call site.
- **Tools in dedicated files** — Keep tool definitions in `tools/` for reuse across agents and type sharing with UI.
- **Approval on destructive actions** — Always set `needsApproval` on tools that modify, delete, or spend resources.
- **Control model output size** — Use `toModelOutput` on tools that return large payloads to avoid wasting context window tokens.
- **Structured output over parsing** — Use `Output.object()` with a Zod schema instead of manually parsing LLM text output.
- **Rerank before stuffing context** — When doing RAG, rerank retrieved documents and pass only the top-N to the model.
- **DevTools in development** — Always enable `devToolsMiddleware` during development for visibility into multi-step agent behavior.
- **Consult latest docs** — Use Context7 MCP (or equivalent) to verify current AI SDK API surfaces, as the SDK evolves rapidly.
