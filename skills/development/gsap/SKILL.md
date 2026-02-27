---
name: gsap
description: Professional animation implementation using GSAP (GreenSock Animation Platform). Use this skill when users request animations, motion design, scroll effects, timeline sequences, interactive animations, or when building animated components, landing pages, or web experiences. Creates production-ready, performant animations with proper easing, timing, and sequencing.
license: MIT License - GSAP has its own license terms for commercial use
---

This skill guides the creation of professional, production-grade animations using GSAP (GreenSock Animation Platform). GSAP is the industry-standard JavaScript animation library known for exceptional performance, reliability, and ease of use.

## Understanding GSAP

GSAP excels at animating CSS properties, SVG, canvas, and generic objects with precise timing and control. It's designed for:
- **Performance**: Hardware-accelerated transforms, minimal reflows
- **Reliability**: Cross-browser consistency, backwards compatibility
- **Power**: Advanced sequencing, easing, and control
- **Professional workflows**: Timeline-based animation orchestration

## Core CDN Setup

Always load GSAP from CDN in this specific order:

```html
<!-- Core GSAP (required) -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>

<!-- ScrollTrigger plugin (for scroll-based animations) -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>

<!-- Other plugins as needed -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/Draggable.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/MotionPathPlugin.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/TextPlugin.min.js"></script>
```

**Note**: Some advanced plugins (SplitText, MorphSVG, DrawSVG) require a GreenSock membership. Stick to freely available plugins unless the user confirms they have access.

## Basic Animation Methods

### gsap.to() - Animate TO values
```javascript
// Animate elements TO these values
gsap.to(".box", {
  x: 100,              // Move right 100px
  y: 50,               // Move down 50px
  rotation: 360,       // Rotate full circle
  opacity: 0.5,        // Fade to 50%
  duration: 1,         // 1 second
  ease: "power2.out"   // Easing function
});
```

### gsap.from() - Animate FROM values
```javascript
// Animate FROM these values to current state
gsap.from(".box", {
  opacity: 0,
  y: -50,
  duration: 1,
  ease: "back.out(1.7)"
});
```

### gsap.fromTo() - Explicit start and end
```javascript
// Define both starting and ending values
gsap.fromTo(".box", 
  { x: -100, opacity: 0 },  // From
  { x: 0, opacity: 1, duration: 1 }  // To
);
```

### gsap.set() - Instant property setting
```javascript
// Set properties immediately (no animation)
gsap.set(".box", { x: 100, opacity: 0 });
```

## Critical Properties & Best Practices

### Transform Properties (GPU-Accelerated)
```javascript
{
  x: 100,              // translateX (pixels or relative)
  y: 100,              // translateY
  xPercent: 50,        // translateX in %
  yPercent: 50,        // translateY in %
  rotation: 360,       // rotate (degrees)
  rotationX: 180,      // rotateX (3D)
  rotationY: 180,      // rotateY (3D)
  scale: 1.5,          // scale uniformly
  scaleX: 2,           // scale width only
  scaleY: 0.5,         // scale height only
  skewX: 10,           // skew horizontally
  skewY: 10,           // skew vertically
  transformOrigin: "center center"  // rotation/scale origin
}
```

### Other Animatable Properties
```javascript
{
  opacity: 0.5,        // transparency
  backgroundColor: "#ff0000",  // colors
  color: "#ffffff",
  width: "500px",      // CSS properties (use strings for units)
  height: "300px",
  borderRadius: "50%",
  filter: "blur(10px)",
  clipPath: "circle(50%)"
}
```

### Essential Animation Parameters
```javascript
{
  duration: 1,         // Animation duration in seconds (default: 0.5)
  delay: 0.5,          // Delay before starting (default: 0)
  ease: "power2.out",  // Easing function (default: "power1.out")
  stagger: 0.1,        // Delay between animating multiple elements
  repeat: 2,           // Number of times to repeat (-1 for infinite)
  repeatDelay: 1,      // Delay between repeats
  yoyo: true,          // Reverse on alternate repeats
  paused: false,       // Start paused?
  immediateRender: false,  // Render immediately for .from()?
  overwrite: "auto"    // How to handle conflicting animations
}
```

## Easing Functions

GSAP includes powerful easing options. Choose based on the desired feel:

```javascript
// Power eases (most common)
"power1.out"    // Subtle deceleration
"power2.out"    // Moderate deceleration (default feel)
"power3.out"    // Strong deceleration
"power4.out"    // Very strong deceleration

// Specialized eases
"back.out(1.7)" // Overshoots then settles (playful)
"elastic.out"   // Bouncy, spring-like
"bounce.out"    // Bounces at the end
"circ.out"      // Circular motion feel
"expo.out"      // Exponential deceleration (dramatic)

// Ease directions
".out"          // Fast start, slow end (most common)
".in"           // Slow start, fast end
".inOut"        // Slow start and end, fast middle

// Linear
"none"          // No easing, constant speed

// Examples of usage
ease: "power2.out"      // Smooth, natural
ease: "back.out(1.7)"   // Playful overshoot
ease: "elastic.out(1, 0.3)"  // Spring effect
```

**Rule of thumb**: Use `power2.out` as default. Use `back` or `elastic` for playful UI. Use `power3/4` for dramatic reveals.

## Timelines - Professional Animation Sequencing

Timelines orchestrate complex animation sequences:

```javascript
// Create a timeline
const tl = gsap.timeline({
  defaults: { duration: 1, ease: "power2.out" },  // Apply to all children
  paused: false,      // Start immediately
  repeat: 0,          // Number of repeats
  yoyo: false,        // Reverse on alternate repeats
  onComplete: () => console.log("Timeline complete")
});

// Add animations to the timeline
tl.to(".hero-title", { opacity: 1, y: 0 })          // Starts at 0s
  .to(".hero-subtitle", { opacity: 1, y: 0 })       // Starts after previous ends
  .to(".hero-cta", { opacity: 1, scale: 1 }, "-=0.3")  // Starts 0.3s before previous ends
  .to(".features", { opacity: 1, stagger: 0.2 })    // Stagger multiple elements
  .to(".background", { scale: 1.1 }, 0);             // Starts at absolute time 0s
```

### Timeline Position Control

```javascript
// Position parameters (4th argument or in object)
tl.to(el, { x: 100 }, "+=1")    // 1 second after previous ends
tl.to(el, { x: 100 }, "-=0.5")  // 0.5 seconds before previous ends
tl.to(el, { x: 100 }, "<")      // Start of previous animation
tl.to(el, { x: 100 }, ">")      // End of previous animation
tl.to(el, { x: 100 }, "myLabel")  // At a named label
tl.to(el, { x: 100 }, 2.5)      // At absolute time 2.5 seconds

// Add labels for complex sequences
tl.add("sceneOne")
  .to(".element1", { x: 100 })
  .to(".element2", { y: 100 })
  .add("sceneTwo")
  .to(".element3", { rotation: 360 }, "sceneTwo+=0.5");
```

### Timeline Control Methods

```javascript
tl.play();        // Play timeline
tl.pause();       // Pause timeline
tl.reverse();     // Reverse playback
tl.restart();     // Restart from beginning
tl.seek(1.5);     // Jump to 1.5 seconds
tl.progress(0.5); // Jump to 50% progress
tl.timeScale(2);  // Play at 2x speed
tl.kill();        // Destroy timeline and stop all animations
```

## ScrollTrigger - Scroll-Based Animations

ScrollTrigger is GSAP's powerful scroll animation plugin:

```javascript
// Register the plugin first
gsap.registerPlugin(ScrollTrigger);

// Basic scroll-triggered animation
gsap.to(".box", {
  x: 500,
  scrollTrigger: {
    trigger: ".box",       // Element that triggers
    start: "top center",   // When top of trigger hits center of viewport
    end: "bottom center",  // When bottom of trigger hits center of viewport
    scrub: true,          // Smooth scrubbing (links to scroll position)
    markers: true,        // Show debug markers (remove in production)
    toggleActions: "play none none reverse"  // onEnter onLeave onEnterBack onLeaveBack
  }
});

// Timeline with ScrollTrigger
const tl = gsap.timeline({
  scrollTrigger: {
    trigger: ".container",
    start: "top top",
    end: "bottom bottom",
    scrub: 1,            // Smooth scrubbing with 1 second delay
    pin: true,           // Pin the trigger element during animation
    anticipatePin: 1     // Prevents jumps in some cases
  }
});

tl.to(".element1", { x: 100 })
  .to(".element2", { rotation: 360 })
  .to(".element3", { opacity: 0 });
```

### ScrollTrigger Start/End Positions

```javascript
start: "top top"       // When trigger top hits viewport top
start: "top center"    // When trigger top hits viewport center
start: "top bottom"    // When trigger top hits viewport bottom
start: "center center" // When trigger center hits viewport center
start: "top 80%"       // When trigger top hits 80% down viewport

// Using offsets
start: "top top+=100"  // 100px after top
start: "top top-=50"   // 50px before top

// Most common patterns
start: "top 80%"       // Trigger near bottom of screen
end: "bottom 20%"      // End near top of screen
```

### ScrollTrigger Properties

```javascript
scrollTrigger: {
  trigger: ".element",    // Element to watch for scroll position
  start: "top center",    // [trigger position] [viewport position]
  end: "bottom top",      // Same format as start
  scrub: true,           // Link to scroll position (boolean or number)
  pin: true,             // Pin element during animation
  pinSpacing: true,      // Add space for pinned element
  snap: 1 / 4,           // Snap to nearest 1/4 of progress
  toggleActions: "play none none reverse",  // Actions on enter/leave
  toggleClass: "active",  // Toggle CSS class
  markers: false,        // Debug markers (development only)
  anticipatePin: 1,      // Prevents visual jumps
  invalidateOnRefresh: true,  // Recalculate on refresh
  onEnter: () => {},     // Callback functions
  onLeave: () => {},
  onEnterBack: () => {},
  onLeaveBack: () => {},
  onUpdate: (self) => {} // Fires continuously during scroll
}
```

### ScrollTrigger Batch Processing

Animate multiple elements efficiently:

```javascript
ScrollTrigger.batch(".feature-card", {
  onEnter: batch => gsap.to(batch, {
    opacity: 1, 
    y: 0, 
    stagger: 0.15,
    ease: "power2.out"
  }),
  start: "top 80%",
  once: true  // Only trigger once
});
```

## Stagger - Animating Multiple Elements

```javascript
// Basic stagger
gsap.to(".item", {
  y: 0,
  opacity: 1,
  duration: 1,
  stagger: 0.2  // 0.2 second delay between each
});

// Advanced stagger configuration
gsap.to(".item", {
  y: 0,
  opacity: 1,
  duration: 1,
  stagger: {
    each: 0.1,        // Delay between each element
    from: "center",   // Start from center (also: "start", "end", "edges", "random", index)
    grid: "auto",     // Or [rows, cols] for grid layouts
    amount: 1,        // Total time for all staggers
    ease: "power2.inOut",  // Easing for the stagger distribution
    repeat: 2,        // Repeat the stagger sequence
    yoyo: true        // Reverse on alternate repeats
  }
});

// Stagger from specific element
gsap.to(".item", {
  scale: 1.2,
  stagger: {
    each: 0.1,
    from: 3  // Start from the 4th element (0-indexed)
  }
});
```

## Common Animation Patterns

### Page Load Animation
```javascript
// Hero section reveal
const heroTL = gsap.timeline({ defaults: { ease: "power3.out" } });

heroTL.from(".hero-bg", { scale: 1.2, duration: 1.5 })
  .from(".hero-title", { y: 100, opacity: 0, duration: 1 }, "-=1")
  .from(".hero-subtitle", { y: 50, opacity: 0, duration: 0.8 }, "-=0.7")
  .from(".hero-cta", { y: 30, opacity: 0, scale: 0.9, duration: 0.6 }, "-=0.5");
```

### Scroll Reveal Cards
```javascript
gsap.registerPlugin(ScrollTrigger);

gsap.utils.toArray(".card").forEach(card => {
  gsap.from(card, {
    y: 100,
    opacity: 0,
    duration: 1,
    ease: "power2.out",
    scrollTrigger: {
      trigger: card,
      start: "top 85%",
      toggleActions: "play none none reverse"
    }
  });
});
```

### Parallax Effect
```javascript
gsap.to(".parallax-bg", {
  y: (i, target) => -ScrollTrigger.maxScroll(window) * target.dataset.speed,
  ease: "none",
  scrollTrigger: {
    start: 0,
    end: "max",
    invalidateOnRefresh: true,
    scrub: 0
  }
});
```

### Horizontal Scroll Section
```javascript
const sections = gsap.utils.toArray(".panel");

gsap.to(sections, {
  xPercent: -100 * (sections.length - 1),
  ease: "none",
  scrollTrigger: {
    trigger: ".container",
    pin: true,
    scrub: 1,
    snap: 1 / (sections.length - 1),
    end: () => "+=" + document.querySelector(".container").offsetWidth
  }
});
```

### Infinite Loop Animation
```javascript
// Seamless infinite horizontal scroll
function horizontalLoop(items, config) {
  items = gsap.utils.toArray(items);
  config = config || {};
  let tl = gsap.timeline({
    repeat: config.repeat || -1,
    paused: config.paused,
    defaults: { ease: "none" },
    onReverseComplete: () => tl.totalTime(tl.rawTime() + tl.duration() * 100)
  });
  
  let length = items.length,
      startX = items[0].offsetLeft,
      times = [],
      widths = [],
      xPercents = [],
      curIndex = 0,
      pixelsPerSecond = (config.speed || 1) * 100;
      
  gsap.set(items, {
    xPercent: (i, el) => {
      let w = widths[i] = parseFloat(gsap.getProperty(el, "width", "px"));
      xPercents[i] = gsap.utils.wrap(0, 100, gsap.getProperty(el, "x", "px") / w * 100 + gsap.getProperty(el, "xPercent"));
      return xPercents[i];
    }
  });
  
  gsap.set(items, { x: 0 });
  
  totalWidth = items[length-1].offsetLeft + xPercents[length-1] / 100 * widths[length-1] - startX + items[length-1].offsetWidth * gsap.getProperty(items[length-1], "scaleX") + (parseFloat(config.paddingRight) || 0);
  
  for (let i = 0; i < length; i++) {
    let item = items[i],
        curX = xPercents[i] / 100 * widths[i],
        distanceToStart = item.offsetLeft + curX - startX,
        distanceToLoop = distanceToStart + widths[i] * gsap.getProperty(item, "scaleX");
    tl.to(item, { xPercent: (curX - distanceToLoop) / widths[i] * 100, duration: distanceToLoop / pixelsPerSecond }, 0)
      .fromTo(item, { xPercent: (curX - distanceToLoop + totalWidth) / widths[i] * 100 }, { xPercent: xPercents[i], duration: (curX - distanceToLoop + totalWidth - curX) / pixelsPerSecond, immediateRender: false }, distanceToLoop / pixelsPerSecond);
  }
  
  return tl;
}

// Usage
const loop = horizontalLoop(".marquee-item", { speed: 1, repeat: -1 });
```

### Hover Animations
```javascript
// Smooth hover effect
const buttons = document.querySelectorAll(".btn");

buttons.forEach(btn => {
  const hoverTL = gsap.timeline({ paused: true });
  
  hoverTL.to(btn, { scale: 1.1, duration: 0.3, ease: "power2.out" })
         .to(btn.querySelector(".icon"), { x: 5, duration: 0.3 }, 0);
  
  btn.addEventListener("mouseenter", () => hoverTL.play());
  btn.addEventListener("mouseleave", () => hoverTL.reverse());
});
```

### Text Reveal Animation
```javascript
// Split text by characters (requires SplitText or manual split)
const text = "Hello World";
const chars = text.split('').map(char => `<span class="char">${char}</span>`).join('');
document.querySelector('.text').innerHTML = chars;

// Animate characters
gsap.from(".char", {
  opacity: 0,
  y: 50,
  rotationX: -90,
  duration: 0.8,
  ease: "back.out(1.7)",
  stagger: 0.03
});
```

## React Integration

### Using useGSAP Hook (Recommended)

```javascript
import { useGSAP } from '@gsap/react';
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

function Component() {
  const container = useRef();
  
  useGSAP(() => {
    // Animations automatically cleaned up on unmount
    gsap.from(".box", { 
      opacity: 0,
      y: 100,
      duration: 1 
    });
    
    gsap.to(".scroll-box", {
      x: 500,
      scrollTrigger: {
        trigger: ".scroll-box",
        start: "top center",
        scrub: true
      }
    });
  }, { scope: container }); // Scope to container
  
  return (
    <div ref={container}>
      <div className="box">Content</div>
      <div className="scroll-box">Scroll me</div>
    </div>
  );
}
```

### useEffect Alternative

```javascript
import { useEffect, useRef } from 'react';
import gsap from 'gsap';

function Component() {
  const boxRef = useRef();
  
  useEffect(() => {
    const ctx = gsap.context(() => {
      gsap.from(boxRef.current, {
        opacity: 0,
        y: 50,
        duration: 1,
        ease: "power2.out"
      });
    });
    
    // Cleanup
    return () => ctx.revert();
  }, []);
  
  return <div ref={boxRef}>Animated Box</div>;
}
```

## Performance Best Practices

### DO's:
- **Use transforms and opacity** - GPU accelerated (x, y, rotation, scale, opacity)
- **Batch similar animations** - Use timelines or single tweens with multiple targets
- **Use will-change sparingly** - `gsap.set(".el", { willChange: "transform" })` before heavy animations
- **Kill unused animations** - `tween.kill()` or `timeline.kill()` when done
- **Use scrub with ScrollTrigger** - Smoother than onUpdate callbacks
- **Set overwrite: "auto"** - Let GSAP handle conflicting animations

### DON'T's:
- **Avoid animating layout properties** - width, height, top, left cause reflows
- **Don't create animations in loops** - Cache and reuse when possible
- **Avoid heavy calculations in onUpdate** - Keep callbacks lightweight
- **Don't use too many simultaneous ScrollTriggers** - Batch when possible
- **Avoid animating large images directly** - Use transform: scale instead of width/height

### Optimization Example:
```javascript
// BAD - Animates layout properties, causes reflows
gsap.to(".box", { width: "500px", height: "300px" });

// GOOD - Uses transforms, GPU accelerated
gsap.to(".box", { scaleX: 2, scaleY: 1.5 });

// BAD - Creates many individual animations
document.querySelectorAll(".item").forEach(item => {
  gsap.to(item, { opacity: 1, y: 0, duration: 1 });
});

// GOOD - Single animation with stagger
gsap.to(".item", { opacity: 1, y: 0, duration: 1, stagger: 0.1 });
```

## Common Gotchas & Solutions

### Issue: Animation doesn't start
```javascript
// Problem: Element not rendered yet
gsap.to(".box", { x: 100 });

// Solution: Use DOMContentLoaded or ensure element exists
document.addEventListener("DOMContentLoaded", () => {
  gsap.to(".box", { x: 100 });
});

// Or in React, use useEffect/useGSAP
```

### Issue: ScrollTrigger not working on page load
```javascript
// Problem: Heights not calculated correctly
// Solution: Refresh after images load
window.addEventListener("load", () => {
  ScrollTrigger.refresh();
});
```

### Issue: Animation jumps on responsive resize
```javascript
// Solution: Refresh ScrollTrigger on resize
ScrollTrigger.addEventListener("refresh", () => {
  // Recalculate positions
});

// Or use matchMedia for responsive animations
gsap.matchMedia().add("(min-width: 768px)", () => {
  // Desktop animations
  gsap.to(".box", { x: 500 });
});
```

### Issue: Timeline repeating unexpectedly
```javascript
// Problem: Repeat on parent timeline affects children
const tl = gsap.timeline({ repeat: -1 }); // Repeats everything

// Solution: Control repeat at child level or use nested timelines
const tl = gsap.timeline();
tl.to(".box", { x: 100, repeat: -1 }); // Only this repeats
```

## Animation Philosophy

### Principles for Professional Motion:
1. **Purpose Over Decoration** - Every animation should serve a purpose (guide attention, provide feedback, reveal content)
2. **Consistency** - Use similar durations and easings for similar interactions
3. **Subtlety** - Less is often more; avoid over-animating
4. **Performance** - Target 60fps, use GPU-accelerated properties
5. **Accessibility** - Respect `prefers-reduced-motion` media query

```javascript
// Respecting motion preferences
const prefersReducedMotion = window.matchMedia("(prefers-reduced-motion: reduce)").matches;

if (!prefersReducedMotion) {
  gsap.to(".element", { 
    y: 100, 
    duration: 1,
    ease: "power2.out"
  });
} else {
  // Instant or simplified animation
  gsap.set(".element", { y: 100 });
}
```

### Timing Guidelines:
- **Micro-interactions**: 0.1-0.3s (hovers, clicks)
- **UI transitions**: 0.3-0.6s (modals, menus)
- **Content reveals**: 0.6-1.2s (page load, scroll reveals)
- **Cinematic effects**: 1.2s+ (hero sections, showcases)

## Complete Example: Landing Page Animation

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>GSAP Landing Page</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: system-ui, sans-serif; overflow-x: hidden; }
    
    .hero { 
      height: 100vh; 
      display: flex; 
      align-items: center; 
      justify-content: center;
      position: relative;
      overflow: hidden;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    }
    
    .hero-bg { 
      position: absolute; 
      width: 100%; 
      height: 100%; 
      background: url('pattern.svg');
      opacity: 0.1;
    }
    
    .hero-content { 
      text-align: center; 
      color: white; 
      z-index: 1;
      max-width: 800px;
      padding: 2rem;
    }
    
    .hero-title { 
      font-size: 4rem; 
      font-weight: 700; 
      margin-bottom: 1rem;
      line-height: 1.1;
    }
    
    .hero-subtitle { 
      font-size: 1.5rem; 
      margin-bottom: 2rem;
      opacity: 0.9;
    }
    
    .hero-cta { 
      display: inline-block;
      padding: 1rem 2.5rem;
      background: white;
      color: #667eea;
      text-decoration: none;
      border-radius: 50px;
      font-weight: 600;
      font-size: 1.1rem;
      transition: transform 0.2s;
    }
    
    .section { 
      min-height: 100vh; 
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 4rem 2rem;
    }
    
    .cards { 
      display: grid; 
      grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
      gap: 2rem;
      max-width: 1200px;
      width: 100%;
    }
    
    .card { 
      background: white;
      padding: 2rem;
      border-radius: 1rem;
      box-shadow: 0 10px 30px rgba(0,0,0,0.1);
    }
    
    .card h3 { 
      font-size: 1.5rem; 
      margin-bottom: 1rem;
      color: #667eea;
    }
  </style>
</head>
<body>
  <section class="hero">
    <div class="hero-bg"></div>
    <div class="hero-content">
      <h1 class="hero-title">Welcome to the Future</h1>
      <p class="hero-subtitle">Experience the power of smooth animations</p>
      <a href="#features" class="hero-cta">Explore Features</a>
    </div>
  </section>
  
  <section class="section" id="features">
    <div class="cards">
      <div class="card">
        <h3>Feature One</h3>
        <p>Amazing functionality that will blow your mind.</p>
      </div>
      <div class="card">
        <h3>Feature Two</h3>
        <p>Incredible performance that exceeds expectations.</p>
      </div>
      <div class="card">
        <h3>Feature Three</h3>
        <p>Beautiful design that captivates users.</p>
      </div>
    </div>
  </section>

  <!-- GSAP Core + ScrollTrigger -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>
  
  <script>
    gsap.registerPlugin(ScrollTrigger);
    
    // Hero animation on page load
    const heroTL = gsap.timeline({ 
      defaults: { ease: "power3.out" } 
    });
    
    heroTL.from(".hero-bg", { 
        scale: 1.3, 
        duration: 2,
        ease: "power2.out"
      })
      .from(".hero-title", { 
        y: 100, 
        opacity: 0, 
        duration: 1 
      }, "-=1.5")
      .from(".hero-subtitle", { 
        y: 50, 
        opacity: 0, 
        duration: 0.8 
      }, "-=0.8")
      .from(".hero-cta", { 
        y: 30, 
        opacity: 0, 
        scale: 0.8, 
        duration: 0.6 
      }, "-=0.6");
    
    // Cards scroll animation
    gsap.from(".card", {
      y: 100,
      opacity: 0,
      duration: 1,
      ease: "power2.out",
      stagger: 0.2,
      scrollTrigger: {
        trigger: ".cards",
        start: "top 80%",
        toggleActions: "play none none reverse"
      }
    });
    
    // Smooth button hover
    const buttons = document.querySelectorAll(".hero-cta");
    buttons.forEach(btn => {
      btn.addEventListener("mouseenter", () => {
        gsap.to(btn, { scale: 1.05, duration: 0.3 });
      });
      btn.addEventListener("mouseleave", () => {
        gsap.to(btn, { scale: 1, duration: 0.3 });
      });
    });
    
    // Refresh ScrollTrigger after all content loads
    window.addEventListener("load", () => {
      ScrollTrigger.refresh();
    });
  </script>
</body>
</html>
```

## Key Reminders

1. **Always load GSAP from CDN** - Use the official GSAP CDN links
2. **Register plugins** - `gsap.registerPlugin(ScrollTrigger)` before use
3. **Use timelines for sequences** - Better control and organization
4. **Prefer transforms and opacity** - GPU accelerated properties
5. **Stagger for multiple elements** - Creates natural, flowing animations
6. **Test ScrollTrigger thoroughly** - Use markers during development
7. **Clean up in React** - Use useGSAP hook or gsap.context()
8. **Respect reduced motion preferences** - Always check prefers-reduced-motion
9. **Start simple, add complexity** - Build animations incrementally
10. **Profile performance** - Use browser dev tools to ensure 60fps

GSAP is extraordinarily powerful. These guidelines help create professional, performant animations that enhance user experience without sacrificing performance or accessibility.
