---
name: adding-animations
description: Add micro-interactions and animations using Framer Motion. Use when user asks about animations, transitions, hover effects, or motion design. MANDATORY for every component.
allowed-tools: Read, Write, Edit, Glob, Grep, Task
user-invocable: true
---

# Adding Animations (MANDATORY)

Every component MUST have Framer Motion animations.

## APEX WORKFLOW

### Phase 0: ANALYZE EXISTING ANIMATIONS

```
Task: explore-codebase
Prompt: "Find existing Framer Motion patterns: variants, timing,
easing, hover effects. Report animation conventions."
```

**RULE:** Match existing animation patterns OR propose migration.

## Standard Patterns

### Container + Stagger (REQUIRED)

```tsx
import { motion } from "framer-motion";

const container = {
  hidden: { opacity: 0 },
  show: { opacity: 1, transition: { staggerChildren: 0.1 } }
};

const item = {
  hidden: { opacity: 0, y: 20 },
  show: { opacity: 1, y: 0 }
};

<motion.div variants={container} initial="hidden" animate="show">
  <motion.div variants={item}>Item 1</motion.div>
  <motion.div variants={item}>Item 2</motion.div>
</motion.div>
```

### Hover Effects (REQUIRED)

```tsx
// Card hover
<motion.div whileHover={{ y: -4 }} transition={{ duration: 0.2 }}>

// Button hover
<motion.button
  whileHover={{ scale: 1.02 }}
  whileTap={{ scale: 0.98 }}
>
```

### Scroll Animation

```tsx
<motion.div
  initial={{ opacity: 0, y: 40 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true, margin: "-100px" }}
/>
```

## Timing Guidelines

| Interaction | Duration | Easing |
|-------------|----------|--------|
| Hover | 50-100ms | ease-out |
| Button press | 100-150ms | ease-out |
| Modal open | 200-300ms | ease-out |
| Page transition | 300-400ms | ease-in-out |

## FORBIDDEN

```tsx
// ❌ Random bouncing loops
animate={{ y: [0, -10, 0] }}
transition={{ repeat: Infinity }}

// ❌ Excessive effects
whileHover={{ scale: 1.2, rotate: 5 }}

// ❌ Slow animations
transition={{ duration: 1.5 }}
```

## Accessibility (MANDATORY)

```tsx
import { useReducedMotion } from "framer-motion";

function Component() {
  const shouldReduce = useReducedMotion();
  return (
    <motion.div
      animate={shouldReduce ? {} : { y: 0 }}
      transition={shouldReduce ? { duration: 0 } : { duration: 0.3 }}
    />
  );
}
```

## Validation

```
[ ] Existing animations analyzed (Phase 0)
[ ] Patterns match existing OR migration proposed
[ ] Stagger on lists/grids
[ ] Hover on all interactive elements
[ ] prefers-reduced-motion respected
[ ] No excessive/random animations
```

## References

- **Motion Patterns**: `../../references/motion-patterns.md` (Framer Motion patterns)
- **Buttons Guide**: `../../references/buttons-guide.md` (hover 50-100ms, press 100-150ms)
- **Cards Guide**: `../../references/cards-guide.md` (whileHover y: -4, shadow transitions)
- **UI Visual Design**: `../../references/ui-visual-design.md` (micro-interactions, 2026 trends)
- **UX Principles**: `../../references/ux-principles.md` (accessibility, reduced motion)
- **Design Patterns**: `../../references/design-patterns.md`
- **Component Examples**: `../../references/component-examples.md`
