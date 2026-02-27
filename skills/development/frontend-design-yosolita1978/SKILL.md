---
name: frontend-design
description: Create distinctive, mobile-first Next.js interfaces with high design quality. Use when building pages, components, or layouts. Generates polished code that avoids generic AI aesthetics.
---

## Tech Stack

- Next.js 14+ App Router
- TypeScript
- Tailwind CSS
- Mobile-first breakpoints (sm → md → lg → xl)

## Mobile-First Rules

1. **Default styles = mobile**. No breakpoint prefix means phone.
2. **Scale up**: `text-sm md:text-base lg:text-lg`
3. **Touch targets**: Minimum 44x44px for interactive elements
4. **Stack by default**: Use `flex-col` then `md:flex-row`
5. **Test at 375px width first**, then 320px for edge cases
6. **Thumb zones**: Place primary actions in bottom 60% of screen

## Low-Bandwidth & Emerging Markets

Design for users with slow 3G, data caps, and older devices.

### Performance Budget

- **First paint**: Under 1.5s on 3G
- **Total page weight**: Under 500KB ideal, 1MB maximum
- **JavaScript**: Minimize client-side JS, prefer Server Components
- **Images**: Under 100KB each, use WebP/AVIF with fallbacks

### Asset Optimization

1. **Use Next.js `<Image>`** — never raw `<img>` tags
2. **Lazy load below-fold content** — `loading="lazy"` or dynamic imports
3. **Subset fonts** — only characters you need, prefer `font-display: swap`
4. **Compress aggressively** — quality 75-80 is usually indistinguishable
5. **Avoid decorative images** — every image must earn its bytes

### Code Splitting

1. **Dynamic imports for routes/modules** — `next/dynamic` with `ssr: false` when appropriate
2. **Split by interaction** — don't load modal code until modal opens
3. **Analyze bundle** — run `next build` and check chunk sizes regularly

### Offline-Tolerant Patterns

1. **localStorage for progress** — save state so users can resume
2. **Graceful degradation** — app works if analytics/fonts fail to load
3. **Optimistic UI** — show success immediately, sync when possible
4. **Cache static assets** — configure proper Cache-Control headers

### Reduce Third-Party Scripts

1. **One analytics tool** — not three
2. **Defer non-critical scripts** — `strategy="lazyOnload"` in next/script
3. **Self-host when possible** — fonts, icons, critical libraries
4. **Audit regularly** — each external script = potential failure point

### Device Constraints

1. **Older Android phones** — test on Chrome DevTools throttling (4x CPU slowdown)
2. **Limited RAM** — avoid memory-heavy animations, large state objects
3. **Small screens** — some users still on 320px width devices
4. **Touch-only** — no hover states as primary interaction

## Design Principles

Before coding, commit to a BOLD aesthetic direction:

- **Tone**: Pick one extreme—brutally minimal, maximalist, retro-futuristic, organic, luxury, playful, editorial, brutalist, art deco, soft/pastel, industrial
- **Differentiation**: What makes this UNFORGETTABLE?

## Typography

- Choose distinctive fonts, never Inter/Arial/Roboto/system fonts
- Pair a display font with a refined body font
- Use Google Fonts or next/font for optimization
- **For low-bandwidth**: Consider system font stacks as fallback-first

## Color & Theme

- Use CSS variables in globals.css
- Dominant color + sharp accent beats evenly-distributed palettes
- Commit to light OR dark, don't hedge
- **High contrast** — many users outdoors on dim screens

## Motion

- CSS-only when possible
- One orchestrated page load > scattered micro-interactions
- Use `animation-delay` for staggered reveals
- **Respect `prefers-reduced-motion`** — some devices struggle with animations

## Layout

- Unexpected compositions: asymmetry, overlap, grid-breaking
- Generous negative space OR controlled density
- Never center everything
- **Single-column default** — complexity only at larger breakpoints

## File Structure
```
src/
  components/    # Reusable components
  app/           # Pages and layouts
  styles/        # Global CSS
```

## Anti-Patterns (NEVER)

- Generic gradients on white backgrounds
- Cookie-cutter card layouts
- Predictable hero + 3-column features
- Space Grotesk (overused)
- Purple/blue tech gradients
- **Heavy client-side rendering for static content**
- **Multiple analytics scripts loading synchronously**
- **Unoptimized images or raw `<img>` tags**
- **Assuming fast, stable internet**