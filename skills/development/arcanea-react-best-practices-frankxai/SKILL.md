---
name: arcanea-react-best-practices
description: React 19 and Next.js 16 performance optimization for the Arcanea platform. Use when writing, reviewing, or refactoring React components, data fetching logic, bundle optimization, or any frontend performance work. Triggers on: React components, Next.js pages, hooks, data fetching, bundle size, re-renders, Server Components, Client Components, hydration. Sourced from Vercel Engineering's official React best practices (57 rules, 8 categories) and adapted for the Arcanea stack.
license: MIT (source: vercel-labs/agent-skills)
metadata:
  author: Arcanea (adapted from Vercel Labs)
  version: "1.0.0"
  stack: Next.js 16, React 19, TypeScript strict, Tailwind CSS, Vercel AI SDK 6
---

# Arcanea React Best Practices

> *"Leyla guards the Gate of Flow at 285 Hz. Your components must flow, not block. Every waterfall is a Flow violation."*

Adapted from Vercel Engineering's 57-rule guide for the Arcanea stack: **Next.js 16 App Router + React 19 + TypeScript strict + Supabase + Vercel AI SDK 6**.

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Eliminating Waterfalls | CRITICAL | `async-` |
| 2 | Bundle Size | CRITICAL | `bundle-` |
| 3 | Server-Side | HIGH | `server-` |
| 4 | Client Data Fetching | MEDIUM-HIGH | `client-` |
| 5 | Re-render Optimization | MEDIUM | `rerender-` |
| 6 | Rendering Performance | MEDIUM | `rendering-` |
| 7 | JavaScript Performance | LOW-MEDIUM | `js-` |
| 8 | Advanced Patterns | LOW | `advanced-` |

---

## 1. Eliminating Waterfalls (CRITICAL)

### `async-parallel` — Use Promise.all for independent fetches
```typescript
// BAD — serial waterfall
const user = await getUser(id)
const posts = await getUserPosts(id) // waits for user unnecessarily

// GOOD — parallel
const [user, posts] = await Promise.all([getUser(id), getUserPosts(id)])
```

### `async-defer-await` — Move await to where it's actually needed
```typescript
// BAD
async function handler() {
  const data = await fetchData() // awaited before branch
  if (condition) return data
  return null
}

// GOOD
async function handler() {
  const dataPromise = fetchData() // start early
  if (!condition) return null
  return await dataPromise // await only when needed
}
```

### `async-suspense-boundaries` — Stream content with Suspense
```tsx
// In Arcanea pages — wrap slow data in Suspense for progressive rendering
export default function LorePage() {
  return (
    <Suspense fallback={<GlassCardSkeleton />}>
      <LoreContent /> {/* streams in independently */}
    </Suspense>
  )
}
```

### `async-api-routes` — Start promises early in route handlers
```typescript
// app/api/guardians/route.ts
export async function GET() {
  const guardianPromise = supabase.from('guardians').select() // start immediately
  // ... validation/auth work ...
  const { data } = await guardianPromise // await late
  return Response.json(data)
}
```

---

## 2. Bundle Size (CRITICAL)

### `bundle-barrel-imports` — Import directly, never from barrel files
```typescript
// BAD — pulls in entire barrel
import { Button, Card, Modal } from '@/components/ui'

// GOOD — direct imports
import { Button } from '@/components/ui/button'
import { Card } from '@/components/ui/card'
```

### `bundle-dynamic-imports` — Lazy-load heavy components
```typescript
// Heavy Arcanea components: GlobeMap, PromptEditor, LoreCanvas
import dynamic from 'next/dynamic'

const LoreCanvas = dynamic(() => import('@/components/lore/LoreCanvas'), {
  loading: () => <GlassCardSkeleton />,
  ssr: false, // client-only canvas
})
```

### `bundle-defer-third-party` — Analytics after hydration
```typescript
// In Arcanea layout — defer non-critical scripts
import { GoogleAnalytics } from '@next/third-parties/google'
// Renders after hydration automatically via next/third-parties
```

---

## 3. Server-Side Performance (HIGH)

### `server-cache-react` — Deduplicate DB calls per request
```typescript
// lib/supabase-cached.ts — use React.cache for per-request dedup
import { cache } from 'react'
import { supabase } from '@/lib/supabase'

export const getGuardian = cache(async (gateName: string) => {
  const { data } = await supabase
    .from('guardians')
    .select()
    .eq('gate', gateName)
    .single()
  return data
})
// Multiple RSCs can call getGuardian('Fire') — only one DB query
```

### `server-auth-actions` — Always authenticate Server Actions
```typescript
// app/actions/prompt.ts
'use server'
import { createServerClient } from '@/lib/supabase-server'

export async function savePrompt(data: PromptData) {
  const supabase = createServerClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) throw new Error('Unauthorized') // NEVER skip this
  // ...
}
```

### `server-parallel-fetching` — Restructure RSC to parallelize
```typescript
// BAD — sequential RSC
async function GuardianPage({ gate }: { gate: string }) {
  const guardian = await getGuardian(gate) // serial
  const godbeast = await getGodbeast(guardian.id) // waits
}

// GOOD — parallel RSC
async function GuardianPage({ gate }: { gate: string }) {
  const [guardian, godbeast] = await Promise.all([
    getGuardian(gate),
    getGodBeastByGate(gate), // use gate directly
  ])
}
```

---

## 4. Client Data Fetching (MEDIUM-HIGH)

### `client-swr-dedup` — SWR/React Query for client fetches
```typescript
// Prefer SWR for client-side Arcanea data
import useSWR from 'swr'

function useGuardians() {
  return useSWR('/api/guardians', fetcher, {
    revalidateOnFocus: false, // lore data doesn't change often
  })
}
```

---

## 5. Re-render Optimization (MEDIUM)

### `rerender-memo` — Memoize expensive Arcanea components
```typescript
// Heavy visual components: GuardianCard, LoreGrid, GodBeastProfile
import { memo } from 'react'

const GuardianCard = memo(function GuardianCard({ guardian }: Props) {
  return <div className="glass-card">...</div>
})
```

### `rerender-derived-state-no-effect` — Derive during render
```typescript
// BAD — unnecessary effect
const [filteredGuardians, setFiltered] = useState([])
useEffect(() => {
  setFiltered(guardians.filter(g => g.element === selectedElement))
}, [guardians, selectedElement])

// GOOD — derive during render
const filteredGuardians = guardians.filter(g => g.element === selectedElement)
```

### `rerender-lazy-state-init` — Lazy init for expensive state
```typescript
// BAD — recalculates every render
const [state, setState] = useState(computeExpensiveDefault())

// GOOD — only runs once
const [state, setState] = useState(() => computeExpensiveDefault())
```

### `rerender-transitions` — Non-urgent updates via startTransition
```typescript
import { useTransition } from 'react'

function GateFilter() {
  const [isPending, startTransition] = useTransition()
  
  return (
    <select onChange={e => startTransition(() => setGate(e.target.value))}>
      {/* Gate filter won't block urgent UI updates */}
    </select>
  )
}
```

---

## 6. Rendering Performance (MEDIUM)

### `rendering-hoist-jsx` — Extract static JSX outside components
```typescript
// BAD — recreated every render
function GuardianList() {
  const header = <h2 className="text-gradient-aurora">The Ten Gates</h2>
  return <div>{header}{items}</div>
}

// GOOD — hoisted static
const HEADER = <h2 className="text-gradient-aurora">The Ten Gates</h2>
function GuardianList() {
  return <div>{HEADER}{items}</div>
}
```

### `rendering-conditional-render` — Ternary, not &&
```typescript
// BAD — renders "0" if count is 0
{count && <Badge>{count}</Badge>}

// GOOD
{count > 0 ? <Badge>{count}</Badge> : null}
```

---

## 7. JavaScript Performance (LOW-MEDIUM)

### `js-index-maps` — Map for repeated lookups
```typescript
// BAD — O(n) per lookup
function getGuardian(gate: string) {
  return guardians.find(g => g.gate === gate)
}

// GOOD — O(1) lookup
const guardianMap = new Map(guardians.map(g => [g.gate, g]))
const getGuardian = (gate: string) => guardianMap.get(gate)
```

### `js-early-exit` — Return early from functions
```typescript
// GOOD Arcanea pattern
function validateGateUnlock(user: User, gate: Gate): ValidationResult {
  if (!user.authenticated) return { valid: false, reason: 'unauthenticated' }
  if (user.gatesOpen < gate.requiredRank) return { valid: false, reason: 'rank' }
  if (gate.locked) return { valid: false, reason: 'sealed' }
  return { valid: true }
}
```

---

## 8. Arcanea-Specific Patterns

### AI SDK 6 — Vercel AI SDK streaming
```typescript
// app/api/guardian-chat/route.ts — AI SDK 6 pattern
import { streamText } from 'ai'
import { google } from '@ai-sdk/google'

export async function POST(req: Request) {
  const { messages } = await req.json()
  const result = streamText({
    model: google('gemini-2.5-flash'),
    messages,
    maxOutputTokens: 2048, // NOT maxTokens (SDK 6 change)
  })
  return result.toUIMessageStreamResponse() // NOT toDataStreamResponse (SDK 6)
}
```

### Supabase Auth in Server Components
```typescript
// Always use server-side auth for SSR
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function getServerUser() {
  const cookieStore = cookies()
  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    { cookies: { get: (name) => cookieStore.get(name)?.value } }
  )
  const { data: { user } } = await supabase.auth.getUser()
  return user
}
```

### Glass Components — Design System compliance
```typescript
// Always use Arcanean Design System classes
// See: .arcanea/design/DESIGN_BIBLE.md

// Glass tiers: glass-subtle (8px) | glass (16px) | glass-strong (24px) | liquid-glass (40px)
// Text: text-gradient-aurora | text-gradient-gold | text-gradient-violet
// Animations: float | pulse-glow | shimmer | cosmic-drift
// FONTS: Inter everywhere (NO Cinzel in new code — MEMORY.md override)

function GuardianCard({ guardian }: Props) {
  return (
    <div className="glass hover-lift glow-card">
      <h3 className="text-gradient-aurora">{guardian.name}</h3>
    </div>
  )
}
```

---

## Quick Checklist

Before any React/Next.js PR in Arcanea:

- [ ] No sequential awaits where Promise.all applies
- [ ] Heavy components use `next/dynamic`
- [ ] No barrel imports — direct imports only
- [ ] Server Actions authenticate with Supabase before any mutation
- [ ] `React.cache()` wraps repeated DB calls in same RSC tree
- [ ] State derived during render, not in useEffect
- [ ] Vercel AI SDK 6: `maxOutputTokens` and `toUIMessageStreamResponse`
- [ ] Glass components use Design Bible classes
- [ ] Fonts: Inter only (no Cinzel in new code)
