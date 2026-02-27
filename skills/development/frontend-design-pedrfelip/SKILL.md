---
name: frontend-design
description: Create distinctive, production-grade frontend interfaces with high design quality. Use this skill when the user asks to build web components, pages, artifacts, posters, or applications (examples include websites, landing pages, dashboards, React components, HTML/CSS layouts, or when styling/beautifying any web UI). Generates creative, polished code and UI design that avoids generic AI aesthetics.
license: Complete terms in LICENSE.txt
---

This skill guides creation of distinctive, production-grade frontend interfaces that avoid generic "AI slop" aesthetics. Implement real working code with exceptional attention to aesthetic details and creative choices.

The user provides frontend requirements: a component, page, application, or interface to build. They may include context about the purpose, audience, or technical constraints.

## Design Thinking

Before coding, understand the context and commit to a BOLD aesthetic direction:
- **Purpose**: What problem does this interface solve? Who uses it?
- **Tone**: Pick an extreme: brutally minimal, maximalist chaos, retro-futuristic, organic/natural, luxury/refined, playful/toy-like, editorial/magazine, brutalist/raw, art deco/geometric, soft/pastel, industrial/utilitarian, etc. There are so many flavors to choose from. Use these for inspiration but design one that is true to the aesthetic direction.
- **Constraints**: Technical requirements (framework, performance, accessibility).
- **Differentiation**: What makes this UNFORGETTABLE? What's the one thing someone will remember?

**CRITICAL**: Choose a clear conceptual direction and execute it with precision. Bold maximalism and refined minimalism both work - the key is intentionality, not intensity.

Then implement working code (HTML/CSS/JS, React, Vue, etc.) that is:
- Production-grade and functional
- Visually striking and memorable
- Cohesive with a clear aesthetic point-of-view
- Meticulously refined in every detail

## Frontend Aesthetics Guidelines

Focus on:
- **Typography**: Choose fonts that are beautiful, unique, and interesting. Avoid generic fonts like Arial and Inter; opt instead for distinctive choices that elevate the frontend's aesthetics; unexpected, characterful font choices. Pair a distinctive display font with a refined body font.
- **Color & Theme**: Commit to a cohesive aesthetic. Use CSS variables for consistency. Dominant colors with sharp accents outperform timid, evenly-distributed palettes.
- **Motion**: Use animations for effects and micro-interactions. Prioritize CSS-only solutions for HTML. Use Motion library for React when available. Focus on high-impact moments: one well-orchestrated page load with staggered reveals (animation-delay) creates more delight than scattered micro-interactions. Use scroll-triggering and hover states that surprise.
- **Spatial Composition**: Unexpected layouts. Asymmetry. Overlap. Diagonal flow. Grid-breaking elements. Generous negative space OR controlled density.
- **Backgrounds & Visual Details**: Create atmosphere and depth rather than defaulting to solid colors. Add contextual effects and textures that match the overall aesthetic. Apply creative forms like gradient meshes, noise textures, geometric patterns, layered transparencies, dramatic shadows, decorative borders, custom cursors, and grain overlays.

NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character.

Interpret creatively and make unexpected choices that feel genuinely designed for the context. No design should be the same. Vary between light and dark themes, different fonts, different aesthetics. NEVER converge on common choices (Space Grotesk, for example) across generations.

**IMPORTANT**: Match implementation complexity to the aesthetic vision. Maximalist designs need elaborate code with extensive animations and effects. Minimalist or refined designs need restraint, precision, and careful attention to spacing, typography, and subtle details. Elegance comes from executing the vision well.

Remember: Claude is capable of extraordinary creative work. Don't hold back, show what can truly be created when thinking outside the box and committing fully to a distinctive vision.

---

## Project-Specific Guidelines (portfolio-v2)

This project is a **backend-focused portfolio** showcasing system design, cloud infrastructure, and scalable APIs. The audience is technical power users: recruiters, senior leads, and fellow engineers who value clarity, density, and precision.

### Design Direction

**Emotional Goal**: Trust + Efficiency + Focus  
**Visual Feel**: Terminal-modern + GitHub dark — clean, technical, precise  
**Personality**: **Precision & Density** — tight spacing, monochrome, information-forward  
**Color Foundation**: Cool foundations (slate, blue-gray) — professional, trustworthy  
**Theme**: Dark mode default (feels technical, focused, premium)  
**Typography**: Humanist sans (SF Pro) — warmer than cold system fonts  

**What makes it memorable**: Subtle delight for nerds — terminal vibes, optimization focus, hands-on Linux energy without being overdone.

### Core Craft Principles

#### The 4px Grid
All spacing uses a 4px base grid:
- `4px` — micro spacing (icon gaps)
- `8px` — tight spacing (within components)
- `12px` — standard spacing (between related elements)
- `16px` — comfortable spacing (section padding)
- `24px` — generous spacing (between sections)
- `32px` — major separation

#### Symmetrical Padding
**TLBR must match.** If top padding is 16px, left/bottom/right must also be 16px. Exception: when content naturally creates visual balance.

#### Border Radius Consistency
Use 2px, 4px, 6px (minimal system). Sharper corners feel technical. Pick a system and commit — don't mix.

#### Depth & Elevation Strategy

**Use borders-only (flat) approach:**
- Clean, technical, dense. Works for utility-focused tools where information density matters.
- Minimal shadows (`0 1px 3px rgba(0,0,0,0.08)` maximum for subtle lift).
- Lean on borders for definition (not shadows).
- Borders at 10-15% opacity for dark backgrounds.

**Example CSS:**
```css
--border: rgba(0, 0, 0, 0.08);
--border-subtle: rgba(0, 0, 0, 0.05);
border: 0.5px solid var(--border);
```

#### Card Design
- Varied internal layouts based on content.
- **Consistent surface treatment**: same border weight, shadow depth, corner radius, padding scale.
- No decorative gradients or excessive effects.

#### Typography Hierarchy
- Headlines: 600 weight, tight letter-spacing (-0.02em)
- Body: 400-500 weight, standard tracking
- Labels: 500 weight, slight positive tracking
- Scale: 11px, 12px, 13px, 14px (base), 16px, 18px, 24px, 32px
- **Monospace for data**: numbers, IDs, codes, timestamps (use `tabular-nums` for alignment)

#### Animation & Motion
- 150ms for micro-interactions, 200-250ms for larger transitions
- Easing: `cubic-bezier(0.25, 1, 0.5, 1)`
- **No spring/bouncy effects** in enterprise UI
- Page load: one well-orchestrated moment with staggered reveals

#### Contrast Hierarchy
Build a four-level system: foreground (primary) → secondary → muted → faint. Use consistently.

#### Color for Meaning Only
- Gray builds structure. Color communicates: status, action, error, success.
- Decorative color is noise.
- Almost monochrome with color reserved for actionable elements (like GitHub).

#### Navigation Context
- Include sidebar or top nav showing location.
- Use same background for sidebar and content (separate with subtle border).
- Add breadcrumbs, page title, or active nav state for grounding.

### Anti-Patterns (Never Do)
- Dramatic drop shadows (`box-shadow: 0 25px 50px...`)
- Large border radius (16px+) on small elements
- Asymmetric padding without clear reason
- Thick borders (2px+) for decoration
- Excessive spacing (margins > 48px between sections)
- Spring/bouncy animations
- Gradients for decoration
- Multiple accent colors in one interface

### Tech Stack

- **Framework**: Next.js (App Router)
- **Runtime**: Bun
- **Styling**: Tailwind CSS
- **UI Components**: shadcn/ui
- **Linter/Formatter**: Biome

**Key UI Components Available:**
- Button, Card, Input, Badge, Separator, Typography (H1-H4, P, Label, Code, MonoText)
- ButtonLink (Next.js Link styled as Button using asChild pattern)
- Always use `cn()` helper for className merging
- Use CSS variables for colors (`bg-background`, `text-foreground`)
- Memoize components with `memo()` and set `displayName`

**Import Pattern:**
```typescript
import { Button, Card, H1, P } from '@/components/ui'
```

**Component File Organization:**
- API Routes: `app/api/**/*.ts`
- Components: `app/components/**/*.tsx`
  - `ui/` — shadcn/ui primitives + typography
  - `layout/` — Header, Footer, Navigation
  - `common/` — Section, SectionHeader, shared
  - `home/`, `projects/`, `blog/`, `about/` — feature-specific
  - `mdx/` — MDX/Markdown rendering
- Lib: `app/lib/**/*.ts`
- Types: `app/types/**/*.ts`

**Biome Settings:**
- 2-space indentation
- Single quotes
- Semicolons always
- Trailing commas in multi-line
- Line width: 100 characters

### The Standard

Every interface should look designed by a team that obsesses over 1-pixel differences. Not stripped — *crafted*. Designed for the specific context of a technical portfolio for power users who value precision, clarity, and density of information without fluff.

The goal: **intricate minimalism with subtle technical personality**. No generic AI aesthetics. Precision meets nerd delight.
