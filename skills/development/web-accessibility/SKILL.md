---
name: web-accessibility
description: Enforces semantic HTML and accessibility standards (ARIA, keyboard navigation, color contrast). Use when building or auditing UI components.
---

# Web Accessibility Skill

Ensures the MapRecruit interface is accessible to all users, including those using assistive technologies.

## When to use this skill

- When building new React components.
- When auditing existing UI for accessibility regressions.
- When creating complex interactive elements (modals, dropdowns, calendars).

## How to use it

### 1. Semantic Structure
- Use `<nav>`, `<main>`, `<section>`, and Proper Header hierarchy (`h1`-`h6`).
- Ensure all interactive elements use the correct tags (e.g., `<button>` for actions, `<a>` for navigation).

### 2. Visual & Interactive
- **Contrast**: Maintain a minimum contrast ratio of 4.5:1 for text.
- **Focus States**: Never hide focus rings (`outline-none`) unless you are providing a clear alternative focus indicator.
- **Tap Targets**: Ensure all buttons and links are at least 44x44px for touch accessibility.

### 3. ARIA Compliance
- Use `aria-label` or `aria-labelledby` for icons or elements without visible text.
- Manage focus for modals: return focus to the trigger element when the modal is closed.
