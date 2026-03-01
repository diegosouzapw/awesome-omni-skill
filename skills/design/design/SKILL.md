---
name: design
description: Design consistency and visual styling for the Svelte client UI. Use when creating or modifying visual elements, colors, typography, buttons, inputs, or cards.
---

# Design

> All code must follow the **Coding Principles** in AGENTS.md (functional, minimal, readable, modular).

## Core principles

- No scrolling — ever. The webview is a fixed sandboxed canvas (min 320×320px).
- Mobile-first. Design for the smallest viewport, then scale up.
- Tailwind only. No `<style>` blocks, no inline `style=` attributes.
- Functional over decorative. Every visual element should serve the user.

---

## Spacing scale

Use Tailwind's default spacing scale. Avoid arbitrary values (`p-[13px]`).

| Use case | Class |
|----------|-------|
| Component padding | `p-4` (16px) |
| Section gap | `gap-4` or `space-y-4` |
| Tight inline gap | `gap-2` |
| Card/panel padding | `p-4` or `p-6` |
| Page horizontal padding | `px-4` |
| Page vertical padding | `py-3` |

---

## Typography

```svelte
<h1 class="text-xl font-bold tracking-tight">Title</h1>
<h2 class="text-base font-semibold">Section</h2>
<p  class="text-sm text-gray-700 dark:text-gray-300">Body</p>
<span class="text-xs text-gray-500">Caption / meta</span>
```

- Base body size: `text-sm` (14px). Never below `text-xs` for readable content.
- Line length: `max-w-sm` or `max-w-xs` for prose.
- No custom fonts — use the system font stack (Tailwind default).

---

## Color

| Role | Light | Dark |
|------|-------|------|
| Background | `bg-white` | `dark:bg-gray-900` |
| Surface / card | `bg-gray-50` | `dark:bg-gray-800` |
| Border | `border-gray-200` | `dark:border-gray-700` |
| Primary text | `text-gray-900` | `dark:text-gray-100` |
| Secondary text | `text-gray-500` | `dark:text-gray-400` |
| Accent / brand | `text-orange-500` | same |
| Destructive | `text-red-500` | same |
| Success | `text-green-500` | same |

Always pair a light and dark variant. Reddit renders in both modes.

---

## Buttons

```svelte
<!-- Primary -->
<button class="w-full rounded-lg bg-orange-500 px-4 py-2.5 text-sm font-semibold text-white
               hover:bg-orange-600 active:scale-95 transition-all disabled:opacity-50
               disabled:cursor-not-allowed focus-visible:outline-none focus-visible:ring-2
               focus-visible:ring-orange-400">
  Submit
</button>

<!-- Secondary -->
<button class="w-full rounded-lg border border-gray-300 dark:border-gray-600 px-4 py-2.5
               text-sm font-semibold text-gray-700 dark:text-gray-200
               hover:bg-gray-100 dark:hover:bg-gray-700 active:scale-95 transition-all">
  Cancel
</button>
```

- Minimum tap target: `min-h-[44px]` or `py-2.5` on `text-sm`.
- Always include `disabled:opacity-50 disabled:cursor-not-allowed`.
- Always include `active:scale-95 transition-all` for tactile feedback.

---

## Inputs

```svelte
<input
  class="w-full rounded-lg border border-gray-300 dark:border-gray-600 bg-white
         dark:bg-gray-800 px-3 py-2 text-sm text-gray-900 dark:text-gray-100
         placeholder:text-gray-400 focus:outline-none focus:ring-2 focus:ring-orange-400"
/>
```

---

## UI states

Every data-driven component must handle all four states:

| State | Pattern |
|-------|---------|
| Loading | `animate-pulse` skeleton or spinner with label |
| Error | Red tinted surface, human-readable message |
| Empty | Neutral muted text, optionally a CTA |
| Success | Normal render; optionally brief `text-green-500` confirmation |

---

## Cards & surfaces

```svelte
<div class="rounded-xl border border-gray-200 dark:border-gray-700
            bg-white dark:bg-gray-800 p-4 shadow-sm">
  <!-- content -->
</div>
```

- `rounded-xl` for cards, `rounded-lg` for inputs/buttons, `rounded-md` for badges.
- `shadow-sm` only — avoid heavy shadows.
- Max two surface levels deep (page → card → no deeper).

---

## Animation

- Only Tailwind built-in transitions: `transition-all`, `transition-colors`, `transition-opacity`.
- Default duration (`150ms`). No custom durations.
- `animate-pulse` for loading, `animate-spin` for spinners.
- No entrance animations.

---

## Checklist before finishing
- [ ] No `<style>` blocks — Tailwind classes only
- [ ] No arbitrary values (`p-[13px]`) — use scale steps
- [ ] Dark mode variants on all color classes
- [ ] Buttons have `disabled`, `active:scale-95`, and `focus-visible:ring` classes
- [ ] Minimum tap target size respected
- [ ] Typography follows the hierarchy (xl → base → sm → xs)
