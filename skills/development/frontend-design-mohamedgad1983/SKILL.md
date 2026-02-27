---
name: frontend-design
description: Create distinctive, production-grade frontend interfaces with high design quality. Use when building web components, pages, dashboards, React components, HTML/CSS layouts, or styling any web UI. Generates creative, polished code that avoids generic AI aesthetics.
---

# Frontend Design Skill

## Purpose
Generate distinctive, production-ready UI that avoids the generic "AI look" (Inter fonts, purple gradients, minimal animations).

## Design Principles

### Typography
- **Avoid**: Inter, Roboto, system defaults
- **Prefer**: Distinctive fonts like Space Grotesk, Outfit, Sora, Manrope, Plus Jakarta Sans
- **Hierarchy**: Use bold size contrasts (48px+ headers, 14-16px body)
- **Weight variety**: Mix weights for visual interest

### Color Strategy
- **Avoid**: Purple gradients, pure black/white, default blues
- **Prefer**: Rich, saturated palettes with personality
- **Dark themes**: Use deep blues (#0a0f1a), warm blacks (#1a1a2e)
- **Accents**: Bold, unexpected colors (coral, electric blue, gold)

### Backgrounds
- **Avoid**: Solid flat colors
- **Prefer**: Atmospheric effects - subtle gradients, noise textures, mesh gradients
- **Techniques**: CSS backdrop-filter, layered gradients, animated backgrounds

### Animation & Motion
- **Avoid**: No animation or jarring transitions
- **Prefer**: Purposeful micro-interactions
- **Elements**: Hover states, loading feedback, scroll reveals
- **Timing**: Use spring physics (0.3-0.5s with easing)

### Layout
- **Avoid**: Generic centered layouts
- **Prefer**: Asymmetric, editorial layouts with visual tension
- **Spacing**: Generous whitespace, consistent rhythm
- **Grids**: CSS Grid for complex layouts, Flexbox for components

## Tech Stack Recommendations

### React Projects
```jsx
// Use Tailwind CSS + shadcn/ui + Framer Motion
import { motion } from 'framer-motion';
import { Button } from '@/components/ui/button';
```

### Styling
- Tailwind CSS for utility-first styling
- CSS custom properties for theming
- shadcn/ui for accessible components

### Animations
- Framer Motion for React
- CSS transitions for simple effects
- Spring-based timing functions

## Component Patterns

### Cards
```jsx
<div className="group relative overflow-hidden rounded-2xl bg-gradient-to-br from-slate-900 to-slate-800 p-6 transition-all hover:scale-[1.02] hover:shadow-2xl">
  <div className="absolute inset-0 bg-gradient-to-r from-blue-500/10 to-purple-500/10 opacity-0 transition-opacity group-hover:opacity-100" />
  {/* content */}
</div>
```

### Buttons
```jsx
<button className="relative overflow-hidden rounded-full bg-gradient-to-r from-indigo-500 to-purple-600 px-8 py-3 font-semibold text-white transition-all hover:shadow-lg hover:shadow-indigo-500/25">
  <span className="relative z-10">Get Started</span>
</button>
```

### Input Fields
```jsx
<input className="w-full rounded-xl border-2 border-slate-700 bg-slate-800/50 px-4 py-3 text-white placeholder-slate-400 backdrop-blur transition-all focus:border-indigo-500 focus:outline-none focus:ring-4 focus:ring-indigo-500/20" />
```

## Dashboard Patterns

### KPI Cards
- Large numbers with trend indicators
- Subtle background gradients
- Icon badges with soft shadows

### Data Tables
- Zebra striping with hover states
- Sticky headers
- Inline actions on hover

### Charts
- Use Recharts or Chart.js
- Match chart colors to theme
- Add subtle grid lines

## Instructions

1. **Analyze the context**: Determine if it's a landing page, dashboard, app, or component
2. **Choose fonts**: Select 1-2 distinctive fonts
3. **Define palette**: Create cohesive color scheme with 1-2 accent colors
4. **Plan layout**: Sketch the visual hierarchy mentally
5. **Build mobile-first**: Start with mobile, enhance for larger screens
6. **Add motion**: Include subtle animations for polish
7. **Test contrast**: Ensure accessibility (WCAG 2.1 AA minimum)

## Examples

### Landing Page
- Hero with gradient mesh background
- Floating navigation with backdrop blur
- CTA buttons with glow effects
- Testimonial cards with avatar borders

### Admin Dashboard
- Sidebar with icon navigation
- Header with search and notifications
- Card-based stats grid
- Data table with filters

### Mobile App UI
- Bottom tab navigation
- Pull-to-refresh patterns
- Swipe gestures
- Haptic feedback indicators

## Anti-Patterns to Avoid

❌ Inter or Roboto fonts everywhere
❌ Purple-to-blue gradients
❌ Pure white backgrounds
❌ No hover states
❌ Generic Bootstrap look
❌ Centered everything
❌ No visual hierarchy
❌ Missing loading states

## Quality Checklist

- [ ] Typography has personality
- [ ] Colors are cohesive and branded
- [ ] Backgrounds have depth
- [ ] Animations are smooth
- [ ] Layout has visual rhythm
- [ ] Components are accessible
- [ ] Mobile responsive
- [ ] Dark mode considered
