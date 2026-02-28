---
name: sixty-design-system
description: Sixty's production design system for enterprise SaaS applications. Use when building UI components, creating React/Next.js interfaces, styling dashboards, implementing light/dark themes, or applying Sixty's glassmorphic dark mode aesthetic. Triggers on frontend development, UI styling, component creation, theme implementation, and any Sixty product interface work.
requires-profile: true
---

# Sixty Design System

---

## STEP 0: Select Model Profile

Before proceeding, ask the user to select which model profile to use:
- **Economy** — Fastest, lowest cost
- **Balanced** — Good balance of speed & accuracy
- **Thorough** — Most accurate, highest cost

Use the `AskUserQuestion` tool with these options.

**Note**: Based on selection, appropriate models will be assigned:
- Economy: Simple styling, minor UI updates
- Balanced: Standard component building, design system implementation
- Thorough: Complex patterns, multi-component systems

---

Production-ready design system with clean minimal light mode and premium glassmorphic dark mode. Built for Next.js/React with Tailwind CSS.

## Core Principles

**Light Mode**: Pure white backgrounds (#FFFFFF, #FCFCFC), high contrast text (gray-900/gray-700), clean borders (gray-200), minimal shadows.

**Dark Mode (Glassmorphism)**: Deep backgrounds (gray-950: #030712), glass cards with `bg-gray-900/80 backdrop-blur-sm`, subtle borders with opacity (`border-gray-700/50`), no shadows.

## Essential Patterns

### Theme-Aware Classes (Use These First)

```tsx
// Backgrounds
"bg-white dark:bg-gray-950"              // Page background
"bg-white dark:bg-gray-900/80 backdrop-blur-sm"  // Cards
"bg-[#FCFCFC] dark:bg-gray-900"          // Secondary surfaces

// Text
"text-gray-900 dark:text-gray-100"       // Primary
"text-gray-700 dark:text-gray-300"       // Secondary
"text-gray-500 dark:text-gray-400"       // Tertiary

// Borders
"border-gray-200 dark:border-gray-700/50"  // Standard
"border-gray-300 dark:border-gray-800/50"  // Emphasized

// Interactive states
"hover:bg-gray-50 dark:hover:bg-gray-800/30"
```

### Card Pattern

```tsx
// Standard card
<div className="bg-white dark:bg-gray-900/80 dark:backdrop-blur-sm 
                border border-gray-200 dark:border-gray-700/50
                rounded-xl p-6 shadow-sm dark:shadow-none">
  {children}
</div>
```

### Button Variants

```tsx
// Primary (blue)
"bg-blue-600 hover:bg-blue-700 text-white rounded-lg px-4 py-2.5 font-medium transition-colors"

// Secondary
"bg-gray-100 dark:bg-gray-800 hover:bg-gray-200 dark:hover:bg-gray-700 
 text-gray-900 dark:text-gray-100 rounded-lg px-4 py-2.5 font-medium"

// Ghost
"bg-transparent hover:bg-gray-100 dark:hover:bg-gray-800/50 
 text-gray-700 dark:text-gray-300 rounded-lg px-4 py-2.5"
```

### Input Fields

```tsx
"bg-white dark:bg-gray-800/50 
 border border-gray-300 dark:border-gray-700/50 
 text-gray-900 dark:text-gray-100 
 rounded-lg px-4 py-2.5 
 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
```

## Setup Requirements

### Dependencies

```bash
npm install tailwindcss class-variance-authority clsx tailwind-merge
```

### tailwind.config.js

```js
module.exports = {
  darkMode: ['class', '[data-theme="dark"]'],
  content: ['./src/**/*.{js,ts,jsx,tsx,mdx}'],
  theme: {
    extend: {
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      },
    },
  },
}
```

### Utility Function (lib/utils.ts)

```typescript
import { clsx, type ClassValue } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

## Quick Reference

| Element | Light | Dark |
|---------|-------|------|
| Page BG | `#FFFFFF` | `gray-950` (#030712) |
| Card BG | `white` + `shadow-sm` | `gray-900/80` + `backdrop-blur-sm` |
| Border | `gray-200` | `gray-700/50` |
| Text Primary | `gray-900` | `gray-100` |
| Text Secondary | `gray-700` | `gray-300` |
| Shadow | `shadow-sm` / `shadow-md` | `shadow-none` |

## Glassmorphism Rules

**DO:**
- Use `backdrop-blur-sm` (4px) for cards/modals, `backdrop-blur-xl` (24px) for premium surfaces
- Combine blur with semi-transparent backgrounds: `bg-gray-900/80`
- Add inset highlights: `inset 0 1px 0 rgba(255,255,255,0.05)`
- Include webkit prefix: `-webkit-backdrop-filter`
- Use `shadow-none` in dark mode

**DON'T:**
- Use backdrop blur without semi-transparent backgrounds
- Stack more than 2-3 glass layers
- Use heavy shadows with glass effects
- Apply glassmorphism in light mode
- Exceed `blur(24px)` for performance

## Icons (Lucide React)

**DO:**
- Use `lucide-react` for all icons
- Match icon stroke width to text weight context
- Use semantic icon names that match their purpose

**DON'T:**
- Use `Sparkles` icon from Lucide - it renders poorly and doesn't match our aesthetic. Use alternatives like `Wand2`, `Stars`, or `Zap` instead

## Detailed References

For complete specifications, load these as needed:

- **[components.md](components.md)**: Full component library with all variants, modals, popovers, navigation, tables
- **[tokens.md](tokens.md)**: Complete color tokens, spacing, typography scales, semantic colors
- **[patterns.md](patterns.md)**: Layout patterns, dashboard grids, responsive breakpoints, animation specs
