---
name: modus-frontend
description: Build production-grade frontend interfaces using the Modus 2.0 Design System with Tailwind CSS. Use this skill when the user asks to build web components, pages, dashboards, or application screens. Enforces the 9-color semantic system, Modus Web Components, Modus Icons, and design system compliance. Framework-agnostic (works with React, Angular, or any framework).
---

This skill guides creation of production-grade frontend interfaces using the **Modus 2.0 Design System**. All output must strictly follow the design system constraints below. The result should be polished, professional, enterprise-grade UI that is fully theme-compatible and design-system compliant.

The user provides frontend requirements: a component, page, dashboard, or interface to build. They may include context about the purpose, audience, or technical constraints.

## Design Thinking

Before coding, understand the context:
- **Purpose**: What problem does this interface solve? Who uses it? (field workers, office managers, engineers, administrators)
- **Layout**: What layout pattern fits? (sidebar + content, top nav + grid, full-width dashboard, split pane, card grid)
- **Information hierarchy**: What's most important? Use font size scale, color weight, and spacing to guide the eye.
- **Theme compatibility**: Will this look correct in both light and dark themes? Never hardcode colors.

The Modus design system philosophy is **clean, consistent, and professional**. Express creativity through thoughtful layout composition, smart use of the 9-color system with opacity variants, precise spacing, and clear visual hierarchy - not through custom fonts, wild color palettes, or decorative flourishes.

## 9-Color Semantic System

ALL colors must come from these 9 semantic tokens. No exceptions.

### Base Colors (5)
| Token | Tailwind Class | Purpose |
|-------|---------------|---------|
| `background` | `bg-background` | Page background |
| `foreground` | `text-foreground` | Primary text |
| `card` | `bg-card` | Card and panel backgrounds |
| `muted` | `bg-muted` | Subtle backgrounds, disabled states |
| `secondary` | `bg-secondary` | Secondary UI elements, dividers |

### Status Colors (4)
| Token | Tailwind Class | Purpose |
|-------|---------------|---------|
| `primary` | `bg-primary`, `text-primary` | Primary actions, info states, links |
| `success` | `bg-success`, `text-success` | Success states, confirmations |
| `warning` | `bg-warning`, `text-warning` | Warning states, caution |
| `destructive` | `bg-destructive`, `text-destructive` | Error states, destructive actions |

### Foreground Variants
Each color has a `-foreground` variant for text placed on top of that color:
- `text-primary-foreground` (text on `bg-primary`)
- `text-card-foreground` (text on `bg-card`)
- `text-muted-foreground` (subdued text)
- `text-secondary-foreground` (text on `bg-secondary`)

### Opacity Variants
Each color supports 4 opacity levels: `-80`, `-60`, `-40`, `-20`

```
text-foreground-80    bg-primary-20    border-warning-60
text-muted-foreground-60    bg-destructive-40    bg-success-20
```

**CRITICAL**: Use `text-foreground-80` NOT `text-foreground/80`. The Tailwind `/` opacity syntax does not work with CSS variables.

### Additional Utility Tokens
| Token | Purpose |
|-------|---------|
| `border` | Default border color (`border-default` class) |
| `ring` | Focus ring color |
| `--radius: 8px` | Default border radius |
| `--radius-card: 16px` | Card border radius |
| `--radius-badge: 4px` | Badge border radius |
| `--chart-1` through `--chart-5` | Data visualization colors |

## Typography

**Font**: Open Sans is the only permitted font. It is loaded globally by the design system.

**Font Scale** (Modus scale):
| Class | Size | Use Case |
|-------|------|----------|
| `text-2xs` | 8px | Micro labels |
| `text-xs` | 10px | Captions, fine print |
| `text-sm` | 12px | Secondary text, labels |
| `text-base` | 14px | Body text (default) |
| `text-lg` | 16px | Subheadings, emphasis |
| `text-xl` | 18px | Section titles |
| `text-2xl` | 20px | Page subtitles |
| `text-3xl` | 24px | Page titles |
| `text-4xl` | 30px | Hero headings |

**Font Weights**: `font-normal` (400), `font-medium` (500), `font-semibold` (600), `font-bold` (700)

## Tailwind CSS Usage

Use Tailwind utility classes for layout, spacing, and sizing. Colors MUST come exclusively from the design system.

**Correct usage**:
```
bg-background text-foreground p-4 rounded-lg flex gap-3
bg-card text-card-foreground border-default shadow-sm
bg-primary text-primary-foreground px-4 py-2 rounded-md
text-muted-foreground-60 text-sm
```

**NEVER use**:
```
bg-blue-500      (generic Tailwind color)
text-gray-600    (generic Tailwind color)
bg-[#1a2b3c]    (hardcoded hex)
text-[rgb(...)]  (hardcoded RGB)
bg-slate-100     (any named Tailwind palette)
```

## Border Utilities

Borders use custom utility classes that work correctly with CSS variables. Do NOT combine Tailwind border utilities with design system colors.

### Default Borders
```
border-default           1px solid border color
border-thick             2px solid border color
border-dashed            1px dashed border color
border-thick-dashed      2px dashed border color
```

### Directional Borders
```
border-top-default       border-bottom-default
border-left-default      border-right-default
```

**CRITICAL**: Use `border-bottom-default` NOT `border-b border-default`. The combined pattern breaks with CSS variables.

### Status Borders
Each status color has full border variants:
```
border-primary           border-thick-primary       border-dashed-primary
border-success           border-thick-success       border-dashed-success
border-warning           border-thick-warning       border-dashed-warning
border-destructive       border-thick-destructive   border-dashed-destructive
```

Plus directional variants: `border-top-primary`, `border-bottom-success`, `border-left-warning`, `border-right-destructive`, etc.

## Icons

Use **Modus Icons** exclusively. The library includes 700+ icons in 3 variants.

**Usage**:
```html
<i class="modus-icons">notifications</i>
<i class="modus-icons">settings</i>
<i class="modus-icons">check_circle</i>
<i class="modus-icons">warning_outline</i>
```

**Variants** (via wrapper component or CSS class):
- Regular (default): line-style icons
- Solid: `modus-icons-solid` - filled icons
- Outlined: `modus-icons-outlined` - outlined icons

**Size**: Default 24px. Adjust with Tailwind `text-[size]` on the `<i>` element or wrapper component props.

**NEVER use**: Font Awesome, Material Icons, Heroicons, Lucide, Phosphor, or any other icon library.

## Component Library

Modus Web Components provide all UI primitives. Use the framework's wrapper components (not raw web component tags).

**Available components include**:
- **Layout**: Card, Accordion, Tabs, Modal, Side Navigation, Navbar, Utility Panel
- **Forms**: Button, Checkbox, Radio, Switch, Text Input, Number Input, Select/Dropdown Menu, Date Picker, Time Picker, File Upload
- **Data**: Table, Data Table, Pagination, Badge, Chip, Progress Bar, Spinner
- **Feedback**: Alert, Toast, Tooltip, Message
- **Navigation**: Breadcrumbs, Menu, Side Navigation, Tabs
- **Media**: Autocomplete, Slider, Sentiment Scale

**Key patterns**:
- Use `ModusDropdownMenu` instead of `ModusSelect` for reliable event handling
- Let Modus components manage their own internal state (e.g., accordion expand/collapse)
- Use slot-based composition for cards (`header`, default content, `footer` slots)
- Modals use dialog-based patterns (`showModal()`/`close()`)

**Checkbox value bug**: The `value` property from checkbox `inputChange` events is inverted. Always invert:
```
const actualChecked = !event.detail.value;
```

## Theme Support

The design system supports 6 themes via the `data-theme` attribute on `<html>`:

| Theme | Mode |
|-------|------|
| `modus-classic-light` | Light |
| `modus-classic-dark` | Dark |
| `modus-modern-light` | Light |
| `modus-modern-dark` | Dark |
| `connect-light` | Light |
| `connect-dark` | Dark |

All semantic color variables automatically update when the theme changes. This means:
- **Never hardcode colors** - always use the semantic tokens
- **Test mental model** in both light and dark: `bg-card` is light in light themes, dark in dark themes
- **Use opacity variants** for subtle layering that works across themes (`bg-primary-20`, `text-foreground-60`)

## HTML Elements

Use `<div>` elements for all layout and content containers. Avoid semantic HTML elements (`h1`, `h2`, `p`, `section`, `article`, `span`, `header`, `footer`, `nav`, `main`) as they can conflict with Tailwind CSS and Modus web component styling.

**Exception**: `<i>` elements are used for Modus Icons.

Apply text styling via Tailwind classes on `<div>` elements:
```html
<div class="text-3xl font-bold text-foreground">Page Title</div>
<div class="text-base text-muted-foreground">Body text content</div>
<div class="text-sm text-foreground-60">Secondary information</div>
```

## Animations

Use CSS animations sparingly for professional transitions:

```css
/* Fade in */
@keyframes fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}

/* Slide in from right */
@keyframes slide-in-right {
  from { transform: translateX(100%); }
  to { transform: translateX(0); }
}
```

Utility classes: `animate-fade-in`, `animate-slide-in-right`

Tailwind transition utilities work well: `transition-all duration-200 ease-out`

Focus on functional transitions (panel open/close, content loading) rather than decorative motion.

## Strict Prohibitions

- No emojis in code or UI text
- No generic Tailwind colors (`blue-500`, `gray-300`, `slate-100`, etc.)
- No hardcoded hex, RGB, HSL, or named CSS colors
- No external icon libraries (Font Awesome, Material Icons, Heroicons, etc.)
- No custom fonts or Google Fonts imports (Open Sans is provided by the design system)
- No `border-b border-default` pattern (use `border-bottom-default`)
- No `text-foreground/80` opacity syntax (use `text-foreground-80`)
- No semantic HTML elements for layout (`h1`, `p`, `section`, `span`, etc.)
- No inline styles with hardcoded color values
- No direct usage of internal `--modus-wc-color-*` variables (use the semantic tokens)

## Design Quality

Within these constraints, produce polished, professional interfaces:

- **Visual hierarchy**: Use font scale (`text-3xl` to `text-xs`), font weight, and color opacity to establish clear information hierarchy
- **Spacing**: Use consistent Tailwind spacing (`gap-2`, `gap-4`, `gap-6`, `p-4`, `p-6`) for rhythm. Be generous with whitespace.
- **Card composition**: Use `bg-card` with `border-default` and `rounded-[--radius-card]` for content grouping
- **Status communication**: Use the 4 status colors purposefully - `primary` for info/action, `success` for positive, `warning` for caution, `destructive` for errors
- **Layering with opacity**: `bg-primary-20` for subtle highlights, `text-foreground-60` for secondary text, `bg-muted` for section backgrounds
- **Data visualization**: Use `--chart-1` through `--chart-5` for charts and graphs
- **Responsive layout**: Use Tailwind responsive prefixes (`sm:`, `md:`, `lg:`) and flexbox/grid utilities
- **Focus states**: Use `ring` color for focus indicators

Remember: Professional design within a constrained system comes from precision - perfect alignment, consistent spacing, purposeful color application, and clear visual hierarchy. The 9-color system with opacity variants provides more than enough range for expressive, polished UI.
