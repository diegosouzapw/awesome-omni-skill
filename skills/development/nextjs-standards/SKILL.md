---
name: nextjs-standards
description: Comprehensive coding standards, verification protocols, and templates for Next.js App Router projects. Auto-loads on Next.js detection.
---

# Next.js Coding Standards Skill

> **Auto-Load Trigger:** Presence of `next.config.*` or `"next"` in `package.json`

## When to Use

- Starting a new Next.js project
- Before any TypeScript/React file edit
- When context seems lost mid-session
- After `/agent_reset` or `/vibe-primeAgent`

---

## 🛑 Verification Protocol (MANDATORY)

### After Every TypeScript/TSX File Edit

```bash
npx tsc --noEmit
```

**If this fails:**
1. DO NOT proceed to next file
2. Fix the type error immediately
3. Re-run until it passes
4. Only then continue

### Before Any Handoff

```bash
python scripts/vibe-verify.py
```

All checks must pass before claiming "done."

---

## The Blueprint & Build Protocol

### Phase 1: Blueprint (Before Coding)
1. Check `docs/features/` for existing patterns
2. Create/update `docs/features/FeatureName.md`
3. Wait for approval before coding

### Phase 2: Build (Implementation)
1. Announce which FR-XXX you're implementing
2. Reference the corresponding issue in `docs/issues/`
3. Implement one step at a time
4. `tsc --noEmit` after every file
5. Mark acceptance criteria as complete

### Phase 3: Finalization
1. Run full verification (`vibe-verify.py`)
2. Update issue file with completion status
3. Generate handoff summary

---

## Full-Stack Type Safety

The AI reliability secret: **TypeScript tells you when you broke something.**

### Core Principle
If you change the backend, the frontend MUST type-check. If type-check fails:
- The change broke something
- Fix it before moving on
- Never ignore type errors

### Stack Alignment
- Server Components fetch data → types flow to client
- API routes return typed responses → frontend consumes them
- Prisma schema changes → regenerate client → type-check

---

## Next.js App Router Rules

1. **Server Components Default** — No `'use client'` unless required
2. **Client Components Sparingly** — Only for interactivity, hooks, browser APIs
3. **Data Fetching** — In async Server Components, not `useEffect`
4. **Route Handlers** — All API in `app/api/.../route.ts`
5. **Caching** — Be explicit: `{ cache: 'no-store' }` or `revalidate = N`

---

## File Structure (Feature-Sliced)

```
src/
├── app/                    # Next.js App Router pages
├── features/               # Business domains
│   └── [FeatureName]/
│       ├── components/     # Feature-specific components
│       ├── hooks/          # Feature-specific hooks
│       └── *.service.ts    # Business logic
├── components/
│   ├── ui/                 # Reusable UI (Button, Card)
│   └── layout/             # Layout components
├── lib/                    # Utilities, clients
└── scripts/                # Automation (vibe-verify.py)
```

---

## Component Rules

1. **200-Line Limit** — Refactor if exceeded
2. **Single Responsibility** — One thing per component
3. **Props Interface** — Always use TypeScript interfaces
4. **Custom Hooks** — Extract stateful logic into `use*` hooks

---

## Styling (Tailwind v4)

> **Why Tailwind for AI Reliability:** Tailwind colocates styles with UI in a single file. Unlike separate `.css` files, AI agents see the complete context (logic + styles + structure) without jumping between files. This dramatically reduces hallucinations and context fragmentation.

```css
@import "tailwindcss";

@theme {
  --color-background: #ffffff;
  --color-foreground: #0b1221;
  --color-border: #e5e7eb;
}

@theme .dark {
  --color-background: #0b1221;
  --color-foreground: #e5e7eb;
}
```

- Use `@theme` tokens, not `tailwind.config` extensions
- Utility-first, no custom CSS files
- Dark mode via `.dark` class on `<html>`

---

## Backend Rules

### Service Layer Pattern
- **Route Handlers** = Controllers (parse request, return response)
- **Services** = Business logic (DB queries, calculations)

### Validation
```typescript
import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});
```

All inputs validated with Zod. No exceptions.

---

## Templates Available

This skill provides templates in `templates/`:

1. **Coding_Guidelines.md** — Copy to `docs/Coding_Guidelines.md`
2. **Issue_Template.md** — Format for FR issues

---

## Quick Reference

| Command | When |
|---------|------|
| `npx tsc --noEmit` | After every TS/TSX edit |
| `python scripts/vibe-verify.py` | Before handoff |
| `npm run lint` | Check code style |
| `npm run build` | Verify production build |

---

## Recovery Protocol

If you break something:

```bash
git status                    # See changed files
git diff                      # See what changed
git checkout -- <file>        # Revert specific file
git stash                     # Save and revert all
```

