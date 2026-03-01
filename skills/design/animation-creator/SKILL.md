---
name: animation-creator
description: CSS animation and transition creator. Generates optimized @keyframes, transitions, and transform sequences with performance best practices and motion accessibility. Use when adding animations, transitions, or micro-interactions.
allowed-tools: Read, Write, Edit
---

# Animation Creator Skill

This skill helps you create performant, accessible CSS animations and transitions. I'll guide you through animation requirements and generate optimized code that respects user preferences and follows performance best practices.

## What I Can Create

### Transitions
- Hover effects
- Focus states
- Button interactions
- Modal/dropdown entrances
- Page transitions
- Property changes

### Keyframe Animations
- Loading spinners
- Progress indicators
- Entrance animations
- Attention-seekers
- Scroll animations
- Complex sequences

### Transform Animations
- Slides
- Fades
- Scales
- Rotations
- 3D effects
- Combined transforms

### Micro-interactions
- Button presses
- Toggle switches
- Checkboxes
- Form validations
- Toast notifications
- Tooltips

## How To Use This Skill

Tell me what you want to animate and I'll create the CSS. I'll ask:

1. **What element** are you animating?
2. **What effect** do you want? (fade, slide, bounce, etc.)
3. **How long** should it take? (duration)
4. **When** should it happen? (on hover, on load, on click?)
5. **Should it repeat?** (once, infinite, number of times)

## Performance Best Practices

I always follow these rules for smooth animations:

### Use Compositor-Only Properties
```css
/* ✓ GOOD - GPU accelerated */
.animated {
  transition: transform 0.3s, opacity 0.3s;
}

/* ❌ BAD - Triggers layout/paint */
.animated {
  transition: width 0.3s, height 0.3s, left 0.3s;
}
```

### Optimize with will-change
```css
.animating {
  will-change: transform, opacity;
}

.animating.complete {
  will-change: auto; /* Remove after animation */
}
```

### Respect Reduced Motion
```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

## Common Animation Examples

### Fade In
```css
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.fade-in {
  animation: fadeIn 0.5s ease-in;
}
```

### Slide In from Left
```css
@keyframes slideInLeft {
  from {
    transform: translateX(-100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

.slide-in-left {
  animation: slideInLeft 0.5s ease-out;
}
```

### Bounce
```css
@keyframes bounce {
  0%, 20%, 50%, 80%, 100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-30px);
  }
  60% {
    transform: translateY(-15px);
  }
}

.bounce {
  animation: bounce 1s ease;
}
```

### Pulse (Attention Seeker)
```css
@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}

.pulse {
  animation: pulse 2s ease infinite;
}
```

### Loading Spinner
```css
@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}
```

### Button Hover
```css
.button {
  transition: all 0.2s ease;
}

.button:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.button:active {
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}
```

### Modal Entrance
```css
@keyframes modalFadeIn {
  from {
    opacity: 0;
    transform: scale(0.9) translateY(-20px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

.modal {
  animation: modalFadeIn 0.3s ease-out;
}

/* Backdrop fade */
@keyframes backdropFadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.modal-backdrop {
  animation: backdropFadeIn 0.3s ease;
}
```

### Shake (Error State)
```css
@keyframes shake {
  0%, 100% {
    transform: translateX(0);
  }
  10%, 30%, 50%, 70%, 90% {
    transform: translateX(-10px);
  }
  20%, 40%, 60%, 80% {
    transform: translateX(10px);
  }
}

.shake {
  animation: shake 0.5s ease;
}
```

### Skeleton Loading
```css
@keyframes shimmer {
  0% {
    background-position: -1000px 0;
  }
  100% {
    background-position: 1000px 0;
  }
}

.skeleton {
  background: linear-gradient(
    90deg,
    #f0f0f0 25%,
    #e0e0e0 50%,
    #f0f0f0 75%
  );
  background-size: 1000px 100%;
  animation: shimmer 2s infinite linear;
}
```

### Slide and Fade Menu
```css
.menu {
  transform: translateX(-100%);
  opacity: 0;
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.menu.open {
  transform: translateX(0);
  opacity: 1;
}
```

## Timing Functions

### Built-in Easings
```css
/* Linear - constant speed */
transition: all 0.3s linear;

/* Ease (default) - slow start and end */
transition: all 0.3s ease;

/* Ease-in - slow start */
transition: all 0.3s ease-in;

/* Ease-out - slow end */
transition: all 0.3s ease-out;

/* Ease-in-out - slow start and end (more pronounced) */
transition: all 0.3s ease-in-out;
```

### Custom Cubic Bezier
```css
/* Bouncy */
transition: transform 0.5s cubic-bezier(0.68, -0.55, 0.265, 1.55);

/* Smooth */
transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);

/* Fast out, slow in */
transition: transform 0.3s cubic-bezier(0, 0, 0.2, 1);
```

## Animation Properties Reference

### Duration
```css
animation-duration: 0.3s;   /* Fast */
animation-duration: 0.5s;   /* Normal */
animation-duration: 1s;     /* Slow */
animation-duration: 2s;     /* Very slow */
```

### Iteration
```css
animation-iteration-count: 1;        /* Once */
animation-iteration-count: 3;        /* Three times */
animation-iteration-count: infinite; /* Forever */
```

### Direction
```css
animation-direction: normal;    /* Forward */
animation-direction: reverse;   /* Backward */
animation-direction: alternate; /* Forward then backward */
```

### Fill Mode
```css
animation-fill-mode: none;      /* No persistence */
animation-fill-mode: forwards;  /* Keep final state */
animation-fill-mode: backwards; /* Start in first keyframe */
animation-fill-mode: both;      /* Both forwards and backwards */
```

### Play State
```css
animation-play-state: running; /* Playing */
animation-play-state: paused;  /* Paused */
```

## Accessibility Considerations

I always include:

### Reduced Motion Support
```css
@media (prefers-reduced-motion: reduce) {
  .animated {
    animation-duration: 0.01ms;
    transition-duration: 0.01ms;
  }
}
```

### Avoid Problematic Patterns
- No rapid flashing (seizure risk)
- No parallax for vestibular disorders
- Provide controls for auto-playing animations
- Keep important animations essential, not decorative

### Focus Indicators
```css
.button:focus-visible {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
  transition: outline-offset 0.2s ease;
}
```

## Performance Tips

✓ Animate `transform` and `opacity` only when possible
✓ Use `will-change` sparingly and temporarily
✓ Avoid animating `width`, `height`, `top`, `left`
✓ Keep animations under 300-500ms for UI interactions
✓ Test on low-end devices
✓ Use `requestAnimationFrame` for JavaScript animations
✓ Remove animations from elements outside viewport

## Example Usage

**You say**: "Create a fade and slide up animation for cards appearing on page load"

**I'll provide**:
```css
@keyframes fadeSlideUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.card {
  animation: fadeSlideUp 0.5s ease-out;
  animation-fill-mode: backwards;
}

/* Stagger the animations */
.card:nth-child(1) { animation-delay: 0.1s; }
.card:nth-child(2) { animation-delay: 0.2s; }
.card:nth-child(3) { animation-delay: 0.3s; }

/* Respect reduced motion */
@media (prefers-reduced-motion: reduce) {
  .card {
    animation: none;
    opacity: 1;
    transform: translateY(0);
  }
}
```

## Just Ask!

Tell me what animation you need:
- "Fade in on page load"
- "Smooth color transition on hover"
- "Loading spinner"
- "Shake on error"
- "Slide in menu from left"
- "Pulse button to draw attention"

I'll create performant, accessible CSS animations for you!
