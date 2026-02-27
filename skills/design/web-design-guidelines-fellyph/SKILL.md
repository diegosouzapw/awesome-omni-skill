---
name: web-design-guidelines
description: Review UI code for Web Interface Guidelines compliance. Use when asked to "review my UI", "check accessibility", "audit design", "review UX", or "check my site against best practices".
argument-hint: <file-or-pattern>
---

# Web Interface Guidelines

Review files for compliance with Web Interface Guidelines AND the Wapuulingo Style Guide.

## How It Works

1. Fetch the latest guidelines from the source URL below
2. Read the specified files (or prompt user for files/pattern)
3. Check against all rules in the fetched guidelines
4. Check against the Wapuulingo Style Guide rules below
5. Output findings in the terse `file:line` format

## Guidelines Source

Fetch fresh guidelines before each review:

```
https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md
```

Use WebFetch to retrieve the latest rules. The fetched content contains all the rules and output format instructions.

---

## Wapuulingo Style Guide

The visual language is **bold, gamified, and high-contrast**, designed to make translation tasks feel like a rewarding game.

### Color Palette

| Element | CSS Variable | Hex Code | Usage |
| :--- | :--- | :--- | :--- |
| **Primary Navy** | `--color-navy` | `#003c56` | Main backgrounds, headers, and nav bars |
| **Mascot Yellow** | `--color-yellow` | `#fecb00` | Primary CTAs, mascot body, and highlights |
| **Success Green** | `--color-green` | `#4ab866` | Completion states, Submit buttons, and progress |
| **Neutral White** | `--color-white` | `#ffffff` | Content cards, input fields, and body text on dark |
| **Status Grey** | `--color-gray` | `#e0e0e0` | Inactive states or background for text fields |

### Typography Rules

- **Font:** Use the `--font-main` variable (Outfit or system fallbacks)
- **Headlines:** Bold or Extra Bold for titles and big numbers
- **Body:** Regular or Medium for labels and instructions
- **Stats:** Large, prominent font sizes for motivation-driving numbers

### UI Components

#### Cards & Containers
- **Radius:** Use `--radius-md` (16px) or `--radius-lg` (24px)
- **Style:** Flat design. Avoid heavy drop shadows; use color contrast for depth
- **Layout:** Vertical stacking with generous padding (20-24px)

#### Buttons & Inputs
- **Primary Button:** Full width, high-contrast background (Yellow or Green)
- **Input Fields:** Subtle grey background with colored border (Green) when active
- **Progress Bar:** Thin horizontal bar at screen top to indicate session completion

### Illustration Style (Wapuu)
- **Placement:** Hero (large center-piece on home), Feedback (small peeking versions)
- **Expressions:** Always positive, enthusiastic, or focused

### Voice & Tone
- **Supportive:** "Great job! More approvals are on the way."
- **Direct:** "Translate to: Portuguese (Brazil)."
- **Gamified:** Use terminology like "XP," "Points Awarded," and "Approved"
- **Concise:** Keep string labels short and actionable

---

## Usage

When a user provides a file or pattern argument:
1. Fetch guidelines from the source URL above
2. Read the specified files
3. Apply all rules from the fetched Web Interface Guidelines
4. Apply all rules from the Wapuulingo Style Guide above
5. Output findings using the format specified in the guidelines

If no files specified, ask the user which files to review.

### Review Checklist

When reviewing UI code, check for:

**Accessibility (Web Interface Guidelines):**
- [ ] Semantic HTML (button for actions, a/Link for navigation)
- [ ] Focus-visible states on all interactive elements
- [ ] prefers-reduced-motion support for animations
- [ ] Proper ARIA labels where needed

**Visual Consistency (Wapuulingo Style):**
- [ ] Colors use CSS variables from the design system
- [ ] Border radius uses design tokens (--radius-sm/md/lg)
- [ ] Buttons follow the full-width, high-contrast pattern
- [ ] Typography hierarchy is clear (headlines bold, body regular)
- [ ] Cards have flat design with color contrast (not heavy shadows)

**Interaction Patterns:**
- [ ] Primary actions use Yellow or Green backgrounds
- [ ] Input fields have Green border when focused
- [ ] Progress indicators are visible during async operations
- [ ] Gamified language in user-facing strings
