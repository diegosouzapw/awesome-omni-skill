---
name: Frontend Design
description: Create sophisticated, luxury UI using Tailwind CSS and shadcn/ui with Black & Rose Gold theme. Use when designing pages, creating components, styling interfaces, implementing the Strictly Dancing design system. Triggers on design, UI, component, styling, shadcn, Tailwind, theme, luxury, elegant, beautiful.
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep]
---

# Frontend Design Skill

## Purpose

Create sophisticated, elegant UI components following the Strictly Dancing **Black & Rose Gold** design system using Tailwind CSS v4 and shadcn/ui.

## When to Use This Skill

- Designing new pages or layouts
- Creating UI components
- Styling interfaces
- Implementing responsive designs
- Adding animations and transitions
- Working with the design system

## Brand Identity

### Philosophy

**Timeless elegance meets modern luxury** - The Strictly Dancing brand combines:
- Classic black for sophisticated authority
- Rose gold accents for warmth and premium feel
- Elegant serif typography for display text
- Clean sans-serif for readability

### Color Palette

#### Primary Colors

| Token | Hex | Usage |
|-------|-----|-------|
| `black` / `foreground` | `#0a0a0a` | Primary text, buttons |
| `white` / `background` | `#fffbf7` | Backgrounds (warm white) |

#### Rose Gold Accent Scale

| Token | Hex | Usage |
|-------|-----|-------|
| `rose-gold-50` | `#fdf4f0` | Subtle tints |
| `rose-gold-100` | `#fbe8e0` | Light backgrounds |
| `rose-gold-200` | `#f5d4c8` | Hover states |
| `rose-gold-300` | `#f0b89a` | Dark mode accent |
| `rose-gold-400` | `#e5a88f` | **Primary rose gold** |
| `rose-gold-500` | `#d4967d` | Medium tone |
| `rose-gold-600` | `#c8856c` | Dark rose gold |
| `rose-gold-700` | `#a66b55` | Shadows |

#### Deep Rose (CTAs)

| Token | Hex | Usage |
|-------|-----|-------|
| `rose-600` | `#e11d48` | Primary CTAs |
| `rose-700` | `#be123c` | CTA hover |

### Typography

**Display Font**: `Cormorant Garamond` (serif)
- Used for: H1-H4, wordmark, special callouts
- Weights: 400 (regular), 500, 600 (semi-bold), 700 (bold)
- Conveys: Elegance, sophistication, dance heritage

**Body Font**: `Inter` (sans-serif)
- Used for: Body text, UI elements, buttons, navigation
- Weights: 400 (regular), 500 (medium), 600 (semi-bold), 700 (bold)
- Conveys: Modern, clean, highly readable

#### Usage in Tailwind

```tsx
// Display headings (serif)
<h1 className="font-display text-4xl font-bold">Find Your Dance Host</h1>

// Body text (sans-serif - default)
<p className="text-base text-muted-foreground">Discover qualified hosts...</p>
```

### Shadows

```css
/* Elegant subtle shadow */
--shadow-elegant: 0 0 40px rgba(229, 168, 143, 0.15);

/* Rose gold glow effect */
--shadow-rose-gold: 0 4px 20px rgba(229, 168, 143, 0.25);
```

Use classes: `glow-elegant` or `glow-rose-gold`

## Component Patterns

### Buttons

```tsx
import { Button } from '@/components/ui/button'

// Primary (black background)
<Button>Book Now</Button>

// Secondary (outline)
<Button variant="outline">View Profile</Button>

// Ghost (minimal)
<Button variant="ghost">Cancel</Button>

// Rose gold accent (custom)
<Button className="btn-rose-gold">Premium Feature</Button>

// Destructive
<Button variant="destructive">Delete</Button>
```

### Cards

```tsx
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card'

<Card className="card-hover">
  <CardHeader>
    <CardTitle className="font-display">Dance Host</CardTitle>
  </CardHeader>
  <CardContent>
    <p className="text-muted-foreground">Content here...</p>
  </CardContent>
</Card>
```

### Forms

```tsx
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'

<div className="space-y-2">
  <Label htmlFor="email">Email</Label>
  <Input id="email" type="email" placeholder="you@example.com" />
</div>
```

### Badges

```tsx
import { Badge } from '@/components/ui/badge'

<Badge>Verified</Badge>
<Badge variant="secondary">New</Badge>
<Badge variant="outline">Popular</Badge>
```

### Toasts (Sonner)

```tsx
import { toast } from 'sonner'

// Success
toast.success('Booking confirmed!')

// Error
toast.error('Payment failed')

// Info
toast.info('New message received')
```

## Custom Utility Classes

### Defined in index.css

```css
/* Rose gold glow effect */
.glow-rose-gold { box-shadow: var(--shadow-rose-gold); }
.glow-elegant { box-shadow: var(--shadow-elegant); }

/* Text gradient */
.text-gradient-rose-gold {
  @apply bg-gradient-to-r from-rose-gold-400 via-rose-gold-500 to-rose-gold-600 bg-clip-text text-transparent;
}

/* Smooth transitions */
.transition-elegant { @apply transition-all duration-300 ease-out; }

/* Card hover effect */
.card-hover { @apply transition-elegant hover:shadow-lg hover:-translate-y-0.5; }

/* Rose gold button */
.btn-rose-gold { @apply bg-rose-gold-400 text-foreground hover:bg-rose-gold-500 transition-elegant; }

/* Gradient border */
.border-gradient-rose-gold { /* Creates rose gold gradient border */ }
```

## Dark Mode

The design system supports dark mode via the `.dark` class on the HTML element.

### Theme Toggle

```tsx
import { ThemeToggle } from '@/components/ui/theme-toggle'

<ThemeToggle />  // Toggles between light/dark/system
```

### Hook

```tsx
import { useTheme } from '@/components/theme-provider'

const { theme, setTheme, resolvedTheme } = useTheme()
```

### Conditional Classes

```tsx
// Using Tailwind's dark: variant
<div className="bg-white dark:bg-card">
  <p className="text-black dark:text-white">Adapts to theme</p>
</div>
```

## Responsive Design

**Mobile-first approach** using Tailwind breakpoints:

| Breakpoint | Min Width | Usage |
|------------|-----------|-------|
| `sm` | 640px | Small tablets |
| `md` | 768px | Tablets |
| `lg` | 1024px | Laptops |
| `xl` | 1280px | Desktops |
| `2xl` | 1536px | Large screens |

```tsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {/* Cards stack on mobile, 2 cols on tablet, 3 on desktop */}
</div>
```

## Animation Patterns

### Elegant Transitions

Use `transition-elegant` for smooth 300ms ease-out animations:

```tsx
<button className="transition-elegant hover:scale-105">
  Hover me
</button>
```

### Dance-Inspired Motion

For special elements, use flowing, graceful animations:

```tsx
// Subtle float animation
<div className="animate-pulse">...</div>

// Custom keyframes in index.css for dance-like motion
@keyframes sway {
  0%, 100% { transform: rotate(-2deg); }
  50% { transform: rotate(2deg); }
}
```

## Accessibility

### Requirements

- WCAG AA contrast ratios (4.5:1 for normal text)
- Focus indicators on all interactive elements
- Proper heading hierarchy
- ARIA labels where needed
- Keyboard navigation support

### Built-in Support

shadcn/ui components include:
- Proper ARIA attributes
- Keyboard navigation
- Focus management
- Screen reader support

## File Locations

| File | Purpose |
|------|---------|
| `frontend/src/index.css` | Design tokens, utilities |
| `frontend/src/components/ui/` | shadcn components |
| `frontend/src/components/theme-provider.tsx` | Dark mode |
| `frontend/public/logo.svg` | Light mode logo |
| `frontend/public/logo-dark.svg` | Dark mode logo |
| `frontend/public/favicon.svg` | Browser favicon |

## Examples

### Hero Section

```tsx
<section className="relative min-h-[60vh] flex items-center justify-center bg-background">
  <div className="container mx-auto px-4 text-center">
    <h1 className="font-display text-5xl md:text-6xl lg:text-7xl font-bold tracking-tight mb-6">
      Find Your Perfect
      <span className="text-gradient-rose-gold"> Dance Host</span>
    </h1>
    <p className="text-xl text-muted-foreground max-w-2xl mx-auto mb-8">
      Connect with qualified dance hosts worldwide for unforgettable experiences.
    </p>
    <div className="flex gap-4 justify-center">
      <Button size="lg">Browse Hosts</Button>
      <Button size="lg" variant="outline">Become a Host</Button>
    </div>
  </div>
</section>
```

### Host Card

```tsx
<Card className="card-hover overflow-hidden">
  <div className="aspect-square relative">
    <img src={host.photo} alt={host.name} className="object-cover w-full h-full" />
    {host.verified && (
      <Badge className="absolute top-3 right-3 bg-rose-gold-400">Verified</Badge>
    )}
  </div>
  <CardContent className="p-4">
    <h3 className="font-display text-xl font-semibold">{host.name}</h3>
    <p className="text-muted-foreground text-sm">{host.location}</p>
    <div className="flex items-center gap-1 mt-2">
      <Star className="h-4 w-4 fill-rose-gold-400 text-rose-gold-400" />
      <span className="font-medium">{host.rating}</span>
      <span className="text-muted-foreground">({host.reviews} reviews)</span>
    </div>
  </CardContent>
</Card>
```

## Integration

This skill works with:
- **TanStack Router** - For page layouts and navigation
- **TanStack Query** - For data-driven UI
- **shadcn/ui** - Component library foundation
