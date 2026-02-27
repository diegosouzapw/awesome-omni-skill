---
name: fct-web-ui-ux
description: UI/UX design intelligence for web and mobile across 13 stacks (React, Next.js, Vue, Svelte, Tailwind, shadcn/ui, etc.). Use when building, designing, reviewing, or optimizing UI/UX for websites, dashboards, apps. Covers styles, palettes, fonts, accessibility, animation, layout.
---

# UI/UX Design Intelligence

Searchable database of UI styles, color palettes, font pairings, and UX guidelines. Generates complete design systems with reasoning.

## Priority Rules

| Priority | Category | Impact |
|----------|----------|--------|
| 1 | Accessibility | CRITICAL |
| 2 | Touch & Interaction | CRITICAL |
| 3 | Performance | HIGH |
| 4 | Layout & Responsive | HIGH |
| 5 | Typography & Color | MEDIUM |
| 6 | Animation | MEDIUM |
| 7 | Style Selection | MEDIUM |
| 8 | Charts & Data | LOW |

## Quick Reference

### Accessibility (CRITICAL)
- `color-contrast` - Minimum 4.5:1 ratio for text
- `focus-states` - Visible focus rings on interactive elements
- `alt-text` - Descriptive alt text for images
- `aria-labels` - aria-label for icon-only buttons
- `keyboard-nav` - Tab order matches visual order

### Touch & Interaction (CRITICAL)
- `touch-target-size` - Minimum 44x44px touch targets
- `loading-buttons` - Disable button during async operations
- `cursor-pointer` - Add cursor-pointer to clickable elements

### Performance (HIGH)
- `image-optimization` - Use WebP, srcset, lazy loading
- `reduced-motion` - Check prefers-reduced-motion

### Layout & Responsive (HIGH)
- `readable-font-size` - Minimum 16px body text on mobile
- `z-index-management` - Define z-index scale (10, 20, 30, 50)

---

## Workflow

### Step 1: Analyze Request

Extract from user request:
- **Product type**: SaaS, e-commerce, portfolio, dashboard, landing page
- **Style keywords**: minimal, playful, professional, elegant, dark mode
- **Industry**: healthcare, fintech, gaming, education
- **Stack**: React, Vue, Next.js, or default to `html-tailwind`

### Step 2: Generate Design System (REQUIRED)

```bash
uv run scripts/search.py "<product_type> <industry> <keywords>" --design-system [-p "Project Name"]
```

Returns: pattern, style, colors, typography, effects, anti-patterns.

**Example:**
```bash
uv run scripts/search.py "beauty spa wellness service" --design-system -p "Serenity Spa"
```

### Step 2b: Persist Design System (Optional)

Save for hierarchical retrieval across sessions:

```bash
uv run scripts/search.py "<query>" --design-system --persist -p "Project Name"
```

Creates:
- `design-system/MASTER.md` — Global Source of Truth
- `design-system/pages/` — Page-specific overrides

With page override:
```bash
uv run scripts/search.py "<query>" --design-system --persist -p "Project Name" --page "dashboard"
```

### Step 3: Detailed Searches (as needed)

```bash
uv run scripts/search.py "<keyword>" --domain <domain> [-n <max_results>]
```

| Need | Domain | Example |
|------|--------|---------|
| More style options | `style` | `--domain style "glassmorphism dark"` |
| Chart recommendations | `chart` | `--domain chart "real-time dashboard"` |
| UX best practices | `ux` | `--domain ux "animation accessibility"` |
| Alternative fonts | `typography` | `--domain typography "elegant luxury"` |
| Landing structure | `landing` | `--domain landing "hero social-proof"` |

### Step 4: Stack Guidelines

```bash
uv run scripts/search.py "<keyword>" --stack html-tailwind
```

Stacks: `html-tailwind` (default), `react`, `nextjs`, `astro`, `vue`, `nuxtjs`, `nuxt-ui`, `svelte`, `swiftui`, `react-native`, `flutter`, `shadcn`, `jetpack-compose`

---

## Search Reference

### Domains

| Domain | Use For |
|--------|---------|
| `product` | Product type recommendations |
| `style` | UI styles, colors, effects |
| `typography` | Font pairings, Google Fonts |
| `color` | Color palettes by product type |
| `landing` | Page structure, CTA strategies |
| `chart` | Chart types, library recommendations |
| `ux` | Best practices, anti-patterns |
| `react` | React/Next.js performance |
| `web` | Web interface guidelines |

---

## Common Rules

### Icons & Visual Elements

| Rule | Do | Don't |
|------|----|----- |
| **No emoji icons** | Use SVG (Heroicons, Lucide) | Use emojis as UI icons |
| **Stable hover states** | Use color/opacity transitions | Use scale transforms |
| **Consistent sizing** | Fixed viewBox (24x24) w-6 h-6 | Mix icon sizes |

### Interaction & Cursor

| Rule | Do | Don't |
|------|----|----- |
| **Cursor pointer** | Add `cursor-pointer` to clickable elements | Leave default cursor |
| **Hover feedback** | Visual feedback (color, shadow) | No indication of interactivity |
| **Smooth transitions** | `transition-colors duration-200` | Instant or >500ms |

### Light/Dark Mode Contrast

| Rule | Do | Don't |
|------|----|----- |
| **Glass card light mode** | `bg-white/80` or higher | `bg-white/10` |
| **Text contrast light** | `#0F172A` (slate-900) | `#94A3B8` (slate-400) |
| **Border visibility** | `border-gray-200` light mode | `border-white/10` |

### Layout & Spacing

| Rule | Do | Don't |
|------|----|----- |
| **Floating navbar** | `top-4 left-4 right-4` spacing | `top-0 left-0 right-0` |
| **Content padding** | Account for fixed navbar | Content behind fixed elements |
| **Consistent max-width** | Same `max-w-6xl` or `max-w-7xl` | Mix container widths |

---

## Pre-Delivery Checklist

### Visual Quality
- [ ] No emojis as icons (use SVG)
- [ ] Consistent icon set (Heroicons/Lucide)
- [ ] Hover states don't cause layout shift
- [ ] Use theme colors directly (bg-primary)

### Interaction
- [ ] All clickable elements have `cursor-pointer`
- [ ] Transitions 150-300ms
- [ ] Focus states visible

### Light/Dark Mode
- [ ] Light mode text contrast 4.5:1 minimum
- [ ] Glass elements visible in light mode
- [ ] Borders visible in both modes

### Layout
- [ ] Responsive at 375px, 768px, 1024px, 1440px
- [ ] No horizontal scroll on mobile
- [ ] No content behind fixed elements

### Accessibility
- [ ] All images have alt text
- [ ] Form inputs have labels
- [ ] `prefers-reduced-motion` respected
