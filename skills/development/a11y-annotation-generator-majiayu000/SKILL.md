---
name: a11y-annotation-generator
description: Adds accessibility annotations (ARIA labels, roles, alt text) to make web content accessible. Use when user asks to "add accessibility", "make accessible", "add aria labels", "wcag compliance", or "screen reader support".
allowed-tools: [Read, Write, Edit, Glob, Grep]
---

# Accessibility Annotation Generator

Automatically adds ARIA labels, roles, alt text, and other accessibility annotations to HTML/JSX/Vue templates.

## When to Use

- "Make this component accessible"
- "Add ARIA labels"
- "Add alt text to images"
- "Make accessible for screen readers"
- "Add accessibility annotations"
- "WCAG compliance"

## Instructions

### 1. Scan for Accessibility Issues

Analyze HTML/JSX/Vue files for common issues:

```bash
# Find images without alt text
grep -r "<img" src/ --include="*.jsx" --include="*.tsx" --include="*.vue" | grep -v "alt="

# Find buttons without labels
grep -r "<button" src/ | grep -v "aria-label"

# Find form inputs without labels
grep -r "<input" src/ | grep -v "aria-label" | grep -v "<label"

# Find interactive elements without role
grep -r "onClick" src/ | grep -v "role="
```

### 2. Add Missing Alt Text

**Images:**
```jsx
// Before
<img src="/logo.png" />
<img src="/photo.jpg" className="avatar" />

// After
<img src="/logo.png" alt="Company Logo" />
<img src="/photo.jpg" alt="Profile photo of John Doe" className="avatar" />

// Decorative images (no alt needed, but must be explicit)
<img src="/decorative-line.png" alt="" role="presentation" />
```

**Background images (CSS):**
```jsx
// Add ARIA label for meaningful background images
<div
  className="hero-banner"
  role="img"
  aria-label="Team collaborating in modern office"
  style={{ backgroundImage: 'url(/hero.jpg)' }}
>
```

### 3. Add ARIA Labels to Interactive Elements

**Buttons:**
```jsx
// Before
<button onClick={handleDelete}>
  <TrashIcon />
</button>

// After
<button
  onClick={handleDelete}
  aria-label="Delete item"
>
  <TrashIcon aria-hidden="true" />
</button>

// Or with visible text
<button onClick={handleDelete}>
  <TrashIcon aria-hidden="true" />
  <span>Delete</span>
</button>
```

**Icon-only buttons:**
```jsx
// Before
<button onClick={handleEdit}>
  <EditIcon />
</button>

// After
<button
  onClick={handleEdit}
  aria-label="Edit profile"
  title="Edit profile"
>
  <EditIcon aria-hidden="true" />
</button>
```

**Links:**
```jsx
// Before
<a href="/settings">
  <SettingsIcon />
</a>

// After
<a href="/settings" aria-label="Go to settings">
  <SettingsIcon aria-hidden="true" />
</a>

// Avoid generic "click here"
// Before
<a href="/docs">Click here</a>

// After
<a href="/docs">Read the documentation</a>
```

### 4. Add Form Accessibility

**Labels for inputs:**
```jsx
// Before
<input type="email" placeholder="Email" />

// After - Method 1: Visible label (preferred)
<label htmlFor="email">Email address</label>
<input
  type="email"
  id="email"
  name="email"
  aria-required="true"
/>

// After - Method 2: aria-label (if no visible label)
<input
  type="email"
  aria-label="Email address"
  aria-required="true"
  placeholder="Email"
/>
```

**Error messages:**
```jsx
// Before
{error && <span className="error">{error}</span>}

// After
<input
  type="email"
  id="email"
  aria-invalid={!!error}
  aria-describedby={error ? "email-error" : undefined}
/>
{error && (
  <span id="email-error" role="alert" className="error">
    {error}
  </span>
)}
```

**Required fields:**
```jsx
<label htmlFor="name">
  Name <span aria-label="required">*</span>
</label>
<input
  type="text"
  id="name"
  required
  aria-required="true"
/>
```

**Field descriptions:**
```jsx
<label htmlFor="password">Password</label>
<input
  type="password"
  id="password"
  aria-describedby="password-requirements"
/>
<div id="password-requirements">
  Must be at least 8 characters with 1 uppercase letter and 1 number
</div>
```

### 5. Add Semantic HTML and Roles

**Navigation:**
```jsx
// Before
<div className="nav">
  <a href="/">Home</a>
  <a href="/about">About</a>
</div>

// After
<nav aria-label="Main navigation">
  <a href="/">Home</a>
  <a href="/about">About</a>
</nav>
```

**Landmarks:**
```jsx
<header role="banner">
  <nav aria-label="Main navigation">...</nav>
</header>

<main role="main">
  <section aria-labelledby="products-heading">
    <h2 id="products-heading">Our Products</h2>
    ...
  </section>
</main>

<aside role="complementary" aria-label="Related articles">
  ...
</aside>

<footer role="contentinfo">
  ...
</footer>
```

**Lists:**
```jsx
// Before
<div className="menu">
  <div>Home</div>
  <div>About</div>
</div>

// After
<ul role="list">
  <li><a href="/">Home</a></li>
  <li><a href="/about">About</a></li>
</ul>
```

### 6. Add Keyboard Navigation Support

**Focusable elements:**
```jsx
// Before
<div onClick={handleClick}>Click me</div>

// After
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
  aria-label="Submit form"
>
  Click me
</div>

// Better: Use actual button
<button onClick={handleClick} aria-label="Submit form">
  Click me
</button>
```

**Skip links:**
```jsx
<a href="#main-content" className="skip-link">
  Skip to main content
</a>

<main id="main-content">
  ...
</main>

// CSS
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: white;
  padding: 8px;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}
```

**Tab order:**
```jsx
// Control tab order with tabIndex
<button tabIndex={1}>First</button>
<button tabIndex={2}>Second</button>

// Or keep natural order (preferred)
<button>First</button>
<button>Second</button>
```

### 7. Add Live Regions for Dynamic Content

**Announcements:**
```jsx
// Status messages
<div role="status" aria-live="polite">
  {statusMessage}
</div>

// Urgent alerts
<div role="alert" aria-live="assertive">
  {errorMessage}
</div>

// Loading state
<div
  role="status"
  aria-live="polite"
  aria-busy={isLoading}
>
  {isLoading ? 'Loading...' : 'Content loaded'}
</div>
```

**Progress indicators:**
```jsx
<div
  role="progressbar"
  aria-valuenow={progress}
  aria-valuemin={0}
  aria-valuemax={100}
  aria-label="Upload progress"
>
  {progress}%
</div>
```

### 8. Add Modal/Dialog Accessibility

**Modal:**
```jsx
<div
  role="dialog"
  aria-modal="true"
  aria-labelledby="modal-title"
  aria-describedby="modal-description"
>
  <h2 id="modal-title">Confirm Delete</h2>
  <p id="modal-description">
    Are you sure you want to delete this item?
  </p>

  <button onClick={handleConfirm}>Confirm</button>
  <button onClick={handleCancel}>Cancel</button>
</div>

// Focus management
useEffect(() => {
  if (isOpen) {
    // Save current focus
    const previousFocus = document.activeElement

    // Focus first element in modal
    modalRef.current?.focus()

    // Trap focus in modal
    const handleKeyDown = (e) => {
      if (e.key === 'Escape') {
        handleClose()
      }
    }

    document.addEventListener('keydown', handleKeyDown)

    return () => {
      document.removeEventListener('keydown', handleKeyDown)
      // Restore focus
      previousFocus?.focus()
    }
  }
}, [isOpen])
```

### 9. Add Dropdown/Menu Accessibility

**Dropdown menu:**
```jsx
<div>
  <button
    aria-expanded={isOpen}
    aria-haspopup="true"
    aria-controls="dropdown-menu"
    onClick={toggleMenu}
  >
    Menu
  </button>

  {isOpen && (
    <ul
      id="dropdown-menu"
      role="menu"
    >
      <li role="menuitem">
        <a href="/profile">Profile</a>
      </li>
      <li role="menuitem">
        <a href="/settings">Settings</a>
      </li>
      <li role="menuitem">
        <button onClick={handleLogout}>Logout</button>
      </li>
    </ul>
  )}
</div>
```

**Accordion:**
```jsx
<div>
  <h3>
    <button
      aria-expanded={isExpanded}
      aria-controls="panel-1"
      id="accordion-header-1"
      onClick={toggle}
    >
      Section 1
    </button>
  </h3>

  {isExpanded && (
    <div
      id="panel-1"
      role="region"
      aria-labelledby="accordion-header-1"
    >
      Content here...
    </div>
  )}
</div>
```

### 10. Add Table Accessibility

**Data tables:**
```jsx
<table>
  <caption>Employee Information</caption>
  <thead>
    <tr>
      <th scope="col">Name</th>
      <th scope="col">Department</th>
      <th scope="col">Email</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">John Doe</th>
      <td>Engineering</td>
      <td>john@example.com</td>
    </tr>
  </tbody>
</table>
```

**Complex tables:**
```jsx
<table>
  <thead>
    <tr>
      <th id="name" scope="col">Name</th>
      <th id="monday" scope="col">Monday</th>
      <th id="tuesday" scope="col">Tuesday</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="john" scope="row">John</th>
      <td headers="john monday">9-5</td>
      <td headers="john tuesday">9-5</td>
    </tr>
  </tbody>
</table>
```

### 11. Framework-Specific Patterns

**React:**
```jsx
import { useEffect, useRef } from 'react'

function AccessibleComponent() {
  const headingRef = useRef(null)

  // Announce page changes to screen readers
  useEffect(() => {
    headingRef.current?.focus()
    document.title = 'New Page Title'
  }, [])

  return (
    <main>
      <h1 ref={headingRef} tabIndex={-1}>
        Page Title
      </h1>
    </main>
  )
}
```

**Vue:**
```vue
<template>
  <button
    @click="handleClick"
    :aria-label="buttonLabel"
    :aria-pressed="isPressed"
  >
    <Icon :aria-hidden="true" />
    {{ text }}
  </button>
</template>

<script>
export default {
  computed: {
    buttonLabel() {
      return `${this.text} (${this.isPressed ? 'active' : 'inactive'})`
    }
  }
}
</script>
```

### Best Practices

**DO:**
- Use semantic HTML first (button, not div with onClick)
- Provide visible labels when possible
- Use aria-label only when no visible label
- Test with keyboard only (no mouse)
- Test with screen reader (NVDA, JAWS, VoiceOver)
- Maintain focus order that makes sense
- Announce dynamic changes

**DON'T:**
- Use positive tabIndex (except -1 for programmatic focus)
- Hide content that should be accessible
- Use color alone to convey information
- Create keyboard traps
- Disable zoom/pinch
- Remove focus outlines without replacement

### Testing Checklist

- [ ] All images have alt text
- [ ] All buttons/links have labels
- [ ] All form inputs have labels
- [ ] Keyboard navigation works
- [ ] Focus visible on all elements
- [ ] Color contrast meets WCAG AA (4.5:1)
- [ ] Screen reader announces content correctly
- [ ] No keyboard traps
- [ ] Skip links work
- [ ] ARIA roles/labels correct
- [ ] Live regions announce updates
- [ ] Modals trap focus correctly
- [ ] Errors announced to screen reader
