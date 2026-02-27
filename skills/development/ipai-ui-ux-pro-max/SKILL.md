---
name: ipai-ui-ux-pro-max
description: "Design intelligence for UI/UX generation and critique; use for layouts, typography, color palettes, tokens, and UX best practices. 50 styles, 21 palettes, 50 font pairings, 20 charts, 9 stacks (React, Next.js, Vue, Svelte, SwiftUI, React Native, Flutter, Tailwind, shadcn/ui). Actions: plan, build, create, design, implement, review, fix, improve, optimize, enhance, refactor, check UI/UX code. Projects: website, landing page, dashboard, admin panel, e-commerce, SaaS, portfolio, blog, mobile app, .html, .tsx, .vue, .svelte. Elements: button, modal, navbar, sidebar, card, table, form, chart. Styles: glassmorphism, claymorphism, minimalism, brutalism, neumorphism, bento grid, dark mode, responsive, skeuomorphism, flat design. Topics: color palette, accessibility, animation, layout, typography, font pairing, spacing, hover, shadow, gradient. Integrations: shadcn/ui MCP for component search and examples."
version: "1.0.0"
tags: [design, ui, ux, tokens, accessibility, typography, color]
---

# UI/UX Pro Max - Design Intelligence

Comprehensive design guide for web and mobile applications. Contains 50+ styles, 97 color palettes, 57 font pairings, 99 UX guidelines, and 25 chart types across 9 technology stacks. Searchable database with priority-based recommendations.

## When to Use This Skill

- Designing new UI (landing pages, dashboards, mobile screens)
- Reviewing UI for UX issues (information architecture, accessibility, copy, hierarchy)
- Choosing color palettes and typography for a project
- Producing token-aligned UI specs (spacing, type scale, color roles)
- Building production-quality components

## How to Use (Prompt Patterns)

### 1. Design Mode
- **Input:** product goal + audience + platform + constraints
- **Output:** layout plan + components + tokens + interaction notes

### 2. Critique Mode
- **Input:** screenshot / HTML / Figma node list + context
- **Output:** prioritized issues + fixes + token-safe suggestions

### 3. Implementation Mode
- **Input:** stack (Next.js/React/Tailwind/OWL/etc.) + design direction
- **Output:** component blueprint + styling guidance

## Design System Generation (REQUIRED first step)

Always start with `--design-system` to get comprehensive recommendations:

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "<product_type> <industry> <keywords>" --design-system [-p "Project Name"]
```

Persist for cross-session use:
```bash
python3 skills/ui-ux-pro-max/scripts/search.py "<query>" --design-system --persist -p "Project Name"
```

### Domain Searches (supplement as needed)

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "<keyword>" --domain <domain> [-n <max_results>]
```

| Domain | Use For | Example |
|--------|---------|---------|
| `product` | Product type recommendations | SaaS, e-commerce, portfolio |
| `style` | UI styles, effects | glassmorphism, minimalism |
| `typography` | Font pairings | elegant, playful, modern |
| `color` | Color palettes | saas, healthcare, fintech |
| `landing` | Page structure, CTAs | hero, pricing, social-proof |
| `chart` | Chart types | trend, comparison, funnel |
| `ux` | Best practices | animation, accessibility |
| `react` | React/Next.js perf | waterfall, bundle, suspense |
| `web` | Web interface | aria, focus, keyboard |

### Stack Guidelines

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "<keyword>" --stack html-tailwind
```

Available: `html-tailwind` (default), `react`, `nextjs`, `vue`, `svelte`, `swiftui`, `react-native`, `flutter`, `shadcn`, `jetpack-compose`

## References (Progressive Disclosure)

See `references/` for detailed catalogs and rulebooks:

- `references/01-styles.md` — visual styles catalog + selection heuristic
- `references/02-color-palettes.md` — palette system, token roles, contrast rules
- `references/03-typography.md` — font pairings, type scale, line-height guidance
- `references/04-layout-spacing.md` — grids, spacing scale, responsive rules
- `references/05-components-patterns.md` — component heuristics + interaction patterns
- `references/06-charts-dataviz.md` — chart selection guide + dashboard rules
- `references/07-ux-guidelines.md` — UX best practices + anti-patterns
- `references/08-accessibility.md` — a11y checklist + WCAG compliance

## Output Contract

- Prefer **tokens/roles** over hard-coded hex colors
- Provide **priority-ranked** recommendations (P0/P1/P2)
- Use SVG icons (Heroicons, Lucide) — never emojis
- Avoid novelty; optimize for clarity and conversion
- Test responsive at 375px, 768px, 1024px, 1440px
