---
name: ui-ux-pro-max
description: 'UI/UX design intelligence. 50 styles, 21 palettes, 50 font pairings, 20 charts, 8 stacks (React, Next.js, Vue, Svelte, SwiftUI, React Native, Flutter, Tailwind). Actions: plan, build, create, design, implement, review, fix, improve, optimize, enhance, refactor, check UI/UX code. Projects: website, landing page, dashboard, admin panel, e-commerce, SaaS, portfolio, blog, mobile app, .html, .tsx, .vue, .svelte. Elements: button, modal, navbar, sidebar, card, table, form, chart. Styles: glassmorphism, claymorphism, minimalism, brutalism, neumorphism, bento grid, dark mode, responsive, skeuomorphism, flat design. Topics: color palette, accessibility, animation, layout, typography, font pairing, spacing, hover, shadow, gradient.'
---

# UI/UX Pro Max - Design Intelligence

CSV database of UI styles, color palettes, font pairings, chart types, product recommendations, UX guidelines, and stack-specific best practices.

## Data Location

All reference data lives in CSV files — read them directly with the Read tool:

```
.claude/skills/ui-ux-pro-max/data/
├── charts.csv        # Chart types & library recommendations
├── colors.csv        # Color palettes by product type
├── landing.csv       # Page structure & CTA strategies
├── products.csv      # Product type recommendations
├── prompts.csv       # AI prompts & CSS keywords
├── styles.csv        # UI styles, colors, effects
├── typography.csv    # Font pairings with Google Fonts
├── ux-guidelines.csv # Best practices & anti-patterns
└── stacks/
    ├── html-tailwind.csv
    ├── react.csv
    ├── nextjs.csv
    ├── vue.csv
    ├── svelte.csv
    ├── swiftui.csv
    ├── react-native.csv
    ├── flutter.csv
    └── nuxt-ui.csv
```

---

## How to Use This Skill

When user requests UI/UX work (design, build, create, implement, review, fix, improve), follow this workflow:

### Step 1: Analyze User Requirements

Extract key information from user request:

- **Product type**: SaaS, e-commerce, portfolio, dashboard, landing page, etc.
- **Style keywords**: minimal, playful, professional, elegant, dark mode, etc.
- **Industry**: healthcare, fintech, gaming, education, etc.
- **Stack**: React, Vue, Next.js, or default to `html-tailwind`

### Step 2: Read Relevant CSV Files

Use the Read tool to load the relevant CSV data files directly.

**Recommended read order:**

1. **products.csv** - Get style recommendations for product type
2. **styles.csv** - Get detailed style guide (colors, effects, frameworks)
3. **typography.csv** - Get font pairings with Google Fonts imports
4. **colors.csv** - Get color palette (Primary, Secondary, CTA, Background, Text, Border)
5. **landing.csv** - Get page structure (if landing page)
6. **charts.csv** - Get chart recommendations (if dashboard/analytics)
7. **ux-guidelines.csv** - Get best practices and anti-patterns
8. **stacks/react.csv** (or relevant stack) - Get stack-specific guidelines

### Step 3: Stack Guidelines (Default: html-tailwind)

If user doesn't specify a stack, **default to `html-tailwind`**.

Available stacks: `html-tailwind`, `react`, `nextjs`, `vue`, `svelte`, `swiftui`, `react-native`, `flutter`

---

## CSV Domain Reference

| CSV File          | Use For                              | Search Keywords                                          |
| ----------------- | ------------------------------------ | -------------------------------------------------------- |
| `products.csv`    | Product type recommendations         | SaaS, e-commerce, portfolio, healthcare, beauty, service |
| `styles.csv`      | UI styles, colors, effects           | glassmorphism, minimalism, dark mode, brutalism          |
| `typography.csv`  | Font pairings, Google Fonts          | elegant, playful, professional, modern                   |
| `colors.csv`      | Color palettes by product type       | saas, ecommerce, healthcare, beauty, fintech, service    |
| `landing.csv`     | Page structure, CTA strategies       | hero, hero-centric, testimonial, pricing, social-proof   |
| `charts.csv`      | Chart types, library recommendations | trend, comparison, timeline, funnel, pie                 |
| `ux-guidelines.csv` | Best practices, anti-patterns      | animation, accessibility, z-index, loading               |
| `prompts.csv`     | AI prompts, CSS keywords             | (style name)                                             |

---

## Tips for Better Results

1. **Be specific with keywords** - "healthcare SaaS dashboard" > "app"
2. **Read multiple CSVs** - Different files reveal different insights
3. **Combine domains** - Style + Typography + Color = Complete design system
4. **Always check UX** - Read ux-guidelines.csv for "animation", "z-index", "accessibility" issues
5. **Use stack CSV** - Get implementation-specific best practices
6. **Iterate** - If first read doesn't match, check related CSV files

---

## Common Rules for Professional UI

These are frequently overlooked issues that make UI look unprofessional:

### Icons & Visual Elements

| Rule                       | Do                                              | Don't                                  |
| -------------------------- | ----------------------------------------------- | -------------------------------------- |
| **No emoji icons**         | Use SVG icons (Heroicons, Lucide, Simple Icons) | Use emojis as UI icons                 |
| **Stable hover states**    | Use color/opacity transitions on hover          | Use scale transforms that shift layout |
| **Correct brand logos**    | Research official SVG from Simple Icons         | Guess or use incorrect logo paths      |
| **Consistent icon sizing** | Use fixed viewBox (24x24) with w-6 h-6          | Mix different icon sizes randomly      |

### Interaction & Cursor

| Rule                   | Do                                                    | Don't                                        |
| ---------------------- | ----------------------------------------------------- | -------------------------------------------- |
| **Cursor pointer**     | Add `cursor-pointer` to all clickable/hoverable cards | Leave default cursor on interactive elements |
| **Hover feedback**     | Provide visual feedback (color, shadow, border)       | No indication element is interactive         |
| **Smooth transitions** | Use `transition-colors duration-200`                  | Instant state changes or too slow (>500ms)   |

### Light/Dark Mode Contrast

| Rule                      | Do                                  | Don't                                   |
| ------------------------- | ----------------------------------- | --------------------------------------- |
| **Glass card light mode** | Use `bg-white/80` or higher opacity | Use `bg-white/10` (too transparent)     |
| **Text contrast light**   | Use `#0F172A` (slate-900) for text  | Use `#94A3B8` (slate-400) for body text |
| **Muted text light**      | Use `#475569` (slate-600) minimum   | Use gray-400 or lighter                 |
| **Border visibility**     | Use `border-gray-200` in light mode | Use `border-white/10` (invisible)       |

### Layout & Spacing

| Rule                     | Do                                  | Don't                                  |
| ------------------------ | ----------------------------------- | -------------------------------------- |
| **Floating navbar**      | Add `top-4 left-4 right-4` spacing  | Stick navbar to `top-0 left-0 right-0` |
| **Content padding**      | Account for fixed navbar height     | Let content hide behind fixed elements |
| **Consistent max-width** | Use same `max-w-6xl` or `max-w-7xl` | Mix different container widths         |

---

## Pre-Delivery Checklist

Before delivering UI code, verify these items:

### Visual Quality

- [ ] No emojis used as icons (use SVG instead)
- [ ] All icons from consistent icon set (Heroicons/Lucide)
- [ ] Brand logos are correct (verified from Simple Icons)
- [ ] Hover states don't cause layout shift
- [ ] Use theme colors directly (bg-primary) not var() wrapper

### Interaction

- [ ] All clickable elements have `cursor-pointer`
- [ ] Hover states provide clear visual feedback
- [ ] Transitions are smooth (150-300ms)
- [ ] Focus states visible for keyboard navigation

### Light/Dark Mode

- [ ] Light mode text has sufficient contrast (4.5:1 minimum)
- [ ] Glass/transparent elements visible in light mode
- [ ] Borders visible in both modes
- [ ] Test both modes before delivery

### Layout

- [ ] Floating elements have proper spacing from edges
- [ ] No content hidden behind fixed navbars
- [ ] Responsive at 320px, 768px, 1024px, 1440px
- [ ] No horizontal scroll on mobile

### Accessibility

- [ ] All images have alt text
- [ ] Form inputs have labels
- [ ] Color is not the only indicator
- [ ] `prefers-reduced-motion` respected
