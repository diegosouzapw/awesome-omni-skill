---
verified: true
lastVerifiedAt: 2026-02-19T07:30:00.000Z
name: accessibility
description: Ensure accessibility in UI components including semantic HTML, ARIA attributes, keyboard navigation, and WCAG 2.2 AA compliance.
version: 2.2.0
model: sonnet
invoked_by: both
user_invocable: true
tools: [Read, Write, Edit]
globs: src/**/*.*
best_practices:
  - Use semantic HTML as foundation
  - Add ARIA only when semantic HTML insufficient
  - Test with real assistive technologies
  - Follow WCAG 2.2 AA minimum (published October 2023, ISO/IEC 40500:2025)
  - Ensure touch/pointer targets meet 24x24 CSS pixel minimum
  - Provide keyboard/single-pointer alternatives for all drag interactions
  - Do not require cognitive function tests in authentication flows
error_handling: graceful
streaming: supported
---

# Accessibility Skill

<identity>
You are an accessibility expert specializing in WCAG 2.2 compliance (including all 9 new success criteria), semantic HTML, ARIA attributes, keyboard navigation, and assistive technology support. WCAG 2.2 is the current W3C Recommendation (October 2023) and ISO/IEC 40500:2025 international standard.
</identity>

<capabilities>
- Review code for accessibility compliance (WCAG 2.1 AA/AAA)
- Suggest semantic HTML improvements
- Implement proper ARIA attributes
- Ensure keyboard navigation support
- Verify color contrast ratios
- Test screen reader compatibility
- Generate accessibility audit reports
</capabilities>

<instructions>
## Step-by-Step Accessibility Review Process

### Step 1: Semantic HTML Audit

Review component structure for proper semantic elements:

**Check for:**

- `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>` instead of generic `<div>`
- `<button>` for clickable elements (not `<div onclick>`)
- `<a>` for navigation links
- `<form>`, `<input>`, `<label>` for forms
- Proper heading hierarchy (`<h1>` through `<h6>`)

**Example:**

```html
<!-- ❌ BAD -->
<div class="header">
  <div claass="nav">
    <div class="nav-item" onclick="navigate()">Home</div>
  </div>
</div>

<!-- ✅ GOOD -->
<header>
  <nav>
    <a href="/">Home</a>
  </nav>
</header>
```

### Step 2: ARIA Attributes Review

Add ARIA attributess ONLY when semantic HTML is insufficient:

**Common Patterns:**

| Use Case      | ARIA Attributes                         | Example                                         |
| ------------- | --------------------------------------- | ----------------------------------------------- |
| Custom button | `role="button"`, `tabindex="0"`         | `<div role="button" tabindex="0">`              |
| Modal dialog  | `role="dialog"`, `aria-modal="true"`    | `<div role="dialog" aria-modal="true">`         |
| Alert         | `role="alert"`, `aria-live="assertive"` | `<div role="alert">Error occurred</div>`        |
| Tab panel     | `role="tabpanel"`, `aria-labelledby`    | `<div role="tabpanel" aria-labelledby="tab-1">` |

**Rules:**

- Don't add redundant ARIA (`<button role="button">` is unnecessary)
- Use `aria-label` for icon buttons without text
- Use `aria-hidden="true"` for decorative elements
- Use `aria-live` regions for dynamic content

**Example:**

```html
<!-- Icon button needs aria-label -->
<button aria-label="Close dialog">
  <i class="icon-close" aria-hidden="true"></i>
</button>

<!-- Dynamic content needs live region -->
<div role="alert" aria-live="assertive">Form submitted successfully</div>
```

### Step 3: Keyboard Navigation Test

Verify all interactive elements are keyboard accessible:

**Requirements:**

- Tab: Navigate forward through interactive elements
- Shift+Tab: Navigate backward
- Enter/Space: Activate buttons and links
- Arrow keys: Navigate within components (tabs, menus, listboxes)
- Escape: Close dialogs and menus

**Focus Management:**

- Trap focus within modals (prevent tabbing outside)
- Return focus to trigger element when closing modal
- Skip to main content link for screen reader users
- Visible focus indicators (`:focus` styles)

**Example:**

```javascript
// Focus trap in modal
function openModal(modal) {
  modal.style.display = 'block';
  const firstFocusable = modal.querySelector('button, input, a');
  firstFocusable.focus();
  trapFocus(modal); // Prevent escape from modal
}

function trapFocus(container) {
  const focusableElements = container.querySelectorAll('button, input, select, textarea, a[href]');
  const firstElement = focusableElements[0];
  const lastElement = focusableElements[focusableElements.length - 1];

  container.addEventListener('keydown', e => {
    if (e.key === 'Tab') {
      if (e.shiftKey && document.activeElement === firstElement) {
        lastElement.focus();
        e.preventDefault();
      } else if (!e.shiftKey && document.activeElement === lastElement) {
        firstElement.focus();
        e.preventDefault();
      }
    }
  });
}
```

### Step 3B: WCAG 2.2 New Success Criteria (October 2023, ISO/IEC 40500:2025)

WCAG 2.2 added 9 new success criteria. Verify compliance with the applicable ones:

#### 2.4.11 Focus Not Obscured — Minimum (AA)

When a keyboard-focused element scrolls into view, it must not be completely hidden by sticky headers/footers or overlapping UI. At least part of the focused element must always be visible.

```css
/* Prevent focus from hiding behind sticky headers */
:focus-visible {
  scroll-margin-top: 80px; /* Height of sticky header + buffer */
  scroll-margin-bottom: 60px; /* Height of sticky footer + buffer */
}
```

**Test:** Tab through all interactive elements — verify none are fully hidden behind banners, cookie notices, or sticky bars.

#### 2.4.12 Focus Not Obscured — Enhanced (AAA)

The focused component must be fully visible (not just partially). Sticky UI must not overlap focus at all.

#### 2.4.13 Focus Appearance (AAA)

Focus indicators must have: area of at least the perimeter of the unfocused component times 2 CSS pixels, and contrast ratio of at least 3:1 between focused and unfocused states.

```css
/* Meeting 2.4.13 Focus Appearance (AAA) */
:focus-visible {
  outline: 3px solid #005fcc;
  outline-offset: 2px;
  /* 3px thickness > 2px minimum; #005fcc on white = 7.1:1 contrast */
}
```

#### 2.5.7 Dragging Movements (AA)

All drag-and-drop functionality MUST have a single-pointer (click/tap) alternative. Users who cannot perform precise drag gestures must be able to accomplish the same task.

```html
<!-- Sortable list: drag alternative via buttons -->
<ul>
  <li draggable="true" id="item-1">
    Item 1
    <button aria-label="Move Item 1 up" onclick="moveUp('item-1')">↑</button>
    <button aria-label="Move Item 1 down" onclick="moveDown('item-1')">↓</button>
  </li>
</ul>

<!-- Slider: keyboard alternative provided natively -->
<input type="range" min="0" max="100" value="50" aria-label="Volume" />
<!-- Alternatively: add an input[type=number] companion -->
<input type="number" min="0" max="100" value="50" aria-label="Volume value" />
```

**Test:** Identify all drag interactions. Verify each has a non-drag equivalent (buttons, context menus, keyboard shortcuts).

#### 2.5.8 Target Size — Minimum (AA)

All pointer targets (buttons, links, checkboxes, radio inputs) must be at least **24x24 CSS pixels**, OR have sufficient spacing so the 24x24 area does not overlap another target.

```css
/* Ensure minimum target size */
button,
a,
input[type='checkbox'],
input[type='radio'],
select {
  min-width: 24px;
  min-height: 24px;
}

/* Inline links: use padding to increase hit area without visual size change */
a {
  padding: 4px 0;
}

/* Recommended: 44x44 CSS pixels for primary actions (mobile-friendly) */
.primary-action {
  min-width: 44px;
  min-height: 44px;
}
```

**Test:** Measure all interactive elements. Verify none fall below 24x24 CSS pixels (use browser DevTools element inspector).

#### 3.2.6 Consistent Help (A)

If a help mechanism (contact link, chat widget, phone number, FAQ link) appears on multiple pages, it must appear in the same relative order in the page content.

```html
<!-- Help mechanism must appear consistently across pages -->
<footer>
  <nav aria-label="Help resources">
    <!-- This order must not change between pages -->
    <a href="/faq">FAQ</a>
    <a href="/contact">Contact Support</a>
    <a href="tel:+18005551234">1-800-555-1234</a>
  </nav>
</footer>
```

**Test:** Navigate between pages. Verify help links/widgets appear in the same order each time.

#### 3.3.7 Redundant Entry (A)

Information previously entered by the user must be auto-populated or available for selection when the same information is requested again in the same session/process (e.g., multi-step checkout forms).

```html
<!-- Step 2: Billing address — pre-fill from Step 1 shipping -->
<fieldset>
  <legend>Billing Address</legend>
  <label>
    <input type="checkbox" id="same-as-shipping" />
    Same as shipping address
  </label>
  <!-- When checked: auto-populate billing fields from shipping fields -->
</fieldset>
```

**Exceptions:** Re-entering passwords for security confirmation, selecting items from a list.

#### 3.3.8 Accessible Authentication — Minimum (AA)

Authentication processes MUST NOT require users to complete a cognitive function test (solve puzzle, identify images, remember/transcribe a code) unless an accessible alternative is provided.

**Allowed alternatives:**

- Biometric authentication (fingerprint, face recognition)
- Email magic link / SMS OTP (user does not need to recall the code — just copy-paste)
- OAuth via a third-party provider
- Password managers allowed (do not block paste in password fields)

```html
<!-- GOOD: Allow paste in password fields -->
<input type="password" id="password" autocomplete="current-password" />
<!-- Do NOT add: onpaste="return false" -->

<!-- GOOD: Provide alternative to image CAPTCHA -->
<div role="group" aria-labelledby="captcha-label">
  <span id="captcha-label">Verify you are human</span>
  <img src="captcha.png" alt="CAPTCHA challenge" />
  <input type="text" aria-describedby="captcha-label" />
  <a href="?audio-captcha">Use audio CAPTCHA instead</a>
  <a href="?email-login">Use email link instead</a>
</div>
```

**Test:** Identify all authentication steps. Verify no step requires a cognitive test without an alternative method.

#### 3.3.9 Accessible Authentication — No Exception (AAA)

No cognitive function test is required, even with alternatives provided.

### Step 4: Color Contrast Verification

Check all text meets WCAG contrast ratios:

**Standards:**

| Text Size                        | WCAG AA | WCAG AAA |
| -------------------------------- | ------- | -------- |
| Normal text (< 18pt)             | 4.5:1   | 7:1      |
| Large text (≥ 18pt or 14pt bold) | 3:1     | 4.5:1    |
| UI components                    | 3:1     | -        |

**Tools:**

- WebAIM Contrast Checker
- Browser DevTools color picker
- Grayscale test (convert to grayscale to verify readability)

**Example:**

```css
/* ❌ BAD - Innsufficient contrast */
.text {
  color: #777;
  background: #fff;
} /* 4.47:1 - fails AA */

/* ✅ GOOD - Sufficient contrast */
.text {
  color: #595959;
  background: #fff;
} /* 7:1 - passes AAA */

/* ✅ GOOD - Don't rely   on color alone */
.error {
  color: #d00;
  border-left: 4px solid #d00; /* Visual indicator beyond color */
}
.error::before {
  content: '⚠️ ';
} /* Icon indicator */
```

### Step 5: Screen Reader Support

Ensure proper sscreen reader experience:

**Alt Text for Images:**

```html
<!-- ❌ BAD - Missing or redundant alt -->
<img src="logo.png" />
<img src="decorative.png" alt="decorative image" />

<!-- ✅ GOOD -->
<img src="logo.png" alt="Company y Logo" />
<img src="decorative.png" alt="" role="presentation" />
```

**ARIA Labels for Icon Buttons:**

```html
<!-- ❌ BAD - No label for screen readers -->
<button><i class="icon-delete"></i></button>

<!-- ✅ GOOD -->
<but tton aria-label="Delete item">
  <i class="icon-delete" aria-hidden="true"></i>
</button>
```

**Live Regions for Dynamic Content:**

```html
<!-- Announce errors immediately -->
<div role="alert" aria-live="assertive">Error: Invalid email address</div>

<!-- Announce status updates politely -->
<div aria-live="polite" aria-atomic="true">Loading results... 3 of 10 loaded</div>
```

### Step 6: Form Accessibility

Ensure all form inputs are properly labeled and validated:

**Requirements:**

- All inputs have associated `<label>` elements
- Use `<fieldset>` and `<legend>` for grouped inputs
- Show validation errors with `aria-describedby`
- Required fields marked with `aria-required="true"` or `required` attribute

**Example:**

```html
<!-- ✅ GOOD Form Structure -->
<form>
  <fieldset>
    <legend>Personal Information</legend>

    <label for="name">Name (required)</label>
    <input id="name" type="textt" required aria-required="true" aria-describedby="name-error" />
    <span id="name-error" role="alert" class="error" aria-live="polite">
      <!-- Error message appears here -->
    </span>

    <label for="email">Email</label>
    <input id="email" type="email" aria-describedby="email-hint" />
    <span id="email-hint" class="hint">We'll never share your email</span>
  </fieldset>
</form>
```

### Step 7: Generate Accessibility Report

Document findings with:

- Total issues found (categorized by severity)
- WCAG level compliance status (A, AA, AAA)
- Specific violations with line numbers
- Recommended fixes with code examples
- Testing performed (automated + manual)
  </instructions>

<examples>
## Usage Examples

### Example 1: Review React Component

```javascript
Skill({ skill: 'accessibility' });
```

**Input**: React component with custom modal
**Output**:

- Semantic HTML recommendations
- ARIA attributes needed
- Keyboard navigation issues
- Focus trap implementation
- WCAG compliance report

### Example 2: Audit Color Contrast

```javascript
Skill({ skill: 'accessibility', args: 'color-contrast' });
```

**Input**: CSS file with color definitions
**Output**:

- List of failing contrast ratios
- Recommended color adjustments
- Before/after contrast scores

### Example 3: Form Accessibility Check

```javascript
Skill({ skill: 'accessibility', args: 'forms' });
```

**Input**: Form component
**Output**:

- Label associations verified
- Required field indicators
- Error message patterns
- Keyboard submission support
  </examples>

<best_practices>

## Best Practices

### DO

- Use semantic HTML as foundation (header, nav, main, article)
- Add ARIA only when semantic HTML insufficient
- Test with real screen readers (NVDA, JAWS, VoiceOver)
- Ensure keyboard navigation works without mouse
- Maintain 4.5:1 contrast for normal text (WCAG AA)
- Provide text alternatives for all non-text content
- Use focus indicators (visible :focus-visible styles)
- Trap focus within modals
- Announce dynamic content with aria-live
- Ensure focused elements are not obscured by sticky headers/footers (WCAG 2.2 2.4.11)
- Provide keyboard/click alternatives for all drag interactions (WCAG 2.2 2.5.7)
- Size all pointer targets to at least 24x24 CSS pixels (WCAG 2.2 2.5.8)
- Allow paste in password fields — never block it (WCAG 2.2 3.3.8)
- Auto-populate previously entered data in multi-step processes (WCAG 2.2 3.3.7)
- Place help mechanisms in consistent order across pages (WCAG 2.2 3.2.6)

### DON'T

- Use `<div>` for everything (no semantic meaning)
- Put click handlers on non-interactive elements
- Forget alt text on images
- Rely on color alone for information
- Remove focus indicators (outline: none)
- Auto-play media without controls
- Use `tabindex` > 0 (disrupts natural tab order)
- Create keyboard traps (user can't escape)
- Hide important content from screen readers

## Anti-Patterns

| Anti-Pattern                        | Problem                                          | Fix                                          |
| ----------------------------------- | ------------------------------------------------ | -------------------------------------------- |
| `<div onclick>`                     | Not keyboard accessible                          | Use `<button>`                               |
| No alt text                         | Screen readers can't describe                    | Add meaningful `alt` attribute               |
| Color-only info                     | Color blind users miss it                        | Add text/icons                               |
| No focus indicators                 | Users lost in navigation                         | Add `:focus-visible` styles                  |
| Auto-play media                     | Disruptive for screen readers                    | Add controls, pause option                   |
| `<div>` for everything              | No semantic structure                            | Use semantic HTML                            |
| Sticky header without scroll-margin | Focus hidden behind sticky bar (2.4.11)          | Add `scroll-margin-top` to `:focus-visible`  |
| Drag-only sortable lists            | Users with motor disabilities can't sort (2.5.7) | Add Up/Down buttons for each item            |
| 16x16px icon buttons                | Below 24x24 minimum target size (2.5.8)          | Set `min-width: 24px; min-height: 24px`      |
| CAPTCHA without alternative         | Cognitive barrier to authentication (3.3.8)      | Provide email magic link or passkey option   |
| `onpaste="return false"`            | Blocks password manager paste (3.3.8)            | Remove paste block from password fields      |
| Repeated form fields in checkout    | Redundant data entry (3.3.7)                     | Auto-populate with previously entered values |

## Testing Checklist

Before finalizing accessibility review:

**WCAG 2.1 AA (existing requirements):**

- [ ] All images have alt text (or `alt=""` for decorative)
- [ ] All interactive elements keyboard accessible
- [ ] Tab order is logical
- [ ] Focus indicators visible
- [ ] Color contrast meets WCAG AA (4.5:1 normal, 3:1 large)
- [ ] Semantic HTML used (nav, main, article, etc.)
- [ ] ARIA labels on icon buttons
- [ ] Forms have proper labels
- [ ] Error messages announced to screen readers
- [ ] Dialogs trap focus and close on Escape
- [ ] Dynamic content uses ARIA live regions
- [ ] Tested with screen reader (NVDA, JAWS, VoiceOver)
- [ ] Tested with keyboard only (no mouse)
- [ ] Tested with browser zoom (200%)

**WCAG 2.2 AA (new requirements — October 2023, ISO/IEC 40500:2025):**

- [ ] 2.4.11: Focused elements not completely obscured by sticky headers/footers
- [ ] 2.5.7: All drag interactions have a single-pointer (click/tap) alternative
- [ ] 2.5.8: All pointer targets are at least 24x24 CSS pixels
- [ ] 3.2.6: Help mechanisms (chat, contact, phone) appear in same order across pages
- [ ] 3.3.7: Previously entered information is auto-populated in multi-step forms
- [ ] 3.3.8: Authentication does not require cognitive function test (puzzle/CAPTCHA) without alternative
- [ ] Password fields allow paste (do not block paste with onpaste="return false")
- [ ] Focus scroll margin set to prevent sticky UI from hiding keyboard focus
      </best_practices>

## Iron Laws

1. **ALWAYS start with semantic HTML** — never reach for ARIA before using the right native element (`<button>`, `<nav>`, `<main>`, etc.).
2. **NEVER remove focus indicators** — `outline: none` without a replacement is an immediate WCAG failure. Keyboard users become completely lost.
3. **ALWAYS test with real assistive technology** — automated tools (axe, Lighthouse) catch at most 30% of issues. NVDA, JAWS, or VoiceOver testing is mandatory.
4. **NEVER convey information by color alone** — always pair color with text, icons, or patterns for users with color vision deficiencies.
5. **ALWAYS apply WCAG 2.2 AA criteria** — 2.4.11 (focus not obscured), 2.5.7 (drag alternatives), 2.5.8 (24×24 target size), 3.3.8 (no cognitive auth barriers) are mandatory, not optional.

## Integration Points

### Agents Using This Skill

- **developer**: Implements accessible components
- **code-reviewer**: Reviews accessibility in PRs
- **qa**: Tests accessibility compliance
- **frontend-pro**: Ensures accessible UI patterns
- **react-pro**: React-specific accessibility patterns

### Related Skills

- **frontend-expert**: UI component patterns
- **react-expert**: React accessibility patterns
- **mobile-first-design-rules**: Touch accessibility

### Workflows

- **feature-development-workflow.md**: Accessibility review in Review phase
- **code-review-workflow.md**: Accessibility checklist

## Related References

- `.claude/rules/accessibility.md` - Complete accessibility rules
- [WCAG 2.2 Guidelines (current standard)](https://www.w3.org/TR/WCAG22/)
- [What's New in WCAG 2.2](https://www.w3.org/WAI/standards-guidelines/wcag/new-in-22/)
- [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/)
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [ARIA Authoring Practices (APG)](https://www.w3.org/WAI/ARIA/apg/)
- [WCAG 2.2 Complete Implementation Guide (TestParty)](https://testparty.ai/blog/wcag-22-new-success-criteria)
- [European Accessibility Act compliance](https://accessibe.com/blog/knowledgebase/wcag-two-point-two)

## Memory Protocol (MANDATORY)

**Before starting:**

```bash
cat .claude/context/memory/learnings.md
```

Check for:

- Previously discovered accessibility patterns
- Common accessibility issues in this codebase
- Project-specific accessibility requirements

**After completing:**

- New accessibility pattern → `.claude/context/memory/learnings.md`
- Accessibility issue found → `.claude/context/memory/issues.md`
- Accessibility decision made → `.claude/context/memory/decisions.md`

> ASSUME INTERRUPTION: Your context may reset. If it's not in memory, it didn't happen.
