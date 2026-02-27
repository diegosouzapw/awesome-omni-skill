---
name: frontend-design
description: "Create distinctive, production-grade frontend interfaces with high design quality. Visual design intelligence, UI/UX aesthetics, and web interface guidelines compliance. Integrates: chrome-devtools, ai-multimodal, media-processing. Capabilities: design analysis, visual hierarchy, color theory, typography, micro-interactions, animation, design systems, accessibility, pattern database search. Actions: analyze, design, create, capture, evaluate, implement, review, fix, improve, optimize, enhance, refactor, check UI/UX code. Use when: building beautiful UIs, analyzing design inspiration, implementing visual hierarchy, adding animations/micro-interactions, creating design systems, evaluating aesthetic quality, capturing design screenshots, reviewing UI code for compliance, building web components, pages, artifacts, posters, or applications."
license: Complete terms in LICENSE.txt
---

# Frontend Design

Create aesthetically beautiful, distinctive, production-grade frontend interfaces by following proven design principles, systematic workflows, and comprehensive analysis techniques. This skill combines design thinking, frontend implementation patterns, searchable pattern databases, and web interface guidelines compliance.

## When to Use

- Building or designing user interfaces (websites, landing pages, dashboards, React components, HTML/CSS layouts)
- Analyzing designs from inspiration websites (Dribbble, Mobbin, Behance, Awwwards)
- Generating design images and evaluating aesthetic quality
- Implementing visual hierarchy, typography, color theory
- Adding micro-interactions and animations
- Creating design documentation and style guides
- Building production-grade frontend interfaces with distinctive aesthetics
- Reviewing UI code for Web Interface Guidelines compliance
- Styling/beautifying any web UI

## Core Philosophy

### Design Thinking

Before coding, understand the context and commit to a BOLD aesthetic direction:
- **Purpose**: What problem does this interface solve? Who uses it?
- **Tone**: Pick an extreme: brutally minimal, maximalist chaos, retro-futuristic, organic/natural, luxury/refined, playful/toy-like, editorial/magazine, brutalist/raw, art deco/geometric, soft/pastel, industrial/utilitarian, etc. There are so many flavors to choose from. Use these for inspiration but design one that is true to the aesthetic direction.
- **Constraints**: Technical requirements (framework, performance, accessibility).
- **Differentiation**: What makes this UNFORGETTABLE? What's the one thing someone will remember?

**CRITICAL**: Choose a clear conceptual direction and execute it with precision. Bold maximalism and refined minimalism both work - the key is intentionality, not intensity.

### Progressive Excellence

**Evidence-Based Aesthetics**: AI lacks aesthetic sense - standards must come from analyzing high-quality examples and aligning with market tastes. Study existing designs, identify patterns, extract principles.

1. **BEAUTIFUL**: Understanding Aesthetics - Study existing designs, identify patterns, extract principles.
   **Reference**: `references/design-principles.md` - Load for visual hierarchy, typography, color theory, white space principles.

2. **RIGHT**: Ensuring Functionality - Beautiful designs lacking usability are worthless. Study design systems, component architecture, accessibility requirements.
   **Reference**: `references/design-principles.md` - Load for design systems, component libraries, WCAG accessibility standards.

3. **SATISFYING**: Micro-Interactions - Incorporate subtle animations with appropriate timing (150-300ms), easing curves (ease-out for entry, ease-in for exit), sequential delays.
   **Reference**: `references/micro-interactions.md` - Load for duration guidelines, easing curves, performance optimization.

4. **PEAK**: Storytelling Through Design - Elevate with narrative elements - parallax effects, particle systems, thematic consistency. Use restraint: "too much of anything isn't good."
   **Reference**: `references/storytelling-design.md` - Load for narrative elements, scroll-based storytelling, interactive techniques.

## Frontend Aesthetics Guidelines

### Typography
Choose fonts that are beautiful, unique, and interesting. Avoid generic fonts like Arial and Inter; opt instead for distinctive choices that elevate the frontend's aesthetics; unexpected, characterful font choices. Pair a distinctive display font with a refined body font.

### Color & Theme
Commit to a cohesive aesthetic. Use CSS variables for consistency. Dominant colors with sharp accents outperform timid, evenly-distributed palettes.

### Motion & Animation
Use animations for effects and micro-interactions. Prioritize CSS-only solutions for HTML. Use Motion library for React when available. Use anime.js for complex animations. Focus on high-impact moments: one well-orchestrated page load with staggered reveals (animation-delay) creates more delight than scattered micro-interactions. Use scroll-triggering and hover states that surprise.

**Reference**: `references/animejs.md` - Load when implementing anime.js v4 animations.

### Spatial Composition
Unexpected layouts. Asymmetry. Overlap. Diagonal flow. Grid-breaking elements. Generous negative space OR controlled density.

### Backgrounds & Visual Details
Create atmosphere and depth rather than defaulting to solid colors. Add contextual effects and textures that match the overall aesthetic. Apply creative forms like gradient meshes, noise textures, geometric patterns, layered transparencies, dramatic shadows, decorative borders, custom cursors, and grain overlays.

**Reference**: `references/implementation-guide.md` - Load for detailed frontend design execution patterns.

## Anti-Patterns: Avoid Generic AI Aesthetics

NEVER use:
- Overused font families (Inter, Roboto, Arial, system fonts)
- Cliched color schemes (particularly purple gradients on white backgrounds)
- Predictable layouts and component patterns
- Cookie-cutter design that lacks context-specific character

**Variation Principle**: No design should be the same. Vary between light and dark themes, different fonts, different aesthetics. NEVER converge on common choices (Space Grotesk, for example) across generations.

Interpret creatively and make unexpected choices that feel genuinely designed for the context.

## Implementation Complexity Matching

Match implementation complexity to the aesthetic vision:
- **Maximalist designs**: Need elaborate code with extensive animations and effects
- **Minimalist/refined designs**: Need restraint, precision, careful attention to spacing, typography, and subtle details
- **Elegance**: Comes from executing the vision well

Then implement working code (HTML/CSS/JS, React, Vue, etc.) that is:
- Production-grade and functional
- Visually striking and memorable
- Cohesive with a clear aesthetic point-of-view
- Meticulously refined in every detail

**IMPORTANT**: Match implementation complexity to the aesthetic vision. Maximalist designs need elaborate code with extensive animations and effects. Minimalist or refined designs need restraint, precision, and careful attention to spacing, typography, and subtle details. Elegance comes from executing the vision well.

## Workflows

### Workflow 1: Capture & Analyze Inspiration

Extract design guidelines from inspiration websites:
1. Browse inspiration sites (Dribbble, Mobbin, Behance, Awwwards)
2. Use **chrome-devtools** skill to capture full-screen screenshots
3. Use **ai-multimodal** skill to analyze and extract design patterns
4. Document findings in project design guidelines

**Reference**: `references/workflows.md` - Load for complete workflow steps and analysis checklist.

### Workflow 2: Generate & Iterate Design Images

Create aesthetically pleasing designs through iteration:
1. Define design prompt with style, colors, typography, audience
2. Use **ai-multimodal** skill to generate design images
3. Evaluate aesthetic quality (must score >= 7/10)
4. Iterate until professional standards met
5. Document final design decisions

**Reference**: `references/workflows.md` - Load for complete iteration process and quality standards.

## UI Pattern Database

Curated reference database of UI/UX patterns, styles, color palettes, font pairings, chart types, product recommendations, UX guidelines, and stack-specific best practices. Use as a lookup tool to find proven design patterns and implementation guidance.

### Prerequisites

Check if Python is installed:

```bash
python3 --version || python --version
```

If Python is not installed, install it based on user's OS:

**macOS:**
```bash
brew install python3
```

**Ubuntu/Debian:**
```bash
sudo apt update && sudo apt install python3
```

**Windows:**
```powershell
winget install Python.Python.3.12
```

### How to Use the Pattern Database

When user requests UI/UX work (design, build, create, implement, review, fix, improve), follow this workflow:

**Step 1: Analyze User Requirements**

Extract key information from user request:
- **Product type**: SaaS, e-commerce, portfolio, dashboard, landing page, etc.
- **Style keywords**: minimal, playful, professional, elegant, dark mode, etc.
- **Industry**: healthcare, fintech, gaming, education, etc.
- **Stack**: React, Vue, Next.js, or default to `html-tailwind`

**Step 2: Search Relevant Domains**

Use `search.py` multiple times to gather comprehensive information. Search until you have enough context.

```bash
python3 .claude/skills/ui-ux-design/scripts/search.py "<keyword>" --domain <domain> [-n <max_results>]
```

**Recommended search order:**

1. **Product** - Get style recommendations for product type
2. **Style** - Get detailed style guide (colors, effects, frameworks)
3. **Typography** - Get font pairings with Google Fonts imports
4. **Color** - Get color palette (Primary, Secondary, CTA, Background, Text, Border)
5. **Landing** - Get page structure (if landing page)
6. **Chart** - Get chart recommendations (if dashboard/analytics)
7. **UX** - Get best practices and anti-patterns
8. **Stack** - Get stack-specific guidelines (default: html-tailwind)

**Step 3: Stack Guidelines (Default: html-tailwind)**

If user doesn't specify a stack, **default to `html-tailwind`**.

```bash
python3 .claude/skills/ui-ux-design/scripts/search.py "<keyword>" --stack html-tailwind
```

Available stacks: `html-tailwind`, `react`, `nextjs`, `vue`, `svelte`, `swiftui`, `react-native`, `flutter`

### Available Domains

| Domain | Use For | Example Keywords |
|--------|---------|------------------|
| `product` | Product type recommendations | SaaS, e-commerce, portfolio, healthcare, beauty, service |
| `style` | UI styles, colors, effects | glassmorphism, minimalism, dark mode, brutalism |
| `typography` | Font pairings, Google Fonts | elegant, playful, professional, modern |
| `color` | Color palettes by product type | saas, ecommerce, healthcare, beauty, fintech, service |
| `landing` | Page structure, CTA strategies | hero, hero-centric, testimonial, pricing, social-proof |
| `chart` | Chart types, library recommendations | trend, comparison, timeline, funnel, pie |
| `ux` | Best practices, anti-patterns | animation, accessibility, z-index, loading |
| `prompt` | AI prompts, CSS keywords | (style name) |

### Tips for Better Results

1. **Be specific with keywords** - "healthcare SaaS dashboard" > "app"
2. **Search multiple times** - Different keywords reveal different insights
3. **Combine domains** - Style + Typography + Color = Complete design system
4. **Always check UX** - Search "animation", "z-index", "accessibility" for common issues
5. **Use stack flag** - Get implementation-specific best practices
6. **Iterate** - If first search doesn't match, try different keywords

## Common Rules for Professional UI

These are frequently overlooked issues that make UI look unprofessional:

### Icons & Visual Elements

| Rule | Do | Don't |
|------|----|----- |
| **No emoji icons** | Use SVG icons (Heroicons, Lucide, Simple Icons) | Use emojis like as UI icons |
| **Stable hover states** | Use color/opacity transitions on hover | Use scale transforms that shift layout |
| **Correct brand logos** | Research official SVG from Simple Icons | Guess or use incorrect logo paths |
| **Consistent icon sizing** | Use fixed viewBox (24x24) with w-6 h-6 | Mix different icon sizes randomly |

### Interaction & Cursor

| Rule | Do | Don't |
|------|----|----- |
| **Cursor pointer** | Add `cursor-pointer` to all clickable/hoverable cards | Leave default cursor on interactive elements |
| **Hover feedback** | Provide visual feedback (color, shadow, border) | No indication element is interactive |
| **Smooth transitions** | Use `transition-colors duration-200` | Instant state changes or too slow (>500ms) |

### Light/Dark Mode Contrast

| Rule | Do | Don't |
|------|----|----- |
| **Glass card light mode** | Use `bg-white/80` or higher opacity | Use `bg-white/10` (too transparent) |
| **Text contrast light** | Use `#0F172A` (slate-900) for text | Use `#94A3B8` (slate-400) for body text |
| **Muted text light** | Use `#475569` (slate-600) minimum | Use gray-400 or lighter |
| **Border visibility** | Use `border-gray-200` in light mode | Use `border-white/10` (invisible) |

### Layout & Spacing

| Rule | Do | Don't |
|------|----|----- |
| **Floating navbar** | Add `top-4 left-4 right-4` spacing | Stick navbar to `top-0 left-0 right-0` |
| **Content padding** | Account for fixed navbar height | Let content hide behind fixed elements |
| **Consistent max-width** | Use same `max-w-6xl` or `max-w-7xl` | Mix different container widths |

## Pre-Delivery Checklist

Before delivering UI code, verify these items:

### Visual Quality
- [ ] No emojis used as icons (use SVG instead)
- [ ] All icons from consistent icon set (Heroicons/Lucide)
- [ ] Brand logos are correct (verified from Simple Icons)
- [ ] Hover states don't cause layout shift
- [ ] Use theme colors directly (bg-primary) not var() wrapper

### Interaction
- [ ] All clickable elements have `cursor-pointer`
- [ ] Hover states provide clear visual feedback
- [ ] Transitions are smooth (150-300ms)
- [ ] Focus states visible for keyboard navigation

### Light/Dark Mode
- [ ] Light mode text has sufficient contrast (4.5:1 minimum)
- [ ] Glass/transparent elements visible in light mode
- [ ] Borders visible in both modes
- [ ] Test both modes before delivery

### Layout
- [ ] Floating elements have proper spacing from edges
- [ ] No content hidden behind fixed navbars
- [ ] Responsive at 320px, 768px, 1024px, 1440px
- [ ] No horizontal scroll on mobile

### Accessibility
- [ ] All images have alt text
- [ ] Form inputs have labels
- [ ] Color is not the only indicator
- [ ] `prefers-reduced-motion` respected

## Web Interface Guidelines Review

Review files for compliance with Web Interface Guidelines.

### How It Works

1. Fetch the latest guidelines from the source URL below
2. Read the specified files (or prompt user for files/pattern)
3. Check against all rules in the fetched guidelines
4. Output findings in the terse `file:line` format

### Guidelines Source

Fetch fresh guidelines before each review:

```
https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md
```

Use WebFetch to retrieve the latest rules. The fetched content contains all the rules and output format instructions.

When a user provides a file or pattern argument:
1. Fetch guidelines from the source URL above
2. Read the specified files
3. Apply all rules from the fetched guidelines
4. Output findings using the format specified in the guidelines

If no files specified, ask the user which files to review.

## Design Documentation

### Create Design Guidelines
Use `assets/design-guideline-template.md` to document:
- Color patterns & psychology
- Typography system & hierarchy
- Layout principles & spacing
- Component styling standards
- Accessibility considerations
- Design highlights & rationale

Save in project `./docs/design-guideline.md`.

### Create Design Story
Use `assets/design-story-template.md` to document:
- Narrative elements & themes
- Emotional journey
- User journey & peak moments
- Design decision rationale

Save in project `./docs/design-story.md`.

## Related Skills Integration

- **ai-multimodal**: Analyze documents, screenshots & videos, generate design images, evaluate aesthetic quality using Gemini API
- **chrome-devtools**: Capture screenshots from inspiration websites, navigate pages, interact with elements
- **media-processing**: Refine generated images (FFmpeg for video, ImageMagick for images)
- **ui-styling**: Implement designs with shadcn/ui components + Tailwind CSS
- **web-frameworks**: Build with Next.js, React, Vue

**Reference**: `references/design-resources.md` - Load for inspiration platforms, design systems, AI tools, MCP integrations.

## Key Principles

1. Aesthetic standards come from humans, not AI - study quality examples
2. Choose a BOLD aesthetic direction and execute with precision
3. Iterate based on analysis - never settle for first output (minimum 7/10 quality)
4. Balance beauty with functionality and accessibility
5. Document decisions for consistency across development
6. Use progressive disclosure in design - reveal complexity gradually
7. Match implementation complexity to aesthetic vision
8. Avoid generic AI aesthetics - create context-specific character

Remember: Claude is capable of extraordinary creative work. Don't hold back, show what can truly be created when thinking outside the box and committing fully to a distinctive vision.
