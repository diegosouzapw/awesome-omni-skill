---
name: a11y-specialist
description: Expert in web accessibility (WCAG 2.1/2.2 AA/AAA compliance), ARIA patterns, keyboard navigation, screen reader testing, color contrast, focus management, and automated accessibility testing
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
---

# Accessibility Specialist

Expert skill for building accessible web applications that comply with WCAG 2.1/2.2 standards. Specializes in ARIA patterns, keyboard navigation, screen reader optimization, automated testing, and inclusive design principles.

## Core Capabilities

### 1. WCAG Compliance
- **Level A**: Basic accessibility (must have)
- **Level AA**: Mid-level accessibility (should have) - Industry standard
- **Level AAA**: Enhanced accessibility (nice to have)
- **WCAG 2.1**: Mobile, low vision, cognitive additions
- **WCAG 2.2**: Latest standards (focus appearance, dragging)
- **Section 508**: US federal requirements
- **ADA**: Americans with Disabilities Act compliance

### 2. ARIA Patterns
- **Roles**: Button, dialog, menu, tablist, combobox, listbox
- **States**: aria-expanded, aria-selected, aria-checked, aria-pressed
- **Properties**: aria-label, aria-labelledby, aria-describedby
- **Live Regions**: aria-live, aria-atomic, aria-relevant
- **Relationships**: aria-owns, aria-controls, aria-flowto
- **Patterns**: WAI-ARIA Authoring Practices Guide (APG)

### 3. Keyboard Navigation
- **Tab Order**: Logical focus order
- **Focus Management**: Focus trap, focus restoration
- **Keyboard Shortcuts**: Arrow keys, Enter, Space, Escape
- **Skip Links**: Skip to main content
- **Focus Indicators**: Visible focus styles
- **Roving Tabindex**: Composite widgets

### 4. Screen Reader Support
- **Semantic HTML**: Use correct elements
- **Alt Text**: Descriptive image alternatives
- **Heading Structure**: Logical h1-h6 hierarchy
- **Landmark Regions**: header, nav, main, aside, footer
- **Announcements**: Dynamic content updates
- **Hidden Content**: aria-hidden, sr-only classes

### 5. Visual Accessibility
- **Color Contrast**: WCAG AA (4.5:1 text, 3:1 UI)
- **Color Independence**: Don't rely on color alone
- **Text Sizing**: Relative units (rem, em)
- **Spacing**: Touch targets 44×44px minimum
- **Motion**: Respect prefers-reduced-motion
- **Zoom**: Support 200% zoom

### 6. Automated Testing
- **axe-core**: Industry standard accessibility engine
- **jest-axe**: Jest integration for unit tests
- **Lighthouse**: Google accessibility audits
- **Pa11y**: CI/CD accessibility testing
- **Testing Library**: Built-in accessibility queries
- **ESLint**: jsx-a11y plugin

### 7. Manual Testing
- **Keyboard Only**: Test without mouse
- **Screen Readers**: NVDA, JAWS, VoiceOver
- **Browser DevTools**: Accessibility tree inspection
- **Color Blindness**: Simulators
- **Zoom Testing**: 200% zoom verification
- **User Testing**: Real users with disabilities

## Workflow

### Phase 1: Accessibility Planning
1. **Define Requirements**
   - WCAG level target (A, AA, AAA)?
   - Legal requirements (Section 508, ADA)?
   - User personas with disabilities?
   - Priority features for accessibility?

2. **Audit Existing Code**
   - Run automated tools (axe, Lighthouse)
   - Manual keyboard testing
   - Screen reader testing
   - Color contrast checks

3. **Create Action Plan**
   - Categorize issues (critical, high, medium, low)
   - Estimate effort
   - Prioritize fixes
   - Set timeline

### Phase 2: Implementation
1. **Semantic HTML**
   - Use correct elements
   - Add ARIA only when needed
   - Structure content logically
   - Use landmarks

2. **Keyboard Support**
   - Add keyboard handlers
   - Manage focus
   - Implement roving tabindex
   - Add skip links

3. **Screen Reader Support**
   - Add ARIA labels
   - Implement live regions
   - Test with real screen readers
   - Fix announced text

4. **Visual Accessibility**
   - Fix color contrast
   - Add focus indicators
   - Ensure text sizing
   - Test with zoom

### Phase 3: Testing & Maintenance
1. **Automated Testing**
   - Unit tests with jest-axe
   - Integration tests
   - CI/CD pipeline checks
   - Regular audits

2. **Manual Testing**
   - Keyboard navigation
   - Screen reader testing
   - User testing
   - Browser compatibility

3. **Documentation**
   - Accessibility statement
   - Known issues
   - Usage guidelines
   - Testing procedures

## Accessibility Patterns

### Accessible Button

```tsx
// AccessibleButton.tsx
import { forwardRef, ButtonHTMLAttributes } from 'react'

interface AccessibleButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  /**
   * Accessible label for screen readers
   * Required if button contains only an icon
   */
  'aria-label'?: string

  /**
   * ID of element that labels this button
   */
  'aria-labelledby'?: string

  /**
   * ID of element that describes this button
   */
  'aria-describedby'?: string

  /**
   * Loading state (shows spinner, disables interaction)
   */
  loading?: boolean
}

export const AccessibleButton = forwardRef<HTMLButtonElement, AccessibleButtonProps>(
  ({ children, loading, disabled, 'aria-label': ariaLabel, ...props }, ref) => {
    return (
      <button
        ref={ref}
        type="button"
        disabled={disabled || loading}
        aria-label={ariaLabel}
        aria-busy={loading}
        aria-disabled={disabled || loading}
        {...props}
      >
        {loading && (
          <span className="spinner" aria-hidden="true">
            {/* Spinner icon */}
          </span>
        )}
        {children}
        {loading && <span className="sr-only">Loading...</span>}
      </button>
    )
  }
)

AccessibleButton.displayName = 'AccessibleButton'

// Usage
<AccessibleButton onClick={handleClick}>
  Save Changes
</AccessibleButton>

// Icon-only button (MUST have aria-label)
<AccessibleButton aria-label="Close dialog" onClick={onClose}>
  <XIcon aria-hidden="true" />
</AccessibleButton>
```

### Accessible Modal/Dialog

```tsx
// AccessibleModal.tsx
import { useEffect, useRef, ReactNode } from 'react'
import { createPortal } from 'react-dom'
import { useFocusTrap } from './hooks/useFocusTrap'
import { useEscapeKey } from './hooks/useEscapeKey'

interface AccessibleModalProps {
  isOpen: boolean
  onClose: () => void
  title: string
  children: ReactNode
  /**
   * ID for the modal title (for aria-labelledby)
   */
  titleId?: string
  /**
   * ID for the modal description (for aria-describedby)
   */
  descriptionId?: string
}

export function AccessibleModal({
  isOpen,
  onClose,
  title,
  children,
  titleId = 'modal-title',
  descriptionId,
}: AccessibleModalProps) {
  const modalRef = useRef<HTMLDivElement>(null)
  const previousActiveElement = useRef<HTMLElement | null>(null)

  // Trap focus inside modal
  useFocusTrap(modalRef, isOpen)

  // Close on Escape
  useEscapeKey(onClose, isOpen)

  useEffect(() => {
    if (isOpen) {
      // Store currently focused element
      previousActiveElement.current = document.activeElement as HTMLElement

      // Prevent body scroll
      document.body.style.overflow = 'hidden'

      // Focus modal
      modalRef.current?.focus()
    } else {
      // Restore body scroll
      document.body.style.overflow = ''

      // Restore focus to previous element
      previousActiveElement.current?.focus()
    }

    return () => {
      document.body.style.overflow = ''
    }
  }, [isOpen])

  if (!isOpen) return null

  return createPortal(
    <div
      className="modal-overlay"
      onClick={onClose}
      role="presentation"
    >
      <div
        ref={modalRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby={titleId}
        aria-describedby={descriptionId}
        className="modal-content"
        onClick={(e) => e.stopPropagation()}
        tabIndex={-1}
      >
        <div className="modal-header">
          <h2 id={titleId}>{title}</h2>
          <button
            onClick={onClose}
            aria-label="Close dialog"
            className="modal-close"
          >
            <span aria-hidden="true">×</span>
          </button>
        </div>

        <div className="modal-body" id={descriptionId}>
          {children}
        </div>
      </div>
    </div>,
    document.body
  )
}

// Usage
<AccessibleModal
  isOpen={isOpen}
  onClose={() => setIsOpen(false)}
  title="Confirm Action"
  descriptionId="modal-desc"
>
  <p id="modal-desc">Are you sure you want to delete this item?</p>
  <button onClick={handleConfirm}>Confirm</button>
  <button onClick={() => setIsOpen(false)}>Cancel</button>
</AccessibleModal>
```

### Focus Trap Hook

```tsx
// hooks/useFocusTrap.ts
import { useEffect, RefObject } from 'react'

const FOCUSABLE_ELEMENTS = [
  'a[href]',
  'button:not([disabled])',
  'textarea:not([disabled])',
  'input:not([disabled])',
  'select:not([disabled])',
  '[tabindex]:not([tabindex="-1"])',
].join(', ')

export function useFocusTrap(ref: RefObject<HTMLElement>, isActive: boolean) {
  useEffect(() => {
    if (!isActive) return

    const element = ref.current
    if (!element) return

    const focusableElements = element.querySelectorAll<HTMLElement>(FOCUSABLE_ELEMENTS)
    const firstFocusable = focusableElements[0]
    const lastFocusable = focusableElements[focusableElements.length - 1]

    const handleTabKey = (e: KeyboardEvent) => {
      if (e.key !== 'Tab') return

      if (e.shiftKey) {
        // Shift + Tab
        if (document.activeElement === firstFocusable) {
          lastFocusable?.focus()
          e.preventDefault()
        }
      } else {
        // Tab
        if (document.activeElement === lastFocusable) {
          firstFocusable?.focus()
          e.preventDefault()
        }
      }
    }

    element.addEventListener('keydown', handleTabKey)

    return () => {
      element.removeEventListener('keydown', handleTabKey)
    }
  }, [ref, isActive])
}
```

### Accessible Form with Live Validation

```tsx
// AccessibleForm.tsx
import { useState, useId, FormEvent } from 'react'

export function AccessibleForm() {
  const [email, setEmail] = useState('')
  const [error, setError] = useState('')
  const [success, setSuccess] = useState('')

  const emailId = useId()
  const errorId = useId()
  const successId = useId()

  const validateEmail = (value: string) => {
    if (!value) {
      setError('Email is required')
      return false
    }
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
      setError('Please enter a valid email address')
      return false
    }
    setError('')
    return true
  }

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault()
    if (validateEmail(email)) {
      setSuccess('Form submitted successfully!')
      // Submit form
    }
  }

  return (
    <form onSubmit={handleSubmit} noValidate>
      <div className="form-field">
        <label htmlFor={emailId}>
          Email Address <span aria-label="required">*</span>
        </label>

        <input
          id={emailId}
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          onBlur={() => validateEmail(email)}
          aria-invalid={!!error}
          aria-describedby={error ? errorId : undefined}
          aria-required="true"
          autoComplete="email"
        />

        {error && (
          <div
            id={errorId}
            role="alert"
            aria-live="polite"
            className="error-message"
          >
            {error}
          </div>
        )}
      </div>

      {success && (
        <div
          id={successId}
          role="status"
          aria-live="polite"
          className="success-message"
        >
          {success}
        </div>
      )}

      <button type="submit">Submit</button>
    </form>
  )
}
```

### Accessible Tabs

```tsx
// AccessibleTabs.tsx
import { useState, useRef, useEffect, KeyboardEvent } from 'react'

interface Tab {
  id: string
  label: string
  content: React.ReactNode
}

interface AccessibleTabsProps {
  tabs: Tab[]
  defaultTab?: string
}

export function AccessibleTabs({ tabs, defaultTab }: AccessibleTabsProps) {
  const [activeTab, setActiveTab] = useState(defaultTab || tabs[0].id)
  const tabRefs = useRef<Map<string, HTMLButtonElement>>(new Map())

  const handleKeyDown = (e: KeyboardEvent, currentIndex: number) => {
    let newIndex = currentIndex

    switch (e.key) {
      case 'ArrowLeft':
        newIndex = currentIndex > 0 ? currentIndex - 1 : tabs.length - 1
        break
      case 'ArrowRight':
        newIndex = currentIndex < tabs.length - 1 ? currentIndex + 1 : 0
        break
      case 'Home':
        newIndex = 0
        break
      case 'End':
        newIndex = tabs.length - 1
        break
      default:
        return
    }

    e.preventDefault()
    const newTab = tabs[newIndex]
    setActiveTab(newTab.id)
    tabRefs.current.get(newTab.id)?.focus()
  }

  return (
    <div className="tabs">
      {/* Tab List */}
      <div role="tablist" aria-label="Content tabs">
        {tabs.map((tab, index) => {
          const isActive = activeTab === tab.id

          return (
            <button
              key={tab.id}
              ref={(el) => {
                if (el) tabRefs.current.set(tab.id, el)
              }}
              role="tab"
              id={`tab-${tab.id}`}
              aria-selected={isActive}
              aria-controls={`panel-${tab.id}`}
              tabIndex={isActive ? 0 : -1}
              onClick={() => setActiveTab(tab.id)}
              onKeyDown={(e) => handleKeyDown(e, index)}
              className={isActive ? 'active' : ''}
            >
              {tab.label}
            </button>
          )
        })}
      </div>

      {/* Tab Panels */}
      {tabs.map((tab) => (
        <div
          key={tab.id}
          role="tabpanel"
          id={`panel-${tab.id}`}
          aria-labelledby={`tab-${tab.id}`}
          hidden={activeTab !== tab.id}
          tabIndex={0}
        >
          {tab.content}
        </div>
      ))}
    </div>
  )
}
```

### Screen Reader Only Text

```css
/* sr-only.css */
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

.sr-only-focusable:focus,
.sr-only-focusable:active {
  position: static;
  width: auto;
  height: auto;
  overflow: visible;
  clip: auto;
  white-space: normal;
}
```

```tsx
// Usage
<button>
  <TrashIcon aria-hidden="true" />
  <span className="sr-only">Delete item</span>
</button>
```

### Skip Links

```tsx
// SkipLinks.tsx
export function SkipLinks() {
  return (
    <div className="skip-links">
      <a href="#main-content" className="skip-link">
        Skip to main content
      </a>
      <a href="#navigation" className="skip-link">
        Skip to navigation
      </a>
      <a href="#footer" className="skip-link">
        Skip to footer
      </a>
    </div>
  )
}
```

```css
/* Skip link styles */
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px;
  text-decoration: none;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}
```

## Automated Testing

### Jest + jest-axe

```tsx
// Button.test.tsx
import { render } from '@testing-library/react'
import { axe, toHaveNoViolations } from 'jest-axe'
import { Button } from './Button'

expect.extend(toHaveNoViolations)

describe('Button Accessibility', () => {
  it('should not have accessibility violations', async () => {
    const { container } = render(<Button>Click me</Button>)
    const results = await axe(container)
    expect(results).toHaveNoViolations()
  })

  it('should have accessible name', () => {
    const { getByRole } = render(<Button>Click me</Button>)
    expect(getByRole('button', { name: 'Click me' })).toBeInTheDocument()
  })

  it('icon-only button should have aria-label', async () => {
    const { container } = render(
      <Button aria-label="Close">
        <XIcon />
      </Button>
    )
    const results = await axe(container)
    expect(results).toHaveNoViolations()
  })

  it('disabled button should have aria-disabled', () => {
    const { getByRole } = render(<Button disabled>Click me</Button>)
    expect(getByRole('button')).toHaveAttribute('aria-disabled', 'true')
  })
})
```

### Testing Library Accessibility Queries

```tsx
// Form.test.tsx
import { render, screen } from '@testing-library/react'
import { userEvent } from '@testing-library/user-event'

describe('Form Accessibility', () => {
  it('should have accessible form fields', () => {
    render(<LoginForm />)

    // Use accessible queries (getByRole, getByLabelText)
    const emailInput = screen.getByLabelText(/email/i)
    const passwordInput = screen.getByLabelText(/password/i)
    const submitButton = screen.getByRole('button', { name: /log in/i })

    expect(emailInput).toBeInTheDocument()
    expect(passwordInput).toBeInTheDocument()
    expect(submitButton).toBeInTheDocument()
  })

  it('should show validation errors with proper ARIA', async () => {
    const user = userEvent.setup()
    render(<LoginForm />)

    const submitButton = screen.getByRole('button', { name: /log in/i })
    await user.click(submitButton)

    // Error should be announced to screen readers
    const errorAlert = screen.getByRole('alert')
    expect(errorAlert).toHaveTextContent(/email is required/i)
  })
})
```

### Lighthouse CI

```yaml
# .lighthouserc.yml
ci:
  collect:
    numberOfRuns: 3
    startServerCommand: 'npm run start'
    url:
      - 'http://localhost:3000'
  assert:
    preset: 'lighthouse:recommended'
    assertions:
      # Accessibility score must be >= 90
      'categories:accessibility':
        - error
        - minScore: 0.9

      # Specific accessibility checks
      'aria-allowed-attr': 'error'
      'aria-required-attr': 'error'
      'aria-valid-attr': 'error'
      'button-name': 'error'
      'color-contrast': 'error'
      'document-title': 'error'
      'html-has-lang': 'error'
      'image-alt': 'error'
      'label': 'error'
      'link-name': 'error'
```

## Best Practices

### Semantic HTML First
```tsx
// ❌ BAD - Div soup
<div onClick={handleClick}>Click me</div>

// ✅ GOOD - Use button
<button onClick={handleClick}>Click me</button>
```

### ARIA is a Last Resort
```tsx
// ❌ BAD - Unnecessary ARIA
<button role="button" aria-label="Submit">Submit</button>

// ✅ GOOD - Native semantics
<button>Submit</button>
```

### Always Provide Text Alternatives
```tsx
// ❌ BAD - Icon without label
<button><XIcon /></button>

// ✅ GOOD - Icon with label
<button aria-label="Close"><XIcon aria-hidden="true" /></button>
```

### Keyboard Accessibility
```tsx
// ✅ Ensure all interactive elements are keyboard accessible
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault()
      handleClick()
    }
  }}
>
  Custom button
</div>
```

### Focus Management
```tsx
// ✅ Manage focus in SPAs
useEffect(() => {
  // Focus heading when page changes
  headingRef.current?.focus()
}, [page])
```

## When to Use This Skill

Activate this skill when you need to:
- Build WCAG compliant components
- Add ARIA attributes correctly
- Implement keyboard navigation
- Create accessible modals/dialogs
- Fix accessibility issues
- Set up automated a11y testing
- Audit accessibility compliance
- Train team on accessibility
- Create accessible forms
- Implement screen reader support

## Integration with Agents

```typescript
// Agent Explore → Scan all components for a11y issues
// a11y-specialist → Apply fixes automatically

// Example workflow:
1. Agent finds: 50 buttons without accessible names
2. a11y-specialist adds aria-label to all
3. Agent verifies: All buttons now accessible
```

## Output Format

When implementing accessibility, provide:
1. **Accessible Component**: WCAG compliant code
2. **ARIA Documentation**: Explain ARIA usage
3. **Keyboard Support**: Document keyboard interactions
4. **Test Suite**: jest-axe tests included
5. **Manual Testing Guide**: How to test with screen readers
6. **Compliance Notes**: WCAG level achieved

Always build interfaces that are usable by everyone, regardless of ability.
