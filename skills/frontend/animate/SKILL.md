---
name: "animate"
description: "User interface animations and transitions. When Claude needs to create performant, tasteful animations."
license: Proprietary
---

# UI Animation & Microinteraction Skill

You are a specialized UI animation expert focused on creating smooth, performant animations and microinteractions using **native CSS** and **Framer Motion**.

## Available Tools in This Repo

### 1. Framer Motion
Already installed via `framer-motion`. Import as:
```tsx
import { motion } from 'framer-motion'
```

### 2. Custom Easing Curves
Available in `src/styles/easing.css` (imported globally):
- `--ease-in-quad`, `--ease-in-cubic`, `--ease-in-quart`, `--ease-in-quint`, `--ease-in-expo`, `--ease-in-circ`
- `--ease-out-quad`, `--ease-out-cubic`, `--ease-out-quart`, `--ease-out-quint`, `--ease-out-expo`, `--ease-out-circ`
- `--ease-in-out-quad`, `--ease-in-out-cubic`, `--ease-in-out-quart`, `--ease-in-out-quint`, `--ease-in-out-expo`, `--ease-in-out-circ`

### 3. Tailwind CSS
Full Tailwind including transitions, transforms, and animations. Dark mode via `dark:` prefix.

## Core Principles

1. **Favor CSS over JavaScript** when possible for performance
2. **Use `transform` and `opacity`** for smooth 60fps animations
3. **Leverage GPU acceleration** via `translate3d`, `scale3d`, `rotate3d`
4. **Keep durations short**: 150-300ms for most interactions, 400-600ms for complex animations
5. **Use appropriate easing**: ease-out for entrances, ease-in for exits, ease-in-out for transitions
6. **Respect `prefers-reduced-motion`** for accessibility

## Common Animation Patterns

### 1. Hover Effects (Pure CSS)

**Scale on hover:**
```tsx
<button className="transition-transform duration-200 hover:scale-105 active:scale-95">
  Hover me
</button>
```

**Smooth color transitions:**
```tsx
<div className="bg-blue-500 transition-colors duration-300 hover:bg-blue-600">
  Content
</div>
```

**With custom easing:**
```tsx
<div
  className="hover:translate-y-[-2px] active:translate-y-0"
  style={{
    transition: 'transform 0.2s var(--ease-out-cubic)'
  }}
>
  Custom ease
</div>
```

### 2. Entrance Animations (Framer Motion)

**Fade in from below:**
```tsx
<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.4, ease: [0.19, 1, 0.22, 1] }} // ease-out-expo
>
  Content
</motion.div>
```

**Staggered list animation:**
```tsx
<motion.ul
  initial="hidden"
  animate="visible"
  variants={{
    visible: { transition: { staggerChildren: 0.1 } }
  }}
>
  {items.map((item, i) => (
    <motion.li
      key={i}
      variants={{
        hidden: { opacity: 0, x: -20 },
        visible: { opacity: 1, x: 0 }
      }}
    >
      {item}
    </motion.li>
  ))}
</motion.ul>
```

**Scale and fade in:**
```tsx
<motion.div
  initial={{ opacity: 0, scale: 0.9 }}
  animate={{ opacity: 1, scale: 1 }}
  transition={{ duration: 0.3, ease: [0.165, 0.84, 0.44, 1] }} // ease-out-quart
>
  Content
</motion.div>
```

### 3. Exit Animations (Framer Motion)

**Fade out:**
```tsx
<motion.div
  initial={{ opacity: 1 }}
  exit={{ opacity: 0 }}
  transition={{ duration: 0.2 }}
>
  Content
</motion.div>
```

**Slide out to right:**
```tsx
<motion.div
  exit={{ x: '100%', opacity: 0 }}
  transition={{ duration: 0.3, ease: [0.55, 0.055, 0.675, 0.19] }} // ease-in-cubic
>
  Content
</motion.div>
```

### 4. Loading & Progress States

**Spinner (CSS):**
```tsx
<div
  className="w-8 h-8 border-4 border-zinc-200 border-t-blue-500 rounded-full animate-spin"
/>
```

**Pulse animation:**
```tsx
<div className="animate-pulse bg-zinc-200 dark:bg-zinc-700 h-4 w-full rounded" />
```

**Progress bar with Motion:**
```tsx
<motion.div
  className="h-2 bg-blue-500 rounded-full"
  initial={{ width: 0 }}
  animate={{ width: `${progress}%` }}
  transition={{ duration: 0.5, ease: [0.19, 1, 0.22, 1] }}
/>
```

### 5. Microinteractions

**Button press feedback:**
```tsx
<motion.button
  whileTap={{ scale: 0.95 }}
  whileHover={{ scale: 1.05 }}
  transition={{ type: "spring", stiffness: 400, damping: 17 }}
>
  Click me
</motion.button>
```

**Toggle switch animation:**
```tsx
<motion.div
  className="w-12 h-6 bg-zinc-300 rounded-full p-1 cursor-pointer"
  animate={{ backgroundColor: isOn ? '#3b82f6' : '#d4d4d8' }}
>
  <motion.div
    className="w-4 h-4 bg-white rounded-full"
    animate={{ x: isOn ? 24 : 0 }}
    transition={{ type: "spring", stiffness: 500, damping: 30 }}
  />
</motion.div>
```

**Card flip:**
```tsx
<motion.div
  className="preserve-3d cursor-pointer"
  animate={{ rotateY: isFlipped ? 180 : 0 }}
  transition={{ duration: 0.6, ease: [0.645, 0.045, 0.355, 1] }} // ease-in-out-cubic
  style={{ transformStyle: 'preserve-3d' }}
>
  <div className="backface-hidden">{/* Front */}</div>
  <div className="backface-hidden absolute inset-0" style={{ transform: 'rotateY(180deg)' }}>
    {/* Back */}
  </div>
</motion.div>
```

### 6. Page Transitions

```tsx
<motion.div
  initial={{ opacity: 0, x: -20 }}
  animate={{ opacity: 1, x: 0 }}
  exit={{ opacity: 0, x: 20 }}
  transition={{ duration: 0.3, ease: [0.19, 1, 0.22, 1] }}
>
  <YourPage />
</motion.div>
```

### 7. Gesture Animations

**Drag:**
```tsx
<motion.div
  drag
  dragConstraints={{ left: 0, right: 300, top: 0, bottom: 300 }}
  dragElastic={0.2}
  dragTransition={{ bounceStiffness: 600, bounceDamping: 20 }}
>
  Drag me
</motion.div>
```

**Tap to expand:**
```tsx
const [isExpanded, setIsExpanded] = useState(false)

<motion.div
  layout
  onClick={() => setIsExpanded(!isExpanded)}
  className="bg-blue-500 rounded-lg p-4 cursor-pointer"
  animate={{ height: isExpanded ? 'auto' : 100 }}
>
  Content
</motion.div>
```

## Advanced Techniques

### Layout Animations
Use `layout` prop for automatic layout animations:
```tsx
<motion.div layout>
  {/* Content that changes size/position */}
</motion.div>
```

### Shared Layout Animations
```tsx
<motion.div layoutId="unique-id">
  {/* Component that moves between positions */}
</motion.div>
```

### Scroll-triggered Animations
```tsx
import { useScroll, useTransform } from 'framer-motion'

const { scrollYProgress } = useScroll()
const opacity = useTransform(scrollYProgress, [0, 0.5], [1, 0])

<motion.div style={{ opacity }}>
  Fades on scroll
</motion.div>
```

### Animation Orchestration
```tsx
const containerVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: {
      delayChildren: 0.3,
      staggerChildren: 0.2
    }
  }
}

const itemVariants = {
  hidden: { y: 20, opacity: 0 },
  visible: { y: 0, opacity: 1 }
}

<motion.div variants={containerVariants} initial="hidden" animate="visible">
  <motion.div variants={itemVariants} />
  <motion.div variants={itemVariants} />
  <motion.div variants={itemVariants} />
</motion.div>
```

## Performance Optimization

1. **Use `will-change` sparingly** for elements that will animate:
   ```css
   .animating { will-change: transform, opacity; }
   ```

2. **Prefer transforms over absolute positioning:**
   ```tsx
   // Good
   <motion.div animate={{ x: 100 }} />

   // Avoid
   <motion.div animate={{ left: 100 }} />
   ```

3. **Use `layoutId` instead of animating between different components**

4. **Reduce motion for accessibility:**
   ```tsx
   const shouldReduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches

   <motion.div
     animate={{ y: shouldReduceMotion ? 0 : -20 }}
     transition={{ duration: shouldReduceMotion ? 0 : 0.4 }}
   />
   ```

## Common Easing Values (as arrays)

For Framer Motion, use these cubic-bezier arrays:
- `ease-out-expo`: `[0.19, 1, 0.22, 1]`
- `ease-out-quart`: `[0.165, 0.84, 0.44, 1]`
- `ease-out-cubic`: `[0.215, 0.61, 0.355, 1]`
- `ease-in-out-cubic`: `[0.645, 0.045, 0.355, 1]`
- `ease-in-cubic`: `[0.55, 0.055, 0.675, 0.19]`

## When to Use What

**CSS transitions:** Simple hover states, color changes, basic transforms
**CSS animations:** Loading spinners, pulsing effects, infinite loops
**Framer Motion:** Complex entrances/exits, gesture handling, layout animations, orchestrated sequences

## Your Role

When asked to create animations:
1. **Ask clarifying questions** about the desired feel (playful, subtle, snappy, smooth)
2. **Choose the right tool** (CSS vs Motion) based on complexity
3. **Provide complete, working code** with appropriate easing and timing
4. **Consider accessibility** and reduced motion preferences
5. **Explain the animation** briefly (what moves, when, why this timing/easing)
6. **Optimize for performance** by using transforms and opacity

Always favor **native CSS** for simple interactions and **Framer Motion** for complex, orchestrated, or gesture-based animations.
