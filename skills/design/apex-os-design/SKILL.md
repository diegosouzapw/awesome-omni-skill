---
name: apex-os-design
description: Generates premium dark-mode UI for Apex OS wellness app (React Native + Expo 54). Use when designing screens, creating components, making layout decisions, generating data visualizations, or writing frontend code. Covers color system, typography, motion, haptics, and component patterns for a Bloomberg-meets-Calm aesthetic. References APEX_OS_PRD_v8.1.md for product logic and APEX_OS_BRAND_GUIDE.md for voice.
---

# Apex OS Design System

## Overview

This skill guides creation of premium, production-grade UI for **Apex OS**: a dark, data-forward, evidence-based wellness operating system for health-conscious professionals.

**Tech stack:** React Native + Expo 54, TypeScript, Reanimated for motion

**Related files:**
- Product behavior: `APEX_OS_PRD_v8.1.md`
- Brand voice: `APEX_OS_BRAND_GUIDE.md`
- Design brief: `APEX_OS_design_brief.md`

## Design Intent

### One-Sentence Brief

Build an interface a busy founder would trust with their health data—premium, dark, data-dense, scientifically credible, and respectful of their time.

### Aesthetic DNA

| Source | Weight | Contribution |
|--------|--------|--------------|
| Bloomberg Terminal | 40% | Data density, professional authority, assumes user intelligence |
| Oura Ring app | 30% | Warm dark mode, "one big thing" focus, color as body-state |
| Linear | 15% | Precision, speed, no wasted pixels |
| Calm | 10% | Breathing space, intentional simplicity |

### User Test

Before finalizing any UI, ask:

> "Would a traveling founder, checking this at 6am in an airport lounge, understand what matters and what to do in under 10 seconds?"

If not, simplify hierarchy, reduce noise, sharpen the primary action.

## Quick Reference

### Core Colors

```
Canvas:       #0F1218   (app background)
Surface:      #181C25   (cards, modals)
Elevated:     #1F2430   (inputs, pressed states)
Subtle:       #2A303D   (borders, dividers)

Teal:         #63E6BE   (primary CTAs, active states)
Blue:         #5B8DEF   (secondary actions, links)
Gold:         #D4A574   (Pro tier, achievements)

Text Primary:   #F6F8FC
Text Secondary: #A7B4C7
Text Muted:     #6C7688

Recovery High:     #4ADE80  (75-100%)
Recovery Moderate: #FBBF24  (60-74%)
Recovery Low:      #F87171  (<60%)
```

For full color system with usage rules, see [colors.md](colors.md).

### Typography Quick Start

```typescript
// Font families
const fonts = {
  ui: 'Inter',           // Headlines, UI text
  data: Platform.select({
    ios: 'SF Mono',
    android: 'JetBrains Mono',
    default: 'monospace'
  })
};

// Key sizes
const type = {
  metric: 48,    // Hero metrics (recovery score)
  h1: 28,        // Screen titles
  h2: 22,        // Section headers
  body: 16,      // Main content
  caption: 12,   // Labels, timestamps
};
```

For full type scale and rules, see [typography.md](typography.md).

### Component Essentials

**Cards:** `surface` background, 1px `subtle` border, 12px radius, 20px padding, subtle elevation

**Elevation:** Cards float with `0 2px 8px rgba(0,0,0,0.12)`. Hero cards (Recovery Score) use `hero` elevation.

**Primary Button:** `teal` background, dark text, 48px min height, 12px radius

**Touch targets:** 44×44pt minimum, 48×48pt preferred

**Spacing:** 8pt grid. Card padding: 20px (was 16). Section gaps: 28px (was 24). Hero sections: 40-56px.

For all component patterns including elevation system, see [components.md](components.md).

## How to Use This Skill

### When Invoked

1. Read this file for visual/interaction constraints
2. Reference PRD for product logic and user flows
3. Reference Brand Guide for copy tone
4. Generate code that follows patterns in referenced files

### Prompting Patterns

**Screen design:**
> "Design the Home (Morning Anchor) screen. Intent: traveling founder should understand recovery + first protocol in 10 seconds. Use Recovery Score Card hero, Today's Protocols list, and brand voice from APEX_OS_BRAND_GUIDE.md."

**Component design:**
> "Create a Protocol Card component following components.md patterns. Show icon, title, adherence dots (5/7), and chevron affordance."

**Data visualization:**
> "Build an HRV trend chart following data-viz.md. 7-day default, teal line with gradient fill, tap for tooltip."

### Degrees of Freedom

| Area | Freedom | Notes |
|------|---------|-------|
| Color palette | Low | Use defined colors only |
| Typography | Low | Inter + monospace for data |
| Spacing | Medium | 8pt grid, adapt to content |
| Layout | Medium | Follow archetypes, adapt to context |
| Animation | Medium | Follow timing guidance, creative within bounds |
| Copy tone | Low | Match "warm expertise" voice exactly |

## Screen Archetypes

### Home (Morning Anchor)

**Goal:** User knows recovery state + first action in 10 seconds.

**Structure:**
1. Header: greeting + date + profile icon
2. Recovery Score Card (hero)
3. "One Big Thing" (AI-curated focus for today)
4. Today's Protocols list (3-5 items)
5. Chat entry point

See [screens.md](screens.md) for full specification.

### Protocol Detail

**Goal:** Understand what to do, then optionally explore why.

**Structure:**
1. Title + icon + category badge
2. Hero visual (geometric line art icon, teal accent, 120×120pt)
3. "The Protocol" (specific steps)
4. Collapsible: Why / Your Data / Science
5. Sticky CTA: "Start Protocol"

**Hero Icon Style:** Geometric line art, 2px stroke, teal (#63E6BE), subtle scale animation on load.

See [screens.md](screens.md) for full specification including icon examples.

### AI Chat

**Goal:** Personalized, data-informed guidance.

**Structure:**
- AI messages: left, surface cards
- User messages: right, accent background
- Thinking state: subtle pulse
- Sticky input bar

See [screens.md](screens.md) for full specification.

### Additional Screens

For MVD (Minimum Viable Day), Weekly Synthesis, Wearable Connection, and Onboarding archetypes, see [screens.md](screens.md).

## Animation & Haptics

**Core principle:** Every animation must confirm action, guide attention, or provide continuity.

**Timing guidance:**
| Interaction | Duration |
|-------------|----------|
| Button press | ~100ms |
| Screen transition | ~250ms |
| Modal appear | ~250ms |
| Metric count-up | ~300-500ms |

**Micro-Delight Moments:**
- Protocol completion: checkmark spring + teal glow pulse + particle burst (3-5 dots)
- Loading: pulsing logo (scale 0.95→1.05, 1.2s), not generic spinner
- Empty states: breathing animation on illustration (scale 0.98→1.02, 3s)
- Recovery score reveal: count-up + ring fill + zone-color glow

For complete motion system, haptic mapping, and micro-delight implementations, see [motion-haptics.md](motion-haptics.md).

## Data Visualization

**Core rules:**
- User understands the "story" in ~5 seconds
- Always label axes, units, time windows
- Teal for primary data, zone colors for recovery states
- Tap → tooltip with exact value

For chart patterns (line trends, progress bars, adherence dots, correlation cards), see [data-viz.md](data-viz.md).

## Accessibility

**Non-negotiable requirements:**
- WCAG AA contrast (≥4.5:1 for text)
- 44×44pt minimum touch targets
- Color never the only signal
- Respect `prefers-reduced-motion`
- Accessible labels on all interactive elements

For complete accessibility checklist, see [accessibility.md](accessibility.md).

## Logo Usage

### Assets Location

Logo files live in `/client/assets/logo/` (NOT in the skill folder).

### Variants

| Variant | File | Use Case |
|---------|------|----------|
| **Chevron only** | `chevron-only.png` | App icon, loading animation, favicon, compact spaces |
| **Vertical** | `vertical_-_Logo-Brand_Name.png` | Splash screen, onboarding, centered layouts |
| **Horizontal** | `horizontal_-_Logo-Brand_Name.png` | Navigation header, login screen, in-app branding |
| **Horizontal + tagline** | `horizontal_-_Logo-Brand_Name-Tagline.png` | Landing page, external marketing only |

### Tagline

**Canonical tagline:** "Your Performance Operating System"

Use tagline version for marketing/external only. Inside the app, use the no-tagline variants.

### Logo Colors

The chevron uses a gradient (do not flatten to solid):
```
Top:    #63E6BE (teal)
Bottom: #5B8DEF (blue)
```

The wordmark:
- "APEX": White (#F6F8FC), bold weight
- "OS": Light gray (#A7B4C7), light weight

**Important:** These logos have white/light wordmarks designed for dark backgrounds. Always place on dark surfaces (#0F1218 canvas or #181C25 surface).

### Sizing Guidelines

| Context | Recommended Size |
|---------|------------------|
| App icon | 1024×1024 source, system scales |
| Navigation header | Height: 32px |
| Splash screen | Width: 40% of screen, centered |
| Loading indicator | 48×48px (chevron only) |

### Clear Space

Minimum padding around logo = height of one chevron stroke.

### Usage Rules

**Do:**
- Place on dark backgrounds only (#0F1218, #181C25)
- Maintain aspect ratio
- Use chevron-only for loading animation (pulsing)

**Don't:**
- Place on light backgrounds (wordmark invisible)
- Stretch or distort
- Add shadows/glows to the logo itself
- Use tagline version inside the app
- Recreate — always use official assets

### Loading Animation

```typescript
import ChevronLogo from '@/assets/logo/chevron-only.png';

// Pulsing loader (see motion-haptics.md for full implementation)
<ApexLoadingIndicator 
  source={ChevronLogo}
  size={48}
/>
```

---

## UI Copy Guidelines

**Voice:** "Warm expertise"—knowledgeable friend who's also a health scientist.

**Do:**
- Lead with action
- Use specific numbers
- Keep praise factual: "HRV up 8% vs last week"

**Don't:**
- Generic wellness clichés
- Guilt-trip about missed days
- Over-celebrate ("You're crushing it!!!")

**Examples:**
| Instead of | Use |
|------------|-----|
| "Amazing job! You smashed your goals!" | "HRV up 10% vs last week. Protocol is working." |
| "Don't break your streak!" | "Morning Light: 5/7 this week." |

## File Reference

| File | Content |
|------|---------|
| [colors.md](colors.md) | Full color system, zone usage, overlay calculations |
| [typography.md](typography.md) | Type scale, font loading, line height rules |
| [components.md](components.md) | Cards, buttons, inputs, navigation patterns |
| [screens.md](screens.md) | All screen archetypes with structure and interaction |
| [data-viz.md](data-viz.md) | Charts, progress bars, correlation cards |
| [motion-haptics.md](motion-haptics.md) | Animation timing, easing, haptic feedback mapping |
| [accessibility.md](accessibility.md) | WCAG compliance, reduced motion, screen reader support |

## Anti-Patterns

**Never:**
- Use light/white backgrounds (breaks dark mode aesthetic)
- Use Inter for data metrics (use monospace)
- Use harsh pure red without amber warning first
- Exceed 350ms for any animation
- Skip haptic feedback on primary actions
- Use "streak" language or guilt-based copy
- Present more than 5 protocols at once without hierarchy
- Use flat cards without elevation (premium apps have depth)
- Use generic spinners (use pulsing logo or skeletons)
- Use emoji for protocol icons in hero positions (use geometric line art)
- Use 16px card padding (too tight—use 20px for premium feel)
