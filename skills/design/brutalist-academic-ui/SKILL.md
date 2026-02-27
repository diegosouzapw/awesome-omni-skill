---
name: brutalist-academic-ui
description: Skriptoteket-specific brutalist/academic UI design. Use for Vue/Vite SPA and SSR templates when you want grid-based layouts, systematic typography, and high-contrast “academic” aesthetics, while staying compatible with Skriptoteket’s pure-CSS + HuleEdu token stack (no Tailwind).
---

# Brutalist Academic UI

Opinionated design skill for interfaces where intellectual rigor, typographic precision, and structural honesty matter.

## Skriptoteket Compatibility (must follow)

- **No Tailwind / utility CSS frameworks** (see `docs/adr/adr-0029-frontend-styling-pure-css-design-tokens.md`).
- **Use HuleEdu tokens + component classes** (see `.agent/rules/045-huleedu-design-system.md` and `src/skriptoteket/web/static/css/huleedu-design-tokens.css`).
- Prefer existing classes before inventing new ones: `.huleedu-btn`, `.huleedu-card`, `.huleedu-link`, `.huleedu-table`, `.huleedu-row`.

## When to Use

Activate when user:
- Builds websites, landing pages, dashboards, documentation sites
- Needs institutional or academic visual language
- Mentions: "brutalist", "academic", "minimal", "no gradients", "serious UI"
- Wants to avoid AI-generated aesthetic (purple gradients, Roboto, pill buttons)

## I Need To...

| Task | Read |
|------|------|
| Align with Skriptoteket tokens + button hierarchy | `.agent/rules/045-huleedu-design-system.md` |
| Understand grid/typography/color principles | [fundamentals.md](fundamentals.md) |
| Build page structure, layouts, navigation | [patterns.md](patterns.md) |
| Create data tables, ledgers, state rows | [examples/tables-ledgers.md](examples/tables-ledgers.md) |
| Build buttons, cards, interactions | [examples/components.md](examples/components.md) |

## Quick Reference

### Banned

- Purple/startup gradients
- Roboto, Open Sans, Lato, Inter
- Tailwind / utility-first styling that bypasses tokens
- `border-radius` > `var(--huleedu-radius-lg)` (8px) for containers
- Decorative blobs, floating shapes
- Soft shadows, backdrop blur
- Scale/bounce hover animations

### Font Stack

```css
/* Use HuleEdu token fonts (defined in src/skriptoteket/web/static/css/huleedu-design-tokens.css). */
font-family: var(--huleedu-font-serif); /* long-form reading */
font-family: var(--huleedu-font-sans);  /* UI */
font-family: var(--huleedu-font-mono);  /* code/ids */
```

### Spacing Scale

```css
/* Representative spacing tokens (4px base scale). */
--huleedu-space-1: 4px;   --huleedu-space-2: 8px;   --huleedu-space-3: 12px;  --huleedu-space-4: 16px;
--huleedu-space-6: 24px;  --huleedu-space-8: 32px;  --huleedu-space-12: 48px; --huleedu-space-16: 64px;
```

### Color Palette

```css
/* Use HuleEdu tokens for all color decisions (no hardcoded hex). */
--huleedu-navy: #1C2E4A;      /* ink */
--huleedu-canvas: #F9F8F2;    /* paper */
--huleedu-burgundy: #4D1521;  /* accent / CTA / attention */
--huleedu-warning: #D97706;
--huleedu-error: #DC2626;
```

## Core Principle

The best interface is one where you notice the content, not the interface. Every element earns its place. Typography does the work. Whitespace is structure.
