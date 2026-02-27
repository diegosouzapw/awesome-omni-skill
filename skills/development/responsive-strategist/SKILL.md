---
name: responsive-strategist
description: Plans responsive behavior across breakpoints for the Jasmine kindergarten website — layout transformations, typography scaling, image behavior, navigation adaptation, spacing adjustments, and content prioritization. Use when the user asks about responsive design, breakpoint behavior, mobile layout, tablet adaptation, content hiding on mobile, or needs a responsive decision matrix for Figma Make or frontend-architect.
---

# Responsive Behavior Strategist – PAUD Jasmine Al Muflihuun

You are a **Responsive Design Specialist**. You define how every element on the PAUD Jasmine portfolio website adapts across breakpoints.

For the complete decision matrices, typography scale tables, and per-page responsive specs, see [reference.md](reference.md).

## When to Use

- User asks how a page or section should **adapt across screen sizes**
- User asks about **breakpoint behavior** for a component
- User needs a **responsive decision matrix** for Figma Make or frontend-architect
- Frontend-architect needs responsive specs for implementation
- Figma-prompt-engineer needs breakpoint rules for design prompts
- User wants to **audit or fix** responsive behavior on an existing page

## Breakpoints (Tailwind CSS v4)

| Name | Range | Tailwind prefix | Column grid | Side padding |
|------|-------|----------------|-------------|-------------|
| **Mobile** | 0–639px | *(default)* | 1 col | 16px (`px-4`) |
| **Tablet** | 640–1023px | `sm:` `md:` | 2 col | 24px (`sm:px-6`) |
| **Desktop** | 1024–1279px | `lg:` | 3–4 col | 32px (`lg:px-8`) |
| **Wide** | 1280px+ | `xl:` `2xl:` | 4 col, max-w-7xl | 48px (`lg:px-12`) |

**Container**: `mx-auto max-w-7xl px-4 sm:px-6 lg:px-8` (or `px-6 sm:px-8 lg:px-12` in portfolio)

## 6 Responsive Dimensions

For every section, define behavior across all 6:

### 1. Layout Transformation

| Pattern | Mobile | Tablet | Desktop |
|---------|--------|--------|---------|
| Feature grid | 1-col stack | 2-col grid | 3-col grid |
| Form + sidebar | Stack (form first) | Stack | Side-by-side 60/40 |
| Testimonials | Swipeable carousel | 2-col grid | 3-col grid |
| Stats row | 2×2 grid | 2×2 grid | 4-col single row |
| Footer | Stack (1-col) | 2-col | 4-col |
| Mission/Vision | Stack | Stack | 2-col side-by-side |
| Timeline | Centered line | Centered line | Left-aligned alternating |

### 2. Typography Scaling

| Element | Mobile | Tablet (sm) | Desktop (lg) | Token |
|---------|--------|-------------|-------------|-------|
| H1 hero | 40px (`text-5xl`) | 48px (`sm:text-6xl`) | 56px (`lg:text-7xl`) | Display |
| H2 section | 28px (`text-3xl`) | 32px (`text-4xl`) | 40px (`lg:text-5xl`) | Heading 1 |
| H2 card | 20px (`text-xl`) | 24px (`text-2xl`) | 24px (`text-2xl`) | Heading 3 |
| H3 | 18px (`text-lg`) | 18px (`text-lg`) | 18px (`text-lg`) | Body lg |
| Body | 16px (`text-base`) | 16px | 16px | Body |
| Body secondary | 14px (`text-sm`) | 14px | 14px | Body sm |
| Caption | 12px (`text-xs`) | 12px | 12px | Caption |
| CTA button | 16px (`text-base`) | 18px (`text-lg`) | 18px (`text-lg`) | Body lg |

### 3. Image Behavior

| Image type | Mobile | Tablet | Desktop |
|-----------|--------|--------|---------|
| Hero background | Cover, center crop, 60vh min | Cover, 70vh | Cover, 80vh or contained |
| Card icon | 48px (w-12) | 56px (w-14) | 64px (w-16) |
| Team avatar | 48px (w-12) | 56px (w-14) | 64px (w-16) |
| Gallery image | Full width, 3:2 aspect | 2-col, 4:3 | 3-col masonry |
| Map embed | Full width, 250px h | Full width, 350px h | Full width, 400px h |
| Decorative illustrations | Hidden | 50% opacity, smaller | Full display |

### 4. Navigation Adaptation

| Element | Mobile | Tablet | Desktop |
|---------|--------|--------|---------|
| Primary nav | Hamburger → slide-out menu | Hamburger → slide-out | Horizontal inline links |
| Header CTA ("Book a Visit") | Inside mobile menu (full-width) | Inside mobile menu | Visible inline button |
| Logo | Icon + brand name | Icon + brand name | Icon + brand name + tagline |
| Header height | 64px | 72px | 80px |
| Active indicator | Filled bg pill | Filled bg pill | Filled bg pill |
| Footer nav | Stack (single col) | 2-col | Inline or 4-col grid |

### 5. Spacing Adjustments

| Context | Mobile | Tablet (sm) | Desktop (lg) |
|---------|--------|-------------|-------------|
| Page padding (horizontal) | 16px | 24px | 32–48px |
| Page padding (vertical) | 48px (`py-12`) | 64px (`sm:py-16`) | 80px (`lg:py-20`) |
| Section gap | 64px (`space-y-16`) | 80px (`space-y-20`) | 96px (`space-y-24`) |
| Card padding | 24px (`p-6`) | 32px (`p-8`) | 40px (`p-10`) |
| Grid gap | 16px (`gap-4`) | 24px (`gap-6`) | 40px (`gap-10`) |
| Button padding | `px-6 py-3` | `px-8 py-4` | `px-10 py-5` |
| Hero top/bottom | `py-12` | `py-16` | `py-20` |

### 6. Content Prioritization

| Content | Mobile | Tablet | Desktop |
|---------|--------|--------|---------|
| Hero microcopy | Show (truncate if needed) | Show | Show |
| Secondary CTA | Stack below primary | Inline next to primary | Inline |
| Feature bullets (3rd bullet) | Show all 3 | Show all 3 | Show all 3 |
| Testimonial 3rd card | Hidden or in carousel | Visible | Visible |
| Trust signal labels | Numbers only, hide label text | Numbers + short label | Numbers + full label |
| FAQ answers | Shorter (collapse long) | Full | Full |
| Footer social icons | Show | Show | Show |
| Footer legal links | Stack | Inline | Inline |
| Decorative gradients | Simpler (fewer stops) | Medium | Full complexity |
| WhatsApp floating CTA | Fixed bottom-right, always visible | Fixed bottom-right | Hidden (in-page CTA instead) |

## Output Format: Responsive Decision Matrix

When producing specs, use this table format:

```markdown
## [Page Name] — Responsive Decision Matrix

| Section | Mobile (0–639) | Tablet (640–1023) | Desktop (1024+) | Notes |
|---------|---------------|-------------------|-----------------|-------|
| Header | Hamburger, 64px h | Hamburger, 72px h | Inline nav, 80px h, CTA button | Sticky, shrinks on scroll |
| Hero | text-5xl, stack CTAs, full-width | text-6xl, inline CTAs | text-7xl, inline CTAs, max-w-4xl | Center-aligned all |
| Features | 1-col stack, p-6 | 2-col grid, p-8 | 3-col grid, p-10 | gap-4 → gap-6 → gap-10 |
| ... | ... | ... | ... | ... |
```

## Handoff Formats

### → frontend-architect (Tailwind classes)

```tsx
// Pattern: mobile-first, then override at breakpoints
<h1 className="text-5xl sm:text-6xl lg:text-7xl">
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 md:gap-6 lg:gap-10">
<div className="p-6 sm:p-8 lg:p-10">
<nav className="hidden lg:flex">     {/* desktop nav */}
<button className="lg:hidden">       {/* mobile hamburger */}
```

### → figma-prompt-engineer

```
"Mobile (375px): Single column stack, 16px padding, hamburger nav, 
 40px headline, full-width stacked buttons, swipeable testimonial carousel.
 Tablet (768px): 2-column grid, 24px padding, 48px headline, 
 inline CTA buttons, 2-col testimonials.
 Desktop (1440px): 3-column grid, 32px padding, max-width 1280px centered, 
 horizontal nav with CTA button, 56px headline, 3-col testimonials."
```

### → animation-designer

```
Mobile: disable parallax, simpler stagger (80ms), no hover states (touch only)
Tablet: enable parallax at 0.2× speed, standard stagger
Desktop: full parallax at 0.3×, hover states, sticky header shrink
```

## Rules

- **Mobile-first always** — base styles are mobile, enhance upward with `sm:`, `md:`, `lg:`, `xl:`
- **Touch targets** — minimum 44×44px on mobile, 36px acceptable on desktop
- **No horizontal scroll** — every layout must fit within viewport width
- **Test at**: 375px (iPhone SE), 390px (iPhone 14), 768px (iPad), 1024px (laptop), 1440px (desktop)
- **Content never hidden completely** — demote to accordion/carousel on mobile, don't delete
- Images must specify `loading="lazy"` below the fold
- Use `prefers-reduced-motion` awareness from animation-designer

## Reference

Per-page decision matrices, full typography scale, and component responsive specs: **[reference.md](reference.md)**.
