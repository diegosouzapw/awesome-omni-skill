---
name: refactor-html
description: Refactor HTML/TSX files to use existing UI components, DaisyUI classes, and semantic colors. Use when (1) refactoring React/TSX page components to use reusable UI components, (2) replacing raw HTML elements with component library equivalents, (3) converting primitive Tailwind colors to semantic DaisyUI colors, (4) extracting repeated styling patterns into components.
---

# Refactor HTML/TSX

Refactor page components to follow project conventions: use existing UI components, DaisyUI classes, and semantic colors.

## Rules

### 1. Use Existing Components

Replace raw HTML with UI components from `packages/ui/src/components/ui/`. See [components.md](references/components.md) for available components and their props.

| Before | After |
|--------|-------|
| `<div className="alert alert-error">` | `<Alert variant="error">` |
| `<button className="btn btn-primary">` | `<Button variant="primary">` |
| `<input className="input input-bordered">` | `<Input />` |
| `<select className="select select-bordered">` | `<Select />` |
| `<fieldset className="fieldset">` | `<Fieldset>` |
| `<span className="loading loading-spinner">` | `<Loading />` |

### 2. Use Semantic Colors

Replace primitive colors with DaisyUI semantic colors:

| Primitive (NG) | Semantic (OK) |
|----------------|---------------|
| `text-red-*` | `text-error` |
| `text-blue-*` | `text-primary` |
| `text-green-*` | `text-success` |
| `bg-gray-100` | `bg-base-100` |
| `border-gray-*` | `border-base-300` |

### 3. Extract Repeated Styles

When styling patterns repeat (e.g., `border-t border-base-300 pt-6`), consider extracting to a component.

## Workflow

1. Read target file
2. Check available components in `packages/ui/src/components/ui/`
3. Identify rule violations
4. Apply refactoring with Edit tool
5. Add necessary imports
6. Report changes made
