---
name: frontend-style-layout
description: Apply consistent styling and layout patterns with Tailwind v4. Use when building page layouts, choosing spacing methods, implementing responsive images, or migrating Tailwind v3 classes to v4. Covers Section Compound Pattern, spacing selection, CLS-free responsive images, and v4 class changes.
---

# Frontend Style & Layout Patterns

Tailwind v4 styling patterns for this project. Covers page structure, spacing, responsive images, and v3→v4 migration.

## 1. Section Compound Pattern

Express page layout structure explicitly using semantic HTML + Tailwind classes at the usage site.

```tsx
// ✅ Layout structure visible at a glance
const OrderListPage = () => (
  <main className="min-h-screen flex flex-col">
    <header className="sticky top-0 z-10 bg-white border-b border-gray-200">
      <OrderListHeader />
    </header>

    <section className="flex-1 flex flex-col gap-4 px-4 py-6">
      <OrderFilterBar />
      <OrderList />
    </section>

    <footer className="sticky bottom-0 bg-white border-t border-gray-200 px-4 py-3">
      <OrderListFooterActions />
    </footer>
  </main>
)
```

### What to Expose vs. Hide

| Expose at usage site                    | Hide in component                  |
| --------------------------------------- | ---------------------------------- |
| Layout structure (flex, grid, position) | Reusable styles (button variants)  |
| Page-specific spacing and sizing        | Internal implementation details    |
| Semantic HTML structure                 | Complexity outside current concern |

- Reusable styles (buttons, cards) → abstract into components.
- Page layout → write directly at the usage site in `screens/` pages.

## 2. Spacing Selection Guide

### Three Methods

**flex gap** — Default for Flexbox containers:

```tsx
<div className="flex flex-col gap-4">
  <OrderCard />
  <OrderCard />
</div>
```

**space-y / space-x** — Sibling elements with consistent spacing:

```tsx
<ul className="space-y-3">
  <li>
    <ReviewItem />
  </li>
  <li>
    <ReviewItem />
  </li>
</ul>
```

**Explicit spacer div** — Single specific gap (e.g., from design spec):

```tsx
<div>
  <OrderHeader />
  <div className="h-8" /> {/* 32px explicit gap */}
  <OrderContent />
</div>
```

### Decision Flow

```
Need spacing?
├── Inside Flexbox container?  → flex gap
├── Repeating sibling spacing? → space-y / space-x
├── Single explicit gap (from design spec)? → spacer div
└── Complex conditional spacing? → individual margin/padding
```

### Notes

- Use gap inside flex containers to avoid margin collapsing.

## 3. Responsive Images — CLS Prevention

Use `aspect-ratio` + `relative` container to maintain original ratio responsively.

```tsx
// ✅ Basic responsive image
const Illustration = () => (
  <div className="flex justify-center">
    <div className="relative w-full aspect-[327/200]">
      <img
        src="/illustrations/order-complete.png"
        alt="Order complete"
        className="object-contain w-full h-full"
        loading="eager"
      />
    </div>
  </div>
)

// ✅ With max-width constraint
const Thumbnail = ({ src, alt }: { src: string; alt: string }) => (
  <div className="relative w-full max-w-sm aspect-square">
    <img src={src} alt={alt} className="object-cover w-full h-full rounded-lg" loading="lazy" />
  </div>
)
```

### Key Properties

| Property                          | Purpose                         |
| --------------------------------- | ------------------------------- |
| `relative`                        | Positioning anchor for children |
| `aspect-[W/H]`                    | Maintain ratio (prevents CLS)   |
| `w-full`                          | Fill parent width               |
| `object-contain` / `object-cover` | Image fit mode                  |

### Loading Strategy

- `loading="eager"`: Above-the-fold images (LCP optimization)
- `loading="lazy"`: Below-the-fold images (initial load optimization)

## 4. Tailwind v4 Class Changes

> This project uses **Tailwind v4**. Do NOT use v3 syntax.

### Import Syntax

```css
/* ❌ v3 (forbidden) */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* ✅ v4 */
@import 'tailwindcss';
```

### Class Migration Table

| v3 (forbidden) | v4 (use this) |
| -------------- | ------------- |
| `shadow-sm`    | `shadow-xs`   |
| `shadow`       | `shadow-sm`   |
| `ring`         | `ring-3`      |
| `blur`         | `blur-sm`     |
| `rounded`      | `rounded-sm`  |

Reference: [Tailwind v4 Upgrade Guide](https://tailwindcss.com/docs/upgrade-guide)
