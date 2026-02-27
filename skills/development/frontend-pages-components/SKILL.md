---
name: frontend-pages-components
description: Build responsive frontend pages, reusable components, layouts, and styling using modern best practices.
---

# Frontend Page & Component Development

## Instructions

1. **Page Structure**
   - Semantic HTML5 structure (header, main, section, footer)
   - Clear content hierarchy
   - Accessibility-first markup (ARIA where needed)

2. **Layout System**
   - Use Flexbox and Grid appropriately
   - Responsive layouts for mobile, tablet, and desktop
   - Consistent spacing and alignment

3. **Components**
   - Reusable and composable UI components
   - Props-driven or data-driven design
   - Separation of structure, style, and behavior

4. **Styling**
   - Modern CSS (Flexbox, Grid, variables)
   - Utility-first or component-based styling
   - Scalable class naming (BEM / Tailwind / CSS Modules)

5. **Responsiveness**
   - Mobile-first approach
   - Breakpoints for key screen sizes
   - Fluid typography and spacing

6. **Interactivity**
   - Hover, focus, and active states
   - Smooth transitions and subtle animations
   - Keyboard and screen-reader friendly

## Best Practices
- Write clean, readable, and maintainable code
- Avoid inline styles unless necessary
- Use design tokens for colors and spacing
- Optimize for performance and accessibility
- Keep components small and single-purpose

## Example Structure

```html
<main class="page">
  <header class="page-header">
    <h1 class="page-title">Page Title</h1>
  </header>

  <section class="content-grid">
    <article class="card">
      <h2 class="card-title">Component Title</h2>
      <p class="card-text">
        This is a reusable UI component.
      </p>
      <button class="btn-primary">Action</button>
    </article>
  </section>

  <footer class="page-footer">
    <p>© 2026 Your Company</p>
  </footer>
</main>
