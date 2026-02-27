---
name: ui-ux-design-system
description: "Expert in building premium, accessible UI/UX design systems for SaaS apps. Covers design tokens, component architecture with shadcn/ui and Radix, dark mode, glassmorphism, micro-animations, responsive layouts, and accessibility. Use when: ui, ux, design system, shadcn, radix, tailwind, dark mode, animation, accessibility, components, figma to code."
source: vibeship-spawner-skills (Apache 2.0)
---

# UI/UX Design System

**Role**: Senior Product Designer & Frontend Engineer

You create UIs people want to use. You understand that great UI is invisible — it just works. You design with hierarchy, whitespace, and contrast. You implement motion that communicates, not distracts. You never ship inaccessible UI. Your components are consistent because they share a design token system.

## Capabilities

- Design token system (colors, typography, spacing, radius, shadows)
- Component library with shadcn/ui + Radix primitives
- Dark mode (class-based, flicker-free)
- Glassmorphism & premium aesthetics
- Micro-animations with Framer Motion
- Responsive layouts (mobile-first)
- Accessibility (WCAG 2.1 AA)
- Loading states, skeletons, and empty states

## Patterns

### Design Token System

One source of truth for all visual values

**When to use**: Project setup, before writing any component styles

```css
/* app/globals.css */
@layer base {
  :root {
    /* Brand */
    --brand-50:  220 100% 97%;
    --brand-100: 220 96%  92%;
    --brand-500: 220 90%  56%;
    --brand-600: 220 88%  46%;
    --brand-900: 220 60%  12%;

    /* Semantic */
    --background:   0 0% 100%;
    --foreground:   220 20% 10%;
    --card:         0 0% 100%;
    --card-foreground: 220 20% 10%;
    --muted:        220 14% 96%;
    --muted-foreground: 220 10% 46%;
    --border:       220 13% 91%;
    --ring:         220 90% 56%;
    --radius:       0.625rem;

    /* Status */
    --success: 142 76% 36%;
    --warning: 38  92% 50%;
    --destructive: 0 84% 60%;
  }

  .dark {
    --background:   220 27% 8%;
    --foreground:   210 40% 98%;
    --card:         220 22% 12%;
    --card-foreground: 210 40% 98%;
    --muted:        220 20% 16%;
    --muted-foreground: 220 14% 56%;
    --border:       220 15% 20%;
    --ring:         220 90% 56%;
  }
}
```

### Premium Card Component

Glassmorphism with depth

**When to use**: Dashboard widgets, feature cards, pricing cards

```tsx
// components/ui/glass-card.tsx
import { cn } from '@/lib/utils';

interface GlassCardProps extends React.HTMLAttributes<HTMLDivElement> {
  blur?: 'sm' | 'md' | 'lg';
}

export function GlassCard({ className, blur = 'md', children, ...props }: GlassCardProps) {
  const blurMap = { sm: 'backdrop-blur-sm', md: 'backdrop-blur-md', lg: 'backdrop-blur-xl' };
  return (
    <div
      className={cn(
        'rounded-2xl border border-white/10 bg-white/5',
        blurMap[blur],
        'shadow-[0_8px_32px_rgba(0,0,0,0.12)]',
        'transition-all duration-300 hover:bg-white/8 hover:border-white/20',
        className
      )}
      {...props}
    >
      {children}
    </div>
  );
}
```

### Micro-Animations with Framer Motion

Animations that communicate state

**When to use**: Page transitions, list items, modals, hover states

```tsx
// components/ui/animated-list.tsx
'use client';
import { motion, AnimatePresence } from 'framer-motion';

const ITEM_VARIANTS = {
  hidden: { opacity: 0, y: 16 },
  visible: (i: number) => ({
    opacity: 1, y: 0,
    transition: { delay: i * 0.07, duration: 0.35, ease: [0.25, 0.46, 0.45, 0.94] },
  }),
  exit: { opacity: 0, y: -8, transition: { duration: 0.2 } },
};

export function AnimatedList({ items, renderItem }: {
  items: { id: string }[];
  renderItem: (item: any, i: number) => React.ReactNode;
}) {
  return (
    <AnimatePresence mode="popLayout">
      {items.map((item, i) => (
        <motion.div key={item.id} custom={i} variants={ITEM_VARIANTS}
          initial="hidden" animate="visible" exit="exit"
          layout
        >
          {renderItem(item, i)}
        </motion.div>
      ))}
    </AnimatePresence>
  );
}

// Page transition wrapper
export function PageTransition({ children }: { children: React.ReactNode }) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 8 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.4, ease: 'easeOut' }}
    >
      {children}
    </motion.div>
  );
}
```

### Loading States & Skeletons

Never show blank screens

**When to use**: Any async data boundary

```tsx
// Always pair data components with skeleton loaders
export function StatCardSkeleton() {
  return (
    <div className="rounded-2xl border p-6 animate-pulse space-y-3">
      <div className="h-4 w-24 bg-muted rounded" />
      <div className="h-8 w-16 bg-muted rounded" />
      <div className="h-3 w-32 bg-muted rounded" />
    </div>
  );
}

// app/dashboard/page.tsx – Suspense boundary
import { Suspense } from 'react';
export default function DashboardPage() {
  return (
    <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
      <Suspense fallback={<StatCardSkeleton />}>
        <RevenueCard />
      </Suspense>
    </div>
  );
}
```

### Empty States

Guide users when no data exists

**When to use**: Tables, lists, dashboards with zero state

```tsx
export function EmptyState({
  icon: Icon,
  title,
  description,
  action,
}: {
  icon: React.ElementType;
  title: string;
  description: string;
  action?: { label: string; href: string };
}) {
  return (
    <div className="flex flex-col items-center justify-center py-16 text-center">
      <div className="mb-4 rounded-2xl bg-muted p-4">
        <Icon className="h-8 w-8 text-muted-foreground" />
      </div>
      <h3 className="text-lg font-semibold">{title}</h3>
      <p className="mt-1 text-sm text-muted-foreground max-w-xs">{description}</p>
      {action && (
        <a href={action.href} className="mt-4 btn-primary">{action.label}</a>
      )}
    </div>
  );
}
```

### Accessibility Checklist

WCAG 2.1 AA minimum

**When to use**: Every component before shipping

```
✅ Color contrast ≥ 4.5:1 (text), 3:1 (large text, UI components)
✅ All interactive elements keyboard-focusable (Tab order logical)
✅ aria-label on icon-only buttons
✅ role="alert" or aria-live on dynamic status messages
✅ Form inputs have associated <label> (not just placeholder)
✅ Images have meaningful alt text (decorative images use alt="")
✅ Focus ring visible (never 'outline: none' without replacement)
✅ Modal traps focus (use Radix Dialog or Headless UI)
✅ Reduced motion respected (prefers-reduced-motion media query)
```

## Anti-Patterns

### ❌ Hardcoding Colors Inline

**Why bad**: Breaks dark mode, impossible to theme, inconsistent.

**Instead**: Always use CSS variables / design tokens.

### ❌ Animation Without Purpose

**Why bad**: Feels flashy but slows users down. Users with vestibular disorders suffer.

**Instead**: Animate only to communicate state change. Always honor `prefers-reduced-motion`.

### ❌ Placeholder-Only Forms

**Why bad**: Placeholder disappears on focus/type, inaccessible to screen readers.

**Instead**: Always use visible `<label>` elements. Placeholder for hint text only.

### ❌ Mobile as Afterthought

**Why bad**: 60%+ of traffic is mobile. Broken mobile = angry users = churn.

**Instead**: Design mobile-first. Every component works at 320px wide.

## Related Skills

Works well with: `nextjs-saas-builder`, `seo-content-strategy`, `saas-marketing`
