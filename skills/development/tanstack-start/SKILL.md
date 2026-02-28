---
name: tanstack-start
description: TanStack Start full-stack React framework patterns. Use when working with file-based routing, API routes, server handlers, TanStack Router layouts, or integrating AI/MCP features in this codebase.
---

# TanStack Start

## File-Based Routing

Routes live in `src/routes/`. The route tree auto-generates to `src/routeTree.gen.ts` - commit this file (required for type-checking).

### Route Types

- `__root.tsx` - Root layout, wraps all routes
- `index.tsx` - Index route for a path segment
- `demo.tsx` - Layout route wrapping all `demo/*.tsx` children (uses `<Outlet />`)
- `demo/*.tsx` - Child routes rendered inside parent layout
- `$param.tsx` - Dynamic route segment

### Layout Routes

Create `foo.tsx` alongside `foo/` directory to wrap child routes:

```tsx
import { createFileRoute, Outlet } from '@tanstack/react-router'

export const Route = createFileRoute('/foo')({
  component: FooLayout,
})

function FooLayout() {
  return (
    <div>
      <Outlet />
    </div>
  )
}
```

## API Routes

Server handlers use `api.*.ts` naming convention:

```tsx
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/demo/api/example')({
  server: {
    handlers: {
      POST: async ({ request }) => {
        const data = await request.json()
        return Response.json({ result: data })
      },
    },
  },
})
```

## AI Chat Integration

### Client Side

Use `@ai-sdk/react` with `DefaultChatTransport`:

```tsx
import { useChat } from '@ai-sdk/react'
import { DefaultChatTransport } from 'ai'

const { messages, sendMessage } = useChat({
  transport: new DefaultChatTransport({
    api: '/demo/api/chat',
  }),
})
```

### Server Side

Use `streamText` from `ai` package with OpenRouter provider:

```tsx
import { convertToModelMessages, streamText } from 'ai'
import { haiku } from '@/lib/openrouter'

const result = await streamText({
  model: haiku,
  messages: convertToModelMessages(messages),
  system: SYSTEM_PROMPT,
  tools,
})

return result.toUIMessageStreamResponse()
```

### OpenRouter Setup

Create provider singleton in `src/lib/openrouter.ts`:

```tsx
import { createOpenRouter } from '@openrouter/ai-sdk-provider'

export const openrouter = createOpenRouter({
  apiKey: process.env.OPENROUTER_API_KEY,
})

export const haiku = openrouter('anthropic/claude-3.5-haiku')
```

## MCP Server

Register tools with Zod schemas in route files:

```tsx
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js'
import z from 'zod'
import { handleMcpRequest } from '@/utils/mcp-handler'

const server = new McpServer({ name: 'my-server', version: '1.0.0' })

server.registerTool(
  'toolName',
  {
    title: 'Tool Title',
    description: 'Tool description',
    inputSchema: { param: z.string().describe('Param description') },
  },
  ({ param }) => ({
    content: [{ type: 'text', text: result }],
  }),
)

export const Route = createFileRoute('/mcp')({
  server: {
    handlers: {
      POST: async ({ request }) => handleMcpRequest(request, server),
    },
  },
})
```

## Path Aliases

`@/*` maps to `./src/*` (configured in tsconfig.json).
