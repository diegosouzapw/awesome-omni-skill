---
name: aria-expert
description: Expert knowledge of WAI-ARIA (Accessible Rich Internet Applications). Use when users ask about ARIA roles, states, properties, accessible name computation, ARIA attributes (aria-label, aria-labelledby, aria-describedby, etc.), widget roles, landmark roles, live regions, ARIA best practices, or how to implement accessible interactive components. Also use for questions about ARIA specifications, API mappings (core-aam, html-aam), digital publishing ARIA (dpub-aria), graphics ARIA, or any ARIA implementation questions.
license: W3C Document License
---

# ARIA Expert

Expert resource for WAI-ARIA (Accessible Rich Internet Applications) specifications and implementation guidance.

## Auto-Initialize

**Before reading any ARIA files**, check if the `repo/aria-spec/` and `repo/apg-repo/` folders exist in this skill directory. If they don't exist, run the setup script:

```bash
cd .cursor/skills/aria-expert && ./setup.sh
```

This clones the official W3C ARIA repository and the ARIA Authoring Practices Guide. Only needs to run once.

## Updating Content

To pull the latest ARIA specifications:

```bash
cd .cursor/skills/aria-expert/repo/aria-spec && git pull
cd .cursor/skills/aria-expert/repo/apg-repo && git pull
```

## Data Source

This skill uses the **official W3C ARIA repository** and **ARIA Authoring Practices Guide (APG)** as git submodules. Content is in HTML format (ReSpec specifications).

### ARIA Authoring Practices Guide (APG)

| Path | Content |
|------|---------||
| `repo/apg-repo/` | ARIA Authoring Practices Guide |
| `repo/apg-repo/content/patterns/` | Design patterns for common widgets |

**Key content in APG:**
- **5 Rules of ARIA Use** - Fundamental principles (see section below)
- **Design Patterns** - Complete examples with code for:
  - Accordion, Alert, Breadcrumb, Button, Carousel, Checkbox, Combobox
  - Dialog (Modal), Disclosure, Feed, Grids, Links, Listbox, Menu, Meter
  - Radio Group, Slider, Spinbutton, Switch, Table, Tabs, Toolbar, Tooltip, Tree View
- **Keyboard Interaction Patterns** - Required keyboard support for each pattern
- **Example Implementations** - Working code examples
- **Landmark Regions** - How to use ARIA landmarks effectively

### Core ARIA Specification

| Path | Content |
|------|---------|
| `repo/aria-spec/index.html` | Main ARIA 1.3 specification - roles, states, properties |

**Key content in index.html:**
- All ARIA roles (button, link, tab, tabpanel, dialog, menu, etc.)
- All ARIA states and properties (aria-label, aria-checked, aria-expanded, etc.)
- Widget roles, composite roles, document structure roles
- Landmark roles (banner, navigation, main, complementary, etc.)
- Live region roles and properties
- Relationship attributes (aria-labelledby, aria-describedby, aria-owns, etc.)

### Accessible Name and Description Computation

| Path | Content |
|------|---------|
| `repo/aria-spec/accname/index.html` | Accessible Name and Description Computation specification |

**Key content in accname:**
- How browsers compute accessible names
- Name calculation algorithm
- Description calculation algorithm
- Text alternative computation
- Hidden vs. visible content in name calculation

### Accessibility API Mappings

| Path | Content |
|------|---------|
| `repo/aria-spec/core-aam/index.html` | Core Accessibility API Mappings - how ARIA maps to platform APIs |
| `repo/aria-spec/html-aam/index.html` | HTML Accessibility API Mappings - HTML element mappings |

**Key content in AAM specs:**
- How ARIA roles map to platform accessibility APIs (MSAA, UIA, ATK, AX API)
- How HTML elements map to accessibility APIs
- Role, state, and property mappings
- Required accessibility tree computations

### Specialized ARIA Modules

| Path | Content |
|------|---------|
| `repo/aria-spec/dpub-aria/index.html` | Digital Publishing ARIA - roles for digital publications |
| `repo/aria-spec/dpub-aam/index.html` | Digital Publishing AAM - mappings for dpub roles |
| `repo/aria-spec/graphics-aria/index.html` | Graphics ARIA - roles for graphics and diagrams |
| `repo/aria-spec/graphics-aam/index.html` | Graphics AAM - mappings for graphics roles |

### Format-Specific Mappings

| Path | Content |
|------|---------|
| `repo/aria-spec/svg-aam/index.html` | SVG Accessibility API Mappings |
| `repo/aria-spec/mathml-aam/index.html` | MathML Accessibility API Mappings |
| `repo/aria-spec/pdf-aam/index.html` | PDF Accessibility API Mappings |

### Documentation

| Path | Content |
|------|---------|
| `repo/aria-spec/README.md` | Repository overview and contribution guidelines |
| `repo/aria-spec/CONTRIBUTING.md` | How to contribute to ARIA specifications |

## How to Answer ARIA Questions

### For ARIA Best Practices, Rules, or Principles

**Answer directly without searching files** - The 5 Rules of ARIA Use are documented in this file above. No need to search the APG repo for these fundamental principles.

### For Design Pattern Questions (e.g., "How to build a tabs widget?", "Dialog keyboard interaction")

1. Read the appropriate pattern file in `repo/apg-repo/content/patterns/`
2. Common patterns:
   - `accordion/` - Accordion pattern
   - `alert/` - Alert pattern  
   - `button/` - Button pattern
   - `dialog-modal/` - Modal dialog
   - `tabs/` - Tabs pattern
   - `menu-button/` - Menu button
   - `combobox/` - Combobox/autocomplete
3. Each pattern includes:
   - Keyboard interaction requirements
   - ARIA roles, states, and properties
   - Working code examples

### For Role Questions (e.g., "button role", "dialog role", "tablist")

1. Read `repo/aria-spec/index.html` and search for the role name
2. The spec contains:
   - Role definition and purpose
   - Required/supported states and properties
   - Inherited states and properties
   - Required context (parent/child roles)
   - Keyboard interaction expectations
   - Examples of proper usage

### For State/Property Questions (e.g., "aria-label", "aria-expanded", "aria-live")

1. Read `repo/aria-spec/index.html` and search for the attribute name
2. The spec contains:
   - Property definition
   - Valid values (true/false, token list, string, etc.)
   - Which roles support the property
   - Usage examples
   - Relationship to other properties

### For Accessible Name Questions

1. Read `repo/aria-spec/accname/index.html`
2. Search for relevant sections:
   - Name calculation algorithm (step-by-step process)
   - Text alternative computation
   - Hidden content handling
   - Priority of name sources (aria-labelledby, aria-label, alt, etc.)

### For API Mapping Questions

1. For ARIA role/property mappings: Read `repo/aria-spec/core-aam/index.html`
2. For HTML element mappings: Read `repo/aria-spec/html-aam/index.html`
3. Search for the specific role or element name
4. The spec shows platform-specific mappings (Windows, macOS, Linux, etc.)

### For Digital Publishing Questions

1. Read `repo/aria-spec/dpub-aria/index.html`
2. Search for roles like: doc-abstract, doc-acknowledgments, doc-bibliography, doc-chapter, etc.

### For Graphics Questions

1. Read `repo/aria-spec/graphics-aria/index.html`
2. Search for roles like: graphics-document, graphics-symbol, graphics-object

## 5 Rules of ARIA Use

**Critical principles for using ARIA correctly (from WAI-ARIA Authoring Practices):**

### Rule 1: Use Native HTML Semantics When Possible
If you can use a native HTML element or attribute with the semantics and behavior already built in, instead of re-purposing an element and adding ARIA, **then do so**.

**Good:** `<button>Click me</button>`  
**Bad:** `<div role="button" tabindex="0">Click me</div>`

### Rule 2: Don't Change Native Semantics
Do not change native semantics, unless you really have to.

**Bad:** `<h1 role="button">Heading button</h1>`  
**Why:** This creates confusion - it looks like a heading but acts like a button.

### Rule 3: All Interactive ARIA Controls Must Be Keyboard Accessible
All interactive ARIA controls must be usable with the keyboard. If you create a widget using `role="button"`, `role="tab"`, etc., it must be keyboard accessible.

**Required:**
- Tab/Shift+Tab to navigate to controls
- Enter/Space to activate buttons and links
- Arrow keys for composite widgets (tabs, menus, etc.)
- Escape to close dialogs and menus

### Rule 4: Don't Hide Focusable Elements
Do not use `role="presentation"` or `aria-hidden="true"` on a focusable element.

**Bad:** `<button aria-hidden="true">Click me</button>`  
**Why:** Screen readers will skip it, but keyboard users can still focus it, creating confusion.

### Rule 5: All Interactive Elements Must Have an Accessible Name
All interactive elements must have an accessible name.

**Methods to provide accessible names:**
- Visible text content: `<button>Submit</button>`
- `aria-label`: `<button aria-label="Close dialog">×</button>`
- `aria-labelledby`: `<button aria-labelledby="label-id">×</button>`
- alt text for images: `<img src="logo.png" alt="Company name">`

### Bonus Principle: "No ARIA is Better Than Bad ARIA"
Incorrect ARIA can make accessibility **worse** for screen reader users than having no ARIA at all. When in doubt, test with real assistive technologies.

---

## Quick Reference

### Common Widget Roles

| Role | Purpose | Key Properties |
|------|---------|----------------|
| button | Clickable button | aria-pressed, aria-expanded |
| link | Hyperlink | aria-expanded |
| checkbox | Checkable input | aria-checked (required) |
| radio | Radio button | aria-checked (required) |
| textbox | Text input | aria-multiline, aria-readonly, aria-required |
| combobox | Combo box widget | aria-expanded (required), aria-controls |
| listbox | List of selectable items | aria-multiselectable |
| option | Selectable option | aria-selected |
| tab | Tab in a tablist | aria-selected (required) |
| tabpanel | Container for tab content | aria-labelledby (required) |
| menu | Menu widget | - |
| menuitem | Menu item | - |
| menuitemcheckbox | Checkable menu item | aria-checked (required) |
| menuitemradio | Radio menu item | aria-checked (required) |
| dialog | Dialog/modal window | aria-modal, aria-labelledby |
| alertdialog | Alert dialog | aria-modal, aria-labelledby |
| slider | Slider control | aria-valuenow (required), aria-valuemin, aria-valuemax |
| progressbar | Progress indicator | aria-valuenow, aria-valuemin, aria-valuemax |

### Landmark Roles

| Role | Purpose | HTML5 Equivalent |
|------|---------|------------------|
| banner | Site header | `<header>` (when not in article/section) |
| navigation | Navigation links | `<nav>` |
| main | Main content | `<main>` |
| complementary | Complementary content | `<aside>` |
| contentinfo | Footer information | `<footer>` (when not in article/section) |
| search | Search functionality | No equivalent |
| region | Significant page section | `<section>` with accessible name |
| form | Form landmark | `<form>` with accessible name |

### Common ARIA Properties

| Property | Type | Purpose |
|----------|------|---------|
| aria-label | String | Provides accessible name |
| aria-labelledby | ID reference(s) | Names element via other element(s) |
| aria-describedby | ID reference(s) | Provides accessible description |
| aria-expanded | true/false/undefined | Indicates expanded state |
| aria-hidden | true/false | Hides content from accessibility tree |
| aria-live | off/polite/assertive | Indicates live region updates |
| aria-atomic | true/false | Announces entire region on change |
| aria-relevant | additions/removals/text/all | What changes to announce |
| aria-controls | ID reference(s) | Element controlled by this element |
| aria-owns | ID reference(s) | Parent-child relationship |
| aria-current | page/step/location/date/time/true/false | Current item in set |
| aria-disabled | true/false | Indicates disabled state |
| aria-invalid | true/false/grammar/spelling | Indicates validation state |
| aria-required | true/false | Indicates required field |

### Document Structure Roles

| Role | Purpose |
|------|---------|
| article | Self-contained composition |
| document | Document content |
| feed | Scrollable list of articles |
| heading | Section heading |
| img | Image (for grouping elements into single image) |
| list | List of items |
| listitem | Item in a list |
| table | Table structure |
| row | Table row |
| cell | Table cell |
| rowheader | Row header cell |
| columnheader | Column header cell |

### Live Region Properties

| Property | Values | Purpose |
|----------|--------|---------|
| aria-live | off, polite, assertive | Announce updates |
| aria-atomic | true, false | Announce whole region |
| aria-relevant | additions, removals, text, all | What to announce |
| aria-busy | true, false | Region is updating |

### Digital Publishing Roles (dpub-aria)

| Role | Purpose |
|------|---------|
| doc-abstract | Abstract/summary |
| doc-acknowledgments | Acknowledgments |
| doc-afterword | Afterword |
| doc-appendix | Appendix |
| doc-backlink | Back link |
| doc-biblioentry | Bibliography entry |
| doc-bibliography | Bibliography |
| doc-biblioref | Bibliography reference |
| doc-chapter | Chapter |
| doc-colophon | Colophon |
| doc-conclusion | Conclusion |
| doc-cover | Cover image |
| doc-credit | Credit statement |
| doc-credits | Credits |
| doc-dedication | Dedication |
| doc-endnote | Endnote |
| doc-endnotes | Endnotes collection |
| doc-epigraph | Epigraph |
| doc-epilogue | Epilogue |
| doc-errata | Errata |
| doc-example | Example |
| doc-footnote | Footnote |
| doc-foreword | Foreword |
| doc-glossary | Glossary |
| doc-glossref | Glossary reference |
| doc-index | Index |
| doc-introduction | Introduction |
| doc-noteref | Note reference |
| doc-notice | Notice |
| doc-pagebreak | Page break |
| doc-pagelist | Page list |
| doc-part | Part |
| doc-preface | Preface |
| doc-prologue | Prologue |
| doc-pullquote | Pull quote |
| doc-qna | Q&A section |
| doc-subtitle | Subtitle |
| doc-tip | Tip |
| doc-toc | Table of contents |

### Graphics Roles (graphics-aria)

| Role | Purpose |
|------|---------|
| graphics-document | Graphics document container |
| graphics-object | Graphics object |
| graphics-symbol | Graphics symbol |

## Attribution

> This skill uses material from the W3C ARIA Repository. Copyright © W3C® (MIT, ERCIM, Keio, Beihang). See [W3C Document License](https://www.w3.org/Consortium/Legal/2015/doc-license).
