---
name: ui-ux-pro-max
description: "UI/UX design intelligence. 50 styles, 21 palettes, 50 font pairings, 20 charts, 8 stacks (React, Next.js, Vue, Svelte, SwiftUI, React Native, Flutter, Tailwind). Actions: plan, build, create, design, implement, review, fix, improve, optimize, enhance, refactor, check UI/UX code. Projects: website, landing page, dashboard, admin panel, e-commerce, SaaS, portfolio, blog, mobile app, .html, .tsx, .vue, .svelte. Elements: button, modal, navbar, sidebar, card, table, form, chart. Styles: glassmorphism, claymorphism, minimalism, brutalism, neumorphism, bento grid, dark mode, responsive, skeuomorphism, flat design. Topics: color palette, accessibility, animation, layout, typography, font pairing, spacing, hover, shadow, gradient."
allowed-tools: Bash, Read, Write, Edit, Grep, Glob
---

# UI/UX Pro Max - Design Intelligence

Searchable database of UI styles, color palettes, font pairings, chart types, product recommendations, UX guidelines, and stack-specific best practices.

## Prerequisites

Check if Python is installed:

```bash
python3 --version || python --version
```

If not installed: `brew install python3` (macOS), `apt install python3` (Ubuntu), `winget install Python.Python.3.12` (Windows)

---

## How to Use This Skill

When user requests UI/UX work (design, build, create, implement, review, fix, improve), follow this workflow:

### Step 1: Analyze User Requirements

Extract key information from user request:
- **Product type**: SaaS, e-commerce, portfolio, dashboard, landing page, etc.
- **Style keywords**: minimal, playful, professional, elegant, dark mode, etc.
- **Industry**: healthcare, fintech, gaming, education, etc.
- **Stack**: React, Vue, Next.js, or default to `html-tailwind`

### Step 2: Search Relevant Domains

Use `search.py` multiple times to gather comprehensive information:

```bash
python3 "$HOME/.claude/skills/ui-ux-pro-max/scripts/search.py" "<keyword>" --domain <domain> [-n <max_results>]
```

**Quick domain reference:**
- `product` - Style recommendations by product type
- `style` - Detailed style guide (colors, effects)
- `typography` - Font pairings with Google Fonts
- `color` - Color palettes
- `landing` - Page structure, CTA strategies
- `chart` - Chart types and libraries
- `ux` - Best practices, anti-patterns

For full domain details, see [reference/search-domains.md](reference/search-domains.md)

### Step 3: Stack Guidelines

If user doesn't specify a stack, **default to `html-tailwind`**.

```bash
python3 "$HOME/.claude/skills/ui-ux-pro-max/scripts/search.py" "<keyword>" --stack html-tailwind
```

Available stacks: `html-tailwind`, `react`, `nextjs`, `vue`, `svelte`, `swiftui`, `react-native`, `flutter`

---

## Example Workflow

**User request:** "Build a landing page for a skincare service"

```bash
# 1. Product type
python3 "$HOME/.claude/skills/ui-ux-pro-max/scripts/search.py" "beauty spa wellness service" --domain product

# 2. Style (based on industry)
python3 "$HOME/.claude/skills/ui-ux-pro-max/scripts/search.py" "elegant minimal soft" --domain style

# 3. Typography
python3 "$HOME/.claude/skills/ui-ux-pro-max/scripts/search.py" "elegant luxury" --domain typography

# 4. Color palette
python3 "$HOME/.claude/skills/ui-ux-pro-max/scripts/search.py" "beauty spa wellness" --domain color

# 5. Landing page structure
python3 "$HOME/.claude/skills/ui-ux-pro-max/scripts/search.py" "hero-centric social-proof" --domain landing

# 6. UX guidelines
python3 "$HOME/.claude/skills/ui-ux-pro-max/scripts/search.py" "animation accessibility" --domain ux

# 7. Stack guidelines (default)
python3 "$HOME/.claude/skills/ui-ux-pro-max/scripts/search.py" "layout responsive" --stack html-tailwind
```

**Then:** Synthesize all search results and implement the design.

---

## Quick Rules

**Icons:** Use SVG (Heroicons, Lucide), never emojis as UI icons
**Hover:** Use color/opacity transitions, not scale transforms
**Cursor:** Add `cursor-pointer` to all clickable elements
**Contrast:** Light mode text minimum `slate-600`, body `slate-900`
**Transitions:** 150-300ms, never instant or >500ms

For complete rules and pre-delivery checklist, see [reference/common-rules.md](reference/common-rules.md)
