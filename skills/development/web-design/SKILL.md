---
name: web-design
description: Review, fix, and enforce web UI quality standards for accessibility, visual design, and performance. Use when reviewing web component files for issues (`/web-design Button.tsx`), fixing accessibility or visual problems (`/web-design fix Button.tsx`), creating new components with quality constraints applied, or loading design constraints for the conversation (`/web-design`). Covers WCAG accessibility, interaction patterns, animation, layout, typography, and prevents common AI-generated UI issues. For web apps built with HTML/CSS/JS, React, Vue, Tailwind, etc.
license: MIT
---

# Web Design

Review, fix, and enforce web UI quality standards.

## Invocation

| Command | Behavior |
|---------|----------|
| `/web-design Component.tsx` | Review file for issues (default mode) |
| `/web-design fix Component.tsx` | Apply minimal targeted fixes |
| `/web-design` | Load constraints for this conversation |

## Mode Behavior

### Review Mode (default)

When a file path is provided and the file exists, perform a full audit:

1. Read the file completely before making assessments
2. Check against all rules in `references/rules.md`
3. Output findings using the format below
4. Offer to fix issues if any are found

**Review Output Format:**

```
═══════════════════════════════════════════════════
DESIGN REVIEW: [filename]
═══════════════════════════════════════════════════

CRITICAL ([n] issues)
─────────────────────
Line [n]: [issue description]
  [code snippet]
  Fix: [specific fix]
  Rule: [rule reference]

SERIOUS ([n] issues)
────────────────────
...

MODERATE ([n] issues)
─────────────────────
...

═══════════════════════════════════════════════════
SUMMARY: [n] critical, [n] serious, [n] moderate
═══════════════════════════════════════════════════
```

### Fix Mode

When invoked with `fix`, apply minimal targeted changes:

1. Prioritize critical issues first
2. Make surgical edits—do not refactor unrelated code
3. Prefer native HTML semantics before adding ARIA
4. Show each fix with before/after

### Create Mode

When the file path doesn't exist:

- If context is provided (user described what they want), create the component with all constraints applied
- If no context, ask: "[filename] doesn't exist. What should this component do?"

### Enforce Mode

When invoked without a file path, acknowledge constraints are loaded and apply them to all UI work in the conversation.

## Stack Defaults

These are the opinionated defaults. Override only when the project uses different tools.

| Category | Default | Notes |
|----------|---------|-------|
| Styling | Tailwind CSS | Use defaults unless custom values exist |
| Animation (JS) | `motion/react` | Formerly framer-motion |
| Animation (CSS) | `tw-animate-css` | For entrance/micro-animations |
| Class logic | `cn` (`clsx` + `tailwind-merge`) | |
| Primitives | Base UI, React Aria, or Radix | Never rebuild keyboard/focus behavior |

**Primitive Rules:**
- Use project's existing primitives first
- Never mix primitive systems in the same interaction surface
- Prefer Base UI for new primitives if stack-compatible
- Never rebuild keyboard or focus behavior unless explicitly requested

## Rule Categories

Rules are organized by priority. See `references/rules.md` for the complete list.

| Priority | Category | Severity |
|----------|----------|----------|
| 1 | Accessible Names | Critical |
| 2 | Keyboard Access | Critical |
| 3 | Focus & Dialogs | Critical |
| 4 | Interactions | High |
| 5 | Forms | High |
| 6 | Animation | Medium |
| 7 | Layout | Medium |
| 8 | Typography | Medium |
| 9 | Performance | Medium |
| 10 | Visual Polish | Low |

## Quick Reference

### Critical (must fix)

- Icon-only buttons must have `aria-label`
- All interactive elements reachable by Tab
- Visible focus states (never `outline-none` without replacement)
- Modals trap focus and restore on close
- No `<div onClick>` for navigation—use `<a>` or `<button>`

### Anti-Slop Rules

These prevent common AI-generated UI issues:

- Never add animation unless explicitly requested
- Never use gradients unless explicitly requested
- Never use purple or multicolor gradients
- Never use glow effects as primary affordances
- Never exceed 200ms for interaction feedback
- Never modify letter-spacing unless explicitly requested
- Never use `h-screen`—use `h-dvh`
- Never block paste in inputs
- Empty states must have one clear next action

### Must Always Apply

- `text-balance` for headings, `text-pretty` for body
- `tabular-nums` for numeric data
- Respect `prefers-reduced-motion`
- Respect safe areas (`env(safe-area-inset-*)`)
- Use AlertDialog for destructive actions
- Show errors inline next to where action happens
