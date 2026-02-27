---
name: frontend-ui
description: Build beautiful, responsive UIs with React, Tailwind CSS, and Framer Motion animations.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Frontend UI Development Masterclass

## Core Philosophy

This skill focuses on creating "premium" web experiences. We don't just build functionality; we build _feeling_. Every interaction should be smooth, every layout responsive, and every aesthetic choice intentional.

## Technology Stack Deep Dive

### 1. React Architecture & Patterns

We use React not just for components, but for **state-driven UI architecture**.

#### Component Composition Patterns

Avoid prop-drilling by using composition (children prop) and slots.

```jsx
// Bad: Prop drilling
<Layout userName={user.name} userAvatar={user.avatar} onLogout={handleLogout} />

// Good: Composition
<Layout
  header={<Header user={user} onLogout={handleLogout} />}
  sidebar={<Sidebar items={navItems} />}
>
  {children}
</Layout>
```

#### Custom Hooks for Logic Separation

Never clutter UI components with complex logic. Extract it.

```javascript
// hooks/useAsync.js
export const useAsync = (asyncFunction, immediate = true) => {
  const [status, setStatus] = useState("idle");
  const [value, setValue] = useState(null);
  const [error, setError] = useState(null);

  const execute = useCallback(() => {
    setStatus("pending");
    setValue(null);
    setError(null);
    return asyncFunction()
      .then((response) => {
        setValue(response);
        setStatus("success");
      })
      .catch((error) => {
        setError(error);
        setStatus("error");
      });
  }, [asyncFunction]);

  useEffect(() => {
    if (immediate) execute();
  }, [execute, immediate]);

  return { execute, status, value, error };
};
```

#### Performance Optimization

- **`useMemo`**: Cache expensive calculations.
- **`useCallback`**: Stabilize function references (crucial for passing props to optimized child components).
- **`React.memo`**: Prevent re-renders of pure components.

### 2. Tailwind CSS Mastery

Tailwind is more than utility classes; it's a design system engine.

#### Arbitrary Values & One-offs

Use `[]` syntax for precise control when design tokens don't fit, but overuse indicates a weak design system.

```html
<div class="top-[17px] w-[calc(100%-20px)] bg-[#1a1a1a]"></div>
```

#### Advanced Selectors

- **Group Hover**: Style child based on parent state.
  ```html
  <div class="group card">
    <p class="text-gray-500 group-hover:text-white transition">Content</p>
  </div>
  ```
- **Peer Modifiers**: Style sibling based on previous sibling state (great for form validation).
  ```html
  <input class="peer invalid:border-red-500" required />
  <p class="invisible peer-invalid:visible text-red-500">Error</p>
  ```
- **Direct Children**: `*:p-4` (applies padding to all direct children).

#### Tailwind Variables (v4+)

Access CSS variables directly in classes if configured, or use arbitrary values to set them.

```html
<div class="--bg-opacity:0.5 bg-black/[var(--bg-opacity)]"></div>
```

### 3. Framer Motion Excellence

Animations should be physics-based, not linear.

#### The `layout` Prop

Magically animate layout changes (reordering lists, expanding cards).

```jsx
<motion.div layout transition={{ type: "spring", stiffness: 300, damping: 30 }}>
  {/* Content that changes size/position */}
</motion.div>
```

#### Variants for Orchestration

Coordinate animations between parent and children.

```jsx
const listVariants = {
  hidden: { opacity: 0 },
  show: {
    opacity: 1,
    transition: {
      staggerChildren: 0.1,
      delayChildren: 0.3,
    },
  },
};

const itemVariants = {
  hidden: { opacity: 0, y: 20 },
  show: { opacity: 1, y: 0 },
};

// Usage
<motion.ul variants={listVariants} initial="hidden" animate="show">
  <motion.li variants={itemVariants}>Item 1</motion.li>
  <motion.li variants={itemVariants}>Item 2</motion.li>
</motion.ul>;
```

#### Gestures

Add tactile feedback easily.

```jsx
<motion.button
  whileHover={{ scale: 1.05 }}
  whileTap={{ scale: 0.95 }}
  transition={{ type: "spring", stiffness: 400, damping: 17 }}
>
  Click Me
</motion.button>
```

## Professional Workflow

### 1. Component scaffolding

Always create components with a future-proof interface.

- use `ts` or `js` with JSDoc
- define `className` prop for overrides using `clsx` and `tailwind-merge`

```javascript
import { cn } from "@/lib/utils"; // standard shadcn/ui utility

export function Button({ className, variant, ...props }) {
  return (
    <button
      className={cn(
        "rounded px-4 py-2 font-medium transition active:scale-95",
        variant === "primary" && "bg-blue-600 text-white hover:bg-blue-700",
        variant === "ghost" && "bg-transparent hover:bg-slate-100",
        className,
      )}
      {...props}
    />
  );
}
```

### 2. Responsive Strategy

- **Mobile First**: default classes are mobile. `md:` is "tablet and up".
- **Container Queries**: Use `@tailwindcss/container-queries` for component-level responsiveness.

### 3. Accessibility (A11y)

- **Radix UI / Headless UI**: Use headless components for complex interactive elements (Dialogs, Dropdowns) to ensure full keyboard navigation and screen reader support.
- **Focus States**: Never remove `outline-none` without adding a custom focus style (`focus-visible:ring-2`).

## Troubleshooting Common Issues

### "My animation jumps/glitches"

- Ensure `layout` animations have stable keys.
- Check if you are animating `height: auto` without `layout` prop.

### "Tailwind classes aren't applying"

- Check `tailwind.config.js` `content` array covers the file path.
- Are you dynamically constructing class names? `bg-${color}-500` will NOT work. Full class names must exist in source code.

### "Re-renders are killing performance"

- Use React DevTools Profiler.
- Check for objects/arrays defined inside component body being passed as props (breaking strict equality checks). Wrap them in `useMemo` or move outside component.
