---
name: apple-design
description: Create Apple-inspired modern, minimalist UI designs with glassmorphism, smooth animations, generous whitespace, and elegant typography. Use when designing portfolio websites, landing pages, hero sections, product showcases, or implementing Apple-style components, dark mode, or responsive layouts.
---

# Apple Design System

Create stunning, modern UI designs inspired by Apple's design language. This design system provides guidelines, components, and patterns for building clean, minimalist interfaces with attention to detail, smooth animations, and premium aesthetics.

## When to Use This Skill

Use this skill when:
- Designing portfolio websites or personal sites
- Creating landing pages or product showcases
- Implementing hero sections with visual impact
- Building card-based layouts for projects or products
- Adding glassmorphism or frosted glass effects
- Implementing smooth, delightful animations
- Creating dark mode compatible designs
- Designing navigation bars, modals, or forms
- Building contact forms or call-to-action sections
- Working with modern CSS features (backdrop-filter, gradients, shadows)

## Quick Start

### 1. Core Design Principles

**Minimalism**: Remove unnecessary elements, focus on content
**Typography**: Large, bold headlines with system fonts
**Colors**: Neutral base with strategic accent colors
**Spacing**: 8px grid system with generous whitespace
**Effects**: Glassmorphism, soft shadows, smooth animations
**Imagery**: High-quality, properly sized images

📖 **Detailed guide**: [design-principles.md](references/design-principles.md)

### 2. Color System

```css
/* Light Mode */
:root {
  --bg-primary: #ffffff;
  --bg-secondary: #f5f5f7;
  --text-primary: #1d1d1f;
  --text-secondary: #86868b;
  --accent-blue: #0071e3;
  --accent-green: #30d158;
  --border-color: rgba(0, 0, 0, 0.1);
}

/* Dark Mode */
@media (prefers-color-scheme: dark) {
  :root {
    --bg-primary: #000000;
    --bg-secondary: #1d1d1f;
    --text-primary: #f5f5f7;
    --text-secondary: #a1a1a6;
    --accent-blue: #0a84ff;
    --border-color: rgba(255, 255, 255, 0.1);
  }
}
```

📖 **Complete palette**: [color-system.md](references/color-system.md)

### 3. Typography

```css
/* System font stack */
font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display',
             'Segoe UI', sans-serif;

/* Fluid responsive typography */
h1 {
  font-size: clamp(2.5rem, 5vw + 1rem, 4.5rem);
  font-weight: 700;
  line-height: 1.1;
  letter-spacing: -0.02em;
}

p {
  font-size: 1rem;
  line-height: 1.6;
}
```

📖 **Typography system**: [typography-system.md](references/typography-system.md)

### 4. Spacing

Use 8px-based spacing scale:

```css
:root {
  --space-2: 0.5rem;   /* 8px */
  --space-4: 1rem;     /* 16px */
  --space-6: 1.5rem;   /* 24px */
  --space-8: 2rem;     /* 32px */
  --space-12: 3rem;    /* 48px */
  --space-16: 4rem;    /* 64px */
  --space-24: 6rem;    /* 96px */
}
```

## Core Components

### Hero Section

```html
<section class="hero">
  <div class="hero-content">
    <h1 class="hero-title">
      Hi, I'm <span class="gradient-text">Your Name</span>
    </h1>
    <p class="hero-subtitle">Developer creating amazing experiences</p>
    <div class="hero-actions">
      <button class="btn btn-primary">View Work</button>
      <button class="btn btn-secondary">Contact</button>
    </div>
  </div>
</section>
```

### Glass Card

```css
.glass-card {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(20px) saturate(180%);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 20px;
  padding: 2rem;
  box-shadow:
    0 8px 32px rgba(0, 0, 0, 0.1),
    inset 0 1px 0 rgba(255, 255, 255, 0.2);
}
```

### Project Card

```css
.project-card {
  border-radius: 24px;
  overflow: hidden;
  background: var(--bg-secondary);
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.08);
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.project-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 12px 48px rgba(0, 0, 0, 0.12);
}
```

### Navigation Bar

```css
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(20px) saturate(180%);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
}
```

### Buttons

```css
.btn {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.875rem 1.75rem;
  border-radius: 980px;
  font-weight: 500;
  border: none;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  min-height: 44px;
}

.btn-primary {
  background: var(--accent-blue);
  color: white;
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(0, 113, 227, 0.3);
}

.btn-secondary {
  background: transparent;
  color: var(--accent-blue);
  border: 2px solid var(--accent-blue);
}

.btn-secondary:hover {
  background: var(--accent-blue);
  color: white;
}
```

📖 **All components**: [components-reference.md](references/components-reference.md)

## Animations

### Hover Effects

```css
/* Lift effect */
.hover-lift {
  transition: transform 300ms ease, box-shadow 300ms ease;
}

.hover-lift:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 48px rgba(0, 0, 0, 0.12);
}

/* Image zoom */
.image-zoom {
  transition: transform 600ms ease;
}

.card:hover .image-zoom {
  transform: scale(1.05);
}
```

### Entrance Animations

```css
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-fade-in-up {
  animation: fadeInUp 600ms ease-out;
}
```

📖 **Animation patterns**: [animation-guide.md](references/animation-guide.md)

## AI Assistant Instructions

When this skill is activated for Apple-inspired design:

### 1. Always Prioritize
- Clean, minimalist aesthetics
- Generous whitespace and breathing room
- Smooth, delightful animations (300-600ms)
- Dark mode compatibility
- Responsive design (mobile-first)
- Accessibility (WCAG AA minimum)

### 2. Component Creation Workflow
1. **Start with semantic HTML structure**
2. **Apply modern CSS** (flexbox, grid, custom properties)
3. **Add glassmorphism** for cards and overlays where appropriate
4. **Implement hover states** with subtle transitions
5. **Ensure dark mode support** using CSS custom properties
6. **Test responsiveness** at mobile, tablet, desktop breakpoints

### 3. Styling Approach
- **Use CSS variables** for all colors and spacing
- **Apply system font stack** for typography
- **Implement soft shadows** for depth (layered shadows)
- **Add smooth transitions** with proper easing
- **Use clamp()** for fluid typography
- **Follow 8px spacing grid** consistently

### 4. Never Do
- Use overly complex animations
- Ignore responsive design
- Sacrifice accessibility for aesthetics
- Use outdated CSS patterns (floats, !important)
- Forget dark mode considerations
- Animate width/height (use transform instead)

### 5. For Portfolio Sites Specifically
- Create **impactful hero sections** with large typography and gradients
- Design **project cards** with image zoom on hover
- Build **clean navigation** with blur effects
- Implement **smooth scroll reveal** animations
- Add **contact forms** with modern, accessible inputs
- Use **glassmorphism** strategically for visual interest

### 6. Code Quality Standards
- Write clean, well-commented code
- Use modern CSS features (grid, clamp, custom properties)
- Optimize for performance (GPU-accelerated properties only)
- Follow accessibility guidelines (ARIA, keyboard navigation, focus states)
- Provide both light and dark mode styles
- Use semantic HTML elements

### 7. When Suggesting Designs
- Provide **complete, working code examples**
- Include **both HTML/JSX and CSS**
- Show **responsive variations** (mobile, tablet, desktop)
- Demonstrate **hover/active states**
- Include **accessibility considerations** (focus states, ARIA labels)
- Reference **templates** from the templates/ folder when applicable

### 8. Reference Documentation
When users need detailed information, direct them to:
- **Design philosophy**: [design-principles.md](references/design-principles.md)
- **Color palettes & gradients**: [color-system.md](references/color-system.md)
- **Typography scale & hierarchy**: [typography-system.md](references/typography-system.md)
- **Animation timing & patterns**: [animation-guide.md](references/animation-guide.md)
- **Component library**: [components-reference.md](references/components-reference.md)

### 9. Templates Available
Ready-to-use templates in `templates/` folder:
- `hero-section.html` - Full-screen hero with gradient
- `project-card.html` - Project showcase cards with hover effects

Pre-built examples in `examples/` folder:
- `color-palette.css` - Complete color system with utilities
- `animations.css` - Animation library with all patterns

## Best Practices

### Performance
- ✅ Animate only `transform` and `opacity`
- ✅ Use `will-change` sparingly
- ✅ Optimize images (WebP, lazy loading)
- ✅ Minimize `backdrop-filter` usage
- ❌ Avoid animating `width`, `height`, `top`, `left`

### Accessibility
- ✅ Maintain 4.5:1 contrast ratio for text
- ✅ Provide focus indicators (2px outline, 2px offset)
- ✅ Use semantic HTML
- ✅ Add ARIA labels for icon-only buttons
- ✅ Respect `prefers-reduced-motion`
- ✅ Minimum touch target: 44x44px

### Responsive Design
- ✅ Mobile-first approach
- ✅ Use `clamp()` for fluid typography
- ✅ Flexible grid layouts with `auto-fit` or `auto-fill`
- ✅ Test on real devices
- ✅ Breakpoints: 640px (sm), 768px (md), 1024px (lg), 1280px (xl)

### Code Organization
- ✅ Use CSS variables for theme values
- ✅ Organize styles by component
- ✅ Keep specificity low (avoid nesting > 3 levels)
- ✅ Document complex calculations
- ✅ Group related properties (positioning, box model, typography, visual)

## Quick Reference

### Shadow Scale
```css
--shadow-sm: 0 1px 2px rgba(0,0,0,0.04), 0 1px 4px rgba(0,0,0,0.04);
--shadow-md: 0 4px 12px rgba(0,0,0,0.08);
--shadow-lg: 0 8px 24px rgba(0,0,0,0.08), 0 16px 32px rgba(0,0,0,0.08);
--shadow-xl: 0 24px 48px rgba(0,0,0,0.12);
```

### Border Radius Scale
```css
--radius-sm: 8px;
--radius-md: 12px;
--radius-lg: 16px;
--radius-xl: 20px;
--radius-2xl: 24px;
--radius-full: 9999px;
```

### Animation Duration
```css
--duration-fast: 150ms;    /* Micro-interactions */
--duration-base: 300ms;    /* Standard transitions */
--duration-slow: 500ms;    /* Complex animations */
```

### Easing Functions
```css
--ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
--ease-out: cubic-bezier(0, 0, 0.2, 1);
--ease-in: cubic-bezier(0.4, 0, 1, 1);
```

## Resources

### Official Apple Resources
- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
- [SF Pro Font](https://developer.apple.com/fonts/)

### Useful Tools
- [Framer Motion](https://www.framer.com/motion/) - React animations
- [Tailwind CSS](https://tailwindcss.com/) - Utility-first CSS
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Coolors](https://coolors.co/) - Color palette generator

### Internal Documentation
- 📖 [Design Principles](references/design-principles.md) - Philosophy and guidelines
- 🎨 [Color System](references/color-system.md) - Palettes, gradients, dark mode
- ✏️ [Typography System](references/typography-system.md) - Fonts, scales, hierarchy
- 🎬 [Animation Guide](references/animation-guide.md) - Timing, easing, patterns
- 🧩 [Components Reference](references/components-reference.md) - Full component library

---

**Remember**: Apple's design philosophy is about removing the unnecessary to let the essential shine. Every element should serve a purpose. Keep it simple, elegant, and focused on the user's needs.
