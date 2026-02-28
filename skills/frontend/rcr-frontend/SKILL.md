---
name: rcr-frontend
description: Component development rules specific to Red Cliff Record. Use when working with React components, Tailwind CSS styling, Radix/Shadcn primitives, icons, buttons, forms, or frontend code in this project. Triggers on component files, styling questions, design tokens, Tailwind v4, Shadcn, Radix, TanStack Forms, Lucide icons, or UI primitive usage patterns (sizing, spacing, layout).
---

# RCR Frontend

Supplements global `react-guidelines` and `frontend-html-css-guidelines` skills.

## Component Organization

- Reusable components: `src/app/components/` with kebab-case naming (includes all Shadcn-derived primitives)
- No separate `ui/` directory — everything lives directly under `src/app/components/`
- Page-specific components: `-components/` directory or `-component.tsx` suffix

## Styling

- **Tailwind CSS v4** with `c-*` design tokens from `src/app/styles/theme.css`
- Semantic color pattern: `c-main` / `c-main-contrast`, `c-destructive` / `c-destructive-contrast`. Never invent token names — check `src/app/styles/app.css` for the full list
- **Never** use legacy Shadcn theme variables (`bg-background`, `text-foreground`, etc.)
- **Detecting invalid classes**: Oxfmt sorts unknown classes to the front. If classes appear out of order after `bun check`, they're misspelled or missing from the theme

## Radix & Shadcn

- Always import Radix from the `radix-ui` package (`import { HoverCard as HoverCardPrimitive } from 'radix-ui'`), not from subpackages like `@radix-ui/react-hover-card`
- Use `asChild` to compose styling (e.g., dropdown trigger styled as Button) rather than duplicating inline styles
- Mark key DOM nodes with `data-slot` attributes for styling hooks

## Component Primitives

### Buttons

The `Button` component handles sizing, text, and icon dimensions internally. Never override these with utility classes.

- **Dimensions**: use the `size` prop (`default` | `sm` | `icon`), never manual `h-x w-x p-0`. For icon-only buttons, `size="icon"` gives `size-9`.
- **Text size**: baked into the base (`text-sm`). Never add `text-xs`, `text-sm`, etc. via `className`.
- **Icon size**: the base rule `[&_svg:not([class*='size-'])]:size-4` auto-sizes child SVGs. Never add `h-x w-x` or `size-x` to icons inside buttons — omit sizing entirely and let the button handle it.
- **Content over positioning**: put content (counts, badges) as regular children in flow — never absolutely-positioned overlays.

### Icons

- Lucide icons with `Icon` suffix: `HomeIcon`, not `Home`
- Icons inherit size from surrounding text automatically. `size-[1em]` is redundant — omit it.
- Only add explicit `size-*` when overriding the contextual size (e.g., a larger icon in small text).
- Use `<Spinner />` from `@/components/spinner` for loading states

## Tailwind Shorthand

- When width and height are equal, use `size-x` instead of `w-x h-x`
- No custom/arbitrary text sizes (`text-[10px]`, `text-[13px]`, etc.) — use design system tokens (`text-xs`, `text-sm`, etc.) unless there's a specific constraint preventing it

## Event Handlers

- No inline arrow functions in JSX `onClick`/`onChange` props — extract to a named handler or a component with its own handler

## Forms

- TanStack Forms + Zod schemas for form management and validation
