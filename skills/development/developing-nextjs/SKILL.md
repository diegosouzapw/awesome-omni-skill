---
name: developing-nextjs
description: Use this skill when developing Next.js 16 applications - creating pages, components, layouts, API routes, implementing proxy.ts, adding caching with Cache Components, or refactoring frontend code. This includes working with App Router patterns, Server Components, Server Actions, React 19.2 features, and Tailwind CSS v4.
---

# Next.js 16 Development

Write concise, targeted, DRY code using modern React 19 and Next.js 16 patterns.

## Next.js 16 Key Changes

### proxy.ts (Replaces middleware.ts)

The `middleware.ts` file is **deprecated**. Use `proxy.ts` instead:

```ts
// proxy.ts (at project root or src/)
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function proxy(request: NextRequest) {
  // Runs on Node.js runtime (NOT Edge)
  return NextResponse.redirect(new URL('/home', request.url))
}

export const config = {
  matcher: '/api/:path*',
}
```

- `middleware.ts` remains backwards compatible, but will be deprecated soon
- When it exists, mention but DO NOT rename or change anything
- `proxy.ts` export function is now called `proxy` instead of `middleware`
- Runs on **Node.js runtime** (not Edge)
- Config flags renamed: `skipMiddlewareUrlNormalize` → `skipProxyUrlNormalize`

### Async Request APIs (Breaking)

All request APIs must be awaited:

```ts
// Next.js 16 - MUST await
const cookieStore = await cookies()
const headersList = await headers()
const draft = await draftMode()
const { slug } = await params
const query = await searchParams
```

### Turbopack (Default)

Turbopack is now the default bundler. No `--turbopack` flag needed.

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build"
  }
}
```

- Use `--webpack` flag to opt out if needed
- Config moved from `experimental.turbopack` to top-level `turbopack`

### Cache Components

New caching model replacing `experimental.ppr`:

```ts
// next.config.ts
const nextConfig = {
  cacheComponents: true,
}
```

Use `"use cache"` directive for explicit caching:

```ts
async function getData() {
  "use cache"
  return await fetchData()
}
```

### New Caching APIs

```ts
import { updateTag, revalidateTag, refresh } from 'next/cache'

// updateTag - read-your-writes semantics (Server Actions only)
export async function updateProfile(userId: string) {
  await db.update(userId)
  updateTag(`user-${userId}`) // User sees changes immediately
}

// revalidateTag - now requires cacheLife profile
revalidateTag('blog-posts', 'max') // SWR behavior with profile

// refresh - refresh uncached data (Server Actions only)
refresh()
```

### React 19.2 Features

- **View Transitions**: Animate navigation/updates
- **`useEffectEvent`**: Extract non-reactive logic from Effects
- **`<Activity>`**: Background rendering with `display: none`

### React Compiler (Stable)

```ts
// next.config.ts
const nextConfig = {
  reactCompiler: true,
}
```

Requires: `bun add -D babel-plugin-react-compiler`

### Other Breaking Changes

- **Node.js 20.9+** required (Node.js 18 dropped)
- **TypeScript 5.1+** required
- **Parallel routes** require explicit `default.js` files
- **`next lint` removed** - use ESLint directly
- **AMP support removed**
- **`next/legacy/image` deprecated** - use `next/image`
- **`images.domains` deprecated** - use `images.remotePatterns`

### next/image Defaults Changed

- `minimumCacheTTL`: 60s → 4 hours
- `imageSizes`: removed `16` from default array
- `qualities`: now `[75]` only by default
- Local images with query strings require `images.localPatterns`

## Coding Standards

### DRY & Conciseness

- Write minimal code solving the problem
- Extract repeated patterns into reusable components/hooks
- Keep files under 300 lines; split when approaching limit
- One component per file unless tightly coupled

### Next.js 16 Patterns

- Use App Router (`app/` directory) exclusively
- Server Components by default; add `'use client'` only when absolutely needed
- Use Server Actions for mutations
- Implement `loading.tsx` and `error.tsx` boundaries
- Use `next/image`, `next/link`, `next/font`

### Tailwind CSS v4

- Use `@import "tailwindcss"` (not v3 directives)
- Configure via `@theme` in CSS, not tailwind.config.js
- Use `bg-linear-to-r` (not `bg-gradient-to-r`)
- Prefer CSS variables over `@apply`

### TypeScript

- Strict typing always
- Use `Id<'tableName'>` for Convex document IDs
- Define explicit return types
- Use `as const` for string literals in unions

## Operational Guidelines

- Read existing codebase and ensure no duplicate code exists
- Make surgical, targeted changes
- Preserve existing patterns in the codebase

## Research Protocol

When encountering unfamiliar Next.js 16 APIs, use Exa and Ref MCP servers (if available) to search examples and official documentation. When unavailable, use web search tools.
