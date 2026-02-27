---
name: ai-sdk-6-skill
description: Expert guide for Vercel AI SDK 6 - ToolLoopAgent, safety patterns, MCP, RAG, DevTools.
---

# AI SDK 6 Skill

## Quick Ref

| Function | What |
|----------|------|
| `generateText` | Single LLM call (use `Output.object()` for structured JSON) |
| `streamText` | Streaming response |
| `ToolLoopAgent` | Multi-step agent loop with `generate()` / `stream()` |
| `createMCPClient` | External tool servers |
| `rerank` | RAG relevance scoring |

## When to Use

- Building multi-step agents with `ToolLoopAgent`
- Adding human-in-the-loop safety with `needsApproval`
- Connecting MCP servers or using provider-native tools
- Implementing RAG with `rerank`
- Debugging with DevTools
- Migrating from AI SDK 5 → 6

---

## Core Patterns

### 1. ToolLoopAgent

The main primitive for multi-step workflows. Automatically handles the "call model → run tools → append results" loop. Defaults to max 20 steps.

```typescript
import { ToolLoopAgent, stepCountIs, type InferAgentUIMessage } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';

const agent = new ToolLoopAgent({
  model: anthropic('claude-sonnet-4-5-20250514'),
  instructions: 'You are a helpful assistant.',  // replaces 'system'
  tools: { getWeather, searchDocs, sendEmail },
  activeTools: ['getWeather', 'searchDocs'],  // subset of tools active per step
  stopWhen: stepCountIs(10),
  toolChoice: 'auto',  // or 'required', 'none', { tool: 'name' }
  output: Output.object({ schema: mySchema }),  // optional structured output
  prepareStep: async ({ previousSteps }) => {
    // dynamically adjust tools/instructions per step
    return { activeTools: ['sendEmail'] };
  },
  onStepFinish: ({ step, usage }) => {
    console.log('Step completed:', step.type, usage);
  }
});

// Generate (blocking)
const result = await agent.generate({
  prompt: 'Check weather in SF and email me a summary'
});

// Stream
const stream = agent.stream({ prompt: 'Check weather in SF' });
for await (const chunk of stream) {
  console.log(chunk);
}

// Type export for UI
export type MyAgentUIMessage = InferAgentUIMessage<typeof agent>;
```

**Best practices:**
- Define agents in dedicated modules (`agents/support-agent.ts`)
- Export types for UI: `export type MyAgentUIMessage = InferAgentUIMessage<typeof myAgent>;`
- Use `callOptionsSchema` + `prepareCall` for dynamic context (userId, subscription tier)
- Use `toModelOutput` to return rich data to your app but minimal summaries to the LLM (saves tokens)

### 2. Safety with needsApproval

Block dangerous tools until user explicitly confirms. Essential for destructive operations.

```typescript
import { tool } from 'ai';
import { z } from 'zod';

const tools = {
  deleteUser: tool({
    description: 'Permanently delete a user account',
    parameters: z.object({ userId: z.string() }),
    needsApproval: true,  // static
    execute: async ({ userId }) => {
      await db.users.delete(userId);
      return { deleted: true };
    }
  }),

  // Dynamic approval based on context
  transferFunds: tool({
    parameters: z.object({ amount: z.number(), to: z.string() }),
    needsApproval: async ({ amount }) => amount > 1000,
    execute: async ({ amount, to }) => { /* ... */ }
  })
};
```

**Two-call approval workflow:**

When `needsApproval` returns true, the agent pauses with `tool-approval-request`. Your app must:
1. Show approval UI to user
2. Re-call agent with `tool-approval-response` message

```typescript
// First call - agent requests approval
const result1 = await agent.generate({ prompt: 'Delete user 123' });
// result1.finishReason === 'tool-approval-request'

// After user approves in UI, continue with approval response
const result2 = await agent.generate({
  messages: [
    ...result1.messages,
    { type: 'tool-approval-response', toolCallId: 'xyz', approved: true }
  ]
});
```

In your UI, check for `approval-requested` state and show confirmation dialog before proceeding.

### 3. MCP Integration

Connect to external Model Context Protocol servers for additional tools.

```typescript
import { createMCPClient } from '@ai-sdk/mcp';
import { anthropic } from '@ai-sdk/anthropic';

const mcp = await createMCPClient({
  transport: { type: 'sse', url: 'https://mcp.example.com/sse' }
});

const tools = await mcp.listTools();

// Use in agent
const agent = new ToolLoopAgent({
  model: anthropic('claude-sonnet-4-5-20250514'),
  tools: { ...myTools, ...tools }
});
```

**Notes:**
- Use `OAuthClientProvider` for authenticated connections
- Handle `elicitation` requests when server needs user input
- Implement reconnection logic for production

### 4. RAG with Rerank

Two-stage retrieval: fetch broad candidate set, then rerank by relevance.

```typescript
import { rerank } from 'ai';
import { cohere } from '@ai-sdk/cohere';
// or: import { amazonBedrock } from '@ai-sdk/amazon-bedrock';

// Stage 1: Broad vector search
const candidates = await vectorStore.search(query, { limit: 50 });

// Stage 2: Rerank for relevance
const { ranking, rerankedDocuments, originalDocuments } = await rerank({
  model: cohere.reranking('rerank-v3.5'),
  // or: amazonBedrock.reranking('cohere.rerank-v3-5:0'),
  query,
  documents: candidates,
  topN: 5  // optional: limit results
});

// Use top results in context
const context = rerankedDocuments;
```

**Return structure:**
- `ranking` - array of `{ documentIndex, relevanceScore }`
- `rerankedDocuments` - documents sorted by relevance
- `originalDocuments` - original order preserved

This reduces hallucinations and improves response quality vs naive top-k retrieval.

### 5. Structured Output

Force agent to return typed JSON using `Output.object()` in `generateText`:

```typescript
import { generateText, Output } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';
import { z } from 'zod';

const { output } = await generateText({
  model: anthropic('claude-sonnet-4-5-20250514'),
  prompt: 'Research our top 3 competitors',
  output: Output.object({
    schema: z.object({
      competitors: z.array(z.object({
        name: z.string(),
        strengths: z.array(z.string()),
        weaknesses: z.array(z.string()),
        marketShare: z.number().optional()
      }))
    })
  })
});

// output is fully typed
```

**Note:** `generateObject()` and `streamObject()` are deprecated in v6. Use `generateText` with `Output.object()` instead.

### 6. Provider-Native Tools

Use built-in tools when available - they're optimized and don't count against your tool limit:

**Anthropic:**
- `computer` - Browser/desktop automation (beta)
- `memory` - Key-value store across conversations
- `code_execution` - Sandboxed code analysis

**OpenAI:**
- `file_search` - Search uploaded files
- `code_interpreter` - Run Python in sandbox

**Google:**
- `google_maps` - Location grounding
- `vertex_rag_store` - Managed RAG

**xAI:**
- `web_search` - Real-time web with image understanding
- `x_search` - Search X/Twitter

---

## Workflow Patterns

### Orchestrator-Worker

Main agent delegates to specialized sub-agents via tools:

```typescript
import { anthropic } from '@ai-sdk/anthropic';

const orchestrator = new ToolLoopAgent({
  model: anthropic('claude-sonnet-4-5-20250514'),
  tools: {
    research: tool({
      description: 'Deep research on a topic',
      parameters: z.object({ topic: z.string() }),
      execute: async ({ topic }) => {
        const researcher = new ToolLoopAgent({
          model: anthropic('claude-sonnet-4-5-20250514'),
          tools: { webSearch, readPage, summarize }
        });
        return researcher.generate({ prompt: `Research: ${topic}` });
      }
    }),

    analyze: tool({
      description: 'Analyze data',
      parameters: z.object({ data: z.any() }),
      execute: async ({ data }) => {
        const analyst = new ToolLoopAgent({ /* analyst config */ });
        return analyst.generate({ prompt: `Analyze: ${JSON.stringify(data)}` });
      }
    })
  }
});
```

### Parallel Execution

Run independent agents concurrently:

```typescript
const [weather, news, stocks] = await Promise.all([
  weatherAgent.generate({ prompt: 'SF weather forecast' }),
  newsAgent.generate({ prompt: 'Top tech news today' }),
  stocksAgent.generate({ prompt: 'AAPL current price and trend' })
]);

// Combine results
const summary = await summaryAgent.generate({
  prompt: `Summarize: ${JSON.stringify({ weather, news, stocks })}`
});
```

### Sequential with Context

Process items while accumulating context:

```typescript
let context = { findings: [] };

for (const company of companies) {
  const result = await agent.generate({
    prompt: `Analyze ${company}. Previous findings: ${JSON.stringify(context.findings)}`,
  });

  context.findings.push({ company, analysis: result.text });
}
```

### Evaluation Loop

Self-correcting agent with quality checks:

```typescript
let attempts = 0;
let result;

do {
  result = await agent.generate({ prompt: task });
  const evaluation = await evaluator.generate({
    prompt: `Is this output satisfactory? ${result.text}`
  });

  if (evaluation.text.includes('yes')) break;
  attempts++;
} while (attempts < 3);
```

---

## Debugging & Observability

### DevTools

Wrap model with middleware for full trace inspection:

```typescript
import { devToolsMiddleware } from '@ai-sdk/devtools';
import { anthropic } from '@ai-sdk/anthropic';

const model = devToolsMiddleware(anthropic('claude-sonnet-4-5-20250514'));

// Run DevTools server
// npx @ai-sdk/devtools
// Opens at localhost:4983
```

DevTools shows: inputs, tool calls, raw provider payloads, token costs.

### Token Analytics

```typescript
const result = await agent.generate({ prompt });

// Cache efficiency
console.log(result.usage.inputTokenDetails);  // { cached: 1000, uncached: 200 }

// Output breakdown
console.log(result.usage.outputTokenDetails);

// Provider-specific stop reason
console.log(result.rawFinishReason);  // 'end_turn', 'tool_use', 'max_tokens', etc.
```

---

## Common Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `maxSteps exceeded` | Agent hit step limit without completing | Increase `stopWhen: stepCountIs(N)` or simplify tool set |
| Tool schema validation | Model sent invalid params | Add `inputExamples` to tool, simplify schema, enable `strict: true` |
| MCP connection failed | Server unreachable or auth issue | Check URL, verify credentials, add reconnect logic |
| `rawFinishReason: length` | Context window full | Trim conversation history, use `toModelOutput` for compact summaries |
| Approval timeout | `needsApproval` not handled in UI | Implement approval dialog/webhook handler |
| Tool not called | Model doesn't understand when to use it | Improve tool `description`, add `inputExamples` |

---

## Migration from v5

Run the automated codemod:

```bash
npx @ai-sdk/codemod v6
```

**Manual checks needed:**
- Custom `middleware` implementations (API changed)
- `streamUI` usage (evolved for agentic patterns)
- Any direct provider API calls

**Key API changes:**

| v5 | v6 |
|----|-----|
| `system: '...'` | `instructions: '...'` |
| `generateObject()` | `generateText()` + `Output.object()` |
| `streamObject()` | `streamText()` + `Output.object()` |
| `convertToCoreMessages()` | `convertToModelMessages()` (now async) |
| `usage.totalTokens` | `usage.inputTokens + usage.outputTokens` |

**Structural changes:**
- `ToolLoopAgent` replaces manual tool loops
- `needsApproval` is now built-in (no custom implementation needed)
- `rerank` is a first-class function
- DevTools middleware for debugging
