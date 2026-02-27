---
name: frontend-design
description: Visual design for Guitar Tone Shootout - design tokens, colors, typography, component patterns, and aesthetic principles. Use for UI decisions and styling.
---

# Frontend Design

**Activation:** design, colors, theme, styling, tokens, UI, visual, dark mode, component style

**Full Reference:** [Wiki: Design/Style-Guide](https://github.com/krazyuniks/guitar-tone-shootout/wiki/Design-Style-Guide)

---

## Design Direction

- **Aesthetic:** Professional audio tool (Quad Cortex / DAW inspired)
- **Theme:** Dark primary
- **Target:** Musicians, audio engineers, content creators
- **Font:** Inter (project standard)

---

## Unified Design Token Architecture

Design tokens are defined **once** in `astro/src/styles/global.css`. The Astro build compiles this CSS and generates a Jinja2 wrapper template that includes the compiled stylesheet.

**Single source of truth:**
```
astro/src/styles/global.css (define tokens here)
    |
    v (Astro build)
astro/dist/_astro/main.[hash].css (compiled CSS)
    |
    v (included by wrapper)
Both static (Astro) and dynamic (Jinja2) pages
```

**All pages share the same compiled CSS.** Static Astro pages and dynamic Jinja2 templates both use identical design tokens through the same stylesheet.

### Usage in Templates

**Jinja2 templates:**
```jinja2
<div class="bg-bg-elevated text-text-primary rounded-lg p-4">
  <h3 class="text-accent-primary font-semibold">{{ item.name }}</h3>
</div>
```

**Astro/React components:**
```tsx
<div className="bg-bg-elevated text-text-primary rounded-lg p-4">
  <h3 className="text-accent-primary font-semibold">{item.name}</h3>
</div>
```

**CSS custom property syntax (when needed):**
```html
<div style="background-color: var(--color-bg-elevated);">
```

---

## Design Tokens

### Background Layers

Three-tier surface system for depth and hierarchy:

| Token | Value | Use |
|-------|-------|-----|
| `--color-bg-base` | `#0a0a0a` | Page background |
| `--color-bg-surface` | `#141414` | Cards, panels |
| `--color-bg-elevated` | `#1f1f1f` | Modals, dropdowns, hover states |

**Tailwind classes:** `bg-bg-base`, `bg-bg-surface`, `bg-bg-elevated`

### Text Colors

| Token | Value | Use |
|-------|-------|-----|
| `--color-text-primary` | `#ffffff` | Primary text, headings |
| `--color-text-secondary` | `#a1a1a1` | Secondary text, labels |
| `--color-text-muted` | `#666666` | Disabled, placeholder text |

**Tailwind classes:** `text-text-primary`, `text-text-secondary`, `text-text-muted`

### Accent Colors

| Token | Value | Use |
|-------|-------|-----|
| `--color-accent-primary` | `#3b82f6` | Blue - CTAs, links |
| `--color-accent-success` | `#22c55e` | Green - success states |
| `--color-accent-warning` | `#f59e0b` | Amber - warnings |
| `--color-accent-error` | `#ef4444` | Red - errors, destructive |

**Tailwind classes:** `text-accent-primary`, `bg-accent-success`, `border-accent-warning`, etc.

### Block Type Colors

Signal chain component identification:

| Token | Value | Use |
|-------|-------|-----|
| `--color-block-di` | `#3b82f6` | Blue - DI/Input blocks |
| `--color-block-amp` | `#f59e0b` | Amber - Amp/NAM blocks |
| `--color-block-cab` | `#22c55e` | Green - Cabinet/IR blocks |
| `--color-block-effect` | `#a855f7` | Purple - Pre-amp pedals |
| `--color-block-post-effect` | `#06b6d4` | Cyan - Post-amp effects |

**Tailwind classes:** `bg-block-di`, `border-block-amp`, `text-block-effect`, etc.

### Typography

| Use | Class | Spec |
|-----|-------|------|
| Body | `text-base font-normal` | 16px, 400 |
| Heading | `text-2xl font-semibold` | 24px, 600 |
| Caption | `text-xs font-medium` | 12px, 500 |
| Mono | `font-mono` | JetBrains Mono |

**Font families:**
- `--font-sans`: Inter
- `--font-mono`: JetBrains Mono

---

## Component Patterns

**Card:**
```html
<div class="bg-bg-surface border border-[#333333] rounded-lg p-4
            hover:bg-bg-elevated transition-colors">
```

**Button:**
```html
<button class="px-4 py-2 bg-accent-primary text-white rounded-md font-medium
               hover:bg-blue-700 disabled:opacity-50">
```

**Block Card (signal chain blocks):**
```tsx
const blockStyles = {
  di:          { border: 'border-block-di/50',          bg: 'bg-block-di/10' },
  amp:         { border: 'border-block-amp/50',         bg: 'bg-block-amp/10' },
  cab:         { border: 'border-block-cab/50',         bg: 'bg-block-cab/10' },
  effect:      { border: 'border-block-effect/50',      bg: 'bg-block-effect/10' },
  postEffect:  { border: 'border-block-post-effect/50', bg: 'bg-block-post-effect/10' },
};
```

---

## Aesthetic Principles

### Do

- **Intentional choices** - Bold maximalism and refined minimalism both work
- **Dominant colors with sharp accents** - Not timid, evenly-distributed palettes
- **Atmosphere and depth** - Gradients, textures, layered transparencies
- **Purposeful motion** - High-impact moments over scattered micro-interactions

### Avoid ("AI Slop")

- Overused fonts without character
- Cliched color schemes (purple gradients on white)
- Predictable layouts and component patterns
- Cookie-cutter design lacking context-specific character

---

## Full Documentation

- [Style Guide](https://github.com/krazyuniks/guitar-tone-shootout/wiki/Design-Style-Guide) - All tokens, accessibility
- [Tools & Workflow](https://github.com/krazyuniks/guitar-tone-shootout/wiki/Design-Tools-and-Workflow) - Showcase, MCP usage
- [Inspiration](https://github.com/krazyuniks/guitar-tone-shootout/wiki/Design-Inspiration) - Reference applications
