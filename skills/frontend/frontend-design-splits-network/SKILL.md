---
name: frontend-design
description: UI/UX design patterns using DaisyUI v5 and TailwindCSS for Splits Network
alwaysApply: false
applyTo:
    - "apps/portal/src/**/*.{tsx,jsx}"
    - "apps/candidate/src/**/*.{tsx,jsx}"
    - "apps/corporate/src/**/*.{tsx,jsx}"
    - "apps/portal/src/**/*.css"
    - "apps/candidate/src/**/*.css"
    - "apps/corporate/src/**/*.css"
---

# Frontend Design Skill

This skill provides UI/UX design patterns and visual implementation guidelines for Splits Network applications using DaisyUI v5 and TailwindCSS.

## Memphis Design System (CRITICAL — Read First)

Memphis pages (`*-memphis/` routes, showcase pages in `apps/corporate/src/app/showcase/`) use the Memphis design system (`@splits-network/memphis-ui`) instead of standard DaisyUI. **The rest of this skill applies to non-Memphis pages only.**

### When to Use Memphis vs Standard DaisyUI

- **Memphis**: Any file in a `*-memphis/` directory, any showcase page, any new page being migrated to Memphis
- **Standard DaisyUI**: Existing non-Memphis pages, utility components in `packages/shared-ui/`

### Memphis Styling Hierarchy (use components and named classes BEFORE raw Tailwind)

1. **Memphis UI React components** (`@splits-network/memphis-ui`) — FIRST. 101 components with correct styling baked in. Check `packages/memphis-ui/src/react/components/index.ts`.
2. **Memphis plugin CSS classes** (`btn`, `badge`, `card`, `input`, `select`, etc.) — named classes with border tiers baked in.
3. **Memphis CSS theme classes** (`bg-coral`, `text-dark`, `border-interactive`) — for elements not covered above.
4. **Local components** (`{feature}-memphis/components/`) — must use memphis-ui primitives internally.
5. **Raw Tailwind** — LAST RESORT, only for layout/spacing/grid. Never for visual styling when a component or class exists.

```tsx
// ❌ WRONG on Memphis pages — raw Tailwind for a button
<button className="bg-coral text-dark border-4 border-dark font-bold uppercase px-6 py-3">Submit</button>

// ✅ CORRECT — use the Memphis UI component
import { Button } from '@splits-network/memphis-ui';
<Button variant="primary" size="md">Submit</Button>

// ✅ ALSO ACCEPTABLE — use the plugin CSS class
<button className="btn btn-coral btn-md">Submit</button>
```

Memphis design rules: no shadows, no rounded corners, no gradients, Memphis palette only (coral, teal, yellow, purple, dark, cream), geometric decorations.

See `.claude/memphis/references/design-principles.md` for full Memphis guidelines.

---

## Core Principles (Standard DaisyUI — Non-Memphis Pages)

### Design System Foundation

- **DaisyUI v5**: Primary component library with semantic components
- **TailwindCSS**: Utility-first CSS framework for custom styling
- **Semantic Colors**: Use DaisyUI color names (`primary`, `secondary`, `base-100`, etc.) for theme compatibility
- **Consistency**: Follow established patterns for predictable user experience

### Visual Hierarchy

- Clear information hierarchy with typography scales
- Appropriate use of color for emphasis and status
- Strategic use of whitespace for content grouping
- Consistent spacing using Tailwind's spacing scale

### Accessibility First

- Semantic HTML elements (`<button>`, `<fieldset>`, `<nav>`)
- ARIA labels where needed
- Keyboard navigation support
- Sufficient color contrast

## Color System

### Semantic Colors (DaisyUI)

```tsx
// ✅ Use semantic colors for theme compatibility
<button className="btn btn-primary">Primary Action</button>
<div className="bg-base-100 text-base-content">Content</div>
<span className="text-error">Error message</span>

// ❌ Avoid hardcoded Tailwind colors for theme-dependent elements
<button className="bg-blue-500 text-white">Bad</button>
```

### Color Categories

**Brand Colors:**

- `primary` - Primary brand actions (submit, confirm)
- `secondary` - Secondary brand actions
- `accent` - Accent highlights

**UI Colors:**

- `base-100` - Page background (lightest)
- `base-200` - Elevated surface
- `base-300` - More elevated surface
- `base-content` - Text on base colors

**Semantic Status:**

- `success` - Success states and confirmations
- `warning` - Warning states and cautions
- `error` - Error states and destructive actions
- `info` - Informational messages

**Neutral:**

- `neutral` - Neutral actions and content
- `neutral-content` - Text on neutral backgrounds

### Color Usage Guidelines

```tsx
// Page backgrounds
<div className="bg-base-100">Main content area</div>

// Cards and elevated surfaces
<div className="card bg-base-200">Card content</div>

// Status indicators
<span className="badge badge-success">Active</span>
<span className="badge badge-warning">Pending</span>
<span className="badge badge-error">Failed</span>

// Text colors
<p className="text-base-content">Body text</p>
<h1 className="text-base-content">Heading</h1>
<span className="text-primary">Link or emphasis</span>
```

## Typography

### Font Scales

```tsx
// Headings
<h1 className="text-4xl font-bold">Page Title</h1>
<h2 className="text-3xl font-semibold">Section Heading</h2>
<h3 className="text-2xl font-semibold">Subsection</h3>
<h4 className="text-xl font-medium">Card Title</h4>

// Body Text
<p className="text-base">Standard body text (16px)</p>
<p className="text-sm">Secondary text (14px)</p>
<p className="text-xs">Caption text (12px)</p>

// Large Text
<div className="text-5xl font-bold">Hero Text</div>
```

### Font Weights

- `font-normal` (400) - Body text
- `font-medium` (500) - Emphasis, labels
- `font-semibold` (600) - Subheadings
- `font-bold` (700) - Headings, strong emphasis

### Text Utilities

```tsx
// Alignment
<p className="text-left">Left aligned</p>
<p className="text-center">Center aligned</p>
<p className="text-right">Right aligned</p>

// Text truncation
<p className="truncate">Long text that gets cut off with ellipsis...</p>
<p className="line-clamp-3">Text limited to 3 lines</p>

// Text transform
<span className="uppercase">Uppercase</span>
<span className="capitalize">Capitalize Each Word</span>
```

## Layout Patterns

### Container Widths

```tsx
// Full width
<div className="w-full">100% width</div>

// Max width with centering
<div className="max-w-7xl mx-auto px-4">Large container</div>
<div className="max-w-4xl mx-auto px-4">Medium container</div>
<div className="max-w-2xl mx-auto px-4">Small container</div>

// Fixed widths
<div className="w-64">256px (16rem)</div>
<div className="w-96">384px (24rem)</div>
```

### Flexbox Layouts

```tsx
// Horizontal with space between
<div className="flex justify-between items-center">
  <div>Left content</div>
  <div>Right content</div>
</div>

// Vertical stack with gap
<div className="flex flex-col gap-4">
  <div>Item 1</div>
  <div>Item 2</div>
</div>

// Centered content
<div className="flex items-center justify-center min-h-screen">
  <div>Centered content</div>
</div>

// Responsive horizontal to vertical
<div className="flex flex-col md:flex-row gap-4">
  <div>Item 1</div>
  <div>Item 2</div>
</div>
```

### Grid Layouts

```tsx
// Equal columns
<div className="grid grid-cols-3 gap-4">
  <div>Column 1</div>
  <div>Column 2</div>
  <div>Column 3</div>
</div>

// Responsive columns
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <div>Item</div>
  <div>Item</div>
  <div>Item</div>
</div>

// Auto-fit columns
<div className="grid grid-cols-[repeat(auto-fit,minmax(250px,1fr))] gap-4">
  <div>Responsive item</div>
</div>
```

### Spacing Scale

Use consistent spacing from Tailwind's scale (0.25rem increments):

- `gap-1` (0.25rem / 4px)
- `gap-2` (0.5rem / 8px)
- `gap-4` (1rem / 16px) - Common for components
- `gap-6` (1.5rem / 24px)
- `gap-8` (2rem / 32px) - Common for sections

```tsx
// Component spacing
<div className="space-y-4">Vertical stack with 1rem gap</div>
<div className="space-x-2">Horizontal inline with 0.5rem gap</div>

// Padding
<div className="p-4">Equal padding all sides</div>
<div className="px-4 py-2">Horizontal 1rem, vertical 0.5rem</div>

// Margins
<div className="mb-6">Margin bottom 1.5rem</div>
<div className="mx-auto">Center horizontally</div>
```

## Component Patterns

### Buttons

```tsx
// Primary actions
<button className="btn btn-primary">Save Changes</button>

// Secondary actions
<button className="btn btn-outline">Cancel</button>
<button className="btn btn-ghost">Dismiss</button>

// Destructive actions
<button className="btn btn-error">Delete</button>

// Button with icon
<button className="btn btn-primary">
  <i className="fa-duotone fa-regular fa-check"></i>
  Confirm
</button>

// Button sizes
<button className="btn btn-xs">Extra Small</button>
<button className="btn btn-sm">Small</button>
<button className="btn btn-md">Medium (default)</button>
<button className="btn btn-lg">Large</button>

// Button group
<div className="join">
  <button className="btn join-item">Left</button>
  <button className="btn join-item">Center</button>
  <button className="btn join-item">Right</button>
</div>

// Loading state
<button className="btn btn-primary" disabled>
  <span className="loading loading-spinner loading-sm"></span>
  Loading...
</button>
```

### Cards

```tsx
// Basic card
<div className="card bg-base-200">
  <div className="card-body">
    <h3 className="card-title">Card Title</h3>
    <p>Card content goes here.</p>
    <div className="card-actions justify-end">
      <button className="btn btn-primary">Action</button>
    </div>
  </div>
</div>

// Card with image
<div className="card bg-base-200">
  <figure>
    <img src="/image.jpg" alt="Description" />
  </figure>
  <div className="card-body">
    <h3 className="card-title">Card with Image</h3>
    <p>Content below image</p>
  </div>
</div>

// Horizontal card (responsive)
<div className="card bg-base-200 sm:card-side">
  <figure className="w-full sm:w-48">
    <img src="/image.jpg" alt="Description" />
  </figure>
  <div className="card-body">
    <h3 className="card-title">Horizontal Card</h3>
    <p>Content beside image on larger screens</p>
  </div>
</div>

// Card grid
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <div className="card bg-base-200">
    <div className="card-body">Content</div>
  </div>
  {/* More cards */}
</div>
```

### Forms (DaisyUI v5 Pattern)

```tsx
// Text input with label
<fieldset className="fieldset">
  <legend className="fieldset-legend">Email Address *</legend>
  <input
    type="email"
    className="input w-full"
    placeholder="you@example.com"
    required
  />
  <p className="fieldset-label">We'll never share your email</p>
</fieldset>

// Select dropdown
<fieldset className="fieldset">
  <legend className="fieldset-legend">Select Role</legend>
  <select className="select w-full">
    <option value="">Choose...</option>
    <option value="recruiter">Recruiter</option>
    <option value="admin">Administrator</option>
  </select>
</fieldset>

// Textarea
<fieldset className="fieldset">
  <legend className="fieldset-legend">Description</legend>
  <textarea
    className="textarea w-full h-24"
    placeholder="Enter details..."
  />
</fieldset>

// Two-column layout
<div className="grid grid-cols-1 md:grid-cols-2 gap-4">
  <fieldset className="fieldset">
    <legend className="fieldset-legend">First Name</legend>
    <input type="text" className="input w-full" />
  </fieldset>
  <fieldset className="fieldset">
    <legend className="fieldset-legend">Last Name</legend>
    <input type="text" className="input w-full" />
  </fieldset>
</div>

// Form actions
<div className="flex gap-2 justify-end mt-6">
  <button type="button" className="btn btn-ghost">Cancel</button>
  <button type="submit" className="btn btn-primary">Save</button>
</div>
```

### Navigation

```tsx
// Top navbar
<nav className="navbar bg-base-200">
  <div className="navbar-start">
    <a className="btn btn-ghost text-xl">Brand</a>
  </div>
  <div className="navbar-center hidden lg:flex">
    <ul className="menu menu-horizontal px-1">
      <li><a>Dashboard</a></li>
      <li><a>Jobs</a></li>
      <li><a>Candidates</a></li>
    </ul>
  </div>
  <div className="navbar-end">
    <button className="btn btn-primary">Sign In</button>
  </div>
</nav>

// Sidebar navigation
<aside className="w-64 bg-base-200 min-h-screen">
  <ul className="menu p-4">
    <li><a><i className="fa-duotone fa-regular fa-house"></i> Dashboard</a></li>
    <li><a><i className="fa-duotone fa-regular fa-briefcase"></i> Jobs</a></li>
    <li><a><i className="fa-duotone fa-regular fa-users"></i> Candidates</a></li>
  </ul>
</aside>

// Breadcrumbs
<div className="breadcrumbs text-sm">
  <ul>
    <li><a>Home</a></li>
    <li><a>Jobs</a></li>
    <li>Software Engineer</li>
  </ul>
</div>

// Tabs
<div role="tablist" className="tabs tabs-box">
  <button role="tab" className="tab tab-active">Overview</button>
  <button role="tab" className="tab">Applications</button>
  <button role="tab" className="tab">Settings</button>
</div>
```

### Tables

```tsx
// Basic table
<div className="overflow-x-auto">
  <table className="table">
    <thead>
      <tr>
        <th>Name</th>
        <th>Email</th>
        <th>Status</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>John Doe</td>
        <td>john@example.com</td>
        <td><span className="badge badge-success">Active</span></td>
        <td>
          <button className="btn btn-ghost btn-xs">Edit</button>
        </td>
      </tr>
    </tbody>
  </table>
</div>

// Zebra striping
<table className="table table-zebra">
  {/* ... */}
</table>

// Pinned columns
<table className="table table-pin-cols">
  <thead>
    <tr>
      <th>Name</th>
      <td>Email</td>
      <td>Phone</td>
      <th>Actions</th>
    </tr>
  </thead>
  {/* Columns with <th> stay pinned when scrolling */}
</table>
```

### Lists

```tsx
// Simple list
<ul className="list">
  <li className="list-row">
    <div>Item label</div>
    <div>Item value</div>
  </li>
  <li className="list-row">
    <div>Another label</div>
    <div>Another value</div>
  </li>
</ul>

// List with icons
<ul className="list">
  <li className="list-row">
    <i className="fa-duotone fa-regular fa-check text-success"></i>
    <div className="list-col-grow">Completed task</div>
    <span className="text-sm text-base-content/60">2 hours ago</span>
  </li>
</ul>

// Card-based list
<div className="space-y-4">
  <div className="card bg-base-200">
    <div className="card-body">
      <div className="flex justify-between items-center">
        <div>
          <h4 className="font-semibold">Item Title</h4>
          <p className="text-sm text-base-content/60">Item description</p>
        </div>
        <button className="btn btn-ghost btn-sm">View</button>
      </div>
    </div>
  </div>
</div>
```

### Badges and Status

```tsx
// Status badges
<span className="badge badge-success">Active</span>
<span className="badge badge-warning">Pending</span>
<span className="badge badge-error">Failed</span>
<span className="badge badge-info">New</span>

// Badge sizes
<span className="badge badge-xs">Extra Small</span>
<span className="badge badge-sm">Small</span>
<span className="badge badge-md">Medium</span>
<span className="badge badge-lg">Large</span>

// Outlined badges
<span className="badge badge-outline badge-primary">Outlined</span>

// In context
<div className="flex items-center gap-2">
  <h3 className="text-lg font-semibold">Job Title</h3>
  <span className="badge badge-success">Open</span>
</div>
```

### Modals and Drawers

```tsx
// Modal with HTML dialog
<button onClick={() => document.getElementById('my_modal').showModal()}>
  Open Modal
</button>

<dialog id="my_modal" className="modal">
  <div className="modal-box">
    <h3 className="text-lg font-bold">Modal Title</h3>
    <p className="py-4">Modal content goes here</p>
    <div className="modal-action">
      <form method="dialog">
        <button className="btn">Close</button>
      </form>
    </div>
  </div>
  <form method="dialog" className="modal-backdrop">
    <button>close</button>
  </form>
</dialog>

// Drawer
<div className="drawer">
  <input id="my_drawer" type="checkbox" className="drawer-toggle" />
  <div className="drawer-content">
    <label htmlFor="my_drawer" className="btn btn-primary drawer-button">
      Open Drawer
    </label>
  </div>
  <div className="drawer-side">
    <label htmlFor="my_drawer" className="drawer-overlay"></label>
    <div className="menu bg-base-200 min-h-full w-80 p-4">
      <h3 className="text-lg font-bold mb-4">Drawer Title</h3>
      {/* Drawer content */}
    </div>
  </div>
</div>
```

### Alerts and Messages

```tsx
// Success alert
<div className="alert alert-success">
  <i className="fa-duotone fa-regular fa-circle-check"></i>
  <span>Your changes have been saved successfully!</span>
</div>

// Error alert
<div className="alert alert-error">
  <i className="fa-duotone fa-regular fa-circle-exclamation"></i>
  <span>Failed to save changes. Please try again.</span>
</div>

// Warning alert
<div className="alert alert-warning">
  <i className="fa-duotone fa-regular fa-triangle-exclamation"></i>
  <span>This action cannot be undone.</span>
</div>

// Info alert
<div className="alert alert-info">
  <i className="fa-duotone fa-regular fa-circle-info"></i>
  <span>New features have been released!</span>
</div>

// Alert with actions
<div className="alert alert-warning">
  <i className="fa-duotone fa-regular fa-triangle-exclamation"></i>
  <span>Your session will expire in 5 minutes.</span>
  <div>
    <button className="btn btn-sm btn-ghost">Dismiss</button>
    <button className="btn btn-sm btn-primary">Extend Session</button>
  </div>
</div>
```

## Loading and Empty States

### Loading Indicators

```tsx
// Spinner
<span className="loading loading-spinner loading-lg"></span>

// Loading dots
<span className="loading loading-dots loading-md"></span>

// Loading in button
<button className="btn btn-primary" disabled>
  <span className="loading loading-spinner loading-sm"></span>
  Loading...
</button>

// Skeleton loading
<div className="space-y-4">
  <div className="skeleton h-8 w-3/4"></div>
  <div className="skeleton h-4 w-full"></div>
  <div className="skeleton h-4 w-5/6"></div>
</div>

// Card skeleton
<div className="card bg-base-200">
  <div className="card-body">
    <div className="skeleton h-6 w-1/2 mb-4"></div>
    <div className="skeleton h-4 w-full mb-2"></div>
    <div className="skeleton h-4 w-3/4"></div>
  </div>
</div>
```

### Empty States

```tsx
// Empty state with icon
<div className="flex flex-col items-center justify-center py-12 text-center">
  <i className="fa-duotone fa-regular fa-inbox text-6xl text-base-content/30 mb-4"></i>
  <h3 className="text-xl font-semibold mb-2">No items yet</h3>
  <p className="text-base-content/60 mb-4">
    Get started by creating your first item
  </p>
  <button className="btn btn-primary">Create Item</button>
</div>

// Empty search results
<div className="text-center py-8">
  <i className="fa-duotone fa-regular fa-magnifying-glass text-4xl text-base-content/30 mb-4"></i>
  <h3 className="text-lg font-semibold mb-2">No results found</h3>
  <p className="text-base-content/60">
    Try adjusting your search filters
  </p>
</div>

// Error state
<div className="flex flex-col items-center justify-center py-12 text-center">
  <i className="fa-duotone fa-regular fa-triangle-exclamation text-6xl text-error mb-4"></i>
  <h3 className="text-xl font-semibold mb-2">Something went wrong</h3>
  <p className="text-base-content/60 mb-4">
    We couldn't load this content. Please try again.
  </p>
  <button className="btn btn-primary">Retry</button>
</div>
```

## Responsive Design

### Breakpoints

Tailwind breakpoints:

- `sm:` - 640px and up
- `md:` - 768px and up
- `lg:` - 1024px and up
- `xl:` - 1280px and up
- `2xl:` - 1536px and up

### Common Responsive Patterns

```tsx
// Hide/show on mobile
<div className="hidden md:block">Desktop only</div>
<div className="block md:hidden">Mobile only</div>

// Responsive columns
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {/* Items */}
</div>

// Responsive text sizes
<h1 className="text-2xl md:text-4xl lg:text-5xl">Responsive Heading</h1>

// Responsive padding
<div className="p-4 md:p-6 lg:p-8">Content</div>

// Responsive flexbox direction
<div className="flex flex-col md:flex-row gap-4">
  <div>Item 1</div>
  <div>Item 2</div>
</div>

// Mobile menu pattern
<div className="navbar bg-base-200">
  <div className="navbar-start">
    <div className="dropdown">
      <label tabIndex={0} className="btn btn-ghost lg:hidden">
        <i className="fa-duotone fa-regular fa-bars"></i>
      </label>
      <ul className="menu dropdown-content mt-3 p-2 shadow bg-base-200 rounded-box w-52">
        <li><a>Dashboard</a></li>
        <li><a>Jobs</a></li>
      </ul>
    </div>
  </div>
  <div className="navbar-center hidden lg:flex">
    <ul className="menu menu-horizontal">
      <li><a>Dashboard</a></li>
      <li><a>Jobs</a></li>
    </ul>
  </div>
</div>
```

## Design Tokens

### Custom Theme Configuration

```css
/* apps/portal/src/app/globals.css */
@import "tailwindcss";
@plugin "daisyui" {
    themes:
        light --default,
        dark --prefersdark;

    /* Or custom theme */
    themes:
        light,
        dark,
        custom --default;
}

/* Custom theme definition */
@plugin "daisyui/theme" {
    name: "custom";
    default: true;
    color-scheme: light;

    --color-primary: oklch(55% 0.3 240);
    --color-secondary: oklch(70% 0.25 200);
    --color-accent: oklch(65% 0.25 160);

    --color-base-100: oklch(98% 0.02 240);
    --color-base-200: oklch(95% 0.03 240);
    --color-base-300: oklch(92% 0.04 240);
    --color-base-content: oklch(20% 0.05 240);

    --radius-box: 0.5rem;
    --radius-field: 0.25rem;
}
```

### Using CSS Variables

```tsx
// Access theme colors in custom styles
<div style={{ backgroundColor: 'var(--color-primary)' }}>
  Custom styled element
</div>

// Or use Tailwind utilities
<div className="bg-primary text-primary-content">
  Theme-aware component
</div>
```

## Best Practices

### Component Composition

```tsx
// ✅ Compose with DaisyUI and Tailwind utilities
<div className="card bg-base-200 shadow-xl">
  <div className="card-body">
    <h2 className="card-title text-2xl">Title</h2>
    <p className="text-base-content/80">Description</p>
  </div>
</div>

// ❌ Don't fight against utility classes
<div style={{ padding: '15px', borderRadius: '12px' }}>
  <h2 style={{ fontSize: '24px' }}>Bad practice</h2>
</div>
```

### Consistency

```tsx
// ✅ Consistent spacing and sizing
<div className="space-y-4">
  <div className="card bg-base-200">Content 1</div>
  <div className="card bg-base-200">Content 2</div>
</div>

// ✅ Consistent button styles for similar actions
<div className="flex gap-2">
  <button className="btn btn-primary">Save</button>
  <button className="btn btn-ghost">Cancel</button>
</div>
```

### Accessibility

```tsx
// ✅ Proper semantic HTML
<nav>
  <ul className="menu">
    <li><a href="/dashboard">Dashboard</a></li>
  </ul>
</nav>

// ✅ ARIA labels for icons
<button aria-label="Delete item" className="btn btn-ghost btn-sm">
  <i className="fa-duotone fa-regular fa-trash"></i>
</button>

// ✅ Focus states
<button className="btn focus:ring-2 focus:ring-primary">
  Accessible Button
</button>
```

### Performance

```tsx
// ✅ Use Tailwind utilities instead of custom CSS
<div className="flex items-center gap-2">Fast</div>

// ❌ Avoid inline styles when utilities exist
<div style={{ display: 'flex', alignItems: 'center', gap: '0.5rem' }}>
  Slower
</div>
```

## Common Mistakes to Avoid

1. **❌ Using Tailwind color names instead of semantic DaisyUI colors**

    ```tsx
    // Bad - won't adapt to theme changes
    <div className="bg-blue-500 text-white">Content</div>

    // Good - theme-aware
    <div className="bg-primary text-primary-content">Content</div>
    ```

2. **❌ Inconsistent spacing**

    ```tsx
    // Bad - random spacing values
    <div className="mb-3">
    <div className="mb-5">
    <div className="mb-7">

    // Good - consistent spacing scale
    <div className="mb-4">
    <div className="mb-4">
    <div className="mb-4">
    ```

3. **❌ Fighting DaisyUI component styles**

    ```tsx
    // Bad - overriding with !important
    <button className="btn !bg-red-500 !text-white">Button</button>

    // Good - use DaisyUI modifiers
    <button className="btn btn-error">Button</button>
    ```

4. **❌ Not using responsive utilities**

    ```tsx
    // Bad - not responsive
    <div className="grid grid-cols-3">

    // Good - responsive columns
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
    ```

## Resources

- [TailwindCSS Documentation](https://tailwindcss.com)
- [Tailwind Color Reference](https://tailwindcss.com/docs/customizing-colors)
- Project: `docs/guidance/form-controls.md` - Form implementation patterns
