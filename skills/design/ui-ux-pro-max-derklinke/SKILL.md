---
name: ui-ux-pro-max
description: Use when designing/reviewing UI or UX for web/mobile products, selecting style/color/typography/layout/animation/chart patterns, or generating stack-specific implementation guidance (React, Next.js, Vue, Svelte, SwiftUI, React Native, Flutter, Tailwind, shadcn/ui).
---

# UI/UX Pro Max - Design Intelligence

Design rules + searchable CLI for style, UX, color, typography, charts, landing patterns, and stack guidance.

## When to Apply

- New UI/page/component design
- UI/UX review and fixes
- Design system setup for a project
- Stack-specific implementation guidance
- Accessibility/performance sanity checks

## Priority Order

| Priority | Category | Impact | Domain |
|---|---|---|---|
| 1 | Accessibility | Critical | `ux` |
| 2 | Touch/Interaction | Critical | `ux` |
| 3 | Performance | High | `ux` |
| 4 | Layout/Responsive | High | `ux` |
| 5 | Typography/Color | Medium | `typography`, `color` |
| 6 | Animation | Medium | `ux` |
| 7 | Style Selection | Medium | `style`, `product` |
| 8 | Charts/Data | Low | `chart` |

## Fast Rule Set

### Accessibility (critical)
- `color-contrast`: >= 4.5:1 for normal text
- `focus-states`: visible keyboard focus
- `alt-text`: meaningful images only
- `aria-labels`: icon-only controls must have labels
- `keyboard-nav`: tab order matches visual order
- `form-labels`: explicit labels, not placeholder-only

### Touch/interaction (critical)
- `touch-target-size`: >= 44x44px
- `hover-vs-tap`: primary action must work on tap/click
- `loading-buttons`: disable during async
- `error-feedback`: field-local and explicit
- `cursor-pointer`: interactive elements only

### Performance (high)
- `image-optimization`: WebP/AVIF + responsive srcset + lazy load
- `reduced-motion`: honor `prefers-reduced-motion`
- `content-jumping`: reserve async space

### Layout/responsive (high)
- `viewport-meta`: `width=device-width, initial-scale=1`
- `readable-font-size`: mobile body >= 16px
- `horizontal-scroll`: no unintentional overflow
- `z-index-management`: explicit scale (`10/20/30/50/...`)

### Typography/color (medium)
- `semantics-first`: type hierarchy follows content roles, not visual decoration
- `line-height`: body typically ~1.2-1.45 for sustained reading; adjust by face/x-height
- `line-length`: prose target ~45-75 chars
- `font-pairing`: heading/body personality alignment
- `glyph-integrity`: true quotes/apostrophes/dashes/ellipsis; no faux styles
- `break-quality`: avoid widows/orphans; monitor rivers/ladders in justified copy

### Animation (medium)
- `duration-timing`: 150-300ms micro interactions
- `transform-performance`: prefer `transform`/`opacity`
- `loading-states`: skeleton or spinner where wait exists

### Style/chart (medium/low)
- `style-match`: style must fit product + audience
- `consistency`: one language across pages
- `no-emoji-icons`: SVG icon sets only
- `chart-type`: match data semantics
- `data-table`: provide accessible tabular fallback

## Workflow (Required)

1. Parse request: product type, keywords, industry, stack.
2. Run `--design-system` first.
3. Add domain searches only if needed.
4. Add stack guidance (`html-tailwind` default).
5. Synthesize and implement.

## Commands

### Prerequisite

```bash
python3 --version || python --version
```

### Required first command

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "<product> <industry> <keywords>" --design-system [-p "Project Name"]
```

### Persist design system (recommended)

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "<query>" --design-system --persist -p "Project Name" [--page "dashboard"]
```

Creates:
- `design-system/MASTER.md`: project baseline
- `design-system/pages/<page>.md`: page-level override

Retrieval rule:
1. Read `design-system/MASTER.md`
2. If `design-system/pages/<page>.md` exists, page file overrides master
3. If absent, master applies exclusively

### Domain search

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "<keyword>" --domain <domain> [-n <max_results>]
```

### Stack search (`html-tailwind` default)

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "<keyword>" --stack html-tailwind
```

### Output format

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "fintech crypto" --design-system
python3 skills/ui-ux-pro-max/scripts/search.py "fintech crypto" --design-system -f markdown
```

## Domains

| Domain | Purpose |
|---|---|
| `product` | Product-type recommendations |
| `style` | Visual language/effects |
| `typography` | Font pairings |
| `color` | Palette by product type |
| `landing` | Page structure + CTA strategy |
| `chart` | Chart type/library guidance |
| `ux` | UX patterns + anti-patterns |
| `react` | React/Next performance patterns |
| `web` | Accessibility + interface guidelines |
| `prompt` | Prompt/CSS keyword guidance |

## Stacks

| Stack | Focus |
|---|---|
| `html-tailwind` | Utility-first responsive UI (default) |
| `react` | Hooks/state/render behavior |
| `nextjs` | Routing/SSR/images |
| `vue` | Composition API/Pinia/router |
| `svelte` | Runes/stores/SvelteKit |
| `swiftui` | View/state/nav/animation |
| `react-native` | Native components/lists/nav |
| `flutter` | Widgets/state/layout/theming |
| `shadcn` | shadcn/ui components + patterns |
| `jetpack-compose` | Composables/state/recomposition |

## Common UI Failure Rules

### Icons/visuals

| Rule | Required | Never |
|---|---|---|
| Icon source | SVG sets (Heroicons/Lucide/Simple Icons) | Emoji as UI icon |
| Hover behavior | Color/opacity transitions | Layout-shifting scale transforms |
| Brand logos | Official vectors | Guessed/inaccurate marks |
| Icon sizing | Consistent viewBox and dimensions | Mixed arbitrary sizes |

### Interaction

| Rule | Required | Never |
|---|---|---|
| Click affordance | `cursor-pointer` on interactive elements | Default cursor on clickable cards |
| Feedback | Visible hover/focus/active state | No interaction feedback |
| Transition | ~150-300ms | Instant or >500ms without reason |

### Light mode contrast

| Rule | Required | Never |
|---|---|---|
| Transparent cards | Readable opacity (e.g. `bg-white/80`) | Low-opacity glass that kills contrast |
| Body text | High contrast (e.g. slate-900) | Light gray body text |
| Muted text | Still readable (>= slate-600 equivalent) | Gray-400 or lighter for body copy |
| Borders | Visible in light and dark | Near-invisible borders |

### Layout

| Rule | Required | Never |
|---|---|---|
| Floating nav | Edge spacing (`top/left/right` offsets) | Hard pin to edges by default |
| Fixed header | Offset content by header height | Hidden content under navbar |
| Containers | Consistent max width scale | Random max-width per section |

## Pre-Delivery Checklist

### Visual
- [ ] No emoji icons
- [ ] One coherent icon set
- [ ] Brand logos verified
- [ ] Hover states do not shift layout
- [ ] Theme tokens used directly
- [ ] Type roles mapped to semantics (display/head/body/meta/data)
- [ ] No faux bold/italic/small-caps or stretched text

### Interaction
- [ ] Clickable elements indicate clickability
- [ ] Hover/focus feedback present
- [ ] Motion duration in expected range
- [ ] Keyboard focus visible

### Color/contrast
- [ ] 4.5:1 minimum for normal text
- [ ] Glass/transparency readable in light mode
- [ ] Borders visible in both modes
- [ ] Tested light and dark mode

### Layout/responsive
- [ ] Edge spacing correct for floating/fixed UI
- [ ] No hidden content behind fixed elements
- [ ] Tested 375/768/1024/1440 widths
- [ ] No horizontal mobile scroll

### Accessibility
- [ ] Meaningful image alt text
- [ ] Form labels present
- [ ] Color not sole state indicator
- [ ] `prefers-reduced-motion` supported
