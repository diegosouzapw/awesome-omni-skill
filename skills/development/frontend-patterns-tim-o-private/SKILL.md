---
name: frontend-patterns
description: React, TypeScript, and UI patterns for webApp/src/. Use when writing or modifying TSX/TS files. Covers Radix Themes (provider only, NOT components), semantic color tokens, React Query hooks, auth from Supabase client, Zustand overlays, react-hook-form + Zod, accessibility (WCAG 2.1 AA), and ADHD-friendly design.
---

# Frontend Patterns

## Principles That Apply

| ID | Rule | Enforcement |
|----|------|-------------|
| A4 | Server state in React Query; client-only state in Zustand | Reviewer |
| A5 | Auth tokens from `supabase.auth.getSession()`, never Zustand | `validate-patterns.sh` BLOCKS in hooks |
| A7 | Every message feature works web + Telegram via shared chat_id | Reviewer |
| A10 | Entity "foo" → `useFooHooks.ts`, `FooComponent.tsx` | Reviewer + `task-completed-gate.sh` |

For full rationale on any principle: `.claude/skills/architecture-principles/reference.md`

## Architecture

```
webApp/src/
  api/hooks/           → React Query hooks (data fetching)
  components/ui/       → Reusable primitives (Card, Modal, Button)
  components/features/ → Domain components (tasks/, auth/)
  features/            → Feature modules (auth/useAuthStore)
  pages/               → Page-level components
  stores/              → Zustand stores
  styles/              → index.css, ui-components.css
```

## Quick Checklist

Before writing frontend code, verify:
- [ ] No `@radix-ui/themes` component imports (only `<Theme>` provider)
- [ ] All colors use semantic tokens (`bg-brand-primary`, `text-text-secondary`)
- [ ] Data fetching via React Query hooks, not useState+useEffect
- [ ] Auth tokens from `supabase.auth.getSession()`, not Zustand
- [ ] Modals via `useOverlayStore`, not local state
- [ ] Loading + error states handled in every data component
- [ ] Forms use react-hook-form + Zod
- [ ] Keyboard support + ARIA labels on interactive elements
- [ ] CSS transitions preferred over framer-motion
- [ ] Path aliases used (`@/`, `@components/` — no relative `../../../`)
- [ ] API base URL uses `import.meta.env.VITE_API_BASE_URL || ''` — never `VITE_API_URL`

## Critical Rule: Radix Themes = Provider Only

```tsx
// ✅ Theme provider in main.tsx
import { Theme } from '@radix-ui/themes';

// ✅ Radix Primitives + Tailwind for components
import * as Dialog from '@radix-ui/react-dialog';

// ❌ NEVER import pre-styled Radix Themes components
import { Button } from '@radix-ui/themes';  // FORBIDDEN
```

## Design Philosophy

This app targets users with ADHD. Prioritize: calm & minimal, clear hierarchy, low friction, encouraging tone, predictable behavior.

## Recipe: Add a New Page

Per A10 (naming predictable from domain model) and F2 (architecture makes standards self-evident):

1. **Page component:** `webApp/src/pages/<Name>Page.tsx`
2. **Route:** Add to router config in `webApp/src/App.tsx` (or equivalent routing file)
3. **Navigation:** Add to `webApp/src/components/navigation/` nav config
4. **Hooks:** If new API data needed: `webApp/src/api/hooks/use<Name>Hooks.ts` (React Query, per A4)
5. **Tests:** `webApp/src/pages/<Name>Page.test.tsx`

Auth guard: wrap with auth check if route requires login (per A5).

## Key Gotchas

1. **Frontend API base URL** — Use `import.meta.env.VITE_API_BASE_URL || ''`, never `VITE_API_URL`.
2. **Supabase env vars** — Frontend reads `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` (VITE_ prefix required).
3. **Color validation disabled** — `validate-colors.js` exists but is removed from build. Don't re-enable without review.
4. **`.gitignore` `lib/` rule** — Root `.gitignore` has `lib/`. `webApp/src/lib/` is negated. New `lib/` directories elsewhere need similar negation.

## Detailed Reference

For full patterns with code examples (color tokens, React Query, auth, forms, accessibility, keyboard shortcuts, animations), see [reference.md](reference.md).
