---
name: frontend-build
description: Production-grade frontend development with distinctive design. Activates for "build UI", "create component", "landing page", "dashboard", "form", "responsive", "tailwind", "frontend", "design", "React", "Next.js" requests.
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep, Task]
---

# Frontend Build Protocol

## When This Skill Activates
- "Build a component", "create UI", "design this"
- "Landing page", "dashboard", "form"
- "Make it responsive", "mobile-friendly"
- "Tailwind", "shadcn", "styling"
- "React", "Next.js", "frontend"

## BEFORE WRITING CODE

Establish a **bold aesthetic direction**. Ask yourself:

1. **Purpose**: What problem does this solve? Who uses it?
2. **Tone**: Minimalist? Maximalist? Retro? Brutalist? Warm? Clinical?
3. **Differentiator**: What ONE thing makes this memorable?
4. **Constraints**: Technical limitations? Brand guidelines?

> "A design with clear intent—whether maximalist or minimal—always beats scattered, unfocused choices."

## Design Philosophy

### The Core Principle
**Distinctive, production-grade interfaces that avoid generic AI aesthetics.**

Every design choice should be intentional:
- If it's minimal, it should feel *deliberately* restrained
- If it's bold, it should feel *confidently* expressive
- Never default to "safe" choices

### Anti-Patterns (AI Slop to Avoid)

| DON'T | DO INSTEAD |
|-------|------------|
| Center everything | Asymmetric, purposeful alignment |
| Purple gradients everywhere | Restrained color with sharp accents |
| Uniform rounded-lg on all | Mix radiuses intentionally |
| Inter font (default) | System fonts or distinctive choices |
| Blue-500 for everything | Custom accent that fits context |
| Generic card grids | Varied, hierarchical layouts |
| Over-animated everything | Strategic motion for key moments |
| Predictable "hero → features → CTA" | Unexpected flow that serves content |

## Typography That Elevates

### Font Pairings (Non-Default)
```css
/* Option 1: Modern Professional */
--font-heading: 'Satoshi', system-ui;
--font-body: 'Inter', system-ui;

/* Option 2: Editorial */
--font-heading: 'Playfair Display', Georgia;
--font-body: 'Source Sans 3', system-ui;

/* Option 3: Technical */
--font-heading: 'Space Grotesk', system-ui;
--font-body: 'IBM Plex Sans', system-ui;

/* Option 4: System (fast, still good) */
--font-heading: system-ui;
--font-body: system-ui;
```

### Type Scale
```tsx
// Limit to 4-5 sizes max
<h1 className="text-4xl md:text-5xl lg:text-6xl font-bold tracking-tight" />
<h2 className="text-2xl md:text-3xl font-semibold" />
<h3 className="text-xl font-medium" />
<p className="text-base text-neutral-600 leading-relaxed" />
<small className="text-sm text-neutral-500" />
```

## Color as Atmosphere

### Curated Palettes (Not Defaults)

**Warm Professional:**
```
Background: #FAF9F7 (warm white)
Surface:    #FFFFFF
Text:       #1A1A19
Accent:     #D97757 (terracotta)
Muted:      #9A9590
Border:     #E8E6E3
```

**Cool Technical:**
```
Background: #F8FAFC
Surface:    #FFFFFF
Text:       #0F172A
Accent:     #3B82F6 (clear blue)
Muted:      #64748B
Border:     #E2E8F0
```

**Dark Mode Done Right:**
```
Background: #0A0A0A
Surface:    #141414
Text:       #FAFAFA
Accent:     #22D3EE (cyan)
Muted:      #71717A
Border:     #27272A
```

### Color Application
```tsx
// Primary actions: accent color
<button className="bg-[#D97757] hover:bg-[#C4684A]">

// Secondary: subtle, don't compete
<button className="bg-neutral-100 text-neutral-900 hover:bg-neutral-200">

// Destructive: red but not alarming
<button className="bg-red-600/90 hover:bg-red-600">
```

## Spatial Composition

### Break the Grid (Intentionally)
```tsx
// Instead of equal columns, use weighted grids
<div className="grid grid-cols-1 lg:grid-cols-[1.4fr,1fr] gap-12">

// Overlap elements for depth
<div className="relative">
  <div className="absolute -top-4 -right-4 w-24 h-24 bg-accent/10 rounded-full" />
  <div className="relative bg-white p-8 rounded-xl">Content</div>
</div>

// Negative space as design element
<section className="py-32 lg:py-48">  {/* Generous vertical space */}
```

### Asymmetric Hero
```tsx
export function Hero() {
  return (
    <section className="relative min-h-[80vh] flex items-center overflow-hidden">
      {/* Background texture */}
      <div className="absolute inset-0 bg-gradient-to-br from-neutral-950 via-neutral-900 to-neutral-950" />

      <div className="relative mx-auto max-w-7xl px-6 py-24">
        <div className="grid lg:grid-cols-[1.3fr,1fr] gap-16 items-center">
          <div className="space-y-8">
            <div className="inline-flex items-center gap-2 px-3 py-1 rounded-full
                          bg-white/10 text-white/80 text-sm">
              <span className="w-1.5 h-1.5 rounded-full bg-emerald-400" />
              Now available
            </div>

            <h1 className="text-5xl lg:text-7xl font-bold text-white tracking-tight">
              Build products
              <br />
              <span className="text-transparent bg-clip-text bg-gradient-to-r
                             from-orange-400 to-amber-400">
                that matter.
              </span>
            </h1>

            <p className="text-lg text-neutral-400 max-w-lg leading-relaxed">
              Ship in days, not months. Focus on what matters while we
              handle the complexity.
            </p>

            <div className="flex flex-wrap gap-4 pt-4">
              <button className="px-6 py-3 bg-white text-neutral-900 rounded-lg
                               font-medium hover:bg-neutral-100 transition-colors">
                Get Started
              </button>
              <button className="px-6 py-3 text-white border border-white/20 rounded-lg
                               font-medium hover:bg-white/10 transition-colors">
                Learn More
              </button>
            </div>
          </div>

          <div className="hidden lg:block relative">
            {/* Visual element - not just an image placeholder */}
            <div className="aspect-square rounded-3xl bg-gradient-to-br
                          from-orange-500/20 to-amber-500/20 backdrop-blur-3xl" />
          </div>
        </div>
      </div>
    </section>
  );
}
```

## Motion with Purpose

### When to Animate
- **Hover states**: Subtle feedback (150-200ms)
- **State changes**: Loading → loaded, collapsed → expanded
- **Entrances**: First-time visibility (staggered, not all at once)
- **High-impact moments**: Success, celebration, key actions

### When NOT to Animate
- Every scroll event
- All elements at once
- Continuous loops (distracting)
- Required interactions (don't delay users)

### Motion Patterns
```tsx
// Micro-interaction: subtle scale
className="transition-transform duration-150 hover:scale-[1.02] active:scale-[0.98]"

// State transition: smooth height
className="transition-all duration-300 ease-out overflow-hidden"

// Entrance: fade up (use sparingly)
const variants = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0, transition: { duration: 0.4 } }
};
```

## Component Library

### Card (Refined)
```tsx
interface CardProps {
  title: string;
  description: string;
  highlight?: boolean;
}

export function Card({ title, description, highlight }: CardProps) {
  return (
    <div className={`
      group relative p-6 rounded-2xl border transition-all duration-200
      ${highlight
        ? 'bg-neutral-900 text-white border-neutral-800 hover:border-neutral-700'
        : 'bg-white border-neutral-200 hover:border-neutral-300 hover:shadow-lg'}
    `}>
      <h3 className={`text-lg font-semibold ${highlight ? 'text-white' : 'text-neutral-900'}`}>
        {title}
      </h3>
      <p className={`mt-2 text-sm leading-relaxed ${highlight ? 'text-neutral-400' : 'text-neutral-600'}`}>
        {description}
      </p>
      <span className={`
        mt-4 inline-flex items-center gap-1 text-sm font-medium
        group-hover:gap-2 transition-all
        ${highlight ? 'text-orange-400' : 'text-orange-600'}
      `}>
        Learn more
        <span aria-hidden>→</span>
      </span>
    </div>
  );
}
```

### Button System
```tsx
const variants = {
  primary: `
    bg-neutral-900 text-white border border-transparent
    hover:bg-neutral-800 active:scale-[0.98]
  `,
  secondary: `
    bg-white text-neutral-900 border border-neutral-200
    hover:bg-neutral-50 hover:border-neutral-300
  `,
  ghost: `
    text-neutral-600 hover:text-neutral-900 hover:bg-neutral-100
  `,
  accent: `
    bg-orange-500 text-white border border-transparent
    hover:bg-orange-600 active:scale-[0.98]
  `,
};

const sizes = {
  sm: 'px-3 py-1.5 text-sm',
  md: 'px-4 py-2 text-sm',
  lg: 'px-6 py-3 text-base',
};

export function Button({
  variant = 'primary',
  size = 'md',
  children,
  ...props
}) {
  return (
    <button
      className={`
        inline-flex items-center justify-center gap-2 rounded-lg font-medium
        transition-all duration-150 disabled:opacity-50 disabled:cursor-not-allowed
        ${variants[variant]} ${sizes[size]}
      `}
      {...props}
    >
      {children}
    </button>
  );
}
```

### Input with Polish
```tsx
export function Input({ label, error, hint, ...props }) {
  return (
    <div className="space-y-1.5">
      {label && (
        <label className="block text-sm font-medium text-neutral-700">
          {label}
        </label>
      )}
      <input
        className={`
          w-full px-3.5 py-2.5 rounded-lg border bg-white text-neutral-900
          placeholder:text-neutral-400 transition-all duration-150
          focus:outline-none focus:ring-2 focus:ring-offset-1
          ${error
            ? 'border-red-500 focus:ring-red-500/20'
            : 'border-neutral-200 hover:border-neutral-300 focus:border-neutral-900 focus:ring-neutral-900/10'}
        `}
        {...props}
      />
      {error && <p className="text-sm text-red-600">{error}</p>}
      {hint && !error && <p className="text-sm text-neutral-500">{hint}</p>}
    </div>
  );
}
```

## Dashboard Layout
```tsx
export function DashboardLayout({ children }) {
  return (
    <div className="min-h-screen bg-neutral-50">
      {/* Sidebar */}
      <aside className="fixed inset-y-0 left-0 w-64 bg-white border-r border-neutral-200">
        <div className="h-16 flex items-center px-6 border-b border-neutral-200">
          <span className="text-lg font-semibold">Dashboard</span>
        </div>
        <nav className="p-4 space-y-1">
          <NavItem icon="home" label="Overview" active />
          <NavItem icon="chart" label="Analytics" />
          <NavItem icon="users" label="Customers" />
          <NavItem icon="settings" label="Settings" />
        </nav>
      </aside>

      {/* Main */}
      <main className="pl-64">
        <header className="sticky top-0 z-10 h-16 bg-white/80 backdrop-blur-sm
                         border-b border-neutral-200 flex items-center px-6">
          <div className="flex-1" />
          <div className="flex items-center gap-4">
            <button className="p-2 hover:bg-neutral-100 rounded-lg transition-colors">
              <BellIcon />
            </button>
            <div className="w-8 h-8 rounded-full bg-neutral-200" />
          </div>
        </header>
        <div className="p-6 lg:p-8">
          {children}
        </div>
      </main>
    </div>
  );
}
```

## Responsive Strategy

**Mobile-first, breakpoint-intentional:**
```tsx
// Not just "add columns at breakpoints"
// Consider what changes at each size

<section className="
  py-12         // Mobile: tighter
  md:py-16      // Tablet: medium
  lg:py-24      // Desktop: generous

  px-4          // Mobile: edge-to-edge
  md:px-8       // Tablet: some breathing room
  lg:px-0       // Desktop: centered container
">
```

## Accessibility Checklist

- [ ] Semantic HTML (button for actions, a for links)
- [ ] Heading hierarchy (h1 → h2 → h3, no skips)
- [ ] Color contrast ≥ 4.5:1 (use WebAIM checker)
- [ ] Focus states visible (focus-visible, not just focus)
- [ ] Alt text for meaningful images
- [ ] Keyboard navigable (Tab, Enter, Escape)
- [ ] aria-labels on icon-only buttons
- [ ] Reduced motion support (@media (prefers-reduced-motion))

## Output Format

```markdown
## Frontend Build: [Component/Page Name]

### Design Direction
- Tone: [Minimal/Bold/Warm/etc.]
- Key differentiator: [One memorable choice]

### Components Created
1. [Component 1] - [Purpose]
2. [Component 2] - [Purpose]

### Styling Approach
- Colors: [Palette used]
- Typography: [Fonts, scale]
- Motion: [Key animations]

### Accessibility
- [ ] Semantic HTML
- [ ] Keyboard nav
- [ ] Contrast checked

### Files
- [path/to/component.tsx]
```
