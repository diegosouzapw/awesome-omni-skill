---
name: distinctive-frontend-design
description: Creates distinctive, production-grade frontend interfaces that avoid generic AI aesthetics. Guides bold aesthetic direction, typography, color, motion, and spatial composition for memorable UI. Use when building frontend components, pages, applications, or interfaces, or when the user asks for UI/UX design, styling, or visually striking interfaces.
---

# Distinctive Frontend Design

Build frontend interfaces with genuine design intent. Every interface should feel like it was crafted by a human designer with a strong point of view — never like default AI output.

## Project Context

This is a **Next.js App Router** project with the following stack:

- **Styling**: Tailwind CSS v4 (CSS-first config via `@theme inline` in `globals.css`)
- **Fonts**: Currently uses Geist Sans & Geist Mono via `next/font/google` (defined in `app/layout.tsx`)
- **UI Components**: Custom components in `components/ui/` (Button, Input, Select, Dropdown, Textarea, Alert) — uses `cn()` utility for class merging, consistent variant/size patterns
- **Icons**: `@mui/icons-material` (Material UI icons package)
- **Animations**: CSS transitions only (no framer-motion installed). Consider adding `motion` (framer-motion) for complex animations.
- **Theme**: Light mode only (`color-scheme: light`), CSS variables at `:root`
- **Auth**: Clerk with Spanish localization (`lang="es"`)
- **Color system**: `--background: #ffffff`, `--foreground: #171717`, blue primary (`blue-500`/`blue-600`), domain-specific color maps (categories, statuses, deal stages)
- **Layout pattern**: `AppLayout` wrapper with `PageHeader` + `PageContent`, white cards on `bg-gray-50/30`, mobile-responsive with bottom nav

### Working Within This Stack

When building new interfaces in this project:

- **Fonts**: Add new display fonts via `next/font/google` in the component or layout that needs them. Set as CSS variable and reference in Tailwind via arbitrary values `font-[var(--font-name)]` or extend `@theme`.
- **Tailwind v4**: Extend colors/tokens in `globals.css` using `@theme inline { }` — no JS config file.
- **Components**: Follow existing patterns — `cn()` for class merging, variant/size props, consistent API shape. Import from `@/components/ui/`.
- **CSS custom properties**: Define at `:root` in `globals.css` or scoped to component.
- **Animations**: Use CSS `@keyframes` + Tailwind `animate-*` for simple cases. For orchestrated sequences, suggest installing `motion` (framer-motion).

## Design Thinking (Before Code)

Before writing any code, spend time on these four questions:

1. **Purpose**: What problem does this solve? Who is the audience?
2. **Tone**: Commit to a BOLD aesthetic direction. Pick one and go deep:
   - Brutally minimal | Maximalist chaos | Retro-futuristic | Organic/natural
   - Luxury/refined | Playful/toy-like | Editorial/magazine | Brutalist/raw
   - Art deco/geometric | Soft/pastel | Industrial/utilitarian | Neo-grotesque
   - Typographic-forward | Glassmorphic depth | Monochrome drama | Warm analog
   - Use these as inspiration — design something true to the chosen direction.
3. **Constraints**: Performance targets, accessibility, consistency with existing app pages.
4. **Differentiator**: What single element makes this UNFORGETTABLE?

Output a brief design brief (2-4 sentences) before implementation. Example:
> "Dashboard for creative professionals. Art deco geometric aesthetic: sharp angles, gold accent on deep navy, Playfair Display headings with DM Sans body. Differentiator: animated geometric border patterns that respond to data state."

## Core Aesthetic Principles

### Typography

Choose fonts that have genuine character. Pair a distinctive display font with a refined body font.

- For curated pairings organized by aesthetic, see [font-pairings.md](font-pairings.md)
- Load via `next/font/google` — set as CSS variable, use in Tailwind classes
- Set a clear typographic scale (e.g., 1.25 or 1.333 ratio)
- Use `letter-spacing`, `line-height`, and `text-transform` intentionally

**NEVER use**: Inter, Roboto, Arial, Open Sans, Lato, Geist, system-ui as display/heading fonts. These are generic defaults. Geist is fine for body/UI text since it's the project default, but headings and hero text deserve character.

### Color & Theme

Commit fully. Dominant colors with sharp accents beat timid, evenly-distributed palettes.

- Define palette as CSS custom properties at `:root` or scoped via `@theme inline`
- Use a clear hierarchy: 1 dominant, 1-2 accents, neutrals
- This project is light-mode only — avoid pure `#fff`; use warm or cool off-whites
- Respect existing color semantics (blue primary for actions, status colors for states)
- New pages/features can have their own sub-palette while staying coherent with the app

### Motion & Interaction

Prioritize high-impact orchestrated moments over scattered micro-interactions.

- **Page load**: Staggered reveals using `animation-delay` on sequential elements
- **Scroll**: Intersection Observer for scroll-triggered entrances
- **Hover states**: Surprising, delightful transitions (scale, color shift, shadow lift, underline animations)
- **CSS animations**: Use `@keyframes` in global CSS or component-scoped styles
- **For complex motion**: Consider adding `motion` (framer-motion) — `npm install motion`
- One well-orchestrated sequence > many small animations

### Spatial Composition

Break predictable layouts. Not every design needs to be wild — but every design needs intentional spatial choices.

- Asymmetric grids, overlapping elements, diagonal flow
- Grid-breaking hero elements or pull-quotes
- Generous negative space OR controlled density — both are valid; pick one
- Consider viewport-relative sizing (`dvh`, `vw`, `clamp()`)
- Work within the existing `AppLayout` > `PageHeader` + `PageContent` structure when building app pages

### Backgrounds & Visual Depth

Create atmosphere. Solid flat colors are a missed opportunity.

- Gradient meshes, noise/grain textures, geometric patterns
- Layered transparencies and backdrop-filter effects
- Dramatic shadows (layered `box-shadow` stacks, not single default shadows)
- Decorative borders, custom dividers, ornamental elements
- Subtle background patterns or textures that reinforce the aesthetic

## Anti-Patterns (AI Slop Checklist)

NEVER produce interfaces with these characteristics:

| Element | Generic (Avoid) | Distinctive (Do This) |
|---------|------------------|-----------------------|
| Fonts | Inter, Roboto, Arial, Geist for headings | Distinctive display + refined body pairing |
| Colors | Purple gradient on white, blue-to-purple | Context-specific palette with clear hierarchy |
| Shadows | `shadow-md` / single box-shadow | Layered shadow stacks or dramatic directional |
| Borders | `rounded-lg border border-gray-200` | Contextual: sharp edges, ornamental, or none |
| Layout | Card grid, centered hero, symmetric | Asymmetric, overlapping, grid-breaking |
| Spacing | Uniform padding everywhere | Rhythmic variation, generous whitespace |
| Backgrounds | Solid white/gray | Textured, gradient, patterned, atmospheric |
| Animations | Fade-in-up on everything | Orchestrated sequences with intentional timing |

## Implementation Guidelines

### Match Complexity to Vision

- **Maximalist designs**: Elaborate code, extensive animations, layered effects, decorative elements
- **Minimal/refined designs**: Restraint and precision in spacing, typography, subtle details
- Elegance = executing the vision well, regardless of complexity level

### Code Quality

- Production-grade: no placeholder content, no TODO comments, no broken states
- Semantic HTML, accessible by default (`aria-labels`, focus states, contrast ratios)
- CSS custom properties for theming consistency
- Responsive: mobile-first, test mental model across breakpoints
- Follow existing component API conventions (`cn()`, variant props, size props)

### Tailwind v4 in This Project

- Extend tokens in `globals.css` via `@theme inline { --color-*: ...; --font-*: ...; }`
- Use arbitrary values `[...]` liberally to break out of default Tailwind system
- Layer custom CSS (in `globals.css` or component-scoped files) for effects Tailwind can't express
- For new fonts: define in `next/font/google`, set CSS variable, reference as `font-[var(--font-name)]`

### New Component Pattern

Follow existing conventions when creating new UI components:

```tsx
import { cn } from '@/lib/utils';

interface Props {
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  className?: string;
  children: React.ReactNode;
}

export function MyComponent({ variant = 'primary', size = 'md', className, children }: Props) {
  return (
    <div className={cn(
      'base-classes-here',
      variant === 'primary' && 'primary-classes',
      size === 'md' && 'md-classes',
      className
    )}>
      {children}
    </div>
  );
}
```

## Variety Mandate

NEVER converge on the same choices across different generations:
- Use different font families each time
- Vary aesthetic directions (don't always pick the same style)
- Different color palettes, layout approaches, animation strategies

Each interface should feel like it came from a different designer with a different perspective.

## Additional Resources

- For curated font pairings by aesthetic, see [font-pairings.md](font-pairings.md)
- For detailed aesthetic directions with visual descriptions, see [aesthetics-reference.md](aesthetics-reference.md)
