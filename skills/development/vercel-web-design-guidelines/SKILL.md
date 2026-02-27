---
name: vercel-web-design-guidelines
description: Reviews UI code for compliance with web interface best practices. Use when auditing accessibility, reviewing UI/UX patterns, checking performance, or improving web design quality. Triggers on "check my site", "audit design", "accessibility review", "UX best practices", or UI code review tasks. Covers 100+ rules for accessibility, performance, and user experience.
license: MIT (Vercel Labs)
---

# Web Interface Guidelines

Reviews UI code for compliance with web interface best practices, auditing code for 100+ rules covering accessibility, performance, and UX.

## When to Apply

- Reviewing UI components for accessibility compliance
- Auditing web pages against best practices
- Checking UX patterns and user interactions
- Improving form validation and error handling
- Optimizing animations and transitions

## Core Categories

| Category | Focus Areas |
|----------|-------------|
| Accessibility | Semantic HTML, ARIA, keyboard navigation, screen readers |
| Performance | Loading, rendering, animations, lazy loading |
| UX Patterns | Forms, feedback, error handling, navigation |
| Visual Design | Typography, spacing, color contrast, responsiveness |
| Interactions | Hover states, focus management, touch targets |

## Key Guidelines

### Accessibility (A11Y)

**Semantic HTML:**
- Use `<button>` for actions, `<a>` for navigation
- Use heading hierarchy (`h1` → `h2` → `h3`)
- Use `<nav>`, `<main>`, `<article>`, `<aside>` landmarks

**Keyboard Navigation:**
- All interactive elements must be focusable
- Visible focus indicators (never `outline: none` without replacement)
- Logical tab order following visual flow
- Escape key closes modals/dropdowns

**ARIA:**
- Use ARIA only when native HTML insufficient
- `aria-label` for icon-only buttons
- `aria-expanded` for expandable content
- `aria-live` for dynamic content updates

**Color & Contrast:**
- Minimum 4.5:1 contrast ratio for text
- Don't convey information by color alone
- Support reduced-motion preferences

### Forms & Validation

**Labels & Instructions:**
- Every input needs a visible `<label>`
- Associate labels with `htmlFor`/`id`
- Provide clear placeholder text as hints, not labels

**Error Handling:**
- Show errors inline near the field
- Use `aria-invalid` and `aria-describedby`
- Don't clear form on error
- Focus first error field on submit

**Input Patterns:**
```tsx
<div className="field">
  <label htmlFor="email">Email</label>
  <input
    id="email"
    type="email"
    aria-invalid={!!error}
    aria-describedby={error ? "email-error" : undefined}
  />
  {error && <span id="email-error" role="alert">{error}</span>}
</div>
```

### Performance

**Loading:**
- Show skeleton loaders, not spinners
- Progressive loading for large content
- Lazy load below-fold images
- Preload critical assets

**Animations:**
- Use CSS transforms and opacity (GPU-accelerated)
- Respect `prefers-reduced-motion`
- Keep animations under 300ms for feedback
- Use `will-change` sparingly

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

### Interactive Elements

**Buttons & Links:**
- Minimum 44x44px touch targets
- Clear hover/focus/active states
- Disabled state visually distinct
- Loading states for async actions

**Focus Management:**
- Trap focus in modals
- Return focus after modal closes
- Skip links for keyboard users

### Responsive Design

- Mobile-first approach
- Fluid typography with `clamp()`
- Touch-friendly spacing on mobile
- Test at common breakpoints (320, 768, 1024, 1440)

## Audit Checklist

When reviewing UI code, verify:

1. [ ] All images have `alt` text
2. [ ] Form inputs have associated labels
3. [ ] Color contrast meets WCAG 2.1 AA
4. [ ] Interactive elements keyboard accessible
5. [ ] Focus indicators visible
6. [ ] Animations respect reduced-motion
7. [ ] Error messages are descriptive
8. [ ] Touch targets are 44px minimum
9. [ ] Headings follow hierarchy
10. [ ] ARIA used correctly (not overused)

## Output Format

Report findings as:
```
file.tsx:42 - Missing alt text on image
file.tsx:67 - Button missing accessible name
file.tsx:89 - Low contrast ratio (2.1:1, needs 4.5:1)
```
