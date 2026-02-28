---
name: jekyll-research-theme
description: Create production-grade, accessible Jekyll themes for researchers conducting "research in public." Generates complete lab notebook-style themes with Tufte-inspired sidenotes, KaTeX math rendering, and WCAG 2.1 AA compliance. Use when building Jekyll themes for scientific journals, experiment logs, field notes, or research documentation sites. Supports collections for organizing experiments and field notes, responsive sidenote rendering (sidebar on desktop, inline on mobile), and full-width layout options.
---

# Jekyll Research Log Theme Generator

Generate a complete, production-ready Jekyll theme designed for researchers who work in public. The theme prioritizes accessibility, elegant typography, and scientific workflows with Tufte-style sidenotes and math rendering.

## Design Philosophy

Create a lab notebook aesthetic that feels like a refined scientist's journal:
- **Typography-first**: Serif display fonts paired with readable body text
- **Breathing room**: Generous whitespace and margins for sidenotes
- **Accessibility-first**: WCAG 2.1 AA compliant by default
- **Responsive sidenotes**: Desktop sidebar positioning, mobile inline rendering
- **Math-ready**: KaTeX integration for equations and formulas

## Theme Structure

Generate a complete Jekyll theme with this directory structure:

```
theme-name/
├── _layouts/
│   ├── default.html
│   ├── experiment.html
│   ├── field-note.html
│   └── home.html
├── _includes/
│   ├── head.html
│   ├── header.html
│   ├── footer.html
│   ├── sidenote.html
│   └── math.html
├── _sass/
│   ├── _base.scss
│   ├── _typography.scss
│   ├── _layout.scss
│   ├── _sidenotes.scss
│   ├── _accessibility.scss
│   └── main.scss
├── assets/
│   ├── css/
│   │   └── main.scss
│   └── js/
│       └── sidenotes.js
├── _config.yml
├── index.html
└── README.md
```

## Core Features Implementation

### 1. Collections Configuration

In `_config.yml`, define collections for content types:

```yaml
collections:
  experiments:
    output: true
    permalink: /experiments/:title/
  field_notes:
    output: true
    permalink: /notes/:title/

defaults:
  - scope:
      path: ""
      type: "experiments"
    values:
      layout: "experiment"
      full_width: false
  - scope:
      path: ""
      type: "field_notes"
    values:
      layout: "field-note"
      full_width: false
```

### 2. Tufte-Style Sidenotes

Implement responsive sidenotes that adapt to viewport:

**HTML Pattern** (`_includes/sidenote.html`):
```html
<label for="sn-{{ include.id }}" class="margin-toggle sidenote-number"></label>
<input type="checkbox" id="sn-{{ include.id }}" class="margin-toggle"/>
<span class="sidenote">{{ include.content }}</span>
```

**CSS Requirements** (`_sass/_sidenotes.scss`):
- Desktop (>768px): Position sidenotes in right margin (float or CSS Grid)
- Mobile (≤768px): Hide margin-toggle checkbox, display sidenotes inline after reference
- Ensure `.sidenote-number` has proper ARIA labels for screen readers
- Use `counter-reset` and `counter-increment` for automatic numbering

**JavaScript Enhancement** (`assets/js/sidenotes.js`):
- Progressive enhancement for keyboard navigation
- Ensure sidenotes are accessible via Tab key
- Add ARIA attributes dynamically if needed

### 3. Full-Width Layout Support

In layout files, check for `full_width` frontmatter:

```liquid
{% if page.full_width %}
  <article class="content-full-width">
{% else %}
  <article class="content-sidenotes">
{% endif %}
```

CSS handles width constraints:
```scss
.content-sidenotes {
  max-width: 55%;  // Leaves room for sidenotes
}

.content-full-width {
  max-width: 90%;  // Full reading width
}
```

### 4. KaTeX Math Rendering

In `_includes/math.html`:
```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.css">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.js"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/contrib/auto-render.min.js"></script>
<script>
  document.addEventListener("DOMContentLoaded", function() {
    renderMathInElement(document.body, {
      delimiters: [
        {left: "$$", right: "$$", display: true},
        {left: "$", right: "$", display: false}
      ]
    });
  });
</script>
```

Include in `<head>` of layouts that need math support.

### 5. Experiment Layout

`_layouts/experiment.html` should include:
- Title and metadata (date, tags, status)
- Hypothesis section
- Methodology section
- Results/observations section
- Sidenote support throughout
- Tag display with links to filtered views

### 6. Field Note Layout

`_layouts/field-note.html` should include:
- Timestamp
- Quick-capture formatting (less structured than experiments)
- Tag support
- Sidenote support
- Optional linking to related experiments

## Accessibility Requirements

### Semantic HTML
- Use proper heading hierarchy (h1 → h2 → h3)
- `<main>`, `<nav>`, `<article>`, `<aside>` for structure
- `<figure>` and `<figcaption>` for images and diagrams

### WCAG 2.1 AA Compliance
- **Color Contrast**: Minimum 4.5:1 for normal text, 3:1 for large text
- **Keyboard Navigation**: All interactive elements accessible via keyboard
- **Focus Indicators**: Visible focus states on all focusable elements
- **Skip Links**: "Skip to main content" link at top of page
- **Alt Text**: All images require alt attributes (enforce in layouts)

### ARIA Attributes
- `aria-label` for icon buttons
- `aria-current="page"` for current navigation item
- `role="navigation"` for nav elements
- `aria-describedby` for sidenote references

### Screen Reader Optimization
- Sidenote numbers should have screen reader text: "sidenote reference"
- Hidden content should use `aria-hidden="true"` or `.visually-hidden` class
- Tables need proper `<caption>` and header markup

## Typography Guidelines

### Font Pairing
Choose distinctive, characterful fonts:
- **Headings**: Serif display font (e.g., Playfair Display, Cormorant Garamond, Crimson Pro)
- **Body**: Readable serif (e.g., Lora, Spectral, Source Serif Pro)
- **Monospace**: For code blocks (e.g., JetBrains Mono, Fira Code)

### Type Scale
Use modular scale for hierarchy:
```scss
$base-font-size: 18px;
$scale-ratio: 1.25; // Major third

h1: $base-font-size * $scale-ratio * $scale-ratio * $scale-ratio;
h2: $base-font-size * $scale-ratio * $scale-ratio;
h3: $base-font-size * $scale-ratio;
body: $base-font-size;
small: $base-font-size / $scale-ratio;
```

### Line Height
- Body text: 1.6-1.8 for readability
- Headings: 1.2-1.3 for tighter spacing
- Code blocks: 1.5 for clarity

## Visual Design

### Color Palette
Lab notebook aesthetic with muted, sophisticated tones:
- **Background**: Off-white or warm cream (#FAFAF8, #F5F5F0)
- **Text**: Deep charcoal or near-black (#1A1A1A, #2C2C2C)
- **Accent**: Muted scientific tones (deep teal, rust, indigo)
- **Borders**: Subtle gray for separation (#D4D4D4)

### Layout Principles
- **Asymmetry**: Sidenotes create natural asymmetry
- **Generous margins**: 10-15% on sides for breathing room
- **Vertical rhythm**: Consistent spacing using modular scale
- **Grid system**: Use CSS Grid for layout structure

### Decorative Elements
- Thin horizontal rules (1px) to separate sections
- Subtle drop caps for experiment introductions (optional)
- Figure numbering in margins
- Date stamps styled like lab notebook entries

## Tagging System

### Implementation
In `_config.yml`:
```yaml
tag_page_layout: tag
tag_page_dir: tags
```

Create `tags/index.html` to list all tags.

For each tag, generate filtered collections view:
```liquid
{% for post in site.experiments %}
  {% if post.tags contains page.tag %}
    <!-- Display post -->
  {% endif %}
{% endfor %}
```

### Display Pattern
- Tags appear below post title as clickable pills
- Tag pages show chronological list of related content
- Support multiple tags per entry (project, subject, methodology)

## Configuration File

Provide sensible defaults in `_config.yml`:

```yaml
title: Research Log
description: Public research and experimentation journal
author: [Your Name]
url: https://example.com
baseurl: ""

# Theme settings
sidenotes: true
math_rendering: true
font_headings: "Crimson Pro"
font_body: "Lora"

# Accessibility
skip_to_content: true
high_contrast_mode: false

# Collections (defined above)

# Defaults (defined above)

# Build settings
markdown: kramdown
kramdown:
  input: GFM
  syntax_highlighter: rouge
```

## README Documentation

Include comprehensive README with:
- Installation instructions
- Configuration options
- How to create experiments vs field notes
- Sidenote syntax examples
- Math rendering examples (inline `$x$` and display `$$x$$`)
- Full-width layout usage
- Accessibility features
- Customization guide

## Example Content Templates

### Experiment Frontmatter
```yaml
---
layout: experiment
title: "Hypothesis Testing: Neural Network Convergence"
date: 2024-01-15
tags: [machine-learning, optimization, neural-networks]
status: ongoing
full_width: false
---
```

### Field Note Frontmatter
```yaml
---
layout: field-note
title: "Initial Observations on Dataset Bias"
date: 2024-01-15T14:30:00
tags: [data-quality, machine-learning]
related_experiment: neural-network-convergence
---
```

## Build Instructions

1. Generate all files in the directory structure outlined above
2. Implement responsive sidenotes with mobile-first approach
3. Include KaTeX CDN links in head for math rendering
4. Ensure all color combinations pass WCAG AA contrast requirements
5. Test keyboard navigation on all interactive elements
6. Validate HTML semantics and ARIA usage
7. Create example content (2 experiments, 3 field notes) to demonstrate features

## Design Constraints

- **No JavaScript dependencies** beyond KaTeX and sidenote enhancements
- **No Jekyll plugins** (GitHub Pages compatible)
- **Mobile-first responsive design**
- **Graceful degradation** if JavaScript disabled
- **Print stylesheet** for archival (sidenotes inline, good page breaks)

## Quality Checklist

Before considering the theme complete, verify:
- [ ] Sidenotes render correctly on desktop (margin) and mobile (inline)
- [ ] Math equations display properly with KaTeX
- [ ] All color contrasts pass WCAG AA (check color contrast linter instuctions below)
- [ ] Keyboard navigation works for all interactive elements
- [ ] Focus indicators are clearly visible
- [ ] Collections generate proper permalinks
- [ ] Tags link to filtered views
- [ ] Full-width layout disables sidenotes appropriately
- [ ] Typography scales harmoniously across devices
- [ ] Code blocks have proper syntax highlighting
- [ ] Example content demonstrates all features

## color contrast linter checking
## Installation

Install the package via pip:

```bash
pip install color-contrast-linter
```

## Usage

### 1. Initialize Configuration

Start by creating a configuration file. Run the `init` command to generate a `.color_pairs.yml` file in your project root:

```bash
cc-lint init
```

This file allows you to define the minimum contrast standard (`AA` or `AAA`) and list the color pairs you want to test.

**Example `.color_pairs.yml`:**

```yaml
min_contrast: AA
pairs:
  - foreground: "#000000"
    background: "#ffffff"
  - foreground: "#767676" # Might fail AA for normal text
    background: "#ffffff"
```

### 2. Run the Linter

Execute the `lint` command to check your configured color pairs for accessibility issues:

```bash
cc-lint lint
```

The tool will analyze each pair and report:
- **Pass/Fail status** based on your `min_contrast` setting.
- **Actual contrast ratio** (e.g., 21.0:1).
- **WCAG Level** achieved (AA, AAA, or Fail).

## Notes

This theme prioritizes researchers who think in experiments and observations. Every design decision should support scientific workflows: hypothesis documentation, iterative refinement, cross-referencing notes, and mathematical notation. The aesthetic should feel like a well-maintained lab notebook—professional, organized, and inviting for long-form reading.
