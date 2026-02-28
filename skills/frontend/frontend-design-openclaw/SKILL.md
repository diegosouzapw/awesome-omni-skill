---
name: Frontend Design
slug: frontend
version: "1.0.2"
homepage: https://clawic.com/skills/frontend
description: Frontend development with React, Next.js, Tailwind CSS. Build landing pages, dashboards, forms, components. Responsive, accessible, performant UI.
changelog: "Renamed to better reflect design-focused capabilities and guidance."
metadata: {"clawdbot":{"emoji":"🖥️","requires":{"bins":[]},"os":["linux","darwin","win32"]}}
---

## When to Use

User needs web UI built. Agent handles landing pages, dashboards, forms, component libraries, and any frontend requiring production polish.

## Quick Reference

| Topic | File |
|-------|------|
| Stack & tooling | `stack.md` |
| Typography rules | `typography.md` |
| Color systems | `colors.md` |
| Mobile patterns | `mobile.md` |
| Animation | `animation.md` |
| Examples | `examples.md` |

## Core Rules

### 1. Mobile-First Always
- Start with mobile layout, enhance upward
- Every grid must collapse to single column
- Touch targets minimum 44x44px
- Test on real devices, not just simulators

### 2. Typography Matters
- Avoid generic fonts (Inter, Roboto, Arial)
- Use dramatic size jumps (2x+), not timid increments
- Body text 16-18px minimum
- See `typography.md` for specific recommendations

### 3. Color with Purpose
- 70-20-10 rule: primary, secondary, accent
- Commit to light OR dark — no muddy mid-grays
- Never solid white backgrounds — add depth
- See `colors.md` for CSS variables and patterns

### 4. Feedback on Every Interaction
- Acknowledge taps within 100ms
- Optimistic updates for instant feel
- Loading states for operations >1s
- Preserve user input on errors

### 5. Accessibility Non-Negotiable
- Color contrast 4.5:1 (text), 3:1 (UI)
- Focus states on all interactive elements
- Semantic HTML (nav, main, section, article)
- Keyboard navigation works for everything

### 6. Performance from Start
- Lazy load below-fold content
- Image placeholders prevent layout shift
- Code split heavy components
- Target LCP <2.5s, CLS <0.1

### 7. One Memorable Element
- Every page needs one unforgettable design choice
- Typography treatment, hero animation, unusual layout
- Timid designs fail — commit to an aesthetic

## Frontend Traps

| Trap | Consequence | Fix |
|------|-------------|-----|
| Generic fonts | Looks like every other site | Use distinctive fonts |
| Solid white backgrounds | Flat, lifeless | Add gradients, grain, depth |
| Mobile as afterthought | Broken for 60% of users | Mobile-first always |
| Form error clears input | User rage | Preserve input, highlight error |
| No loading states | User thinks broken | Show progress immediately |
| Timid type scale | No visual hierarchy | Use 2x+ jumps for headlines |

## Scope

This skill ONLY:
- Provides frontend patterns and guidelines
- Recommends stack and tooling choices
- Guides responsive implementation

This skill NEVER:
- Makes network requests
- Accesses user data
- Stores any information

## Security & Privacy

This skill is read-only guidance. No data is collected, sent, or stored.

## Feedback

- If useful: `clawhub star frontend`
- Stay updated: `clawhub sync`
