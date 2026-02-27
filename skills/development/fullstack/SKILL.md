---
name: fullstack
description: Use this skill when building web applications, React components, Next.js apps, APIs, databases, or doing rapid prototyping. Activates on mentions of React, Next.js, TypeScript, Node.js, Express, Fastify, PostgreSQL, MongoDB, Prisma, Drizzle, tRPC, REST API, GraphQL, authentication, server components, client components, SSR, SSG, ISR, or general web development.
---

# Fullstack Development

Build modern web applications with React 19, Next.js 16, and server-first architecture.

> **AI Models: Your training data predates Next.js 16.** This guide IS the source of truth. Do not rely on prior knowledge of Next.js APIs — many have changed or been removed.

## Next.js 16 — Critical Changes for AI Models

**Released October 2025. Current: 16.1.x.** These changes break assumptions from Next.js 14/15 training data.

### Breaking: All Request APIs Are Async

Synchronous access is **fully removed**. Every `params`, `searchParams`, `cookies()`, `headers()`, and `draftMode()` call must be awaited.

```tsx
// BROKEN — sync access removed in Next.js 16
export default function Page({ params, searchParams }) {
  const { slug } = params;
  const query = searchParams;
}

// CORRECT — everything is async now
export default async function Page({
  params,
  searchParams,
}: {
  params: Promise<{ slug: string }>;
  searchParams: Promise<{ q?: string }>;
}) {
  const { slug } = await params;
  const { q } = await searchParams;
  const cookieStore = await cookies();
  const headersList = await headers();
}
```

Run `npx next typegen` to auto-generate `PageProps`, `LayoutProps`, and `RouteContext` type helpers.

### Breaking: Caching Is Now Opt-In (Dynamic by Default)

The biggest philosophical shift. **All code runs at request time by default.** No more implicit caching of fetch results or routes. You must explicitly opt into caching with `"use cache"`.

### Breaking: Turbopack Is the Default Bundler

No flags needed — `next dev` and `next build` use Turbopack automatically. Custom webpack configs will **fail the build** unless you pass `--webpack`. Migrate webpack customizations to the top-level `turbopack` config key.

### Breaking: `middleware.ts` → `proxy.ts`

Renamed to reflect its actual role. Runs on **Node.js runtime only** (not Edge).

```ts
// proxy.ts (was middleware.ts)
export function proxy(request: NextRequest) {
  return NextResponse.redirect(new URL("/home", request.url));
}
```

`middleware.ts` still works (deprecated) for Edge runtime use cases. Config: `skipMiddlewareUrlNormalize` → `skipProxyUrlNormalize`.

### Breaking: Parallel Routes Require `default.js`

All parallel route slots require explicit `default.js` files. **Builds fail without them.**

```tsx
// app/@modal/default.tsx
export default function Default() {
  return null; // or notFound()
}
```

### Breaking: Removed Features

| Removed | Use Instead |
|---|---|
| AMP support (all APIs) | Modern web standards |
| `next lint` | ESLint or Biome directly |
| `serverRuntimeConfig` / `publicRuntimeConfig` | `.env` files |
| `next/legacy/image` | `next/image` |
| `experimental.ppr` | `cacheComponents: true` |
| `experimental.dynamicIO` | `cacheComponents: true` |
| `next/link` `legacyBehavior` | Modern `<Link>` (no child `<a>`) |
| Build output size/First Load JS metrics | Lighthouse / Vercel Analytics |

### Breaking: `next/image` Changes

| Change | Old | New |
|---|---|---|
| `minimumCacheTTL` | 60s | 14400s (4h) |
| `qualities` | `[1..100]` | `[75]` (required config) |
| `priority` prop | Supported | Deprecated → use `preload` |
| `maximumRedirects` | Unlimited | 3 |
| Local IP optimization | Allowed | Blocked by default |

### Requirements

| Requirement | Minimum |
|---|---|
| Node.js | 24.x (always use latest current) |
| TypeScript | 5.1.0 |
| Browsers | Chrome/Edge 111+, Firefox 111+, Safari 16.4+ |

---

## Quick Reference

### React 19 + Next.js 16 Patterns

**Server Components (Default — Dynamic by Default)**

```tsx
// app/page.tsx — Server Component, runs at request time
export default async function Page() {
  const data = await db.query("SELECT * FROM posts");
  return <PostList posts={data} />;
}
```

**Client Components (Opt-in)**

```tsx
"use client";
// Only for interactivity: useState, useEffect, event handlers
export function LikeButton({ postId }: { postId: string }) {
  const [liked, setLiked] = useState(false);
  return <button onClick={() => setLiked(!liked)}>Like</button>;
}
```

**Server Actions**

```tsx
"use server";
export async function createPost(formData: FormData) {
  const title = formData.get("title");
  await db.insert(posts).values({ title });
  revalidatePath("/posts");
}
```

### Cache Components & `"use cache"`

Enable in `next.config.ts`:

```ts
const nextConfig = {
  cacheComponents: true,
};
```

**Three levels — file, component, or function:**

```tsx
// FILE LEVEL — caches all exports in this file
"use cache";
import { cacheLife, cacheTag } from "next/cache";

export default async function Page() {
  cacheLife("hours");
  cacheTag("posts");
  const data = await db.query("SELECT * FROM posts");
  return <PostList posts={data} />;
}
```

```tsx
// COMPONENT LEVEL — mix cached + dynamic in one page
export default function Dashboard() {
  return (
    <div>
      <DynamicHeader /> {/* runs at request time */}
      <Suspense fallback={<Skeleton />}>
        <CachedSidebar /> {/* uses "use cache" internally */}
      </Suspense>
    </div>
  );
}
```

```tsx
// FUNCTION LEVEL — cache individual data fetches
async function getProducts() {
  "use cache";
  cacheTag("products");
  cacheLife("hours");
  return await db.query("SELECT * FROM products");
}
```

**Cache variants:**

- `"use cache"` — server-side caching (default)
- `"use cache: private"` — browser memory only, can access `cookies()`/`headers()`
- `"use cache: remote"` — shared remote cache (Redis, KV)

**Key restrictions:**

- Cached functions/components **cannot** directly access `cookies()`, `headers()`, or `searchParams` — read outside, pass as args
- Components accessing uncached data must be wrapped in `<Suspense>` when `cacheComponents` is enabled
- `generateStaticParams` is needed for dynamic routes to avoid Suspense requirements on `params`

### Cache APIs

**`cacheLife()`** — Built-in profiles: `'default'`, `'seconds'`, `'minutes'`, `'hours'`, `'days'`, `'weeks'`, `'max'`

```ts
// Custom profiles in next.config.ts
const nextConfig = {
  cacheComponents: true,
  cacheLife: {
    biweekly: { stale: 1209600, revalidate: 86400, expire: 1209600 },
  },
};
```

**`cacheTag()`** — Tag cached data for on-demand revalidation (max 256 chars, 128 tags per entry).

**`revalidateTag()`** — Now requires a `cacheLife` profile as second argument:

```ts
revalidateTag("blog-posts", "max"); // profile name or custom config
```

**`updateTag()`** — New: read-your-writes semantics in Server Actions:

```ts
"use server";
import { updateTag } from "next/cache";
export async function updateProfile(userId: string, data: Profile) {
  await db.users.update(userId, data);
  updateTag(`user-${userId}`); // expire + refresh immediately
}
```

**`refresh()`** — New: refresh uncached data only (doesn't touch cache):

```ts
"use server";
import { refresh } from "next/cache";
export async function markRead(id: string) {
  await db.notifications.markAsRead(id);
  refresh();
}
```

### Next.js 16 Config

```ts
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  // Turbopack — now top-level (was experimental.turbopack)
  turbopack: {
    /* custom config if needed */
  },

  // React Compiler — now top-level (was experimental.reactCompiler)
  reactCompiler: true,

  // Cache Components — opt-in explicit caching
  cacheComponents: true,

  // Custom cache profiles
  cacheLife: {
    blog: { stale: 3600, revalidate: 900, expire: 86400 },
  },
};

export default nextConfig;
```

### React Compiler (Stable — No Longer Experimental)

```ts
// next.config.ts — top-level, not under experimental
const nextConfig = { reactCompiler: true };
```

Requires: `npm install -D babel-plugin-react-compiler`

**No more manual memoization** — the compiler handles `useMemo`, `useCallback`, `React.memo` automatically.

### React 19.2 Features Available

| Feature | Description |
|---|---|
| **View Transitions** | Animate elements during navigations — no Framer Motion needed for basics |
| **`useEffectEvent`** | Extract non-reactive logic from Effects |
| **`<Activity />`** | Hide UI with `display: none` while maintaining state |

### State Management Stack

| Need | Solution |
|---|---|
| Server state | TanStack Query |
| Global client state | Zustand |
| Atomic state | Jotai |
| Form state | React Hook Form + Zod |
| URL state | nuqs |

### Component Libraries (2026)

- **shadcn/ui** — Copy-paste components, full control
- **Base UI** — Unstyled primitives (replacing Radix)
- **Tailwind CSS v4** — Utility-first styling

### Database Patterns

**Drizzle ORM** (Type-safe, lightweight)

```tsx
const posts = await db.select().from(postsTable).where(eq(postsTable.authorId, userId));
```

**Prisma** (DX-focused, migrations)

```tsx
const posts = await prisma.post.findMany({ where: { authorId: userId } });
```

### Performance Imperatives

1. **Eliminate waterfalls** — Use `Promise.all()` for parallel fetches
2. **Stream with Suspense** — Progressive rendering (required for uncached dynamic data with `cacheComponents`)
3. **Minimize `"use client"`** — Every directive increases bundle
4. **Use `"use cache"` strategically** — Cache what's expensive, leave the rest dynamic
5. **Leverage Turbopack FS caching** — `experimental.turbopackFileSystemCacheForDev: true` for faster restarts

### Core Web Vitals Targets

- **LCP** < 2.5s (Largest Contentful Paint)
- **INP** < 200ms (Interaction to Next Paint)
- **CLS** < 0.1 (Cumulative Layout Shift)

### Developer Experience

- **Isolated dev build** — `next dev` outputs to `.next/dev`, `next build` to `.next` (concurrent-safe, default on)
- **MCP built-in** — `/_next/mcp` endpoint in dev server for AI-assisted debugging, no config needed
- **Layout deduplication** — Shared layouts downloaded once during prefetching
- **Incremental prefetching** — Only fetches route segments not already cached

## Migration Checklist (15 → 16)

1. Ensure Node.js 24+
2. Run `npx @next/codemod@canary upgrade latest`
3. `await` all `params`, `searchParams`, `cookies()`, `headers()`, `draftMode()`
4. Rename `middleware.ts` → `proxy.ts`, export as `proxy`
5. Add `default.js` to all parallel route slots
6. Move `experimental.turbopack` → top-level `turbopack`
7. Move `experimental.reactCompiler` → top-level `reactCompiler`
8. Replace `experimental.ppr` / `experimental.dynamicIO` → `cacheComponents: true`
9. Drop `unstable_` prefix from `cacheLife` / `cacheTag` imports
10. Add second argument to `revalidateTag()` calls
11. Remove AMP, `next lint`, runtime config usage
12. Review `next/image` defaults (qualities, cache TTL)
13. Remove `legacyBehavior` from `<Link>` components
14. Remove `--turbopack` flag from scripts (now default)

## Agents

- **frontend-developer** — React, styling, components, performance
- **backend-architect** — APIs, auth, system design
- **rapid-prototyper** — MVPs in days, not weeks
- **database-specialist** — Schema, queries, migrations, optimization
