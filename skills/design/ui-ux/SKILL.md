---
name: ui-ux
description: "UI/UX design intelligence. Activated when: designing UI, building pages, creating components, choosing colors, picking fonts, reviewing UX, building dashboards, landing pages, web design, mobile design, accessibility review, dark mode, light mode. Styles, color palettes, font pairings, UX guidelines, chart types across 13 stacks. Always delivers a structured Changes Made summary."
---

# UI/UX

Searchable design intelligence database for web and mobile applications. Generates complete design systems with style, color palette, typography, and UX guidelines. BM25 search engine across 13 technology stacks.

## When to Activate

- User requests UI/UX work: design, build, create, implement, review, fix, improve
- User asks for color palettes, font pairings, or style recommendations
- User builds a landing page, dashboard, portfolio, or any web/mobile interface
- User mentions dark mode, accessibility, responsive design, or layout issues
- User asks to review or audit existing UI for quality issues

---

## Rule Categories by Priority

| Priority | Category | Impact | Domain |
|----------|----------|--------|--------|
| 1 | Accessibility | CRITICAL | `ux` |
| 2 | Touch & Interaction | CRITICAL | `ux` |
| 3 | Performance | HIGH | `ux` |
| 4 | Layout & Responsive | HIGH | `ux` |
| 5 | Typography & Color | MEDIUM | `typography`, `color` |
| 6 | Animation | MEDIUM | `ux` |
| 7 | Style Selection | MEDIUM | `style`, `product` |
| 8 | Charts & Data | LOW | `chart` |

## Quick Reference

### Accessibility (CRITICAL)

- `color-contrast` — Minimum 4.5:1 ratio for normal text
- `focus-states` — Visible focus rings on interactive elements
- `alt-text` — Descriptive alt text for meaningful images
- `aria-labels` — aria-label for icon-only buttons
- `keyboard-nav` — Tab order matches visual order
- `form-labels` — Use label with for attribute

### Touch & Interaction (CRITICAL)

- `touch-target-size` — Minimum 44x44px touch targets
- `hover-vs-tap` — Use click/tap for primary interactions
- `loading-buttons` — Disable button during async operations
- `cursor-pointer` — Add cursor-pointer to clickable elements

### Performance (HIGH)

- `image-optimization` — Use WebP, srcset, lazy loading
- `reduced-motion` — Check prefers-reduced-motion
- `content-jumping` — Reserve space for async content

### Layout & Responsive (HIGH)

- `viewport-meta` — width=device-width initial-scale=1
- `readable-font-size` — Minimum 16px body text on mobile
- `horizontal-scroll` — Ensure content fits viewport width
- `z-index-management` — Define z-index scale (10, 20, 30, 50)

---

## Prerequisites

```bash
python3 --version || python --version
```

If not installed — **Windows:** `winget install Python.Python.3.12` | **macOS:** `brew install python3` | **Linux:** `sudo apt install python3`

---

## Workflow

### Progress
- [ ] Analyzed user requirements (product type, style, industry, stack)
- [ ] Generated design system via `--design-system`
- [ ] Supplemented with domain searches (if needed)
- [ ] Applied stack guidelines
- [ ] Implemented UI with design system values
- [ ] Ran Pre-Delivery Checklist
- [ ] Delivered Changes Made summary

### Step 1: Analyze User Requirements

Extract from the user's request:

| Field | Extract |
|-------|---------|
| **Product type** | SaaS, e-commerce, portfolio, dashboard, landing page |
| **Style keywords** | minimal, playful, professional, elegant, dark mode |
| **Industry** | healthcare, fintech, gaming, education, beauty |
| **Stack** | React, Vue, Next.js — default to `html-tailwind` |

### Step 2: Generate Design System (REQUIRED)

**Always start with `--design-system`** to get comprehensive recommendations with reasoning:

```bash
python3 skills/ui-ux/scripts/search.py "<product_type> <industry> <keywords>" --design-system [-p "Project Name"]
```

This command:
1. Searches 5 domains in parallel (product, style, color, landing, typography)
2. Applies reasoning rules from `ui-reasoning.csv` to select best matches
3. Returns complete design system: pattern, style, colors, typography, effects
4. Includes anti-patterns to avoid

**Example:**
```bash
python3 skills/ui-ux/scripts/search.py "beauty spa wellness service" --design-system -p "Serenity Spa"
```

### Step 2b: Persist Design System (Optional)

Save for hierarchical retrieval across sessions with `--persist`:

```bash
python3 skills/ui-ux/scripts/search.py "<query>" --design-system --persist -p "Project Name"
```

Creates `design-system/MASTER.md` (source of truth) and `design-system/pages/` for overrides.

**With page override:** Add `--page "dashboard"` to create page-specific deviations.

**Retrieval rule:** Check `pages/<page>.md` first → if exists, overrides Master → otherwise use Master exclusively.

### Step 3: Supplement with Detailed Searches

After the design system, use domain searches for additional detail:

```bash
python3 skills/ui-ux/scripts/search.py "<keyword>" --domain <domain> [-n <max_results>]
```

| Need | Domain | Example |
|------|--------|---------|
| More style options | `style` | `"glassmorphism dark"` |
| Chart recommendations | `chart` | `"real-time dashboard"` |
| UX best practices | `ux` | `"animation accessibility"` |
| Alternative fonts | `typography` | `"elegant luxury"` |
| Landing structure | `landing` | `"hero social-proof"` |

### Step 4: Stack Guidelines (Default: html-tailwind)

```bash
python3 skills/ui-ux/scripts/search.py "<keyword>" --stack html-tailwind
```

Available: `html-tailwind`, `react`, `nextjs`, `vue`, `svelte`, `swiftui`, `react-native`, `flutter`, `shadcn`, `jetpack-compose`

---

## Search Reference

### Domains

| Domain | Use For | Example Keywords |
|--------|---------|------------------|
| `product` | Product type recommendations | SaaS, portfolio, healthcare |
| `style` | UI styles, colors, effects | glassmorphism, dark mode, brutalism |
| `typography` | Font pairings, Google Fonts | elegant, playful, modern |
| `color` | Color palettes by product type | saas, ecommerce, fintech |
| `landing` | Page structure, CTA strategies | hero, testimonial, pricing |
| `chart` | Chart types, library recs | trend, comparison, funnel |
| `ux` | Best practices, anti-patterns | animation, accessibility, z-index |
| `react` | React/Next.js performance | suspense, memo, rerender |
| `web` | Web interface guidelines | aria, focus, semantic |

### Output Formats

```bash
# ASCII box (default)
python3 skills/ui-ux/scripts/search.py "fintech crypto" --design-system

# Markdown
python3 skills/ui-ux/scripts/search.py "fintech crypto" --design-system -f markdown
```

---

## Constraints

- Do **not** use emoji as UI icons — always use SVG icons (Heroicons, Lucide, Simple Icons)
- Do **not** skip Step 2 (design system generation) — it's required before implementing any UI
- Do **not** use `scale` transforms for hover states — they cause layout shift; use color/opacity instead
- Do **not** hardcode colors — always derive from the generated design system palette
- Do **not** leave interactive elements without `cursor-pointer`
- Do **not** ignore `prefers-reduced-motion` for animations

---

## Example

**Input:** "Build a landing page for a luxury spa brand"
**Workflow:**
```bash
python3 skills/ui-ux/scripts/search.py "luxury spa wellness service" --design-system -p "Serenity Spa"
```
**Output:** Design system with elegant/organic style, warm muted palette (sage, cream, rose gold), Cormorant Garamond + Montserrat typography, subtle fade animations, and anti-patterns to avoid (no neon colors, no aggressive CTAs).

---

## Common Rules for Professional UI

### Icons & Visual Elements

| Do | Don't |
|----|-------|
| Use SVG icons (Heroicons, Lucide, Simple Icons) | Use emojis 🎨 🚀 as UI icons |
| Use color/opacity transitions on hover | Use scale transforms that shift layout |
| Research official SVG from Simple Icons | Guess or use incorrect logo paths |
| Use fixed viewBox (24x24) with w-6 h-6 | Mix different icon sizes randomly |

### Interaction & Cursor

| Do | Don't |
|----|-------|
| `cursor-pointer` on all clickable elements | Default cursor on interactive elements |
| Visual feedback on hover (color, shadow, border) | No indication element is interactive |
| `transition-colors duration-200` | Instant or too-slow transitions (>500ms) |

### Light/Dark Mode Contrast

| Do | Don't |
|----|-------|
| `bg-white/80` or higher opacity in light mode | `bg-white/10` (too transparent) |
| `#0F172A` (slate-900) for light mode text | `#94A3B8` (slate-400) for body text |
| `border-gray-200` in light mode | `border-white/10` (invisible) |

### Layout & Spacing

| Do | Don't |
|----|-------|
| `top-4 left-4 right-4` floating navbar | Stick navbar to `top-0 left-0 right-0` |
| Account for fixed navbar height | Let content hide behind fixed elements |
| Consistent `max-w-6xl` or `max-w-7xl` | Mix different container widths |

---

## Error Handling

If the search script fails:
1. Verify Python is installed: `python3 --version || python --version`
2. Check the script path is correct: `ls skills/ui-ux/scripts/`
3. Run with help: `python3 skills/ui-ux/scripts/search.py --help`
4. Verify data files exist: `ls skills/ui-ux/data/`
5. If on Windows, try `python` instead of `python3`
6. Re-read the relevant section of this SKILL.md

---

## Pre-Delivery Checklist

### Visual Quality
- [ ] No emojis used as icons (use SVG instead)
- [ ] All icons from consistent icon set (Heroicons/Lucide)
- [ ] Brand logos correct (verified from Simple Icons)
- [ ] Hover states don't cause layout shift

### Interaction
- [ ] All clickable elements have `cursor-pointer`
- [ ] Hover states provide clear visual feedback
- [ ] Transitions are smooth (150-300ms)
- [ ] Focus states visible for keyboard navigation

### Light/Dark Mode
- [ ] Light mode text has sufficient contrast (4.5:1 minimum)
- [ ] Glass/transparent elements visible in light mode
- [ ] Borders visible in both modes

### Layout
- [ ] Floating elements have proper spacing from edges
- [ ] No content hidden behind fixed navbars
- [ ] Responsive at 375px, 768px, 1024px, 1440px
- [ ] No horizontal scroll on mobile

### Accessibility
- [ ] All images have alt text
- [ ] Form inputs have labels
- [ ] Color is not the only indicator
- [ ] `prefers-reduced-motion` respected

### Changes Made Summary
- [ ] **Changes Made summary delivered to user** (see below)

---

## Changes Made (MANDATORY)

> [!CAUTION]
> **This is NON-NEGOTIABLE. Every single UI/UX task MUST end with a "Changes Made" summary.** This is the LAST thing you do — present it before any other closing remarks. Never skip this step, even for small changes.

After completing any UI/UX work, present the user with a structured summary:

```markdown
## ✅ Changes Made

| # | File | Action | Detail |
|---|------|--------|--------|
| 1 | `index.html` | Created / Modified | <what was done> |
| 2 | `styles.css` | Created / Modified | <what was done> |
| 3 | `components/nav.html` | Created | <what was done> |

**Summary:** <1-2 sentence recap of the overall change>
```

**Rules:**
- List **every** file created, modified, or deleted
- Action column: `Created`, `Modified`, `Deleted`, `Moved`, or `Renamed`
- Detail column must be specific — not "updated file" but "added responsive navbar with dark mode toggle"
- End with a 1-2 sentence **Summary** of the overall impact

**Worked example:**

```markdown
## ✅ Changes Made

| # | File | Action | Detail |
|---|------|--------|--------|
| 1 | `index.html` | Modified | Added floating navbar with glassmorphism effect, hero section with gradient overlay, testimonials grid |
| 2 | `styles.css` | Modified | Implemented design system tokens (--primary, --accent), added dark mode media query, responsive breakpoints at 375/768/1024px |
| 3 | `assets/hero-bg.webp` | Created | Generated hero background image with WebP format |

**Summary:** Built a luxury spa landing page using the Serenity Spa design system with dark mode support and responsive layout.
```
