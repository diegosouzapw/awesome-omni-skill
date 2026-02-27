---
name: frontend-design
description: Frontend design system for the Orient. Use this skill when doing ANY UI work including creating components, building pages, adding forms, styling elements, or working with buttons, cards, modals, inputs, tables, or layouts. Triggers on "create component", "build page", "add form", "button", "card", "modal", "style", "layout", "UI", "frontend", "display data", "show results", "dashboard", or any visual/interface development task.
---

# Frontend Design System

This project uses a **developer-grade** design aesthetic inspired by Vercel, Linear, and shadcn/ui.

## Core Principles

1. **Black & White Dominant** - Color only for semantic status (success/error/warning)
2. **High Information Density** - Clean but compact
3. **Border-Based Hierarchy** - Avoid heavy shadows
4. **Monospace for Data** - IDs, dates, timestamps, code always in `font-mono`

## Tech Stack

- **React + Vite** for apps
- **Tailwind CSS v3.4+** for styling
- **CSS Variables** with semantic classes (`bg-background` not `bg-white`)
- **Dark Mode First** - Class-based via `.dark` on `<html>`

## Semantic Color Tokens

**NEVER use hardcoded colors.** Use semantic tokens:

| Token                     | Purpose                |
| ------------------------- | ---------------------- |
| `bg-background`           | Page background        |
| `text-foreground`         | Primary text           |
| `bg-card`                 | Cards/panels           |
| `border-border`           | All borders            |
| `bg-muted`                | Secondary backgrounds  |
| `text-muted-foreground`   | Secondary text, labels |
| `bg-primary`              | Primary actions        |
| `text-primary-foreground` | Text on primary        |

## Typography

- **UI Text**: `font-sans` (Inter)
- **Data/Code**: `font-mono` (JetBrains Mono) - MANDATORY for:
  - IDs, UUIDs, hashes
  - Dates and timestamps
  - Status codes
  - Logs/console output
  - Metric values

## Component Patterns

### Buttons

```tsx
// Primary
<button className="h-9 px-4 rounded-md bg-primary text-primary-foreground text-sm font-medium hover:bg-primary/90">
  Action
</button>

// Secondary
<button className="h-9 px-4 rounded-md bg-secondary text-secondary-foreground text-sm font-medium hover:bg-secondary/80">
  Cancel
</button>

// Ghost (for icons/menus)
<button className="h-9 px-3 rounded-md hover:bg-accent hover:text-accent-foreground">
  <Icon />
</button>
```

### Cards

```tsx
<div className="rounded-xl border border-border bg-card text-card-foreground shadow-sm">
  <div className="p-6 pb-2">
    <h3 className="font-semibold text-foreground">Title</h3>
    <span className="font-mono text-xs text-muted-foreground">ID: abc123</span>
  </div>
  <div className="p-6 pt-0">{/* Content */}</div>
</div>
```

### Inputs

```tsx
<input
  className="h-9 w-full rounded-md border border-input bg-transparent px-3 py-1 text-sm shadow-sm focus-visible:ring-1 focus-visible:ring-ring"
  placeholder="Enter value..."
/>
```

### Tables

```tsx
<div className="rounded-md border border-border">
  <table className="w-full">
    <thead>
      <tr className="border-b border-border bg-muted/50">
        <th className="p-4 text-left text-xs uppercase font-medium text-muted-foreground">
          Column
        </th>
      </tr>
    </thead>
    <tbody>
      <tr className="border-b border-border hover:bg-muted/50">
        <td className="p-4 align-middle font-mono text-sm">value</td>
      </tr>
    </tbody>
  </table>
</div>
```

## Quick Reference

| Element       | Classes                                             |
| ------------- | --------------------------------------------------- |
| Page bg       | `bg-background`                                     |
| Card          | `rounded-xl border border-border bg-card shadow-sm` |
| Button height | `h-9`                                               |
| Border radius | `rounded-md` (6px)                                  |
| Text sizes    | `text-sm` (body), `text-xs` (labels)                |
| Monospace     | `font-mono`                                         |
| Muted text    | `text-muted-foreground`                             |

## Do's and Don'ts

**DO:**

- Use `border-border` for all dividing lines
- Use `text-muted-foreground` for metadata/labels
- Use `font-mono` for any data values
- Ensure sufficient contrast

**DON'T:**

- Use shadows to define hierarchy (use borders)
- Use `rounded-full` for buttons (except icon-only)
- Mix serif fonts
- Use hardcoded colors like `bg-white` or `bg-gray-100`

## Reference Files

For detailed information:

- **Full design system**: See [references/design-system.md](references/design-system.md)
- **Component examples**: See [references/components.md](references/components.md)
