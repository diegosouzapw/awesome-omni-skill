---
name: ai-integration
description: AI/LLM integration patterns - Claude API, fal.ai, streaming, tool use
triggers:
  - ai integration
  - claude api
  - anthropic
  - fal.ai
  - llm
  - streaming ai
  - tool use
  - yapay zeka
  - AI entegrasyon
---

# AI Integration Skill

Patterns for integrating AI services into Next.js applications: Anthropic Claude API, fal.ai, streaming, tool use, and cost optimization.

## 1. Anthropic Claude API - Messages

**File:** `src/lib/ai/anthropic.ts`

```typescript
import Anthropic from '@anthropic-ai/sdk'

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
})

export async function generateCompletion({
  systemPrompt,
  userMessage,
  maxTokens = 1024,
  model = 'claude-sonnet-4-20250514',
}: {
  systemPrompt: string
  userMessage: string
  maxTokens?: number
  model?: string
}) {
  const response = await anthropic.messages.create({
    model,
    max_tokens: maxTokens,
    system: systemPrompt,
    messages: [
      { role: 'user', content: userMessage },
    ],
  })

  const textBlock = response.content.find((block) => block.type === 'text')
  if (!textBlock || textBlock.type !== 'text') {
    throw new Error('No text response from Claude')
  }

  return {
    text: textBlock.text,
    usage: {
      inputTokens: response.usage.input_tokens,
      outputTokens: response.usage.output_tokens,
    },
    stopReason: response.stop_reason,
  }
}
```

---

## 2. Streaming Responses (Next.js API Route)

**File:** `src/app/api/ai/chat/route.ts`

```typescript
import Anthropic from '@anthropic-ai/sdk'
import { NextRequest } from 'next/server'
import { auth } from '@/lib/auth'
import { z } from 'zod'

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
})

const ChatSchema = z.object({
  messages: z.array(z.object({
    role: z.enum(['user', 'assistant']),
    content: z.string().min(1).max(100000),
  })),
  systemPrompt: z.string().max(10000).optional(),
})

export async function POST(request: NextRequest) {
  const { userId } = await auth()
  if (!userId) {
    return new Response('Unauthorized', { status: 401 })
  }

  const body = await request.json()
  const result = ChatSchema.safeParse(body)
  if (!result.success) {
    return Response.json(
      { error: 'Validation failed', details: result.error.flatten() },
      { status: 400 }
    )
  }

  const { messages, systemPrompt } = result.data

  const stream = await anthropic.messages.stream({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 4096,
    system: systemPrompt ?? 'You are a helpful assistant.',
    messages,
  })

  const encoder = new TextEncoder()

  const readable = new ReadableStream({
    async start(controller) {
      try {
        for await (const event of stream) {
          if (
            event.type === 'content_block_delta' &&
            event.delta.type === 'text_delta'
          ) {
            const chunk = `data: ${JSON.stringify({ text: event.delta.text })}\n\n`
            controller.enqueue(encoder.encode(chunk))
          }
        }

        const finalMessage = await stream.finalMessage()
        const done = `data: ${JSON.stringify({
          done: true,
          usage: {
            inputTokens: finalMessage.usage.input_tokens,
            outputTokens: finalMessage.usage.output_tokens,
          },
        })}\n\n`
        controller.enqueue(encoder.encode(done))
        controller.close()
      } catch (error) {
        console.error('[AI] Stream error:', error)
        const errorChunk = `data: ${JSON.stringify({ error: 'Stream failed' })}\n\n`
        controller.enqueue(encoder.encode(errorChunk))
        controller.close()
      }
    },
  })

  return new Response(readable, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      Connection: 'keep-alive',
    },
  })
}
```

### Client-Side Stream Consumer

```typescript
'use client'

export function useAIStream() {
  const [isStreaming, setIsStreaming] = useState(false)
  const [streamedText, setStreamedText] = useState('')
  const abortRef = useRef<AbortController | null>(null)

  const startStream = useCallback(async (
    messages: Array<{ role: 'user' | 'assistant'; content: string }>,
    systemPrompt?: string
  ) => {
    setIsStreaming(true)
    setStreamedText('')

    abortRef.current = new AbortController()

    try {
      const response = await fetch('/api/ai/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ messages, systemPrompt }),
        signal: abortRef.current.signal,
      })

      if (!response.ok) throw new Error(`HTTP ${response.status}`)
      if (!response.body) throw new Error('No response body')

      const reader = response.body.getReader()
      const decoder = new TextDecoder()
      let buffer = ''

      while (true) {
        const { done, value } = await reader.read()
        if (done) break

        buffer += decoder.decode(value, { stream: true })
        const lines = buffer.split('\n\n')
        buffer = lines.pop() ?? ''

        for (const line of lines) {
          if (!line.startsWith('data: ')) continue
          const data = JSON.parse(line.slice(6))

          if (data.text) {
            setStreamedText((prev) => prev + data.text)
          }
          if (data.done) {
            setIsStreaming(false)
          }
          if (data.error) {
            throw new Error(data.error)
          }
        }
      }
    } catch (error) {
      if ((error as Error).name !== 'AbortError') {
        console.error('[AI] Stream consumer error:', error)
      }
    } finally {
      setIsStreaming(false)
    }
  }, [])

  const stopStream = useCallback(() => {
    abortRef.current?.abort()
    setIsStreaming(false)
  }, [])

  return { streamedText, isStreaming, startStream, stopStream }
}
```

---

## 3. Tool Use (Structured Output with Zod)

**File:** `src/lib/ai/structured.ts`

```typescript
import Anthropic from '@anthropic-ai/sdk'
import { z } from 'zod'

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
})

const ExtractedProductSchema = z.object({
  name: z.string(),
  price: z.number(),
  currency: z.string(),
  features: z.array(z.string()),
  category: z.enum(['electronics', 'clothing', 'food', 'other']),
})

type ExtractedProduct = z.infer<typeof ExtractedProductSchema>

export async function extractProductInfo(
  rawText: string
): Promise<ExtractedProduct> {
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1024,
    tools: [
      {
        name: 'extract_product',
        description: 'Extract structured product information from raw text.',
        input_schema: {
          type: 'object' as const,
          properties: {
            name: { type: 'string', description: 'Product name' },
            price: { type: 'number', description: 'Price as a number' },
            currency: { type: 'string', description: 'ISO 4217 currency code' },
            features: {
              type: 'array',
              items: { type: 'string' },
              description: 'Key product features',
            },
            category: {
              type: 'string',
              enum: ['electronics', 'clothing', 'food', 'other'],
              description: 'Product category',
            },
          },
          required: ['name', 'price', 'currency', 'features', 'category'],
        },
      },
    ],
    tool_choice: { type: 'tool', name: 'extract_product' },
    messages: [
      {
        role: 'user',
        content: `Extract product information from this text:\n\n${rawText}`,
      },
    ],
  })

  const toolUseBlock = response.content.find(
    (block) => block.type === 'tool_use'
  )
  if (!toolUseBlock || toolUseBlock.type !== 'tool_use') {
    throw new Error('No tool use response from Claude')
  }

  return ExtractedProductSchema.parse(toolUseBlock.input)
}
```

### Multi-Turn Tool Use (Agentic)

```typescript
export async function runAgentLoop({
  systemPrompt,
  userMessage,
  tools,
  toolHandlers,
  maxIterations = 10,
}: {
  systemPrompt: string
  userMessage: string
  tools: Anthropic.Tool[]
  toolHandlers: Record<string, (input: unknown) => Promise<string>>
  maxIterations?: number
}) {
  const messages: Anthropic.MessageParam[] = [
    { role: 'user', content: userMessage },
  ]

  for (let i = 0; i < maxIterations; i++) {
    const response = await anthropic.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 4096,
      system: systemPrompt,
      tools,
      messages,
    })

    // If the model stopped without tool use, return the final text
    if (response.stop_reason === 'end_turn') {
      const textBlock = response.content.find((b) => b.type === 'text')
      return textBlock?.type === 'text' ? textBlock.text : ''
    }

    // Process tool calls
    messages.push({ role: 'assistant', content: response.content })

    const toolResults: Anthropic.ToolResultBlockParam[] = []
    for (const block of response.content) {
      if (block.type !== 'tool_use') continue

      const handler = toolHandlers[block.name]
      if (!handler) {
        toolResults.push({
          type: 'tool_result',
          tool_use_id: block.id,
          content: `Error: Unknown tool "${block.name}"`,
          is_error: true,
        })
        continue
      }

      try {
        const result = await handler(block.input)
        toolResults.push({
          type: 'tool_result',
          tool_use_id: block.id,
          content: result,
        })
      } catch (error) {
        toolResults.push({
          type: 'tool_result',
          tool_use_id: block.id,
          content: `Error: ${(error as Error).message}`,
          is_error: true,
        })
      }
    }

    messages.push({ role: 'user', content: toolResults })
  }

  throw new Error('Agent loop exceeded max iterations')
}
```

---

## 4. Prompt Caching

```typescript
export async function cachedCompletion({
  systemPrompt,
  userMessage,
}: {
  systemPrompt: string
  userMessage: string
}) {
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 4096,
    system: [
      {
        type: 'text',
        text: systemPrompt,
        cache_control: { type: 'ephemeral' },
      },
    ],
    messages: [{ role: 'user', content: userMessage }],
  })

  return {
    text: response.content.find((b) => b.type === 'text')?.type === 'text'
      ? (response.content.find((b) => b.type === 'text') as Anthropic.TextBlock).text
      : '',
    usage: {
      inputTokens: response.usage.input_tokens,
      outputTokens: response.usage.output_tokens,
      cacheCreationTokens: response.usage.cache_creation_input_tokens ?? 0,
      cacheReadTokens: response.usage.cache_read_input_tokens ?? 0,
    },
  }
}
```

### Caching Large Context (Documents, Knowledge Base)

```typescript
// Cache a large document as part of the conversation
const response = await anthropic.messages.create({
  model: 'claude-sonnet-4-20250514',
  max_tokens: 4096,
  system: [
    {
      type: 'text',
      text: 'You are an expert analyst.',
    },
  ],
  messages: [
    {
      role: 'user',
      content: [
        {
          type: 'text',
          text: largeDocumentText, // 50k+ tokens
          cache_control: { type: 'ephemeral' },
        },
        {
          type: 'text',
          text: 'Summarize the key findings from this document.',
        },
      ],
    },
  ],
})
```

---

## 5. fal.ai Integration

**File:** `src/lib/ai/fal.ts`

```typescript
import { fal } from '@fal-ai/client'

fal.config({
  credentials: process.env.FAL_KEY,
})

// Image Generation
export async function generateImage({
  prompt,
  negativePrompt,
  width = 1024,
  height = 1024,
  model = 'fal-ai/flux/dev',
}: {
  prompt: string
  negativePrompt?: string
  width?: number
  height?: number
  model?: string
}) {
  const result = await fal.subscribe(model, {
    input: {
      prompt,
      negative_prompt: negativePrompt,
      image_size: { width, height },
      num_images: 1,
      enable_safety_checker: true,
    },
    logs: true,
    onQueueUpdate: (update) => {
      if (update.status === 'IN_PROGRESS' && update.logs) {
        for (const log of update.logs) {
          console.log(`[fal] ${log.message}`)
        }
      }
    },
  })

  return {
    imageUrl: result.data.images[0].url,
    seed: result.data.seed,
    requestId: result.requestId,
  }
}

// Video Generation
export async function generateVideo({
  imageUrl,
  prompt,
  duration = 5,
  model = 'fal-ai/kling-video/v1.5/pro/image-to-video',
}: {
  imageUrl: string
  prompt: string
  duration?: number
  model?: string
}) {
  const result = await fal.subscribe(model, {
    input: {
      image_url: imageUrl,
      prompt,
      duration: `${duration}` as '5' | '10',
    },
    logs: true,
    onQueueUpdate: (update) => {
      if (update.status === 'IN_PROGRESS' && update.logs) {
        for (const log of update.logs) {
          console.log(`[fal] ${log.message}`)
        }
      }
    },
  })

  return {
    videoUrl: result.data.video.url,
    requestId: result.requestId,
  }
}
```

### fal.ai Webhook Pattern (Async Processing)

```typescript
// Submit job with webhook callback
export async function submitImageGeneration({
  prompt,
  callbackUrl,
  metadata,
}: {
  prompt: string
  callbackUrl: string
  metadata: Record<string, string>
}) {
  const { request_id } = await fal.queue.submit('fal-ai/flux/dev', {
    input: {
      prompt,
      image_size: { width: 1024, height: 1024 },
      num_images: 1,
    },
    webhookUrl: callbackUrl,
  })

  return { requestId: request_id }
}

// Webhook handler
// app/api/webhooks/fal/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { db } from '@/lib/db'
import { generations } from '@/lib/db/schema'
import { eq } from 'drizzle-orm'

export async function POST(request: NextRequest) {
  const body = await request.json()

  const { request_id, status, payload } = body

  if (status === 'OK') {
    await db
      .update(generations)
      .set({
        status: 'completed',
        resultUrl: payload.images[0].url,
        completedAt: new Date(),
      })
      .where(eq(generations.requestId, request_id))
  } else {
    await db
      .update(generations)
      .set({
        status: 'failed',
        error: payload?.detail ?? 'Unknown error',
      })
      .where(eq(generations.requestId, request_id))
  }

  return NextResponse.json({ received: true })
}
```

---

## 6. Rate Limiting and Retry

**File:** `src/lib/ai/retry.ts`

```typescript
interface RetryOptions {
  maxRetries: number
  baseDelay: number
  maxDelay: number
}

const DEFAULT_RETRY: RetryOptions = {
  maxRetries: 3,
  baseDelay: 1000,
  maxDelay: 30000,
}

export async function withRetry<T>(
  fn: () => Promise<T>,
  options: Partial<RetryOptions> = {}
): Promise<T> {
  const { maxRetries, baseDelay, maxDelay } = { ...DEFAULT_RETRY, ...options }

  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn()
    } catch (error) {
      const isRetryable =
        error instanceof Error &&
        ('status' in error
          ? [429, 500, 502, 503, 529].includes(
              (error as Error & { status: number }).status
            )
          : error.message.includes('rate limit') ||
            error.message.includes('overloaded'))

      if (!isRetryable || attempt === maxRetries) {
        throw error
      }

      // Extract retry-after header if available
      const retryAfter =
        'headers' in error
          ? Number(
              (error as Error & { headers: Record<string, string> }).headers?.[
                'retry-after'
              ]
            ) * 1000
          : 0

      const delay = retryAfter || Math.min(baseDelay * 2 ** attempt, maxDelay)
      const jitter = delay * (0.5 + Math.random() * 0.5)

      console.warn(
        `[AI] Retry ${attempt + 1}/${maxRetries} after ${Math.round(jitter)}ms`
      )
      await new Promise((resolve) => setTimeout(resolve, jitter))
    }
  }

  throw new Error('Unreachable')
}

// Usage
const result = await withRetry(() =>
  generateCompletion({
    systemPrompt: 'You are helpful.',
    userMessage: 'Hello',
  })
)
```

### Per-User Rate Limiting

```typescript
// lib/ai/rate-limit.ts
import { Redis } from '@upstash/redis'

const redis = Redis.fromEnv()

export async function checkAIRateLimit(
  userId: string,
  {
    maxRequests = 50,
    windowSeconds = 3600,
  }: { maxRequests?: number; windowSeconds?: number } = {}
): Promise<{ allowed: boolean; remaining: number; resetAt: Date }> {
  const key = `ai:rate:${userId}`
  const now = Math.floor(Date.now() / 1000)
  const windowStart = now - windowSeconds

  // Remove old entries and count current window
  await redis.zremrangebyscore(key, 0, windowStart)
  const count = await redis.zcard(key)

  if (count >= maxRequests) {
    const oldest = await redis.zrange(key, 0, 0, { withScores: true })
    const resetAt = new Date(
      ((oldest[0]?.score ?? now) + windowSeconds) * 1000
    )
    return { allowed: false, remaining: 0, resetAt }
  }

  await redis.zadd(key, { score: now, member: `${now}:${crypto.randomUUID()}` })
  await redis.expire(key, windowSeconds)

  return {
    allowed: true,
    remaining: maxRequests - count - 1,
    resetAt: new Date((now + windowSeconds) * 1000),
  }
}
```

---

## 7. Token Counting and Context Management

```typescript
// lib/ai/tokens.ts
import Anthropic from '@anthropic-ai/sdk'

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
})

export async function countTokens(
  messages: Anthropic.MessageParam[],
  systemPrompt?: string
): Promise<number> {
  const result = await anthropic.messages.countTokens({
    model: 'claude-sonnet-4-20250514',
    system: systemPrompt,
    messages,
  })
  return result.input_tokens
}

// Context window management
const MODEL_LIMITS: Record<string, number> = {
  'claude-sonnet-4-20250514': 200000,
  'claude-opus-4-20250514': 200000,
  'claude-haiku-3-20250307': 200000,
}

export async function trimConversation({
  messages,
  systemPrompt,
  model = 'claude-sonnet-4-20250514',
  maxOutputTokens = 4096,
  reserveRatio = 0.8,
}: {
  messages: Anthropic.MessageParam[]
  systemPrompt?: string
  model?: string
  maxOutputTokens?: number
  reserveRatio?: number
}): Promise<Anthropic.MessageParam[]> {
  const limit = MODEL_LIMITS[model] ?? 200000
  const maxInput = Math.floor(limit * reserveRatio) - maxOutputTokens

  let tokenCount = await countTokens(messages, systemPrompt)

  // Trim oldest messages (keep system + first user message + recent history)
  const trimmed = [...messages]
  while (tokenCount > maxInput && trimmed.length > 2) {
    trimmed.splice(1, 2) // Remove oldest user+assistant pair
    tokenCount = await countTokens(trimmed, systemPrompt)
  }

  return trimmed
}
```

---

## 8. OpenAI SDK Pattern (Reference)

```typescript
// lib/ai/openai.ts
import OpenAI from 'openai'

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
})

export async function generateEmbedding(text: string): Promise<number[]> {
  const response = await openai.embeddings.create({
    model: 'text-embedding-3-small',
    input: text,
  })
  return response.data[0].embedding
}

export async function generateEmbeddings(
  texts: string[]
): Promise<number[][]> {
  const response = await openai.embeddings.create({
    model: 'text-embedding-3-small',
    input: texts,
  })
  return response.data.map((d) => d.embedding)
}
```

---

## 9. Error Handling

```typescript
// lib/ai/errors.ts
import Anthropic from '@anthropic-ai/sdk'

export function handleAIError(error: unknown): {
  message: string
  retryable: boolean
  statusCode: number
} {
  if (error instanceof Anthropic.APIError) {
    switch (error.status) {
      case 400:
        return {
          message: 'Invalid request to AI service',
          retryable: false,
          statusCode: 400,
        }
      case 401:
        return {
          message: 'AI service authentication failed',
          retryable: false,
          statusCode: 500,
        }
      case 429:
        return {
          message: 'AI service rate limited. Please try again shortly.',
          retryable: true,
          statusCode: 429,
        }
      case 529:
        return {
          message: 'AI service is temporarily overloaded. Please try again.',
          retryable: true,
          statusCode: 503,
        }
      default:
        return {
          message: 'AI service error',
          retryable: error.status >= 500,
          statusCode: 502,
        }
    }
  }

  return {
    message: 'Unexpected AI error',
    retryable: false,
    statusCode: 500,
  }
}

// Usage in API route
export async function POST(request: NextRequest) {
  try {
    const result = await generateCompletion({ ... })
    return Response.json(result)
  } catch (error) {
    const { message, statusCode } = handleAIError(error)
    console.error('[AI] Error:', error)
    return Response.json({ error: message }, { status: statusCode })
  }
}
```

---

## Cost Optimization

| Strategy | Savings | When to Use |
|----------|---------|-------------|
| Prompt caching | 90% on cached tokens | Repeated system prompts, large documents |
| Haiku for simple tasks | 80-95% vs Opus | Classification, extraction, simple Q&A |
| Sonnet for most tasks | 50-80% vs Opus | Code generation, analysis, tool use |
| Batch API | 50% | Non-real-time processing, bulk operations |
| Shorter prompts | Linear | Always optimize prompt length |
| Token counting | Prevents waste | Before sending large contexts |

### Model Selection Helper

```typescript
type TaskComplexity = 'simple' | 'moderate' | 'complex'

function selectModel(complexity: TaskComplexity): string {
  switch (complexity) {
    case 'simple':
      return 'claude-haiku-3-20250307'
    case 'moderate':
      return 'claude-sonnet-4-20250514'
    case 'complex':
      return 'claude-opus-4-20250514'
  }
}
```

---

## Checklist

```
[ ] API key stored in env var (never NEXT_PUBLIC_)
[ ] Auth check before every AI endpoint
[ ] Input validation with Zod
[ ] Rate limiting per user
[ ] Retry with exponential backoff for 429/5xx
[ ] Streaming for long responses
[ ] Token counting before large context sends
[ ] Proper error handling with handleAIError
[ ] Cost tracking via usage response fields
[ ] Prompt caching for repeated system prompts
[ ] Generic errors to client, detailed to server logs
[ ] AbortController support for client-side streaming
[ ] tsc --noEmit = 0 errors
```

---

## Dependencies

```bash
npm install @anthropic-ai/sdk @fal-ai/client
# Optional
npm install openai @upstash/redis
```

## Red Flags (STOP)

| If You See | Fix |
|------------|-----|
| `NEXT_PUBLIC_ANTHROPIC_API_KEY` | Move to server-only env var |
| No auth check on AI endpoint | Add auth() check |
| No rate limiting | Add per-user rate limits |
| Catching errors silently | Use handleAIError, log to server |
| Hardcoded model strings everywhere | Use selectModel() or constants |
| No streaming for chat UI | Use ReadableStream + SSE |
| Sending full conversation without trim | Use trimConversation() |
| fal.ai polling in a loop | Use fal.subscribe or webhooks |

---

## 10. OpenAI GPT-4o Integration

```typescript
// lib/ai/openai.ts
import OpenAI from 'openai'

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
})

export async function chatCompletion({
  systemPrompt,
  userMessage,
  model = 'gpt-4o',
  maxTokens = 4096,
}: {
  systemPrompt: string
  userMessage: string
  model?: string
  maxTokens?: number
}) {
  const response = await openai.chat.completions.create({
    model,
    max_tokens: maxTokens,
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userMessage },
    ],
  })

  return {
    text: response.choices[0].message.content ?? '',
    usage: {
      promptTokens: response.usage?.prompt_tokens ?? 0,
      completionTokens: response.usage?.completion_tokens ?? 0,
    },
    finishReason: response.choices[0].finish_reason,
  }
}
```

### Function Calling

```typescript
export async function functionCall<T>({
  systemPrompt,
  userMessage,
  functionName,
  functionDescription,
  parameters,
}: {
  systemPrompt: string
  userMessage: string
  functionName: string
  functionDescription: string
  parameters: Record<string, unknown>
}) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userMessage },
    ],
    tools: [
      {
        type: 'function',
        function: {
          name: functionName,
          description: functionDescription,
          parameters,
        },
      },
    ],
    tool_choice: { type: 'function', function: { name: functionName } },
  })

  const toolCall = response.choices[0].message.tool_calls?.[0]
  if (!toolCall) throw new Error('No function call response')

  return JSON.parse(toolCall.function.arguments) as T
}
```

---

## 11. Embeddings + pgvector RAG Pipeline

### Drizzle Schema with pgvector

```typescript
// lib/db/schema.ts
import { pgTable, text, vector, bigint, timestamp } from 'drizzle-orm/pg-core'

export const documents = pgTable('documents', {
  id: bigint('id', { mode: 'number' }).primaryKey().generatedAlwaysAsIdentity(),
  content: text('content').notNull(),
  embedding: vector('embedding', { dimensions: 1536 }),
  metadata: text('metadata'),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
})
```

### Enable pgvector Extension

```sql
-- Migration: enable pgvector
CREATE EXTENSION IF NOT EXISTS vector;
```

### Generate Embeddings

```typescript
// lib/ai/embeddings.ts
import OpenAI from 'openai'

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY })

export async function generateEmbedding(text: string): Promise<number[]> {
  const response = await openai.embeddings.create({
    model: 'text-embedding-3-small',
    input: text,
  })
  return response.data[0].embedding
}

export async function generateEmbeddings(texts: string[]): Promise<number[][]> {
  // Batch in chunks of 100
  const results: number[][] = []
  for (let i = 0; i < texts.length; i += 100) {
    const batch = texts.slice(i, i + 100)
    const response = await openai.embeddings.create({
      model: 'text-embedding-3-small',
      input: batch,
    })
    results.push(...response.data.map((d) => d.embedding))
  }
  return results
}
```

### Similarity Search

```typescript
// lib/ai/search.ts
import { db } from '@/lib/db'
import { documents } from '@/lib/db/schema'
import { cosineDistance, desc, gt, sql } from 'drizzle-orm'
import { generateEmbedding } from './embeddings'

export async function similaritySearch({
  query,
  limit = 5,
  minSimilarity = 0.7,
}: {
  query: string
  limit?: number
  minSimilarity?: number
}) {
  const queryEmbedding = await generateEmbedding(query)

  const similarity = sql<number>`1 - (${cosineDistance(documents.embedding, queryEmbedding)})`

  const results = await db
    .select({
      id: documents.id,
      content: documents.content,
      metadata: documents.metadata,
      similarity,
    })
    .from(documents)
    .where(gt(similarity, minSimilarity))
    .orderBy(desc(similarity))
    .limit(limit)

  return results
}
```

### Full RAG Pipeline

```typescript
// lib/ai/rag.ts
import { similaritySearch } from './search'
import { chatCompletion } from './openai'

export async function ragQuery({
  question,
  systemPrompt = 'You are a helpful assistant. Answer based on the provided context.',
  maxContextDocs = 5,
}: {
  question: string
  systemPrompt?: string
  maxContextDocs?: number
}) {
  // 1. Search for relevant documents
  const relevantDocs = await similaritySearch({
    query: question,
    limit: maxContextDocs,
  })

  // 2. Build augmented prompt
  const context = relevantDocs
    .map((doc, i) => `[${i + 1}] ${doc.content}`)
    .join('\n\n')

  const augmentedMessage = `Context:\n${context}\n\nQuestion: ${question}`

  // 3. Generate response
  const response = await chatCompletion({
    systemPrompt,
    userMessage: augmentedMessage,
  })

  return {
    answer: response.text,
    sources: relevantDocs,
    usage: response.usage,
  }
}
```

### Ingest Documents

```typescript
// lib/ai/ingest.ts
import { db } from '@/lib/db'
import { documents } from '@/lib/db/schema'
import { generateEmbeddings } from './embeddings'

export async function ingestDocuments(
  docs: Array<{ content: string; metadata?: string }>
) {
  const contents = docs.map((d) => d.content)
  const embeddings = await generateEmbeddings(contents)

  const rows = docs.map((doc, i) => ({
    content: doc.content,
    embedding: embeddings[i],
    metadata: doc.metadata ?? null,
  }))

  await db.insert(documents).values(rows)

  return { ingested: rows.length }
}
```

### Dependencies

```bash
npm install openai
# pgvector extension must be enabled in PostgreSQL
```
