---
name: ui-ux-pro-max
description: Use when building UI that needs systematic design decisions, accessibility compliance, UX best practices, or pre-delivery quality checks - complements designing-frontends for production-grade interfaces
---

# UI/UX Pro Max - Design Intelligence

## Overview

Systematic design guide with priority-based UX rules, accessibility requirements, and pre-delivery checklists. Use alongside `@designing-frontends` for creative direction.

**Core principle:** Apply rules by priority (CRITICAL → HIGH → MEDIUM → LOW) and verify with checklist before delivery.

## When to Use

- Building UI components that need accessibility compliance
- Reviewing code for UX issues (touch targets, focus states, contrast)
- Implementing responsive layouts
- Adding animations and interactions
- Preparing UI for production delivery
- Choosing typography, colors, or chart types systematically

**Use `@designing-frontends` for:** Creative direction, bold aesthetics, unique visual identity

**Use `@ui-ux-pro-max` for:** Systematic rules, accessibility, checklists, production polish

## Quick Reference - Rules by Priority

### CRITICAL - Always Apply

| Rule | Requirement |
|------|-------------|
| **color-contrast** | Minimum 4.5:1 ratio for normal text |
| **focus-states** | Visible focus rings on all interactive elements |
| **alt-text** | Descriptive alt text for meaningful images |
| **aria-labels** | `aria-label` for icon-only buttons |
| **keyboard-nav** | Tab order matches visual order |
| **form-labels** | Use `<label>` with `for` attribute |
| **touch-target-size** | Minimum 44x44px touch targets |
| **hover-vs-tap** | Use click/tap for primary interactions |
| **loading-buttons** | Disable button during async operations |
| **error-feedback** | Clear error messages near the problem |
| **cursor-pointer** | Add `cursor-pointer` to all clickable elements |

### HIGH - Apply for Production

| Rule | Requirement |
|------|-------------|
| **image-optimization** | Use WebP, srcset, lazy loading |
| **reduced-motion** | Check `prefers-reduced-motion` |
| **content-jumping** | Reserve space for async content |
| **viewport-meta** | `width=device-width, initial-scale=1` |
| **readable-font-size** | Minimum 16px body text on mobile |
| **horizontal-scroll** | Ensure content fits viewport width |
| **z-index-management** | Define z-index scale (10, 20, 30, 50) |

### MEDIUM - Apply for Polish

| Rule | Requirement |
|------|-------------|
| **line-height** | Use 1.5-1.75 for body text |
| **line-length** | Limit to 65-75 characters per line |
| **font-pairing** | Match heading/body font personalities |
| **duration-timing** | Use 150-300ms for micro-interactions |
| **transform-performance** | Use `transform`/`opacity`, not `width`/`height` |
| **loading-states** | Skeleton screens or spinners |
| **style-consistency** | Use same style across all pages |
| **no-emoji-icons** | Use SVG icons, not emojis |

### LOW - Apply When Relevant

| Rule | Requirement |
|------|-------------|
| **chart-type** | Match chart type to data type |
| **chart-colors** | Use accessible color palettes |
| **data-table** | Provide table alternative for chart accessibility |

## Common Professional UI Rules

### Interaction & Cursor

```css
/* All clickable elements */
.clickable {
  cursor: pointer;
  transition: colors 200ms;
}
```

### Light/Dark Mode Contrast

| Element | Light Mode | Dark Mode |
|---------|------------|-----------|
| Glass background | `bg-white/80` | `bg-white/10` |
| Text primary | `#0F172A` | `#F8FAFC` |
| Text secondary | `#475569` | `#94A3B8` |
| Borders | `border-gray-200` | `border-white/10` |

### Layout & Spacing

- **Floating elements**: Use `top-4 left-4 right-4` not `top-0 left-0 right-0`
- **Content width**: Use `max-w-6xl` or `max-w-7xl` for readability
- **Responsive breakpoints**: Test at 375px, 768px, 1024px, 1440px

## Pre-Delivery Checklist

Run through before delivering any UI code:

### Visual Quality
- [ ] No emojis used as icons (use SVG instead)
- [ ] All icons from consistent icon set (Heroicons/Lucide)
- [ ] Brand logos verified (use Simple Icons for tech logos)
- [ ] Hover states don't cause layout shift
- [ ] Use theme colors directly (`bg-primary`) not `var()` wrapper

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
- [ ] Responsive at 375px, 768px, 1024px, 1440px
- [ ] No horizontal scroll on mobile

### Accessibility
- [ ] All images have alt text
- [ ] Form inputs have labels
- [ ] Color is not the only indicator
- [ ] `prefers-reduced-motion` respected

## Design Reference Data

This skill includes comprehensive design databases in the `data/` folder:

### Available References
- **`data/styles.md`** - 68 design styles with colors, effects, CSS implementation
- **`data/colors.md`** - 96 color palettes by product type (SaaS, e-commerce, healthcare, etc.)
- **`data/typography.md`** - 57 font pairings with Google Fonts imports and Tailwind config

### How to Use
When building UI, read the relevant data file for recommendations:
- Need a style? → Read `data/styles.md` for Glassmorphism, Neubrutalism, Bento Grid, etc.
- Need colors? → Read `data/colors.md` for product-specific palettes
- Need fonts? → Read `data/typography.md` for curated pairings with imports

### Example Workflow
1. User asks for "fintech dashboard"
2. Read `data/colors.md` → Find "Financial Dashboard" palette
3. Read `data/styles.md` → Find "Data-Dense Dashboard" or "Executive Dashboard" style
4. Read `data/typography.md` → Find "Financial Trust" or "Dashboard Data" pairing
5. Apply the recommendations with the UX rules from this skill

## Tips for Better Results

1. **Be specific** - "healthcare SaaS dashboard" > "app"
2. **Check accessibility first** - CRITICAL rules before polish
3. **Test both modes** - Light and dark before delivery
4. **Use the checklist** - Run through every item before shipping

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Missing `cursor-pointer` on buttons | Add to all clickable elements |
| Low contrast in light mode | Test with contrast checker (4.5:1 minimum) |
| Emojis as icons | Replace with SVG from Lucide/Heroicons |
| Touch targets too small | Ensure minimum 44x44px |
| No loading states | Add skeleton or spinner for async |
| Layout shift on hover | Use `transform` instead of `width`/`margin` |
| Missing focus states | Add visible `:focus-visible` styles |

## Source

Adapted from [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) - contains 50+ styles, 97 color palettes, 57 font pairings, 99 UX guidelines, and 25 chart types.
