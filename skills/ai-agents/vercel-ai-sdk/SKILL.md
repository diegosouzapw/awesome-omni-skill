---
name: vercel-ai-sdk
description: Comprehensive guide to Vercel AI SDK for building AI-powered applications with text generation, streaming, tool calling, autonomous agents, and React UI integration. Use when working with AI SDK, Vercel AI, useChat, streamText, generateText, generateObject, tool calling, LLM tools, AI agents, streaming chat, chatbots, AI Elements, PromptInput, Message components, streamUI, or building conversational interfaces with language models. Focuses on Anthropic Claude with extended thinking, prompt caching, and code execution.
---

# Vercel AI SDK Expert

Build production-ready AI applications with streaming, tool calling, autonomous agents, and polished UI components. This skill covers the AI SDK Core (text generation, structured output, tools), AI SDK UI (React hooks), AI SDK RSC (experimental server components), and AI Elements (pre-built UI components).

## Installation

```bash
# Core SDK + Anthropic provider (recommended)
npm install ai @ai-sdk/anthropic zod

# React hooks for UI
npm install @ai-sdk/react

# AI Elements components (requires shadcn/ui)
npx ai-elements@latest add message prompt-input conversation
```

**Prerequisites:**

- Node.js 18+
- Next.js (App Router recommended)
- Anthropic API key: `ANTHROPIC_API_KEY` in `.env.local`
- For AI Elements: shadcn/ui, Tailwind CSS 4, React 19

## Quick Start: Chat with Tools

```typescript
// app/api/chat/route.ts
import { anthropic } from "@ai-sdk/anthropic";
import { streamText, tool } from "ai";
import { z } from "zod";

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = streamText({
    model: anthropic("claude-sonnet-4-5"),
    messages,
    tools: {
      getWeather: tool({
        description: "Get weather for a location",
        parameters: z.object({
          location: z.string(),
        }),
        execute: async ({ location }) => ({
          temperature: 72,
          condition: "sunny",
        }),
      }),
    },
  });

  return result.toDataStreamResponse();
}
```

```typescript
// app/page.tsx
'use client';
import { useChat } from '@ai-sdk/react';
import { Message, PromptInput } from '@/components/ai-elements';

export default function Chat() {
  const { messages, sendMessage, status } = useChat();

  return (
    <div>
      {messages.map(msg => (
        <Message key={msg.id} from={msg.role}>
          {msg.content}
        </Message>
      ))}
      <PromptInput
        onSubmit={({ text }) => sendMessage({ content: text })}
        disabled={status === 'streaming'}
      />
    </div>
  );
}
```

## Core Concepts

### 1. Text Generation

**`streamText()`** - Stream responses for interactive UIs

- Real-time token streaming to client
- Tool calling with automatic execution
- Callbacks: `onChunk`, `onFinish`, `onStepFinish`
- Returns: `toDataStreamResponse()`, `textStream`, `fullStream`

**`generateText()`** - Synchronous generation for non-interactive tasks

- Await full completion (drafts, summaries, agents)
- Tool calling with multiple rounds
- Returns: `text`, `toolCalls`, `toolResults`, `usage`, `steps`

**`generateObject()`** - Structured output with Zod schema validation

- Extract typed data from unstructured input
- Returns: `object`, `usage`, `warnings`

See [references/core-api.md](references/core-api.md) for full API reference.

### 2. Tool Calling

Define tools with `tool()` helper:

```typescript
tool({
  description: "Present options to user as buttons",
  parameters: z.object({
    question: z.string(),
    options: z.array(z.string()),
  }),
  execute: async ({ question, options }) => {
    return { selected: null }; // UI-only tool
  },
});
```

**Key Patterns:**

- **Multi-step**: Use `maxSteps` or `stopWhen` for tool loops
- **UI-only tools**: Return minimal data, render in client
- **Programmatic tools** (Anthropic): Tools callable from code execution

See [references/agents.md](references/agents.md) for workflow patterns.

### 3. Autonomous Agents

**`ToolLoopAgent`** - Multi-step autonomous agent

```typescript
import { ToolLoopAgent } from "ai/agents";

const agent = new ToolLoopAgent({
  model: anthropic("claude-sonnet-4-5"),
  instructions: systemPrompt,
  tools: { searchDocs, updateSpec, scheduleCall },
  stopWhen: (result) => {
    // Stop when UI-only tool called
    return result.steps.some(
      (step) =>
        "toolCalls" in step &&
        step.toolCalls.some((tc) => tc.toolName === "scheduleCall"),
    );
  },
});

const result = await agent.execute({ messages });
return createAgentUIStreamResponse({ result }).toUIMessageStreamResponse();
```

See [references/agents.md](references/agents.md) for loop control and oyster patterns.

### 4. React Hooks (AI SDK UI)

**`useChat()`** - Complete chat interface state management

- **Messages**: Array of `UIMessage` with parts (text, tool-call, tool-result)
- **`sendMessage()`**: Send with optional files, metadata
- **Status**: `'ready' | 'submitted' | 'streaming' | 'error'`
- **Transports**: DefaultChatTransport (HTTP), DirectChatTransport (in-process)

```typescript
const { messages, sendMessage, status, stop, reload } = useChat({
  api: "/api/chat",
  onToolCall: ({ toolCall }) => {
    if (toolCall.toolName === "updateSpec") {
      setSpec(toolCall.input);
    }
  },
});
```

See [references/ui-hooks.md](references/ui-hooks.md) for full hook APIs.

### 5. AI Elements Components

Pre-built, customizable components built on shadcn/ui:

- **`<Message>`** - Container with role-based styling (`from="user" | "assistant"`)
- **`<MessageResponse>`** - Markdown rendering with syntax highlighting
- **`<PromptInput>`** - Input with file upload, voice, submit button
- **`<Conversation>`** - Auto-scrolling container with empty states
- **`<Reasoning>`** - Collapsible thinking/reasoning display
- **`<Tool>`** - Tool execution visualization

Install: `npx ai-elements@latest add <component-name>`

See [references/elements-components.md](references/elements-components.md) for full component APIs.

### 6. React Server Components (Experimental)

**⚠️ Not production-ready** - Use AI SDK UI for production apps.

**`streamUI()`** - Stream React components from server

```typescript
const result = streamUI({
  model: anthropic('claude-sonnet-4-5'),
  prompt: 'Show me a chart',
  tools: {
    showChart: tool({
      parameters: z.object({ data: z.array(z.number()) }),
      generate: function* ({ data }) {
        yield <Spinner />;
        return <Chart data={data} />;
      },
    }),
  },
});

return result.value; // ReactNode
```

See [references/rsc-api.md](references/rsc-api.md) for RSC APIs.

## Anthropic-Specific Features

While AI SDK is provider-agnostic, Anthropic Claude offers unique capabilities:

### Extended Thinking

Models reason through complex problems before responding:

```typescript
streamText({
  model: anthropic("claude-sonnet-4"),
  providerOptions: {
    anthropic: {
      thinking: { type: "enabled", budgetTokens: 12000 },
    },
  },
});
```

Render in UI with `<Reasoning>` component or access `reasoningText` in results.

### Prompt Caching

Cache system prompts, long contexts, or tool definitions:

```typescript
const systemMessage: CoreSystemMessage = {
  role: "system",
  content: largeContext,
  experimental_providerMetadata: {
    anthropic: { cacheControl: { type: "ephemeral", ttl: "1h" } },
  },
};
```

Reduces cost and latency for repeated contexts (>1024 tokens).

### Code Execution

Run Python in sandboxed containers:

```typescript
import { anthropic } from "@ai-sdk/anthropic";

streamText({
  model: anthropic("claude-sonnet-4-5"),
  tools: {
    codeExecution: anthropic.tools.codeExecution_20250825(),
    // Programmatic tools - callable from code
    searchDocs: tool({
      parameters: z.object({ query: z.string() }),
      execute: async ({ query }) => searchResults,
      experimental_allowedCallers: ["code_execution_20250825"],
    }),
  },
  providerOptions: {
    anthropic: {
      forwardAnthropicContainerIdFromLastStep: true, // Persist container
    },
  },
});
```

See [references/anthropic-features.md](references/anthropic-features.md) for provider-defined tools, MCP connectors, context management, and more.

## Error Handling

AI SDK provides robust error handling patterns:

```typescript
try {
  const result = await streamText({ model, messages });
} catch (error) {
  if (error instanceof APICallError) {
    console.error("API Error:", error.statusCode, error.message);
  } else if (error instanceof InvalidToolArgumentsError) {
    console.error("Tool Error:", error.toolName, error.cause);
  }
}
```

**Patterns:**

- **Retries**: Wrap calls with exponential backoff
- **Partial results**: Handle `toolResults` even on errors
- **Tool fallbacks**: Catch in `execute()`, return error state
- **Stream interruption**: Use `stop()` from useChat

See [references/error-handling.md](references/error-handling.md) for comprehensive patterns.

## Common Workflows

1. **Simple Chat**: `useChat()` + `streamText()` with message API
2. **Chat with Tools**: Add `tools` object, render with custom UI
3. **Autonomous Agent**: `ToolLoopAgent` with `stopWhen` control
4. **Structured Extraction**: `generateObject()` with Zod schema
5. **UI Streaming**: `createAgentUIStreamResponse()` for tool streaming
6. **File Upload**: `PromptInput` with file handling, multi-modal messages

## Architecture Patterns

### Server-Side (API Route)

```typescript
// app/api/chat/route.ts
import { anthropic } from "@ai-sdk/anthropic";
import { streamText } from "ai";

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = streamText({
    model: anthropic("claude-sonnet-4-5"),
    messages,
    system: buildSystemPrompt(),
    tools: createTools(),
  });

  return result.toDataStreamResponse();
}
```

### Client-Side (React)

```typescript
// components/chat.tsx
'use client';
import { useChat } from '@ai-sdk/react';
import { Message, PromptInput, Conversation } from '@/components/ai-elements';

export function Chat() {
  const { messages, sendMessage, status } = useChat({ api: '/api/chat' });

  return (
    <Conversation>
      {messages.map(msg => (
        <Message key={msg.id} from={msg.role}>
          <MessageResponse>{msg.content}</MessageResponse>
        </Message>
      ))}
      <PromptInput onSubmit={({ text }) => sendMessage({ content: text })} />
    </Conversation>
  );
}
```

## Progressive Disclosure

- **Start here**: Quick start example, core concepts
- **Deep dive**: Reference files for specific APIs
- **Real examples**: `examples/` folder with production patterns
- **Anthropic features**: When you need advanced capabilities

## Reference Files

Load as needed for detailed documentation:

- [core-api.md](references/core-api.md) - generateText, streamText, generateObject, tool definitions
- [ui-hooks.md](references/ui-hooks.md) - useChat, useObject, transports, message handling
- [agents.md](references/agents.md) - ToolLoopAgent, stopWhen patterns, oyster workflows
- [rsc-api.md](references/rsc-api.md) - streamUI, state management (experimental)
- [elements-components.md](references/elements-components.md) - Full component prop tables
- [anthropic-features.md](references/anthropic-features.md) - Thinking, caching, code execution, provider tools
- [error-handling.md](references/error-handling.md) - Retry patterns, fallbacks, validation

## Examples

See [examples/](examples/) for complete implementations:

- [tool-registry.tsx](examples/tool-registry.tsx) - Custom tool UI rendering
- [chat-with-tools.tsx](examples/chat-with-tools.tsx) - Full chat with tool calling
- [streaming-object.tsx](examples/streaming-object.tsx) - Structured data extraction
- [elements-chat.tsx](examples/elements-chat.tsx) - Chat with AI Elements
- [anthropic-thinking.tsx](examples/anthropic-thinking.tsx) - Reasoning display
- [agent-workflow.tsx](examples/agent-workflow.tsx) - Multi-step autonomous agent

## Best Practices

1. **Use `streamText()` for UIs**, `generateText()` for non-interactive tasks
2. **Define tools clearly** - Descriptive names, precise parameter schemas
3. **Control loops** - Use `stopWhen` or `maxSteps` to prevent runaway agents
4. **Cache aggressively** - System prompts, tool definitions, long contexts (Anthropic)
5. **Handle errors gracefully** - Retries, fallbacks, partial results
6. **Throttle UI updates** - `experimental_throttle` for smooth rendering
7. **Type everything** - Use Zod for tool parameters and structured output
8. **Test tools independently** - Mock `execute` functions for reliability
9. **Monitor usage** - Track tokens, costs, errors in callbacks
10. **Progressive enhancement** - Start simple, add tools/agents as needed

## Resources

- **Docs**: https://ai-sdk.dev/docs
- **AI Elements**: https://ai-sdk.dev/elements
- **Examples**: https://github.com/vercel/ai
- **Anthropic**: https://docs.anthropic.com
