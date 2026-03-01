---
name: anthropic-streaming-patterns
description: Use when integrating Claude API with streaming responses, implementing tool execution in streams, tracking API costs, or encountering streaming errors - provides Anthropic SDK 0.30.1+ patterns with mandatory cost monitoring
---

# Anthropic Claude API Streaming Patterns

## Overview

Claude API integration with streaming, tool execution, and cost tracking using Anthropic SDK.

**Core principle:** Stream (don't buffer). Track costs. Handle tools correctly.

**Announce at start:** "I'm using the anthropic-streaming-patterns skill for Claude API integration."

## When to Use

- Implementing Claude API service (Task 3.4)
- Implementing streaming responses
- Implementing tool execution within streams
- Tracking API costs
- Debugging streaming issues

## Quick Reference

| Pattern | SDK Method | Purpose |
|---------|-----------|---------|
| Initialize | messages.stream() | Start streaming |
| Text deltas | stream.on('text') | Receive text chunks |
| Tool start | stream.on('contentBlockStart') | Tool use begins |
| Tool input | stream.on('contentBlockDelta') | Accumulate params |
| Tool complete | stream.on('contentBlockStop') | Execute tool |
| Stream end | stream.on('message') | Calculate costs |
| Errors | stream.on('error') | Handle failures |

## Streaming Pattern (Complete)

```typescript
const stream = client.messages.stream({
  model: 'claude-sonnet-4-20250514',
  max_tokens: 8192,
  messages: messageHistory,
  tools: toolDefinitions,
});

let currentToolUse = null;
let accumulatedInput = '';

// Text deltas → forward to client
stream.on('text', (text) => {
  sendToClient({type: 'content_delta', delta: text});
});

// Tool use started
stream.on('contentBlockStart', (block) => {
  if (block.type === 'tool_use') {
    currentToolUse = {name: block.name, id: block.id};
    accumulatedInput = '';
    sendToClient({type: 'tool_execution', tool: block.name});
  }
});

// Tool input accumulation
stream.on('contentBlockDelta', (delta) => {
  if (delta.type === 'input_json_delta' && currentToolUse) {
    accumulatedInput = delta.partial_json;
  }
});

// Tool execution
stream.on('contentBlockStop', async () => {
  if (currentToolUse) {
    const input = JSON.parse(accumulatedInput);
    const result = await executeTool(currentToolUse.name, input);
    
    sendToClient({type: 'tool_result', result});
    currentToolUse = null;
  }
});

// Stream complete with usage
stream.on('message', (message) => {
  if (message.usage) {
    const inputCost = (message.usage.input_tokens / 1000) * 0.003;
    const outputCost = (message.usage.output_tokens / 1000) * 0.015;
    
    saveSessionCost(sessionId, {
      inputTokens: message.usage.input_tokens,
      outputTokens: message.usage.output_tokens,
      cost: inputCost + outputCost
    });
  }
});

stream.on('error', (error) => {
  logger.error('Streaming error:', error);
  sendToClient({type: 'error', error: error.message});
});

await stream.finalMessage(); // Wait for completion
```

## Cost Tracking (MANDATORY)

```typescript
const PRICING = {
  input: 0.003,  // $0.003 per 1k tokens
  output: 0.015, // $0.015 per 1k output tokens
};

// Calculate per message
const cost = {
  input: (inputTokens / 1000) * PRICING.input,
  output: (outputTokens / 1000) * PRICING.output,
  total: inputCost + outputCost
};

// Aggregate per session
sessionCosts.push(cost);
const sessionTotal = sessionCosts.reduce((sum, c) => sum + c.total, 0);
```

## Error Handling

```typescript
try {
  const stream = await client.messages.stream({...});
} catch (error) {
  if (error.status === 429) {
    // Rate limit - wait and retry
    await delay(60000);
    return retry();
  } else if (error.status === 401) {
    // Auth error
    throw new Error('Invalid API key');
  } else {
    logger.error(error);
    throw error;
  }
}
```

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| "Buffering is simpler" | WRONG. Streaming provides real-time UX. Required. |
| "Cost tracking is optional" | WRONG. Users need visibility. Prevents surprise bills. |
| "I can figure out SDK" | WRONG. Event handling is subtle. Use proven patterns. |
| "Error handling later" | WRONG. Streams fail. Handle from start. |

### ❌ WRONG: Buffering

```typescript
const response = await client.messages.create({...}); // Buffering
const fullText = response.content[0].text;
sendToClient(fullText);
```

### ✅ CORRECT: Streaming

```typescript
const stream = await client.messages.stream({...});
stream.on('text', (delta) => sendToClient({type: 'content_delta', delta}));
```

## Red Flags

- "Buffering is easier" → WRONG. Stream for real-time.
- "Cost tracking is overhead" → WRONG. Mandatory feature.
- "Skip error handling" → WRONG. Streams fail often.

## Integration

- **Use FOR**: Task 3.4 (claude.service.ts)
- **Use WITH**: `@claude-mobile-cost-tracking`
- **Integrate**: Task 3.11 (cost.service.ts)
