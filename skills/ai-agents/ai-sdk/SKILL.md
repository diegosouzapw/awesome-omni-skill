---
name: ai-sdk
description: Vercel AI SDK reference for building AI-powered applications. Use when implementing text/object generation (generateText, streamText, generateObject, streamObject), building chatbots with useChat/useCompletion hooks, defining tools with Zod schemas, creating agents with ToolLoopAgent, or integrating with AI providers (OpenAI, Anthropic, Google, etc.).
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Bash
---

# AI SDK

The AI SDK is Vercel's TypeScript toolkit for building AI-powered applications with React, Next.js, Vue, Svelte, Node.js, and more.

## When to Use This Skill

Use this skill when:
- Generating text or structured data with LLMs
- Building chatbot UIs with streaming
- Implementing tool calling and function execution
- Creating AI agents that use tools in a loop
- Integrating with AI providers (OpenAI, Anthropic, Google, etc.)
- Working with useChat, useCompletion, or useObject hooks

## Documentation

See the `docs/2025-12-02/` directory for complete AI SDK documentation:

### Getting Started
- `00-introduction/index.mdx` - Overview and core concepts
- `02-getting-started/` - Framework-specific quickstarts (Next.js, Svelte, Vue, Node.js)
- `02-foundations/` - Prompts, providers, tools, streaming fundamentals

### AI SDK Core
- `03-ai-sdk-core/01-overview.mdx` - Core API overview
- `03-ai-sdk-core/05-generating-text.mdx` - Text generation with generateText/streamText
- `03-ai-sdk-core/10-generating-structured-data.mdx` - Structured output with generateObject/streamObject
- `03-ai-sdk-core/15-tools-and-tool-calling.mdx` - Tool definitions and execution
- `03-ai-sdk-core/16-mcp-tools.mdx` - MCP (Model Context Protocol) tools
- `03-ai-sdk-core/40-middleware.mdx` - Request/response middleware

### Agents
- `03-agents/01-overview.mdx` - Agent fundamentals
- `03-agents/02-building-agents.mdx` - Building agents with ToolLoopAgent
- `03-agents/03-workflows.mdx` - Structured workflow patterns
- `03-agents/04-loop-control.mdx` - stopWhen and prepareStep control

### AI SDK UI (React/Vue/Svelte Hooks)
- `04-ai-sdk-ui/01-overview.mdx` - UI hooks overview
- `04-ai-sdk-ui/02-chatbot.mdx` - useChat hook for chat interfaces
- `04-ai-sdk-ui/03-chatbot-tool-usage.mdx` - Tools in chatbots
- `04-ai-sdk-ui/05-completion.mdx` - useCompletion for text completion
- `04-ai-sdk-ui/08-object-generation.mdx` - useObject for streaming JSON
- `04-ai-sdk-ui/50-stream-protocol.mdx` - Stream protocol details

### AI SDK RSC (React Server Components)
- `05-ai-sdk-rsc/01-overview.mdx` - RSC overview
- `05-ai-sdk-rsc/02-streaming-react-components.mdx` - Streaming components

### Reference
- `07-reference/01-ai-sdk-core/` - Core API reference
- `07-reference/02-ai-sdk-ui/` - UI hooks reference
- `07-reference/05-ai-sdk-errors/` - Error types

## Quick Reference

### Core Functions

```typescript
import { generateText, streamText, generateObject, streamObject } from 'ai';

// Generate text
const { text } = await generateText({
  model: anthropic('claude-sonnet-4-5-20241022'),
  prompt: 'Write a haiku about coding',
});

// Stream text
const result = streamText({
  model: anthropic('claude-sonnet-4-5-20241022'),
  prompt: 'Write a story',
});
for await (const chunk of result.textStream) {
  console.log(chunk);
}

// Generate structured data
const { object } = await generateObject({
  model: anthropic('claude-sonnet-4-5-20241022'),
  schema: z.object({
    name: z.string(),
    age: z.number(),
  }),
  prompt: 'Generate a person',
});

// Stream structured data
const { partialObjectStream } = streamObject({
  model: anthropic('claude-sonnet-4-5-20241022'),
  schema: z.object({ items: z.array(z.string()) }),
  prompt: 'List 5 fruits',
});
```

### Tool Definition

```typescript
import { tool } from 'ai';
import { z } from 'zod';

const weatherTool = tool({
  description: 'Get the weather for a location',
  inputSchema: z.object({
    location: z.string().describe('City name'),
  }),
  execute: async ({ location }) => {
    return { temperature: 72, condition: 'sunny' };
  },
});

// Use with generateText/streamText
const result = await generateText({
  model: anthropic('claude-sonnet-4-5-20241022'),
  tools: { weather: weatherTool },
  prompt: 'What is the weather in San Francisco?',
});
```

### Agent (ToolLoopAgent)

```typescript
import { ToolLoopAgent, stepCountIs, tool } from 'ai';

const agent = new ToolLoopAgent({
  model: anthropic('claude-sonnet-4-5-20241022'),
  tools: {
    search: tool({ /* ... */ }),
    calculate: tool({ /* ... */ }),
  },
  stopWhen: stepCountIs(10), // Max 10 steps
});

const result = await agent.generate({
  prompt: 'Research and calculate...',
});
```

### useChat Hook (React)

```typescript
import { useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from 'ai';

function Chat() {
  const { messages, sendMessage, status, stop } = useChat({
    transport: new DefaultChatTransport({ api: '/api/chat' }),
  });

  return (
    <>
      {messages.map(m => (
        <div key={m.id}>
          {m.role}: {m.parts.map(p => p.type === 'text' ? p.text : null)}
        </div>
      ))}
      <form onSubmit={e => {
        e.preventDefault();
        sendMessage({ text: input });
      }}>
        <input disabled={status !== 'ready'} />
      </form>
    </>
  );
}
```

### API Route (Next.js)

```typescript
import { streamText, convertToModelMessages, UIMessage } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';

export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json();

  const result = streamText({
    model: anthropic('claude-sonnet-4-5-20241022'),
    system: 'You are a helpful assistant.',
    messages: convertToModelMessages(messages),
  });

  return result.toUIMessageStreamResponse();
}
```

### Providers

```typescript
// Official providers
import { anthropic } from '@ai-sdk/anthropic';
import { openai } from '@ai-sdk/openai';
import { google } from '@ai-sdk/google';
import { mistral } from '@ai-sdk/mistral';

// Use models
const model = anthropic('claude-sonnet-4-5-20241022');
const model = openai('gpt-4o');
const model = google('gemini-1.5-flash');
```

### Prompt Types

```typescript
// Text prompt
await generateText({
  model,
  prompt: 'Hello!',
});

// System + prompt
await generateText({
  model,
  system: 'You are a helpful assistant.',
  prompt: 'Hello!',
});

// Message array
await generateText({
  model,
  messages: [
    { role: 'user', content: 'Hi!' },
    { role: 'assistant', content: 'Hello!' },
    { role: 'user', content: 'How are you?' },
  ],
});

// Multi-modal (images)
await generateText({
  model,
  messages: [{
    role: 'user',
    content: [
      { type: 'text', text: 'Describe this image' },
      { type: 'image', image: fs.readFileSync('./image.png') },
    ],
  }],
});
```

### Status Values (useChat)

- `submitted` - Message sent, awaiting response stream
- `streaming` - Response actively streaming
- `ready` - Complete, ready for new message
- `error` - Error occurred

### Stream Result Properties

```typescript
const result = streamText({ model, prompt });

// Async iterables
result.textStream      // Stream of text chunks
result.fullStream      // Full event stream with types

// Promises (resolve when complete)
result.text            // Full generated text
result.toolCalls       // Tool calls made
result.toolResults     // Tool execution results
result.usage           // Token usage
result.finishReason    // Why generation stopped

// Response helpers
result.toUIMessageStreamResponse() // For useChat
result.toTextStreamResponse()      // Plain text stream
```

## Source

Documentation downloaded from: https://github.com/vercel/ai/tree/main/content/docs
