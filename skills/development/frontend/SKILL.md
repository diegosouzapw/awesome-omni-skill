---
name: frontend
description: Apply distinctive frontend design to React + TailwindCSS apps. Use when building UI components, pages, or improving visual design. Breaks generic "AI slop" patterns.
allowed-tools: Read, Write, Edit, Glob, WebSearch, WebFetch
user-invocable: true
arguments: "[component-or-page-name]"
---
# Frontend Design System

Apply this guidance when building UI in apps created with app-builder. The goal: distinctive, cohesive design that avoids generic AI patterns.

### Weight Contrast

Use extreme weight differences. Not `400` vs `600`—use `100` vs `900`, `200` vs `800`.

```tsx
<h1 className="font-black text-5xl tracking-tight">Bold Statement</h1>
<p className="font-light text-lg tracking-wide">Lighter supporting text</p>
```

### Add to Tailwind config or use inline styles for font-family.

## Color Strategy

**Reject:** Evenly-distributed palettes, purple gradients, teal-to-blue
**Embrace:** Dominant color with sharp accents, drawn from real aesthetics

### Usage in Tailwind

```tsx
<div className="bg-[--color-surface] text-[--color-text]">
  <span className="text-[--color-accent]">Highlighted</span>
  <p className="text-[--color-text-muted]">Secondary info</p>
</div>
```

## Motion

**Reject:** Scattered micro-interactions, framer-motion for everything
**Embrace:** One orchestrated moment, CSS-first, purposeful

### Page Load Animation (CSS-only)

```css
@keyframes fade-up {
  from {
    opacity: 0;
    transform: translateY(8px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-in {
  animation: fade-up 0.5s ease-out forwards;
}

.animate-delay-100 { animation-delay: 100ms; }
.animate-delay-200 { animation-delay: 200ms; }
.animate-delay-300 { animation-delay: 300ms; }
```

### Staggered Reveal Pattern

```tsx
<main className="space-y-6">
  <h1 className="animate-in opacity-0">Welcome</h1>
  <p className="animate-in animate-delay-100 opacity-0">Subtitle</p>
  <div className="animate-in animate-delay-200 opacity-0">
    <Button>Get Started</Button>
  </div>
</main>
```

### Interactive States (subtle)

```tsx
<Button className="
  transition-all duration-150
  hover:translate-y-[-1px] hover:shadow-lg
  active:translate-y-0 active:shadow-md
">
  Click me
</button>
```

## Backgrounds & Atmosphere

**Reject:** Solid `bg-white` or `bg-gray-900`
**Embrace:** Layered depth, subtle gradients, geometric patterns

### Gradient Mesh (dark mode)

```tsx
<div className="relative min-h-screen bg-[#0a0a0f] overflow-hidden">
  {/* Ambient gradient blobs */}
  <div className="absolute top-0 left-1/4 w-96 h-96 bg-purple-500/10 rounded-full blur-3xl" />
  <div className="absolute bottom-0 right-1/4 w-96 h-96 bg-orange-500/10 rounded-full blur-3xl" />

  {/* Content */}
  <div className="relative z-10">
    {/* ... */}
  </div>
</div>
```

### Subtle Grid Pattern

```css
.bg-grid {
  background-image:
    linear-gradient(to right, rgba(255,255,255,0.03) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(255,255,255,0.03) 1px, transparent 1px);
  background-size: 24px 24px;
}
```

### Noise Texture (add grain)

```css
.bg-noise {
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.65' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%' height='100%' filter='url(%23noise)'/%3E%3C/svg%3E");
  opacity: 0.03;
}
```

## Component Patterns

### Card with Glow Border

```tsx
<div className="
  relative p-6 rounded-xl
  bg-[--color-surface-raised]
  border border-white/5
  hover:border-[--color-accent]/30
  transition-colors
">
  <div className="absolute inset-0 rounded-xl bg-gradient-to-r from-[--color-accent]/10 to-transparent opacity-0 hover:opacity-100 transition-opacity" />
  <div className="relative">
    {/* Content */}
  </div>
</div>
```

### Status Badge

```tsx
<Badge className="
  inline-flex items-center gap-1.5 px-2.5 py-1
  text-xs font-medium tracking-wide uppercase
  rounded-full
  bg-emerald-500/10 text-emerald-400
">
  <span className="w-1.5 h-1.5 rounded-full bg-emerald-400 animate-pulse" />
  Active
</Badge>
```

### Input with Focus Ring

```tsx
<Input className="
  w-full px-4 py-3
  bg-[--color-surface] text-[--color-text]
  border border-white/10 rounded-lg
  placeholder:text-[--color-text-muted]
  focus:outline-none focus:ring-2 focus:ring-[--color-accent]/50 focus:border-transparent
  transition-shadow
" />
```

## Quick Start Checklist

When building a new page/component:

1. [ ] Pick a font pairing from the table above
2. [ ] Define CSS variables for your color theme
3. [ ] Add one staggered entrance animation
4. [ ] Use a background with depth (gradient blobs, grid, or noise)
5. [ ] Ensure weight contrast in typography (not just size)
6. [ ] Add subtle hover states with transforms (not just color changes)

## Anti-Patterns to Avoid

- Generic shadows (`shadow-lg` without context)
- Rainbow gradients or purple-to-blue
- Identical font weights throughout
- Solid color backgrounds without texture
- Motion on every element
- Default Tailwind colors without customization

## Component Library

Shadcn UI components are available at `libs/frontend/src/components/ui/`. Use these as building blocks:

```tsx
import { Button } from "@vaatun/frontend/components/ui/button";
import { Input } from "@vaatun/frontend/components/ui/input";
import { Card, CardHeader, CardContent } from "@vaatun/frontend/components/ui/card";
```

Apply the design principles above to customize these base components.