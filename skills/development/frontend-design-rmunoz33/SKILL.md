---
name: frontend-design
description: Create frontend components and pages for the Fables & Sagas D&D app. Use this skill when building UI for the dnd-homebrew-frontend project. Generates code that matches the existing dark fantasy aesthetic, theme system, and component patterns.
---

This skill guides creation of frontend interfaces for the **Fables & Sagas** D&D Solo app. All output must integrate seamlessly with the existing codebase and its dark medieval fantasy aesthetic.

## Tech Stack

- **Framework**: Next.js 16 with React 19, TypeScript
- **Styling**: Tailwind CSS v4 with DaisyUI v5
- **State**: Zustand (`useDnDStore` from `@/stores/useStore`)
- **Icons**: `lucide-react` (primary), `@mdi/react` + `@mdi/js` (supplemental)
- **Toasts**: `sonner` (dark theme, custom styled — see layout.tsx)
- **Lists**: `react-virtuoso` for virtualized scrolling
- **Markdown**: `react-markdown`
- **Fonts loaded via `next/font/google`**:
  - `MedievalSharp` — display/headings (import from `@/app/components/medievalFont`)
  - `EB_Garamond` — secondary display
  - `Geist` / `Geist_Mono` — body text (CSS variables `--font-geist-sans`, `--font-geist-mono`)

## Theme: "fables"

DaisyUI custom theme (`data-theme="fables"`) defined in `src/app/globals.css`:

| Token | Value | Usage |
|-------|-------|-------|
| `primary` | `#d4af37` (antique gold) | Headings, accents, borders, CTA |
| `primary-content` | `#0a0a0a` | Text on primary backgrounds |
| `secondary` | `#8b1a1a` (deep crimson) | Secondary actions, danger accents |
| `secondary-content` | `#f0e6c8` | Text on secondary backgrounds |
| `accent` | `#e6c44c` (bright gold) | Highlights, hover states |
| `neutral` | `#1a1a1a` | Card/panel backgrounds |
| `neutral-content` | `#d4c9a8` (parchment) | Body text on neutral |
| `base-100` | `#0a0a0a` (near-black) | Page background |
| `base-200` | `#1a1a1a` | Elevated surfaces |
| `base-300` | `#2a2a2a` | Higher elevation |
| `base-content` | `#d4c9a8` | Default text |
| `error` | `#dc2626` | Destructive actions |
| `success` | `#22c55e` | Confirmations |

**Always use DaisyUI semantic tokens** (`text-primary`, `bg-base-200`, `btn-primary`, etc.) instead of hardcoded colors.

## Custom Tailwind Utilities

Defined in `src/app/globals.css`:

- `bg-login-vignette` — radial gradient overlay for cinematic scenes
- `bg-gradient-radial`, `bg-gradient-conic` — custom gradient utilities
- `.animate-fade-in-up`, `.animate-fade-in-up-delay-1/2/3` — staggered entrance animations
- `.text-glow-gold` — gold text-shadow glow effect for dramatic headings

## Component Patterns

Follow these conventions from existing code:

- **"use client"** directive on all interactive components
- **DaisyUI classes** for base components: `btn`, `input`, `input-bordered`, `modal`, `drawer`, `card`, `badge`, `tooltip`
- **Opacity modifiers** for subtle layering: `bg-base-200/50`, `border-primary/15`, `text-base-content/30`
- **Section headers** use `medievalFont.className` with gold decorative horizontal rules (see `SectionHeader.tsx` pattern)
- **Modals** use fixed overlay with `bg-black bg-opacity-50`, centered card with `border-2`
- **Inputs** use: `input input-bordered w-full bg-base-200/50 border-primary/15 text-base-content placeholder:text-base-content/30 focus:border-primary/40 focus:outline-none`
- **Drawers** slide from right with backdrop blur

## Aesthetic Direction

This app has a **dark medieval fantasy** aesthetic — think weathered parchment text on near-black backgrounds, gold filigree accents, and deep crimson highlights. The tone is immersive and atmospheric, like opening an ancient tome by candlelight.

When designing new components:
- Lean into the **dark + gold** contrast
- Use `medievalFont` for section titles and dramatic text
- Apply subtle opacity and blur for depth (`backdrop-blur-sm`, `/50` opacity modifiers)
- Use staggered `animate-fade-in-up` for page entrances
- Add `text-glow-gold` sparingly for emphasis on hero elements
- Keep backgrounds dark (`base-100`, `base-200`) with gold (`primary`) as the accent color
- Borders should be subtle: `border-primary/15` to `border-primary/40`

## File Organization

- Components: `src/app/components/{Feature}/ComponentName.tsx`
- Hooks: `src/hooks/useHookName.ts`
- Stores: `src/stores/useStore.ts` (single Zustand store)
- API routes: `src/app/api/{endpoint}/route.ts`
- Services: `src/services/`
