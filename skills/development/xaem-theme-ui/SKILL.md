---
name: xaem-theme-ui
description: Two-pass theme generation and code translation pipeline. Pass 1 generates high-entropy visual themes (colour palettes, glow intensities, animation timing, typography weights). Pass 2 translates themes into CSS variables, TypeScript design tokens, Tailwind config, and Framer Motion presets. Complements scientific-luxury by handling creative generation before constraint enforcement.
license: MIT
metadata:
  author: NodeJS-Starter-V1
  version: '1.0.0'
  locale: en-AU
---

# XAEM Theme UI - Theme Generation & Code Translation

Two-pass pipeline for generating high-entropy visual themes and translating them into implementable code. Works upstream of `scientific-luxury` constraint enforcement.

## Description

XAEM Theme UI adds a creative generation layer to the design system. While `scientific-luxury` enforces structural constraints (OLED black, 0.5px borders, `rounded-sm`, physics-based easings), it does not generate themes. This skill fills that gap with a two-pass pipeline:

1. **Pass 1 (Generate)**: Create high-entropy visual themes -- spectral colour palettes, glow intensities, animation timing scales, typography weight distributions, opacity hierarchies
2. **Pass 2 (Translate)**: Convert generated themes into implementable code -- CSS variables, TypeScript design tokens, Tailwind config extensions, Framer Motion preset overrides, component style maps

The structural skeleton remains immutable. Only the visual skin is customisable.

## When to Apply

### Positive Triggers

- Generating new colour palettes or theme variations
- Creating design tokens for a new feature area
- Building theme presets (dark variants, branded themes)
- User mentions: "theme", "palette", "colour scheme", "design tokens", "generate theme"
- Translating a mood board or colour brief into code
- Customising glow intensities, animation timing, or opacity scales

### Negative Triggers

- Enforcing existing design constraints (use `scientific-luxury` instead)
- Reviewing component layout compliance (use `scientific-luxury` instead)
- Optimising algorithm performance (use `council-of-logic` instead)
- Writing backend logic, API routes, or database schemas

## The Two Laws

### Law 1: Generate Then Translate

Every theme goes through both passes sequentially. Never ship a palette without its code translation. Never write design token code without a generation rationale.

### Law 2: Never Violate Structural Constraints

The following are **immutable** and cannot be overridden by any theme:

| Constraint       | Locked Value                               | Reason                          |
| ---------------- | ------------------------------------------ | ------------------------------- |
| Background       | `#050505` (OLED Black)                     | Foundation of visual hierarchy  |
| Border width     | `0.5px`                                    | Single-pixel precision          |
| Border radius    | `rounded-sm` (2px)                         | Sharp, professional appearance  |
| Animation engine | Framer Motion only                         | Physics-based motion            |
| Easing curves    | Physics-based (`outExpo`, `snappy`, etc.)  | No linear transitions           |
| Layout pattern   | Timeline / asymmetrical                    | No symmetrical card grids       |
| Status icons     | Breathing orbs                             | No Lucide/FontAwesome for status|

---

## Pass 1: Theme Generation

### 1.1 Colour Palette Generation

Every theme defines exactly **six spectral colours** mapped to semantic roles:

| Role      | Semantic Purpose                | Default Hex |
| --------- | ------------------------------- | ----------- |
| Primary   | Active, in-progress, CTA        | `#00F5FF`   |
| Success   | Completed, approved, positive   | `#00FF88`   |
| Warning   | Verification, awaiting, caution | `#FFB800`   |
| Danger    | Error, failed, rejected         | `#FF4444`   |
| Accent    | Escalation, human intervention  | `#FF00FF`   |
| Neutral   | Pending, inactive, disabled     | `#6B7280`   |

#### Palette Strategies

**Analogous** -- Colours within 30 degrees on the colour wheel. Produces harmonious, low-contrast themes suited for ambient dashboards.

```
Example: Primary #00F5FF → Success #00FFC8 → Warning #88FF00
```

**Complementary** -- Pairs colours from opposite sides of the wheel. High contrast, suited for alert-heavy interfaces.

```
Example: Primary #00F5FF → Accent #FF0A55
```

**Triadic** -- Three colours evenly spaced at 120 degrees. Balanced vibrancy for data-rich views.

```
Example: Primary #00F5FF → Warning #FF8800 → Accent #AA00FF
```

#### Palette Constraints

- All colours must have sufficient contrast against `#050505` (minimum 4.5:1 for text)
- No pastel or desaturated colours (saturation must be >= 70%)
- No pure white (`#FFFFFF`) as a spectral colour
- Each colour must be visually distinct at 50% opacity over OLED black

### 1.2 Glow Intensity Curves

Glow is the primary visual feedback mechanism. Three intensity tiers:

| Tier   | Inner Opacity | Outer Opacity | Spread (px) | Use Case                |
| ------ | ------------- | ------------- | ----------- | ----------------------- |
| Low    | 20%           | 10%           | 20          | Subtle ambient presence |
| Medium | 40%           | 20%           | 40          | Default interactive     |
| High   | 60%           | 30%           | 60          | Alert, active focus     |

Themes may adjust these values within these bounds:

- Inner opacity: 15%--65%
- Outer opacity: 8%--35%
- Spread: 12px--80px

### 1.3 Animation Timing Scale

Themes define a timing scale with six duration values:

| Token     | Default (s) | Range (s)   | Purpose                    |
| --------- | ----------- | ----------- | -------------------------- |
| `fast`    | 0.2         | 0.15--0.3   | Micro-interactions         |
| `normal`  | 0.4         | 0.3--0.6    | Standard transitions       |
| `slow`    | 0.6         | 0.5--0.9    | Deliberate, emphasised     |
| `breathe` | 2.0         | 1.5--3.0    | Breathing pulse loops      |
| `pulse`   | 1.5         | 1.0--2.5    | Glow pulse cycles          |
| `ambient` | 3.0         | 2.0--4.0    | Background ambient effects |

All durations must use physics-based easing. Linear timing is **banned**.

### 1.4 Typography Weight Distribution

Themes select from the permitted weight range:

| Token         | Default | Range     | Usage              |
| ------------- | ------- | --------- | ------------------ |
| `heroWeight`  | 200     | 100--300  | Hero titles        |
| `titleWeight` | 300     | 200--400  | Section headings   |
| `bodyWeight`  | 400     | 300--500  | Body text          |
| `dataWeight`  | 500     | 400--600  | Monospace data     |
| `labelWeight` | 400     | 300--500  | Uppercase labels   |

Font families are **locked**: JetBrains Mono (data), Inter/SF Pro (editorial).

### 1.5 Opacity Hierarchy Tuning

Themes define a six-step text opacity scale and a three-step border opacity scale:

```
Text:   primary(0.85-0.95) → secondary(0.65-0.75) → tertiary(0.45-0.55)
        → muted(0.35-0.45) → subtle(0.25-0.35) → ghost(0.15-0.25)

Border: visible(0.08-0.12) → subtle(0.04-0.08) → ghost(0.02-0.04)
```

---

## Pass 2: Code Translation

### 2.1 CSS Variable Output

```css
:root {
  /* Spectral Palette */
  --spectral-primary: #00F5FF;
  --spectral-success: #00FF88;
  --spectral-warning: #FFB800;
  --spectral-danger: #FF4444;
  --spectral-accent: #FF00FF;
  --spectral-neutral: #6B7280;

  /* Glow Intensities */
  --glow-low-inner: 20;
  --glow-low-outer: 10;
  --glow-med-inner: 40;
  --glow-med-outer: 20;
  --glow-high-inner: 60;
  --glow-high-outer: 30;

  /* Animation Timing */
  --duration-fast: 0.2s;
  --duration-normal: 0.4s;
  --duration-slow: 0.6s;
  --duration-breathe: 2s;
  --duration-pulse: 1.5s;
  --duration-ambient: 3s;

  /* Text Opacity Hierarchy */
  --text-primary: rgba(255, 255, 255, 0.9);
  --text-secondary: rgba(255, 255, 255, 0.7);
  --text-tertiary: rgba(255, 255, 255, 0.5);
  --text-muted: rgba(255, 255, 255, 0.4);
  --text-subtle: rgba(255, 255, 255, 0.3);
  --text-ghost: rgba(255, 255, 255, 0.2);

  /* Border Opacity Hierarchy */
  --border-visible: rgba(255, 255, 255, 0.1);
  --border-subtle: rgba(255, 255, 255, 0.06);
  --border-ghost: rgba(255, 255, 255, 0.03);
}
```

### 2.2 TypeScript Design Token File

```typescript
export const THEME = {
  spectral: {
    primary: '#00F5FF',
    success: '#00FF88',
    warning: '#FFB800',
    danger: '#FF4444',
    accent: '#FF00FF',
    neutral: '#6B7280',
  },
  glow: {
    low: { inner: '20', outer: '10', spread: 20 },
    medium: { inner: '40', outer: '20', spread: 40 },
    high: { inner: '60', outer: '30', spread: 60 },
  },
  durations: {
    fast: 0.2,
    normal: 0.4,
    slow: 0.6,
    breathe: 2.0,
    pulse: 1.5,
    ambient: 3.0,
  },
  weights: {
    hero: 200,
    title: 300,
    body: 400,
    data: 500,
    label: 400,
  },
} as const;
```

### 2.3 Tailwind Theme Extension

```typescript
// tailwind.config.ts (theme.extend block)
{
  colors: {
    spectral: {
      primary: 'var(--spectral-primary)',
      success: 'var(--spectral-success)',
      warning: 'var(--spectral-warning)',
      danger: 'var(--spectral-danger)',
      accent: 'var(--spectral-accent)',
      neutral: 'var(--spectral-neutral)',
    },
  },
  transitionDuration: {
    fast: 'var(--duration-fast)',
    normal: 'var(--duration-normal)',
    slow: 'var(--duration-slow)',
  },
}
```

### 2.4 Framer Motion Preset Overrides

```typescript
import { THEME } from './design-tokens';

export const themeMotion = {
  fadeInLeft: {
    initial: { opacity: 0, x: -20 },
    animate: { opacity: 1, x: 0 },
    transition: {
      duration: THEME.durations.normal,
      ease: [0.19, 1, 0.22, 1],
    },
  },
  breathing: {
    animate: { opacity: [1, 0.6, 1], scale: [1, 1.05, 1] },
    transition: {
      duration: THEME.durations.breathe,
      repeat: Infinity,
      ease: 'easeInOut',
    },
  },
  glowPulse: (colour: string) => ({
    animate: {
      boxShadow: [
        `0 0 0 ${colour}00`,
        `0 0 ${THEME.glow.medium.spread}px ${colour}${THEME.glow.medium.inner}`,
        `0 0 0 ${colour}00`,
      ],
    },
    transition: { duration: THEME.durations.pulse, repeat: Infinity },
  }),
};
```

### 2.5 Status Colour Mapping

```typescript
import { THEME } from './design-tokens';

export const STATUS_MAP = {
  pending: THEME.spectral.neutral,
  in_progress: THEME.spectral.primary,
  awaiting_verification: THEME.spectral.warning,
  verification_in_progress: THEME.spectral.warning,
  verification_passed: THEME.spectral.success,
  verification_failed: THEME.spectral.danger,
  completed: THEME.spectral.success,
  failed: THEME.spectral.danger,
  blocked: THEME.spectral.warning,
  escalated_to_human: THEME.spectral.accent,
} as const;
```

---

## Existing Design Tokens Reference

The current baseline lives at `apps/web/lib/design-tokens.ts`. Key exports:

- `SPECTRAL` -- Six-colour palette (cyan, emerald, amber, red, magenta, grey)
- `BACKGROUNDS` -- OLED black foundation with elevation steps
- `TEXT` -- Six-step text opacity hierarchy
- `BORDERS` -- Three-step border opacity hierarchy
- `EASINGS` -- Four physics-based cubic-bezier curves
- `DURATIONS` -- Six animation duration tokens
- `MOTION_PRESETS` -- Framer Motion animation presets
- `TW` -- Tailwind class helper constants

All generated themes must be compatible with this token structure.

---

## Theme Presets

### Scientific Luxury (Default)

The baseline theme. All values match `apps/web/lib/design-tokens.ts`.

```
Primary: #00F5FF (Cyan)    Success: #00FF88 (Emerald)
Warning: #FFB800 (Amber)   Danger:  #FF4444 (Red)
Accent:  #FF00FF (Magenta) Neutral: #6B7280 (Grey)
Glow: standard | Timing: standard | Weights: standard
```

### Midnight Aurora

Cool-shifted palette with extended glow spread for ambient dashboards.

```
Primary: #00D4FF (Ice Blue)   Success: #00E87A (Mint)
Warning: #FF9F00 (Deep Amber) Danger:  #FF3355 (Crimson)
Accent:  #CC44FF (Violet)     Neutral: #5A6270 (Slate)
Glow: high-spread (50px med)  Timing: slower (+20%)
Weights: lighter (hero 100, title 200)
```

### Solar Flare

Warm-shifted palette with snappy timing for alert-dense interfaces.

```
Primary: #FFAA00 (Solar Gold) Success: #44FF66 (Neon Green)
Warning: #FF6600 (Flame)      Danger:  #FF2222 (Hot Red)
Accent:  #FF44AA (Hot Pink)   Neutral: #7A7A80 (Warm Grey)
Glow: tight (spread 16px)     Timing: faster (-15%)
Weights: heavier (data 600, title 400)
```

### Deep Ocean

Desaturated blue-green palette with slow, ambient timing for monitoring views.

```
Primary: #0099CC (Deep Teal)  Success: #00CC99 (Sea Green)
Warning: #CCAA00 (Muted Gold) Danger:  #CC3344 (Muted Red)
Accent:  #9944CC (Deep Purple) Neutral: #556677 (Ocean Grey)
Glow: diffuse (spread 70px)   Timing: slowest (+30%)
Weights: standard
```

---

## Anti-Patterns

| Anti-Pattern                                    | Why It Fails                                   | Correct Approach                                              |
| ----------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------- |
| Overriding `#050505` background in theme config | Breaks OLED black foundation                   | Background is structural, not thematic. Leave it locked.      |
| Using `transition: linear` in generated presets | Violates physics-based motion requirement      | Use `outExpo`, `snappy`, or `smooth` cubic-bezier curves      |
| Generating pastel or low-saturation colours     | Insufficient contrast on OLED black            | Maintain >= 70% saturation, >= 4.5:1 contrast ratio           |
| Producing `grid-cols-2` or `grid-cols-4` layout | Symmetrical grids banned by structural rules   | Theme controls colour/timing, not layout. Layout is locked.   |
| Setting `rounded-lg` or `rounded-xl` in tokens  | Violates sharp corner constraint               | `rounded-sm` is structural. Themes cannot modify border radius.|
| Using CSS `@keyframes` instead of Framer Motion | Bypasses physics-based animation engine        | All motion via Framer Motion presets                          |
| Generating white text at 100% opacity           | Harsh, breaks opacity hierarchy                | Maximum text opacity is 0.95; use the six-step scale          |

## Checklist

Before merging any generated theme:

- [ ] Six spectral colours defined with hex values
- [ ] All colours pass 4.5:1 contrast against `#050505`
- [ ] Glow intensities within permitted bounds (inner 15-65%, outer 8-35%)
- [ ] Animation durations within permitted ranges (0.15s--4.0s)
- [ ] Typography weights within permitted ranges (100--600)
- [ ] Text opacity scale has six steps, all within bounds
- [ ] Border opacity scale has three steps, all within bounds
- [ ] CSS variables generated and valid
- [ ] TypeScript design token file compiles without errors
- [ ] Tailwind config extension is syntactically correct
- [ ] Framer Motion presets use physics-based easing only
- [ ] Status colour mapping covers all `AgentStatus` values
- [ ] No structural constraints overridden (background, borders, radius, layout)
- [ ] Australian English used throughout (colour, not color)

## Response Format

```
[AGENT_ACTIVATED]: XAEM Theme UI
[PHASE]: {Generation | Translation | Review}
[PASS]: {1 - Generate | 2 - Translate}
[STATUS]: {in_progress | awaiting_verification | complete}

{theme generation output or code translation}

[NEXT_ACTION]: {what happens next}
```

## Integration Points

| System               | Relationship                                                        |
| -------------------- | ------------------------------------------------------------------- |
| `scientific-luxury`  | Downstream consumer. XAEM generates, scientific-luxury enforces.    |
| `council-of-logic`   | Shannon check on token count -- no redundant variables.             |
| `design-tokens.ts`   | Baseline reference. Generated themes must be structurally compatible.|
| `genesis-orchestrator`| Theme generation may be triggered during SECTION_D (frontend shell).|

## Australian Localisation

| Element  | Format             | Example           |
| -------- | ------------------ | ----------------- |
| Date     | DD/MM/YYYY         | 13/02/2026        |
| Time     | H:MM am/pm         | 2:30 pm           |
| Timezone | AEST/AEDT          | 2:30 pm AEDT      |
| Currency | AUD ($)            | $1,234.56         |
| Spelling | Australian English | colour, behaviour |

All generated tokens, comments, and documentation must use Australian English spelling conventions: colour (not color), customise (not customize), optimise (not optimize), analyse (not analyze).

---

**XAEM THEME UI - GENERATION PIPELINE ACTIVE**
