---
name: frontend-design
description: Create distinctive, production-grade frontend interfaces with high design quality. Use this skill when the user asks to build web components, pages, or applications. Generates creative, polished code that avoids generic AI aesthetics.
allowed-tools: AskUserQuestion
license: Complete terms in LICENSE.txt
---

This skill guides creation of distinctive, production-grade frontend interfaces that avoid generic "AI slop" aesthetics.

**Always implement, never just suggest**: Create complete, working code with full styling, animations, and interactions. If the purpose or aesthetic direction is unclear, use AskUserQuestion to clarify before proceeding.

## Existing Projects: Adapt First

Before designing anything, discover the project's existing frontend conventions:

1. **Styling approach**: Check for Tailwind (`tailwind.config`), CSS Modules, styled-components, Sass, or plain CSS. Use what exists.
2. **Component library**: Look for shadcn/ui, Radix, MUI, Chakra, or custom components. Extend existing patterns.
3. **Design tokens**: Find existing color variables, spacing scales, typography settings. Use them.
4. **Existing components**: Search for similar components to match patterns (naming, props, structure).

**Match the codebase**: Use existing utility classes, follow established naming conventions, extend the design system rather than fighting it. A component that looks native to the codebase is better than one that's technically impressive but stylistically alien.

**When extending**: If the existing system lacks something you need, add it in a way that feels native (e.g., add a new Tailwind color to the config rather than using arbitrary values).

## Greenfield Projects: Bold Aesthetics

For new projects without existing conventions, commit to a BOLD aesthetic direction:

- **Purpose**: What problem does this interface solve? Who uses it?
- **Tone**: Pick an extreme: brutally minimal, maximalist chaos, retro-futuristic, organic/natural, luxury/refined, playful/toy-like, editorial/magazine, brutalist/raw, art deco/geometric, soft/pastel, industrial/utilitarian, etc. There are so many flavors to choose from. Use these for inspiration but design one that is true to the aesthetic direction.
- **Constraints**: Technical requirements (framework, performance, accessibility).
- **Differentiation**: What makes this UNFORGETTABLE? What's the one thing someone will remember?

Choose a clear conceptual direction and execute it with precision. Bold maximalism and refined minimalism both work - the key is intentionality, not intensity.

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

Avoid generic AI-generated aesthetics: overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character.

Interpret creatively and make unexpected choices that feel genuinely designed for the context. No design should be the same. Vary between light and dark themes, different fonts, different aesthetics. Don't converge on common choices (Space Grotesk, for example) across generations.

Match implementation complexity to the aesthetic vision. Maximalist designs need elaborate code with extensive animations and effects. Minimalist or refined designs need restraint, precision, and careful attention to spacing, typography, and subtle details. Elegance comes from executing the vision well.

Remember: Claude is capable of extraordinary creative work. Don't hold back, show what can truly be created when pushing beyond conventions and committing fully to a distinctive vision.

## Summary

- **Existing projects**: Match the codebase first - discover and extend existing patterns
- **Greenfield projects**: Commit to a bold aesthetic direction with intentionality
- **Every design**: Typography, color, motion, composition, and atmosphere should feel distinctive and memorable
