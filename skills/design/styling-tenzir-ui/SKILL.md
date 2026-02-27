---
name: styling-tenzir-ui
description: Provides Tenzir design system tokens and component specifications. Use when building UI components, styling with CSS/Tailwind, choosing colors, typography, spacing, or implementing buttons, inputs, tags/badges, toasts, and other Tenzir UI elements.
---

# Styling Tenzir UI

This skill provides Tenzir's brand and style guidelines for frontend development.

## Contents

- [Design System Overview](#design-system-overview)
- [Quick Reference](#quick-reference)
- [When to Load References](#when-to-load-references)

## Design System Overview

Tenzir's design system provides consistent styling across all Tenzir products.

When implementing frontend components, always reference the appropriate section:

| Aspect         | Reference                                      | Description                                    |
| -------------- | ---------------------------------------------- | ---------------------------------------------- |
| Typography     | [typography.md](./typography.md)               | Font families, sizes, weights, line heights    |
| Colors         | [colors.md](./colors.md)                       | Brand colors, semantic colors, neutrals        |
| Spacing        | [spacing.md](./spacing.md)                     | Padding and margin scale                       |
| Shadows        | [shadows.md](./shadows.md)                     | Elevation and shadow styles                    |
| Border Radius  | [border-radius.md](./border-radius.md)         | Corner radius tokens (always 5px)              |
| Buttons        | [buttons.md](./buttons.md)                     | All button styles (standard, delete, floating) |
| Dropdown       | [dropdown.md](./dropdown.md)                   | Dropdown trigger with chevron                  |
| Hyperlinks     | [hyperlinks.md](./hyperlinks.md)               | Link styling with underline                    |
| Segmented Ctrl | [segmented-control.md](./segmented-control.md) | Toggle between mutually exclusive options      |
| Input Field    | [input-field.md](./input-field.md)             | Text input with title and states               |
| Search Input   | [search-input.md](./search-input.md)           | Search field with icon and clear               |
| Checkbox       | [checkbox.md](./checkbox.md)                   | Square multi-select control                    |
| Radio Button   | [radio-button.md](./radio-button.md)           | Circular single-select control                 |
| Toggle Switch  | [toggle-switch.md](./toggle-switch.md)         | Binary on/off switch                           |
| Tag            | [tag.md](./tag.md)                             | Colored labels for categorization              |
| Badge          | [badge.md](./badge.md)                         | Small uppercase status indicators              |
| Tab Bar        | [tab-bar.md](./tab-bar.md)                     | Horizontal navigation tabs                     |
| Toast          | [toast.md](./toast.md)                         | Transient notification messages                |

## Quick Reference

### Fonts

- **Inter** - Corporate font for UI text
- **JetBrains Mono** - Monospace font for code

### Usage Guidelines

1. **Always use design tokens** - Never hardcode pixel values; use the token names (e.g., `text-sm`, `text-base`)
2. **Respect the type scale** - Use the defined sizes; don't create custom sizes
3. **Match weight to purpose** - Use semi-bold for headings, medium for emphasis, regular for body text
4. **Apply letter-spacing** - Larger text sizes (2xl+) require negative letter-spacing

### Spacing

- **Scale:** 0, 0.5, 1, 2, 3, 4, 5, 6, 7, 8, 10, 16, 20
- **Base unit:** 4px (1 = 0.25rem = 4px)
- **Common:** p-2 (8px) for inputs, p-4 (16px) for containers, gap-4 (16px) for sections

### Colors

- **Primary:** Blue (`#0A54FF`) and Green (`#29E06C`)
- **Neutrals:** 11-shade grey scale from `neutral-800` (black) to `neutral-50` (white)
- **Semantic:** Success (green), Warning (yellow), Error (red)
- **Graph colors:** Blue, Lightblue, Purple, Pink, Orange, Yellow (in order)

### Shadows

- **shadow-l** - Sidepanels and modals (highest elevation)
- **shadow-m** - Popups
- **shadow-s** - Tooltips and toasts
- **shadow-xs** - Subtle elevation (lowest)

### Buttons

- **Primary** - Solid blue (`blue-500`), main actions
- **Secondary** - Outlined (`neutral-250` border), alternative actions
- **Tertiary** - Text-only, low-emphasis actions
- **Sizes:** XS (24px), S (28px), M (32px), L (36px)

### Inputs

- **Input Field** - M (32px), L (36px) heights; neutral-100 background
- **Search** - 32px height with search icon and clear button
- **Checkbox** - 16px square, 5px radius
- **Radio Button** - 16px circle
- **Toggle Switch** - 32x20px with sliding knob
- **Focus ring** - 3px primary-200 for inputs, 2px for controls

### Tags & Badges

- **Tag** - 24px height, 9 colors, add/remove variants
- **Badge** - 16px height, 8px uppercase text, 9 colors (same palette as tags)

### Navigation

- **Tab Bar** - 48px height, 2px active indicator, notification counter support

### Feedback

- **Toast** - 48px min-height, shadow-s, optional icon/subtitle/button/progress bar

## CSS Custom Properties Naming Convention

When implementing the design system in CSS, use the `--tnz-` prefix for all custom properties:

```css
:root {
  /* Typography */
  --tnz-font-sans: "Inter Variable", "Inter", system-ui, sans-serif;
  --tnz-font-mono: "JetBrains Mono Variable", "JetBrains Mono", monospace;

  /* Font Sizes */
  --tnz-text-xxs: 0.625rem; /* 10px */
  --tnz-text-xs: 0.75rem; /* 12px */
  --tnz-text-sm: 0.875rem; /* 14px */
  --tnz-text-base: 1rem; /* 16px */
  --tnz-text-lg: 1.125rem; /* 18px */
  --tnz-text-xl: 1.25rem; /* 20px */
  --tnz-text-2xl: 1.5rem; /* 24px */
  --tnz-text-3xl: 1.875rem; /* 30px */
  --tnz-text-4xl: 2.25rem; /* 36px */
  --tnz-text-5xl: 3rem; /* 48px */

  /* Border Radius - 5px is the standard */
  --tnz-radius: 5px;

  /* Spacing Scale (base unit: 4px) */
  --tnz-space-0: 0;
  --tnz-space-0-5: 0.125rem; /* 2px */
  --tnz-space-1: 0.25rem; /* 4px */
  --tnz-space-1-5: 0.375rem; /* 6px */
  --tnz-space-2: 0.5rem; /* 8px */
  --tnz-space-3: 0.75rem; /* 12px */
  --tnz-space-4: 1rem; /* 16px */
  --tnz-space-5: 1.25rem; /* 20px */
  --tnz-space-6: 1.5rem; /* 24px */
  --tnz-space-7: 1.75rem; /* 28px */
  --tnz-space-8: 2rem; /* 32px */
  --tnz-space-10: 2.5rem; /* 40px */
  --tnz-space-16: 4rem; /* 64px */
  --tnz-space-20: 5rem; /* 80px */

  /* Shadows */
  --tnz-shadow-xs: 0px 8px 16px -8px #0e10171a, 0px 3px 6px -3px #0e10171a;
  --tnz-shadow-s: 0px 8px 16px -8px #0e101733, 0px 3px 6px -3px #0e101733;
  --tnz-shadow-m: 0px 10px 20px -8px #0e101733, 0px 4px 8px -6px #0e101733;
  --tnz-shadow-l: 0px 20px 40px -16px #0e101733, 0px 8px 16px -8px #0e101733;

  /* Transitions */
  --tnz-transition-fast: 0.15s ease;
  --tnz-transition-base: 0.2s ease;
  --tnz-transition-slow: 0.3s ease;

  /* Colors - see colors.md for full palette */
  /* Primary Blue */
  --tnz-primary-500: #0a54ff;
  --tnz-primary-600: #0043e0;
  /* ... etc */

  /* Neutrals */
  --tnz-neutral-50: #fdfdfe;
  --tnz-neutral-100: #f7f8fa;
  --tnz-neutral-200: #f0f1f5;
  /* ... etc */
}
```

**Important:** Always use design tokens via CSS custom properties. Never hardcode values like `font-family: 'Inter'` or `border-radius: 5px` - use `var(--tnz-font-sans)` and `var(--tnz-radius)` instead.

## When to Load References

Load the specific reference file when you need detailed specifications:

- Working on typography? Read [typography.md](./typography.md)
- Implementing colors? Read [colors.md](./colors.md)
- Adding shadows/elevation? Read [shadows.md](./shadows.md)
- Building buttons? Read [buttons.md](./buttons.md)
- Adding spacing? Read [spacing.md](./spacing.md)
