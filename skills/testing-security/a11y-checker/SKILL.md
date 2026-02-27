---
name: a11y-checker
description: Accessibility audit for CSS covering focus styles, color contrast, text sizing, screen reader support, and WCAG compliance. Provides actionable fixes. Use when auditing accessibility or fixing a11y issues.
allowed-tools: Read, Grep, Glob
---

# Accessibility Checker Skill

This skill performs comprehensive CSS accessibility audits based on WCAG 2.2 guidelines. I'll identify issues and provide specific, actionable fixes to make your styles more accessible.

## What I Check

### Visual Accessibility
- Color contrast ratios (text, UI components)
- Text sizing and readability
- Focus indicators
- Visual hierarchy
- Spacing and target sizes

### Motion & Animation
- Reduced motion support
- Animation safety (no seizure triggers)
- Auto-playing content
- Parallax effects

### Interaction
- Keyboard navigation
- Focus management
- Interactive element states
- Touch target sizes

### Screen Readers
- Visually hidden but accessible content
- Display/visibility usage
- Content order
- Skip links

## WCAG 2.2 Guidelines I Follow

### Level A (Must Have)
- **1.4.1**: Color not sole indicator
- **2.1.1**: Keyboard accessible
- **2.4.1**: Skip navigation
- **3.2.1**: Consistent on focus

### Level AA (Should Have)
- **1.4.3**: Contrast minimum (4.5:1)
- **1.4.11**: Non-text contrast (3:1)
- **2.4.7**: Focus visible
- **2.5.5**: Target size (44x44px)

### Level AAA (Best Practice)
- **1.4.6**: Contrast enhanced (7:1)
- **1.4.8**: Visual presentation
- **2.4.8**: Location indication

## Common Issues & Fixes

### 1. Missing Focus Indicators

#### ❌ Problem
```css
/* Removes default focus outline */
button:focus {
  outline: none;
}

/* No visible focus indicator */
.link:focus {
  text-decoration: underline;
}
```

#### ✓ Solution
```css
/* Clear, visible focus indicator */
button:focus-visible {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}

/* High contrast focus ring */
.link:focus-visible {
  outline: 2px solid currentColor;
  outline-offset: 4px;
  text-decoration: underline;
}

/* Fallback for browsers without :focus-visible */
button:focus {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}

button:focus:not(:focus-visible) {
  outline: none;
}
```

### 2. Insufficient Color Contrast

#### ❌ Problem
```css
/* 2.85:1 contrast - fails AA */
.text-muted {
  color: #999999;
  background: #ffffff;
}

/* 3.2:1 contrast - fails AA for normal text */
.button {
  color: #3b82f6;
  background: #ffffff;
}
```

#### ✓ Solution
```css
/* 7.03:1 contrast - passes AAA */
.text-muted {
  color: #666666;
  background: #ffffff;
}

/* 7.02:1 contrast - passes AAA */
.button {
  color: #2563eb;
  background: #ffffff;
}

/* Alternative: Use for large text only */
.large-text {
  font-size: 18px;  /* or 14px bold */
  color: #3b82f6;   /* 4.52:1 - passes AA for large text */
}
```

### 3. Tiny Touch Targets

#### ❌ Problem
```css
/* 20x20px - too small */
.icon-button {
  width: 20px;
  height: 20px;
  padding: 0;
}
```

#### ✓ Solution
```css
/* 44x44px - meets WCAG 2.5.5 */
.icon-button {
  width: 44px;
  height: 44px;
  padding: 12px;  /* 20px icon + 12px padding each side */
}

/* Or use minimum size */
.icon-button {
  min-width: 44px;
  min-height: 44px;
  padding: 0.75rem;
}
```

### 4. No Reduced Motion Support

#### ❌ Problem
```css
/* Animated without considering preferences */
.animated {
  animation: spin 2s infinite;
  transition: all 0.5s ease;
}
```

#### ✓ Solution
```css
/* Respects user preferences */
.animated {
  animation: spin 2s infinite;
  transition: all 0.5s ease;
}

@media (prefers-reduced-motion: reduce) {
  .animated {
    animation-duration: 0.01ms;
    animation-iteration-count: 1;
    transition-duration: 0.01ms;
  }
}

/* Or completely remove animations */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-play-state: paused !important;
    transition-duration: 0.01ms !important;
  }
}
```

### 5. Hidden from Screen Readers

#### ❌ Problem
```css
/* Hidden from everyone including screen readers */
.error-message {
  display: none;
}

.announcement {
  visibility: hidden;
}
```

#### ✓ Solution
```css
/* Visually hidden but accessible to screen readers */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

/* Hidden until shown (keeps in DOM) */
.error-message {
  position: absolute;
  left: -9999px;
  opacity: 0;
}

.error-message.visible {
  position: static;
  opacity: 1;
}
```

### 6. Small Text

#### ❌ Problem
```css
/* 10px - too small to read */
.caption {
  font-size: 10px;
}

/* Fixed pixel sizes don't scale */
body {
  font-size: 14px;
}
```

#### ✓ Solution
```css
/* Minimum 12px, uses relative units */
.caption {
  font-size: 0.75rem;  /* 12px if root is 16px */
}

/* Uses relative units for scalability */
body {
  font-size: 1rem;  /* 16px default, scales with user preferences */
}

.small {
  font-size: 0.875rem;  /* 14px */
}

/* Or use clamp for fluid sizing */
body {
  font-size: clamp(1rem, 2vw, 1.125rem);
}
```

### 7. Insufficient Line Height

#### ❌ Problem
```css
/* 1.1 line height - too tight */
.text {
  line-height: 1.1;
}
```

#### ✓ Solution
```css
/* 1.5 line height - WCAG recommendation */
.text {
  line-height: 1.5;
}

/* Adjust for different text sizes */
.heading {
  line-height: 1.2;  /* Tighter for large text */
}

.body {
  line-height: 1.5;  /* Normal for body text */
}

.caption {
  line-height: 1.4;  /* Slightly tighter for small text */
}
```

### 8. CSS-Only Interactions

#### ❌ Problem
```css
/* Hover-only dropdown - not keyboard accessible */
.nav-item:hover .dropdown {
  display: block;
}

/* CSS-only toggle - not accessible */
#toggle:checked + .content {
  display: block;
}
```

#### ✓ Solution
```css
/* Keyboard accessible dropdown */
.nav-item:hover .dropdown,
.nav-item:focus-within .dropdown {
  display: block;
}

/* Or better: use JavaScript for complex interactions */

/* For toggle, ensure keyboard access */
.toggle-button:focus + .content,
.toggle-button[aria-expanded="true"] + .content {
  display: block;
}
```

## Accessibility Audit Checklist

### Color & Contrast
- [ ] Text has 4.5:1 contrast (AA) or 7:1 (AAA)
- [ ] Large text has 3:1 contrast (AA) or 4.5:1 (AAA)
- [ ] UI components have 3:1 contrast
- [ ] Focus indicators have 3:1 contrast
- [ ] Information not conveyed by color alone

### Typography
- [ ] Base font size at least 16px
- [ ] Line height at least 1.5 for body text
- [ ] Text can be resized to 200% without loss
- [ ] Max line length 70-80 characters
- [ ] Sufficient letter and word spacing

### Focus & Keyboard
- [ ] All interactive elements keyboard accessible
- [ ] Visible focus indicators on all elements
- [ ] Focus order follows logical reading order
- [ ] No keyboard traps
- [ ] Skip navigation links available

### Touch & Interaction
- [ ] Touch targets at least 44x44px
- [ ] Adequate spacing between clickable elements
- [ ] Hover states also have focus states
- [ ] No hover-only interactions

### Motion & Animation
- [ ] Respects prefers-reduced-motion
- [ ] No auto-playing animations
- [ ] No flashing/strobing (3+ times per second)
- [ ] Animations can be paused

### Screen Readers
- [ ] Important content not hidden with display:none
- [ ] Decorative images hidden with CSS
- [ ] Content order makes sense
- [ ] Visually hidden text uses proper technique

## Complete Accessible Component Example

```css
/* Accessible button component */
.button {
  /* Size & spacing */
  min-width: 44px;
  min-height: 44px;
  padding: 0.75rem 1.5rem;
  font-size: 1rem;  /* 16px */

  /* Contrast */
  background: #2563eb;  /* 7.02:1 on white */
  color: white;         /* 9.52:1 on button bg */
  border: 2px solid transparent;

  /* Typography */
  font-weight: 600;
  line-height: 1;
  text-align: center;

  /* Visual */
  border-radius: 0.375rem;
  cursor: pointer;
  transition: all 0.2s ease;
}

/* Hover state */
.button:hover {
  background: #1d4ed8;  /* Darker, higher contrast */
  transform: translateY(-1px);
}

/* Focus state */
.button:focus-visible {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}

/* Active state */
.button:active {
  transform: translateY(0);
}

/* Disabled state */
.button:disabled {
  background: #93c5fd;
  color: #1e40af;
  cursor: not-allowed;
  opacity: 0.7;
}

/* Reduced motion */
@media (prefers-reduced-motion: reduce) {
  .button {
    transition-duration: 0.01ms;
  }

  .button:hover {
    transform: none;
  }
}

/* High contrast mode */
@media (prefers-contrast: high) {
  .button {
    border-color: currentColor;
  }
}
```

## Testing Tools I Recommend

- **Chrome DevTools** - Lighthouse accessibility audit
- **axe DevTools** - Browser extension for accessibility testing
- **WAVE** - Web accessibility evaluation tool
- **Color Contrast Analyzer** - Desktop tool for contrast checking
- **Keyboard testing** - Navigate with Tab, Arrow keys, Enter, Escape
- **Screen readers** - NVDA (Windows), JAWS (Windows), VoiceOver (Mac/iOS)

## Example Usage

**You**: "Audit this button for accessibility issues"

```css
.button {
  font-size: 12px;
  padding: 5px 10px;
  color: #999;
  background: white;
  border: 1px solid #ccc;
}

.button:focus {
  outline: none;
}
```

**I'll respond**:

### Issues Found:

1. **🔴 CRITICAL: No focus indicator**
   - WCAG 2.4.7 (Level AA)
   - Fix: Add visible outline on :focus-visible

2. **🔴 CRITICAL: Insufficient contrast**
   - Color #999 on white = 2.85:1 (fails 4.5:1 requirement)
   - WCAG 1.4.3 (Level AA)
   - Fix: Use #666 for 5.74:1 contrast

3. **🟠 IMPORTANT: Small touch target**
   - 25x18px (estimated)
   - WCAG 2.5.5 recommends 44x44px
   - Fix: Increase padding

4. **🟠 IMPORTANT: Small text size**
   - 12px may be difficult to read
   - Fix: Use at least 14px (0.875rem) or 16px (1rem)

### Corrected Code:
```css
.button {
  /* Size */
  min-width: 44px;
  min-height: 44px;
  padding: 0.75rem 1.5rem;

  /* Typography */
  font-size: 1rem;  /* 16px */
  font-weight: 500;

  /* Colors - AAA compliant */
  color: #1f2937;  /* 14.07:1 on white */
  background: white;
  border: 2px solid #6b7280;  /* 5.77:1 on white */

  /* Interaction */
  cursor: pointer;
  transition: all 0.2s ease;
}

.button:hover {
  border-color: #374151;
  background: #f9fafb;
}

.button:focus-visible {
  outline: 2px solid #2563eb;
  outline-offset: 2px;
}

@media (prefers-reduced-motion: reduce) {
  .button {
    transition-duration: 0.01ms;
  }
}
```

## Just Ask!

Request an accessibility audit:
- "Check this CSS for accessibility issues"
- "Audit my button styles"
- "Review focus indicators"
- "Check color contrast"
- "Verify WCAG compliance"

I'll identify problems and provide fixes!
