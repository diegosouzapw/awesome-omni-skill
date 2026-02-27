---
name: frontend-shadcn
description: Frontend development using Vite + React + shadcn/ui + Tailwind CSS + React Router v7. Use when creating new frontend projects, adding UI components, implementing routing, styling with Tailwind, or working with shadcn/ui component library.
---

# Frontend ShadCN Stack

Modern React frontend stack:

- **Vite** - Build tooling
- **React 19** - UI framework
- **TypeScript** - Type safety
- **Tailwind CSS v4** - Utility-first styling (`@tailwindcss/vite` plugin)
- **shadcn/ui** - Component library (New York style)
- **React Router v7** - Client-side routing

## Environment Setup

Use [asdf](https://asdf-vm.com/) to manage Node.js versions:

```bash
# Install Node.js plugin (one-time)
asdf plugin add nodejs

# Set project Node.js version
asdf set nodejs latest:22
```

This creates a `.tool-versions` file in the project root that ensures consistent Node.js versions across the team.

## Reference Files

| File | When to Use |
|------|-------------|
| [setup-guide.md](references/setup-guide.md) | Starting a new project from scratch |
| [patterns.md](references/patterns.md) | Implementing features, understanding architecture |
| [maplibre.md](references/maplibre.md) | Working with MapLibre GL JS maps |
| [mcp-tools.md](references/mcp-tools.md) | Looking up docs, adding components via MCP |

## Quick Reference

### Commands

```bash
# Add shadcn component
npx shadcn@latest add <component> -y

# Dev server
npm run dev

# Type check
npm run type-check
```

### Key Imports

```typescript
// React Router v7 - use 'react-router' NOT 'react-router-dom'
import { Routes, Route, Link, useLocation, useSearchParams } from 'react-router'

// Path alias - @/ maps to src/
import { Button } from '@/components/ui/button'
import { cn } from '@/lib/utils'
```

### Conditional Classes

```typescript
import { cn } from '@/lib/utils'

<div className={cn(
  "base-classes",
  condition && "conditional-classes",
  variant === "primary" && "variant-classes"
)} />
```

### Common Components

```bash
# Layout
npx shadcn@latest add sidebar card separator -y

# Forms
npx shadcn@latest add button input form select checkbox -y

# Feedback
npx shadcn@latest add dialog alert toast -y

# Navigation
npx shadcn@latest add dropdown-menu tabs tooltip -y
```

### Project Structure

```
src/
├── components/
│   ├── ui/           # shadcn/ui components (auto-generated)
│   ├── AppShell.tsx  # Main layout with header
│   └── AppSidebar.tsx
├── pages/            # Route page components
├── hooks/            # Custom hooks
├── lib/
│   └── utils.ts      # cn() helper
├── App.tsx           # Route definitions
├── main.tsx          # Entry point with BrowserRouter
└── index.css         # Tailwind + shadcn theme
```

### Tailwind Patterns

```typescript
// Common utility patterns
"flex items-center gap-4"           // Flexbox with gap
"bg-muted text-muted-foreground"    // Muted backgrounds
"border-b bg-background"            // Borders and backgrounds
"h-screen overflow-auto"            // Full height scrolling
"space-y-4"                         // Vertical spacing
```

### Common Gotchas

- **Package management**: Use `npm install <pkg>` not manual `package.json` edits
- **shadcn components**: Use `npx shadcn@latest add <component> -y`
- **React Router imports**: Use `react-router` NOT `react-router-dom`
