---
name: motion-animation
description: Motion (Framer Motion) animation library integration for Remotion - smooth, production-grade UI animations
metadata:
  tags: motion, framer-motion, animation, react-animation, transitions, gestures, spring-physics
  category: animation
  version: 1.0.0
---

# Motion Animation for Remotion

Integrate Motion (formerly Framer Motion) animation library with Remotion for smooth, declarative, production-grade animations in your video projects.

## When to use

Use this skill when you need to:

      - Create smooth prop-based animations in Remotion compositions
      - Build complex animation sequences with variants and orchestration
      - Leverage spring physics for natural, organic motion
      - Implement gesture-based animations (useful for preview/development)
      - Animate SVG paths and shapes with precision
      - Use Motion's declarative API alongside Remotion's frame-based timeline
      - Create reusable animation patterns and presets

      ## When NOT to use

      - For precise frame-by-frame control (use Remotion's `useCurrentFrame()` and `interpolate()` instead)
      - For simple fade in/out animations (use Remotion's native `interpolate()` for better performance)
      - When you need exact synchronization with Remotion's timeline (Motion runs in real-time)

      ## How to use

      Read individual rule files for detailed explanations, code examples, and best practices:

      ### Core Concepts

      - **rules/motion-basics.md** - Installing, configuring, and basic usage of Motion in Remotion
      - **rules/motion-with-remotion.md** - Synchronizing Motion animations with Remotion's timeline

      ### Animation Techniques

      - **rules/motion-animate.md** - Using Motion's animate prop, variants system, and animation control
      - **rules/motion-spring.md** - Spring animations, physics-based timing, and natural motion
      - **rules/motion-transitions.md** - Transition effects, easing functions, and animation orchestration
      - **rules/motion-timing.md** - Timing control, delays, duration, and stagger effects

      ### Advanced Features

      - **rules/motion-layout.md** - Layout animations, AnimatePresence, and shared layout transitions
      - **rules/motion-gestures.md** - Hover, tap, drag, and other gesture-based animations
      - **rules/motion-svg.md** - SVG path animations, morphing, and drawing effects
      - **rules/motion-performance.md** - Optimization techniques and performance best practices

      ## Installation

      ```bash
      npm install framer-motion
      # or
      yarn add framer-motion
      # or
      pnpm add framer-motion
      ```

      ## Quick Example

      ```tsx
      import { AbsoluteFill, useCurrentFrame, useVideoConfig } from "remotion";
      import { motion } from "framer-motion";

      export const MotionExample: React.FC = () => {
        const frame = useCurrentFrame();
          const { fps } = useVideoConfig();

              // Sync Motion with Remotion's timeline
                const progress = Math.min(frame / fps, 1);

                    return (
                        <AbsoluteFill style={{ backgroundColor: "white" }}>
                              <motion.div
                                      animate={{
                                                scale: progress,
                                                          opacity: progress,
                                                                  }}
                                                                          transition={{
                                                                                    type: "spring",
                                                                                              stiffness: 100,
                                                                                                        damping: 20,
                                                                                                                }}
                                                                                                                        style={{
                                                                                                                                  width: 200,
                                                                                                                                            height: 200,
                                                                                                                                                      backgroundColor: "#0099ff",
                                                                                                                                                                borderRadius: 20,
                                                                                                                                                                        }}
                                                                                                                                                                              />
                                                                                                                                                                                  </AbsoluteFill>
                                                                                                                                                                                    );
                                                                                                                                                                                    };
                                                                                                                                                                                    ```
                                                                                                                                                                                    
                                                                                                                                                                                    ## Best Practices
                                                                                                                                                                                    
                                                                                                                                                                                    1. **Sync with Remotion**: Always sync Motion animations with `useCurrentFrame()` for predictable video rendering
                                                                                                                                                                                    2. **Use variants**: Leverage Motion's variant system for reusable, organized animation states
                                                                                                                                                                                    3. **Optimize performance**: Use `will-change` CSS property sparingly and prefer GPU-accelerated properties
                                                                                                                                                                                    4. **Test renders**: Always test your animations in actual Remotion renders, not just in preview
                                                                                                                                                                                    5. **Combine wisely**: Use Motion for complex declarative animations, Remotion for precise timeline control
                                                                                                                                                                                    
                                                                                                                                                                                    ## Resources
                                                                                                                                                                                    
                                                                                                                                                                                    - [Motion Documentation](https://motion.dev)
                                                                                                                                                                                    - [Remotion Documentation](https://remotion.dev)
                                                                                                                                                                                    - [Motion Examples](https://motion.dev/examples)
                                                                                                                                                                                    - [Integration Guide](./docs/integration-guide.md)
