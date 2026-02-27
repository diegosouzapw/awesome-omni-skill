---
name: frontend-design
description: Create distinctive, production-grade frontend interfaces with high design quality. Use this skill when the user asks to build web components, pages, artifacts, posters, or applications (examples include websites, landing pages, dashboards, React components, HTML/CSS layouts, or when styling/beautifying any web UI). Generates creative, polished code and UI design that avoids generic AI aesthetics.
license: Complete terms in LICENSE.txt
---

This skill guides creation of distinctive, production-grade frontend interfaces that avoid generic "AI slop" aesthetics. Implement real working code with exceptional attention to aesthetic details and creative choices.

The user provides frontend requirements: a component, page, application, or interface to build. They may include context about the purpose, audience, or technical constraints.

## Next.js Stack Context (Vercel)

As a Next.js expert at Vercel, you should produce LLM-friendly guidance and implementation that is:
- Accurate to modern Next.js conventions (App Router, RSC, edge/runtime boundaries, data fetching).
- Explicit about tradeoffs, constraints, and recommended defaults.
- Practical and copy-ready, with explanations that help other LLMs and humans follow your intent.

### Related Rules and Skills (Use Them Together)

When working in this stack, also consult:
- `configs/stack/nextjs/react-best-practices/SKILL.md` for React architecture, rendering, data, and performance guidance.
- `configs/stack/nextjs/composition-patterns/SKILL.md` for React composition patterns (compound components, context providers, state lifting, avoiding boolean prop proliferation).
- `configs/stack/nextjs/web-interface-guidelines/SKILL.md` for UI/UX and web interface expectations.
- `configs/stack/nextjs/agentation/SKILL.md` for adding Agentation visual feedback toolbar to Next.js projects.
- `configs/stack/nextjs/rules/next-js.mdc` and `configs/stack/nextjs/rules/typescript.mdc` for always-applied stack rules.
- `configs/rules/` for repo-wide rules that apply across stacks.

### Workflow (use this order)

1. Classify the task: routing/data, UI, performance, or infra.
2. Decide server vs client components; minimize `use client`.
3. Choose data strategy (RSC fetch, route handlers, server actions).
4. Implement UI with accessibility and responsive constraints.
5. Validate loading/error states and edge cases.
6. Add tests or checks appropriate to the change.

### Review Checklist

- RSC/Client boundaries are explicit and minimal.
- Data fetching uses the right layer and cache strategy.
- UI meets accessibility and responsive expectations.
- Loading/error/empty states are covered.
- Build and runtime constraints respected (edge/runtime).

### Local Resources in This Folder

Use these bundled resources when they match the task:
- `react-best-practices/README.md`, `react-best-practices/metadata.json`, and `react-best-practices/AGENTS.md` for scope, triggers, and agent guidance.
- `react-best-practices/rules/*.md` for detailed React performance, rendering, and data patterns referenced by that skill.
- `composition-patterns/SKILL.md` and `composition-patterns/AGENTS.md` for compound component patterns, context interfaces, and state lifting strategies.
- `composition-patterns/rules/*.md` for individual composition rules (architecture, state, patterns, React 19 APIs).
- `web-interface-guidelines/command.md` and `web-interface-guidelines/AGENTS.md` for UI/UX command usage and agent context.
- `agentation/SKILL.md` for integrating Agentation UI feedback.
- `rules/next-js.mdc` and `rules/typescript.mdc` for always-applied Next.js + TypeScript rules.

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
