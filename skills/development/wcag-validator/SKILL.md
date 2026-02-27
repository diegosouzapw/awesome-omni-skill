---
name: wcag-validator
description: Automatically validate Moodle templates, JavaScript, and CSS for WCAG 2.1 Level AA accessibility compliance. Checks semantic HTML, ARIA patterns, keyboard navigation, color contrast, and screen reader compatibility. Activates when working with Mustache templates, AMD modules, or discussing accessibility, a11y, WCAG, screen readers, or keyboard navigation.
allowed-tools: Read, Grep, Bash
---

# WCAG 2.1 AA Validator Skill

## Automatic Activation

This skill activates when:
- Working with Mustache templates (*.mustache files)
- Editing AMD JavaScript modules (amd/src/*.js)
- Modifying CSS/SCSS files
- User mentions: "accessibility", "a11y", "WCAG", "screen reader", "keyboard", "contrast", "ARIA"
- Implementing forms, modals, dynamic content
- Creating interactive UI components

## Validation Checklist

### 1. HTML Semantics & Structure

**✓ Proper Document Structure**
```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Descriptive Page Title</title>
    </head>
    <body>
        <!-- Content -->
    </body>
</html>
```

**✓ Heading Hierarchy**
- Single h1 per page
- No skipped levels (h2 → h4 ❌)
- Logical document outline

**✓ Semantic Elements**
- `<nav>`, `<main>`, `<aside>`, `<footer>`, `<article>`, `<section>`
- Not `<div>` for everything

**✓ Landmarks**
```html
<header role="banner">
<nav role="navigation" aria-label="Main">
<main role="main">
<aside role="complementary">
<footer role="contentinfo">
```

### 2. Images & Media

**✓ Alt Text**
```html
<!-- Informative image -->
<img src="chart.png" alt="Bar chart showing 60% increase in completion rates">

<!-- Decorative image -->
<img src="divider.png" alt="" role="presentation">

<!-- Linked image -->
<a href="/course/view.php?id=1">
    <img src="icon.png" alt="Go to Introduction to Programming course">
</a>
```

**✓ Complex Images**
```html
<img src="diagram.png"
     alt="Process flowchart"
     aria-describedby="diagram-desc">
<div id="diagram-desc" class="sr-only">
    Detailed description: The process starts with...
</div>
```

### 3. Forms & Inputs

**✓ Label Association**
```html
<!-- Explicit association -->
<label for="username">Username</label>
<input type="text" id="username" name="username">

<!-- Implicit association -->
<label>
    Email
    <input type="email" name="email">
</label>
```

**✓ Required Fields**
```html
<label for="folder">
    Folder name
    <span class="text-danger" aria-label="required">*</span>
</label>
<input type="text"
       id="folder"
       required
       aria-required="true">
```

**✓ Error Handling**
```html
<div class="form-group {{#error}}has-error{{/error}}">
    <label for="email">Email</label>
    <input type="email"
           id="email"
           aria-invalid="{{#error}}true{{/error}}"
           aria-describedby="{{#error}}email-error{{/error}}">
    {{#error}}
    <div id="email-error" class="text-danger" role="alert">
        {{error}}
    </div>
    {{/error}}
</div>
```

**✓ Field Instructions**
```html
<label for="password">Password</label>
<input type="password"
       id="password"
       aria-describedby="password-requirements">
<small id="password-requirements">
    Must be at least 8 characters with one number
</small>
```

### 4. Interactive Elements

**✓ Buttons**
```html
<!-- Text button -->
<button type="button">Save Changes</button>

<!-- Icon button -->
<button type="button" aria-label="Delete file">
    <i class="fa fa-trash" aria-hidden="true"></i>
</button>

<!-- Loading state -->
<button type="submit" aria-busy="true">
    <span class="spinner" aria-hidden="true"></span>
    Loading...
</button>
```

**✓ Links**
```html
<!-- Descriptive text -->
<a href="file.pdf">Download assignment guidelines (PDF, 2MB)</a>

<!-- Icon link -->
<a href="/edit" aria-label="Edit folder settings">
    <i class="fa fa-edit" aria-hidden="true"></i>
</a>

<!-- External link -->
<a href="https://example.com"
   target="_blank"
   rel="noopener noreferrer">
    External resource
    <span class="sr-only">(opens in new window)</span>
</a>
```

**✓ Skip Links**
```html
<a href="#main-content" class="sr-only sr-only-focusable">
    Skip to main content
</a>
```

### 5. ARIA Patterns

**✓ Live Regions**
```html
<!-- Polite announcements (non-urgent) -->
<div aria-live="polite" aria-atomic="true" class="sr-only"></div>

<!-- Assertive announcements (urgent) -->
<div aria-live="assertive" aria-atomic="true" class="sr-only"></div>

<!-- Status updates -->
<div role="status" aria-live="polite">
    File uploaded successfully
</div>

<!-- Alerts -->
<div role="alert">
    Error: Connection failed
</div>
```

**✓ Dialogs/Modals**
```html
<div role="dialog"
     aria-labelledby="dialog-title"
     aria-describedby="dialog-desc"
     aria-modal="true">
    <h2 id="dialog-title">Confirm Deletion</h2>
    <p id="dialog-desc">Are you sure you want to delete this file?</p>
    <button type="button" class="btn-danger">Delete</button>
    <button type="button" class="btn-secondary">Cancel</button>
</div>
```

**✓ Tabs**
```html
<div role="tablist" aria-label="Content views">
    <button role="tab"
            aria-selected="true"
            aria-controls="tree-panel"
            id="tree-tab">
        Tree View
    </button>
    <button role="tab"
            aria-selected="false"
            aria-controls="table-panel"
            id="table-tab"
            tabindex="-1">
        Table View
    </button>
</div>
<div role="tabpanel"
     id="tree-panel"
     aria-labelledby="tree-tab">
    <!-- Tree view content -->
</div>
<div role="tabpanel"
     id="table-panel"
     aria-labelledby="table-tab"
     hidden>
    <!-- Table view content -->
</div>
```

### 6. Keyboard Navigation

**✓ Focus Management**
```javascript
// ✅ Trap focus in modal
const trapFocus = (element) => {
    const focusableElements = element.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    const firstElement = focusableElements[0];
    const lastElement = focusableElements[focusableElements.length - 1];

    element.addEventListener('keydown', (e) => {
        if (e.key === 'Tab') {
            if (e.shiftKey && document.activeElement === firstElement) {
                e.preventDefault();
                lastElement.focus();
            } else if (!e.shiftKey && document.activeElement === lastElement) {
                e.preventDefault();
                firstElement.focus();
            }
        }
    });
};

// ✅ Return focus after modal closes
let previousFocus = null;

const openModal = (modal) => {
    previousFocus = document.activeElement;
    modal.showModal();
    modal.querySelector('button').focus();
};

const closeModal = (modal) => {
    modal.close();
    if (previousFocus) {
        previousFocus.focus();
    }
};
```

**✓ Keyboard Event Handlers**
```javascript
// ✅ Handle both click and keyboard
element.addEventListener('click', handleAction);
element.addEventListener('keydown', (e) => {
    if (e.key === 'Enter' || e.key === ' ') {
        e.preventDefault();
        handleAction();
    }
});

// ✅ Escape to close
dialog.addEventListener('keydown', (e) => {
    if (e.key === 'Escape') {
        closeDialog();
    }
});
```

### 7. Color & Contrast

**✓ Contrast Ratios (WCAG AA)**
- Normal text (< 18pt): 4.5:1 minimum
- Large text (≥ 18pt or 14pt bold): 3.0:1 minimum
- UI components & graphics: 3.0:1 minimum

```css
/* ❌ Poor contrast (2.8:1) */
.text {
    color: #999;
    background: #fff;
}

/* ✅ Good contrast (4.7:1) */
.text {
    color: #666;
    background: #fff;
}

/* ✅ Excellent contrast (7.0:1) */
.text {
    color: #333;
    background: #fff;
}
```

**✓ Don't Rely on Color Alone**
```html
<!-- ❌ Color only -->
<span style="color: red;">Error</span>

<!-- ✅ Color + icon + text -->
<span class="text-danger">
    <i class="fa fa-exclamation-circle" aria-hidden="true"></i>
    Error: Invalid input
</span>
```

### 8. Dynamic Content

**✓ Screen Reader Announcements**
```javascript
// ✅ Announce loading state
const announceLoading = () => {
    const liveRegion = document.querySelector('[aria-live="polite"]');
    liveRegion.textContent = 'Loading files...';
};

// ✅ Announce completion
const announceComplete = (count) => {
    const liveRegion = document.querySelector('[aria-live="polite"]');
    liveRegion.textContent = `${count} files loaded successfully`;
    setTimeout(() => {
        liveRegion.textContent = '';
    }, 1000);
};

// ✅ Announce errors
const announceError = (message) => {
    const liveRegion = document.querySelector('[aria-live="assertive"]');
    liveRegion.textContent = `Error: ${message}`;
};
```

## Validation Workflow

### Step 1: Automated Scan
```bash
# Use grep to find potential issues
grep -r "onclick=" templates/  # Check for click handlers on non-buttons
grep -r "<img" templates/ | grep -v "alt="  # Find images without alt
grep -r "<input" templates/ | grep -v "label"  # Find unlabeled inputs
```

### Step 2: Template Analysis
For each `.mustache` file:
1. Check heading hierarchy
2. Verify form label associations
3. Ensure buttons have accessible text
4. Validate ARIA usage
5. Check color contrast in CSS

### Step 3: JavaScript Analysis
For each AMD module:
1. Check keyboard event handlers
2. Verify focus management
3. Validate live region updates
4. Check for focus traps
5. Ensure escape key handling

### Step 4: Generate Report
```
♿ Accessibility Validation Report

File: templates/folder_view.mustache
Status: ❌ FAILED (3 issues)

Issues:
❌ Line 45: Image missing alt attribute
   <img src="{{icon}}">
   Fix: <img src="{{icon}}" alt="{{iconDescription}}">

❌ Line 78: Button not keyboard accessible
   <div onclick="deleteFile()">Delete</div>
   Fix: <button type="button" onclick="deleteFile()">Delete</button>

❌ Line 102: Form input missing label
   <input type="text" name="foldername">
   Fix: <label for="folder-{{id}}">Folder name</label>
        <input type="text" id="folder-{{id}}" name="foldername">

Recommendations:
- Add aria-live region for file loading status
- Implement keyboard navigation for file list
- Add skip link to file content

WCAG 2.1 AA Compliance: 68% → Target: 100%
```

## Common Patterns

### Accessible Card Component
```html
<div class="card" role="region" aria-labelledby="card-title-{{id}}">
    <div class="card-header">
        <h3 id="card-title-{{id}}">{{title}}</h3>
    </div>
    <div class="card-body">
        <p>{{description}}</p>
    </div>
    <div class="card-footer">
        <a href="{{url}}"
           class="btn btn-primary"
           aria-label="View details for {{title}}">
            View Details
        </a>
    </div>
</div>
```

### Accessible Data Table
```html
<table class="table">
    <caption>List of course files</caption>
    <thead>
        <tr>
            <th scope="col">Filename</th>
            <th scope="col">Size</th>
            <th scope="col">Modified</th>
            <th scope="col">Actions</th>
        </tr>
    </thead>
    <tbody>
        {{#files}}
        <tr>
            <th scope="row">{{name}}</th>
            <td>{{size}}</td>
            <td>{{modified}}</td>
            <td>
                <button type="button"
                        class="btn btn-sm"
                        aria-label="Download {{name}}">
                    Download
                </button>
            </td>
        </tr>
        {{/files}}
    </tbody>
</table>
```

## Integration

- Auto-validates on template writes/edits
- Triggered by `/m:a11y` command
- Pre-commit hook validation
- Part of CI/CD pipeline

## References

- WCAG 2.1 Quick Reference
- ARIA Authoring Practices Guide
- Moodle Accessibility Guidelines
