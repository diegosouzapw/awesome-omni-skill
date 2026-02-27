---
name: frontend-governance
description: Enforces Contemplative design system and Anti-Slop protocols for all UI generation
activation: automatic
triggers:
  - "apps/web/src/**/*.tsx"
  - "apps/web/src/**/*.css"
  - component
  - page
  - layout
  - styling
  - design
---

# Frontend Governance Skill

## Design Philosophy: Contemplative

UI must exhibit restraint, intentionality, and quiet confidence. Reject the generic SaaS aesthetic. Every element earns its place through purpose, not decoration.

## Semantic Token System

All styling MUST use the project's semantic tokens. Direct color values are forbidden.

### Background Layers

| Token              | Usage                        | Tailwind Class |
| ------------------ | ---------------------------- | -------------- |
| `--color-void`     | Page background, deepest     | `bg-void`      |
| `--color-surface`  | Card backgrounds, containers | `bg-surface`   |
| `--color-elevated` | Modals, popovers, dropdowns  | `bg-elevated`  |

### Text Hierarchy

| Token                    | Usage                     | Tailwind Class        |
| ------------------------ | ------------------------- | --------------------- |
| `--color-text-primary`   | Headings, body text       | `text-text-primary`   |
| `--color-text-secondary` | Muted text, labels        | `text-text-secondary` |
| `--color-text-tertiary`  | Disabled, hints, captions | `text-text-tertiary`  |

### Accent Colors

| Token                  | Usage                     | Tailwind Class      |
| ---------------------- | ------------------------- | ------------------- |
| `--color-accent-warm`  | Warm highlights, warnings | `text-accent-warm`  |
| `--color-accent-cool`  | Links, interactive states | `text-accent-cool`  |
| `--color-accent-dream` | Purple accent, special    | `text-accent-dream` |

### Typography

| Token            | Usage                  | Tailwind Class |
| ---------------- | ---------------------- | -------------- |
| `--font-heading` | Display text, headings | `font-heading` |
| `--font-data`    | Numbers, code, tabular | `font-data`    |

## Forbidden Patterns (Anti-Slop)

The following patterns are explicitly banned. Their presence indicates "AI slop" and must be rejected:

### Visual Anti-Patterns

- **Gradient Crutch**: `bg-gradient-to-*` used decoratively without semantic purpose
- **Rounded-XL Addiction**: Excessive `rounded-xl`, `rounded-2xl`, `rounded-3xl` on every element
- **Skeleton Aesthetic**: Gray placeholder boxes as permanent design elements
- **Shadow Spam**: Multiple layered shadows (`shadow-lg shadow-xl`) for "depth"
- **Border Radius Inconsistency**: Mixing radius values without system

### Color Anti-Patterns

- **Hardcoded Colors**: `bg-gray-900`, `text-white`, `#1a1a1a`, `rgb(...)`, `hsl(...)`
- **Tailwind Palette Direct**: `bg-blue-500`, `text-slate-400` (use semantic tokens)
- **Opacity Hacks**: `bg-black/50` instead of proper elevation tokens

### Layout Anti-Patterns

- **Flexbox Soup**: Nested flex containers without clear hierarchy
- **Magic Numbers**: `mt-[47px]`, `w-[312px]` (use spacing scale)
- **Container Nesting**: Excessive wrapper divs for spacing

### Typography Anti-Patterns

- **Font Size Chaos**: Arbitrary text sizes without hierarchy
- **Weight Overload**: Bold everything for emphasis
- **Line Height Neglect**: Default line heights on dense text

## Component Guidelines

### Prefer Radix/shadcn Primitives

When available, use Radix UI primitives or shadcn/ui components. These provide:

- Accessibility out of the box
- Keyboard navigation
- Focus management
- ARIA compliance

### Server Components by Default

All components are React Server Components unless they require:

- Event handlers (`onClick`, `onChange`)
- React hooks (`useState`, `useEffect`)
- Browser APIs (`window`, `document`)

Client components must import `client-only` at the top.

## Design Intent Protocol

Before generating any UI component, emit a `<design_intent>` block in the conversation:

```
<design_intent>
Component: [Name]
Purpose: [Single sentence]
Tokens: [List semantic tokens to be used]
Hierarchy: [Visual hierarchy description]
Interactions: [If any]
</design_intent>
```

**CRITICAL:** The `<design_intent>` block is ephemeral. It appears in conversation only and is NEVER committed to the codebase. It is a meta-cognitive tool for ensuring intentional design, not a code artifact.

## Validation Checklist

Before presenting any UI code:

- [ ] All colors use semantic tokens (no hardcoded values)
- [ ] Typography uses `--font-heading` or `--font-data` where appropriate
- [ ] No forbidden patterns present
- [ ] Component has clear visual hierarchy
- [ ] Spacing uses Tailwind scale (no magic numbers)
- [ ] Server/Client boundary is correct
