---
name: tap
description: Create markdown presentations with live code execution
version: 1.0.0
metadata:
  author: tap-sh
  tags: presentations, slides, markdown, live-code
globs:
  - "**/*.md"
---

# Tap - Presentations for Developers

Tap is a CLI tool that transforms markdown files into beautiful, interactive presentations with live code execution.

## When to Use This Skill

Use Tap when building:
- Technical presentations with code demos
- Database query demonstrations with live results
- CLI tool tutorials and walkthroughs
- Workshop materials with executable examples
- Conference talks with code walkthroughs

## Key Capabilities

1. **Markdown-first** - Write slides in familiar markdown syntax
2. **Live code execution** - Run SQL, shell commands, Python, Node.js, and more directly in slides
3. **Beautiful themes** - 5 built-in themes (paper, noir, aurora, phosphor, poster)
4. **11 layouts** - From title slides to code-focused layouts
5. **Presenter mode** - Speaker notes, timer, and slide preview
6. **Export options** - Build static sites or export to PDF

## Quick Start

```bash
# Create a new presentation
tap new my-talk

# Start the dev server
tap dev my-talk.md

# Build for production
tap build my-talk.md

# Export to PDF
tap pdf my-talk.md
```

## Basic Slide Structure

```markdown
---
title: My Presentation
theme: paper
---

# First Slide

Content here.

---

# Second Slide

More content.
```

## Rule Index

This skill includes detailed rules for:

1. **getting-started** - Installation and first presentation
2. **writing-slides** - Markdown syntax, slide separators, speaker notes
3. **frontmatter** - Global config options (title, theme, transitions)
4. **layouts** - All 11 layouts with slot markers
5. **slide-directives** - Per-slide HTML comment syntax
6. **animations** - Transitions and fragment reveals
7. **code-blocks** - Syntax highlighting, line highlighting, diffs
8. **live-code** - Drivers for SQLite, MySQL, PostgreSQL, shell
9. **themes** - 5 built-in themes and customization
10. **cli** - tap new/dev/build/serve/pdf commands
11. **best-practices** - Presentation design tips
12. **mermaid** - Mermaid diagram support (flowcharts, sequence, ER, etc.)
13. **ai-images** - Gemini AI image generation from prompts

## Important Patterns

### Slide Separators
Use `---` on its own line to separate slides:
```markdown
# Slide 1

---

# Slide 2
```

### Slide Directives
Use HTML comments with YAML for per-slide settings:
```markdown
<!--
layout: two-column
transition: fade
-->
```

### Live Code Execution
Add `{driver: 'drivername'}` to code blocks:
````markdown
```sql {driver: sqlite, connection: demo}
SELECT * FROM users;
```
````

### Column Separator
For multi-column layouts, use `|||` to separate content:
```markdown
Left content

|||

Right content
```
