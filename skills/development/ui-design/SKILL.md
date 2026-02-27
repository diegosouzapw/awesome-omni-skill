---
name: ui-design
description: React UI design principles and conventions. Load this skill when building or modifying any user interface. Covers component design, application type detection, Mantine (business apps) and Tailwind (consumer apps), accessibility, responsiveness, and state management patterns.
license: MIT
compatibility: opencode
---

## Determine application type first

Before writing any UI code, determine whether this is a **business-facing** or **consumer-facing** application. This determines the entire styling and component approach.

**Business-facing** (internal tools, dashboards, admin panels, B2B SaaS, data-heavy interfaces):
- Use **Mantine** as the component library
- Style using Mantine's styling system (`styles`, `classNames`, `sx`, CSS Modules with Mantine tokens)
- Do not introduce Tailwind

**Consumer-facing** (marketing sites, consumer products, public-facing apps, brand-driven experiences):
- Use **Tailwind CSS** for all styling
- Use **Radix UI** for unstyled accessible primitives (dialogs, dropdowns, tooltips, etc.)
- Do not introduce Mantine

Do not mix the two systems. Pick one per application and be consistent.

---

## Visual standards

These rules apply regardless of whether you are using Mantine or Tailwind. They express the difference between an interface that looks considered and one that looks generated. Follow them without exception.

### Typography

- Establish a scale of at least three distinct sizes: heading, body, and label/caption. Never use a single font size throughout a UI.
- Body text uses a line height of 1.5. Headings use 1.2–1.3.
- Constrain line length for readable text to 60–80 characters (`max-w-prose` in Tailwind, `maw` in Mantine). Never let paragraphs stretch full width.
- Heading hierarchy is meaningful and sequential — do not skip levels (h1 → h3) or use heading tags for visual size alone.
- Use font weight to establish hierarchy: one heavy weight for primary headings, one medium weight for subheadings, regular for body. Do not use more than three weights on a single screen.

### Spacing

- Derive all spacing from a base-8 scale: 4, 8, 16, 24, 32, 48, 64px. No arbitrary values.
- Spacing between related elements is smaller than spacing between unrelated elements. A label and its input are closer together than two separate form fields.
- Every section of a page has clear breathing room. Content that is cramped to the edges reads as unfinished.
- Padding inside a container is consistent on all sides unless there is a deliberate reason to differ.

### Colour

- Every UI has one primary action colour used consistently for all primary buttons and key interactive elements. It does not appear decoratively.
- Limit accent colours to two or three across the entire interface. More than three competing colours creates visual noise.
- Background, surface, and border colours form a clear hierarchy: page background is the darkest (or lightest), cards/panels sit one step above, inputs and interactive surfaces one step above that.
- Never use colour alone to convey meaning — pair it with an icon, label, or pattern. This applies to status indicators, validation states, and charts.
- All text meets WCAG AA contrast minimums: 4.5:1 for body text, 3:1 for large text and UI components.

### Visual hierarchy

- Every screen has one primary action. It is visually dominant. Secondary actions are visually subordinate.
- The most important content on a page has the most visual weight — through size, contrast, or position, not decoration.
- Decorative elements (dividers, background patterns, icons used ornamentally) are subtle. They must never compete with content.
- Empty space is intentional. Do not fill it. Whitespace is structure.

### Interactive elements

- Every button, link, input, and interactive element has four explicit states: default, hover, focus, and disabled. None of these are left to browser defaults alone.
- Focus states are always visible and never suppressed with `outline: none` without a replacement style.
- Primary buttons are filled. Secondary buttons are outlined or ghost. Destructive actions use a distinct colour (typically red). These conventions are consistent throughout the application.
- Click targets for interactive elements are at minimum 44×44px on touch surfaces.

### Iconography

- Icons are used to reinforce meaning, not replace text in ambiguous contexts. If an icon's meaning is not immediately obvious, it has a visible label or tooltip.
- Icon sizes are consistent within a context — navigation icons are all the same size, inline icons are all the same size.
- Icons are sourced from a single library throughout the application. Do not mix icon sets.

### Forms

- Every input has a visible label above it. Placeholder text is not a substitute for a label.
- Validation errors appear adjacent to the field they relate to, not only at the top of the form.
- Required fields are marked consistently — either all required fields are marked, or all optional fields are marked. Never both.
- Submission buttons are disabled or show a loading state while a request is in flight.
- Multi-field forms group related fields visually (billing address fields together, personal info together).

### Feedback and communication

- Every user action that triggers an async operation shows a loading indicator scoped to that action, not a full-page spinner.
- Success and error feedback is specific. "Profile updated" beats "Success". "Email already in use" beats "Error".
- Destructive actions (delete, archive, revoke) require confirmation. The confirmation UI names the specific thing being destroyed.
- Toasts and notifications are used for transient feedback. Persistent errors live inline near the relevant content.

---

## Component design principles

**One component, one responsibility.** A component should do one thing at one level of abstraction. If a component manages data fetching, layout, and user interaction simultaneously, break it apart.

**Separate concerns between container and presentational components.** Data fetching, state management, and business logic live in container components or hooks. Presentational components receive data and callbacks via props and render UI. Presentational components are easy to test, easy to reuse, and easy to reason about.

**Props are the interface.** Design component props the way you design a function signature — with intention. Required props should be necessary. Optional props should have sensible defaults. Avoid prop bags (`options: {}`) that obscure what a component actually needs.

**Prefer composition over configuration.** Rather than a single component with many boolean flags (`showHeader`, `compact`, `withBorder`), prefer composing smaller components together. Flags are a sign a component is doing too much.

```tsx
// Avoid
<DataTable showFooter compact withBorder headerAction={<Button />} />

// Prefer
<DataTable>
  <DataTable.Header action={<Button />} />
  <DataTable.Body compact />
  <DataTable.Footer />
</DataTable>
```

**Never put logic in JSX.** Extract conditionals and transformations into variables or functions before the return statement. JSX should read like a description of the UI, not a program.

---

## Business apps: Mantine conventions

Use Mantine's component library as-is before building custom components. Reach for `Table`, `DataTable` (mantine-datatable), `Modal`, `Drawer`, `Select`, `MultiSelect`, `DatePicker`, `Notifications`, `Menu`, `Tabs` before writing your own.

Style with Mantine's system in this order of preference:
1. `classNames` prop with CSS Modules for component-level overrides
2. Mantine CSS variables and tokens for consistency with the theme
3. `styles` prop for one-off inline style needs
4. Never use arbitrary hex values or hardcoded spacing — use theme tokens

```tsx
// Good — uses theme tokens
<Box p="md" bg="gray.0" style={{ borderRadius: 'var(--mantine-radius-sm)' }}>

// Bad — hardcoded values
<Box style={{ padding: '16px', background: '#f8f9fa', borderRadius: '4px' }}>
```

Define the theme in one place. Never override Mantine defaults ad hoc in individual components — extend the theme centrally.

Forms use `@mantine/form`. Validate on submit by default, with field-level validation on change after the first submit attempt.

---

## Consumer apps: Tailwind conventions

Use Tailwind utility classes exclusively. Do not write custom CSS unless absolutely necessary (complex animations, third-party overrides).

Use Radix UI primitives for interactive components that require accessibility: `Dialog`, `DropdownMenu`, `Select`, `Tooltip`, `Popover`, `Tabs`, `Accordion`. Style them with Tailwind `className` props.

Establish a design token vocabulary in `tailwind.config.ts` upfront — brand colours, spacing scale, typography scale. Use these tokens throughout. Do not use arbitrary values (`w-[347px]`) except for one-off pixel-perfect requirements.

```tsx
// Good — uses design tokens
<button className="bg-brand-500 hover:bg-brand-600 text-white px-4 py-2 rounded-md text-sm font-medium">

// Avoid — arbitrary values
<button className="bg-[#4F46E5] w-[120px]">
```

Organise Tailwind classes in a consistent order: layout → sizing → spacing → typography → colour → border → effects. Use a Prettier plugin (`prettier-plugin-tailwindcss`) to enforce this automatically.

---

## Accessibility

Accessibility is not optional. Every interactive element must be keyboard navigable and screen reader compatible.

- All images have meaningful `alt` text. Decorative images use `alt=""`.
- Form inputs have associated labels — use `htmlFor` / `id` pairing or wrap the input in the label.
- Interactive elements that are not native `<button>` or `<a>` must have `role` and `aria-*` attributes.
- Colour is never the sole means of conveying information.
- Focus states are always visible — never `outline: none` without a custom focus style.
- Use semantic HTML. `<button>` for actions, `<a>` for navigation, `<nav>`, `<main>`, `<section>`, `<header>`, `<footer>` for structure.

When using Mantine, accessibility is largely handled by the library. When using Radix, the primitives are accessible by default — do not override their ARIA attributes without understanding the implications.

---

## Responsiveness

Design for mobile first. Write base styles for small screens and add breakpoint overrides for larger screens.

In Mantine: use responsive props (`p={{ base: 'sm', md: 'lg' }}`) and `useMediaQuery` hook.

In Tailwind: use mobile-first breakpoint prefixes (`sm:`, `md:`, `lg:`).

Do not hardcode widths for content containers. Use `max-w-*` with `mx-auto` in Tailwind or Mantine's `Container` component.

---

## State management

Local UI state (open/closed, form values, selected tab) lives in `useState` or `useReducer` in the component that owns it.

Shared UI state (current user, theme, notifications, permissions) lives in React context or a lightweight global store (Zustand). Use context for state that changes infrequently. Use Zustand for state that changes often or is accessed by many components.

Server state lives in a data-fetching library — use React Query (`@tanstack/react-query`). Do not replicate server state into `useState`. React Query is the cache.

Do not reach for Redux. It is not justified for new projects.

---

## Loading and error states

Every data-dependent component must handle three states: loading, error, and success. There is no fourth option.

Use skeleton loaders (Mantine's `Skeleton`, or a Tailwind pulse animation) for loading states — not spinners in the middle of content that already has a known shape.

Error states must be actionable. "Something went wrong" with a retry button beats a raw error message.

Do not show empty states and loading states at the same time.

---

## Performance

Lazy load routes using `React.lazy` and `Suspense`. Do not bundle the entire application into one chunk.

Memoize with `useMemo` and `useCallback` only when there is a measured performance problem. Premature memoization adds noise and obscures intent.

Virtualise long lists (100+ rows) using `@tanstack/react-virtual` or `mantine-datatable`'s built-in virtualisation.

Images use correct dimensions, modern formats (WebP, AVIF), and lazy loading (`loading="lazy"`) unless above the fold.

---

## Screenshots

After completing any UI changes, take screenshots of the affected routes and attach them to the task log. This is a required part of the definition of done for all frontend tasks.

### Setup

If `@playwright/test` is not already a dev dependency in the project, install it:

```bash
pnpm add -D @playwright/test
pnpm exec playwright install chromium --with-deps
```

### Screenshots must capture the full interaction, not just the page load

Before deciding what to screenshot, think through the complete user interaction your change introduced or affected. The default page state is only a baseline. The screenshots that actually demonstrate the feature are the ones taken during the interaction — after a button click, after a field reveals, after validation fires, after a success state appears.

Ask yourself: **what would a reviewer need to see to confirm this feature works?** Take screenshots of those states. There will often be several.

Common interaction states to capture:
- The default page state on arrival
- A revealed input, panel, or section after a button click
- An open modal, drawer, or dropdown
- A validation error state after a failed submission
- A success confirmation after a completed action
- Intermediate steps in a multi-step flow

### Workflow

1. **Start the dev server** if it is not already running:

   ```bash
   pnpm dev &
   DEV_PID=$!
   ```

   Wait for the server to be ready before proceeding. Poll the base URL until it responds or time out after 30 seconds.

2. **Think through the interaction** your change introduces. Identify every state a reviewer would need to see to confirm the feature works.

3. **Take screenshots** using a Playwright script, scripting each interaction step before capturing:

   ```js
   import { chromium } from '@playwright/test'
   const browser = await chromium.launch()
   const page = await browser.newPage()
   await page.setViewportSize({ width: 1280, height: 800 })
   await page.goto('http://localhost:PORT/your-route')
   await page.waitForLoadState('networkidle')

   // Default state
   await page.screenshot({ path: 'agent-logs/.../home_default_desktop.png', fullPage: true })

   // After clicking the trigger
   await page.click('button:has-text("Add Item")')
   await page.waitForSelector('label:has-text("Item name")')
   await page.screenshot({ path: 'agent-logs/.../home_item-form-open_desktop.png', fullPage: true })

   // After triggering validation
   await page.click('button:has-text("Save")')
   await page.waitForSelector('text=Item name is required')
   await page.screenshot({ path: 'agent-logs/.../home_validation-error_desktop.png', fullPage: true })

   await browser.close()
   ```

   Repeat at mobile viewport (390×844) for states where responsive behaviour is relevant.

4. **Stop the dev server**:

   ```bash
   kill $DEV_PID 2>/dev/null
   ```

5. **Reference all screenshots in the task log**:

   ```markdown
   ## Screenshots

   ![Default state](./home_default_desktop.png)
   ![Item form open](./home_item-form-open_desktop.png)
   ![Validation error](./home_validation-error_desktop.png)
   ```

### File structure and naming

All task output lives in a single folder. Screenshots sit alongside the task log, named to describe the UI state they show:

```
agent-logs/
  2024-03-15-14-32_add-item-input/
    task.md
    home_default_desktop.png
    home_item-form-open_desktop.png
    home_item-form-open_mobile.png
    home_validation-error_desktop.png
    home_item-saved_desktop.png
```

Name screenshots as `route_state-description_viewport.png`. The state description should be specific enough that someone reading the task log knows what they're about to see before opening the image.

### What to screenshot

- Every route you created or modified
- Every meaningful UI state within those routes, not just the page load
- Mobile viewport for any states where responsive behaviour is relevant

Do not screenshot routes you did not touch. Do not take a single page-load screenshot and call it done if the feature only appears after interaction.