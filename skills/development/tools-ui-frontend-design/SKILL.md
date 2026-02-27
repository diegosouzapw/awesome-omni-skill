---
name: tools-ui-frontend-design
description: Create distinctive, production-grade frontend interfaces grounded in this repo's design system. Use when asked to build web components, pages, or applications. Combines bold creative direction with token-constrained implementation.
---

# Frontend Design

Build distinctive UI that is both creatively intentional and design-system-compliant.

This skill bridges the Claude `frontend-design` plugin's creative philosophy with this repo's concrete design token system, brand language, and component library. The plugin provides the "think boldly" mindset; this skill constrains it to the actual design system.

## Philosophy (from the Claude frontend-design plugin)

- Commit to a BOLD aesthetic direction before coding
- Avoid generic AI aesthetics (purple gradients, Inter/Roboto, predictable grid layouts)
- Every design choice should be intentional and defensible
- High-impact micro-interactions and motion where they serve the user
- Atmosphere and depth through composition, not through arbitrary values

## Constraints (this repo's design system)

ALL design work MUST use the repo's token system. The tokens and brand dossier define the creative canvas — boldness comes from HOW you compose them, not from bypassing them.

### Mandatory References

Load these before any design work:

| What | Where | Load when |
|------|-------|-----------|
| Token quick-ref | `.claude/skills/lp-design-system/SKILL.md` | Always — first thing |
| Brand dossier | `docs/business-os/strategy/<BIZ>/brand-dossier.user.md` | If it exists for the business |
| Theme tokens | `packages/themes/<theme>/src/tokens.ts` | Before any design |
| Base tokens | `packages/themes/base/src/tokens.ts` | Always (fallback + reference) |
| Component catalog | `docs/design-system-handbook.md` | When composing layouts |
| Typography & color | `docs/typography-and-color.md` | When choosing fonts/colors |
| Business registry | `docs/business-os/strategy/businesses.json` | To resolve app → business |

### Hard Rules

1. ALL colors via semantic tokens (`bg-primary`, `text-fg`, `bg-accent`, etc.) — never arbitrary hex
2. ALL typography via font tokens (`font-sans`, `font-heading`, `font-mono`) — never import external fonts
3. ALL spacing via 8-pt rhythm (`p-4`, `gap-6`, `space-y-4`) — never arbitrary px
4. ALL radius via token scale (`rounded-md`, `rounded-lg`) — never arbitrary
5. ALL shadows via token scale (`shadow-sm`, `shadow-md`, `shadow-lg`)
6. Use `cn()` from `@acme/design-system/utils/style` for class merging
7. Use existing `@acme/design-system` components before creating new ones
8. Mobile-first responsive: base → `md:` (768px) → `lg:` (1024px) → `xl:` (1280px)

### Where Boldness Lives

The plugin's creative philosophy applies to:
- **Composition**: How you arrange components, use whitespace, create visual hierarchy — asymmetry, overlap, and diagonal flow are encouraged through layout choices (grid-cols, order, absolute positioning), not through arbitrary spacing values
- **Motion**: CSS transitions/animations on existing token values (opacity, transform, scale) — use `transition-*` utilities and `@keyframes` with token-derived values
- **Density context**: Choose `.context-operations` / `.context-consumer` / `.context-hospitality` intentionally — this changes the entire feel without breaking the system
- **Brand expression**: Each business has a distinct brand dossier — lean into its personality, voice, and visual identity. The dossier IS the aesthetic direction
- **Component selection**: Choose the right atom/molecule from the design system, don't default to the obvious choice. A `StatCard` grid tells a different story than a `DataGrid` table
- **Surface layering**: Use `surface-1`, `surface-2`, `surface-3` to create depth and hierarchy — this is how the design system provides atmosphere

The plugin's creative philosophy does NOT override:
- Color values (use tokens)
- Font choices (use tokens)
- Spacing values (use 8-pt rhythm)
- Border radius (use token scale)
- Creating custom CSS properties outside the design system

## Workflow

### Step 1: Resolve the Business

Read `docs/business-os/strategy/businesses.json` to find the target business for the work.

| Business | Key Apps | Theme |
|----------|----------|-------|
| BRIK | brikette, reception, prime | `packages/themes/prime/` |
| PLAT | platform-core, design-system, cms, dashboard | `packages/themes/base/` |
| BOS | business-os | `packages/themes/base/` |
| PIPE | product-pipeline | `packages/themes/base/` |
| XA | xa | `packages/themes/base/` |
| HEAD | cochlearfit | `packages/themes/base/` |
| PET | (no apps yet) | `packages/themes/base/` |
| HBAG | cover-me-pretty, handbag-configurator | `packages/themes/base/` |

Load the brand dossier: `docs/business-os/strategy/<BIZ>/brand-dossier.user.md`

**If no brand dossier exists** (common for PLAT, BOS, PIPE, XA): use base theme tokens directly from `packages/themes/base/src/tokens.ts` as the design reference. No redirection needed — the design system tokens provide a complete and correct palette. Reserve `/lp-assessment-bootstrap <BIZ>` for operating businesses that will have a distinct brand identity.

### Step 2: Load Design System Context

1. Read `.claude/skills/lp-design-system/SKILL.md` for the token quick-reference
2. Read `packages/themes/<theme>/src/tokens.ts` for the business's concrete token values
3. Read `docs/design-system-handbook.md` for available components
4. If the feature is complex, also read `docs/typography-and-color.md`

### Step 3: Design Direction

With the brand dossier and tokens loaded, establish a creative direction:

- What density context fits? (`operations` for dashboards, `consumer` for marketing, `hospitality` for booking)
- What surface layering creates the right depth? (flat vs elevated vs deeply layered)
- What motion principles serve the user? (subtle transitions vs dramatic reveals)
- How does the brand personality translate to composition choices?

### Step 4: Build

Implement using:
- `@acme/design-system` components (atoms, molecules, primitives)
- Semantic Tailwind tokens (never arbitrary values)
- `cn()` for conditional class merging
- Proper TypeScript types for all props

### Step 5: Verify

After building, run `/lp-design-qa` to audit:
- Token compliance (no arbitrary values)
- Brand dossier alignment
- Accessibility (contrast ratios, keyboard navigation, ARIA)
- Responsive behavior across breakpoints

### Step 6: Document (optional)

For significant features that benefit from visual documentation, suggest `/lp-visual` to create diagrams showing the component architecture, state flows, or interaction patterns.

## Anti-Patterns

| Do NOT | Do instead |
|--------|------------|
| `bg-[#FF6B35]` | `bg-primary` (resolved from theme tokens) |
| `font-['Plus Jakarta Sans']` | `font-heading` (resolved from theme tokens) |
| `p-[13px]` | `p-3` (8-pt rhythm: 12px) |
| `rounded-[7px]` | `rounded-md` (token scale) |
| Import Google Fonts | Use `font-sans`, `font-heading`, `font-mono` |
| Create new CSS custom properties | Add tokens to `packages/themes/base/src/tokens.ts` |
| `style={{ color: '#333' }}` | `className="text-fg"` |
| Build a custom button | Use `@acme/design-system/shadcn/Button` |
