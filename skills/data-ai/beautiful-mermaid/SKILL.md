---
name: beautiful-mermaid
description: Render Mermaid diagrams as pure ASCII text (default) or themed SVGs using the beautiful-mermaid renderer. Use when the user asks for a Mermaid diagram (flowchart, sequence, class, state, or ER), wants a diagram file generated, or needs terminal-friendly ASCII output.
---

# Beautiful Mermaid

## Overview

Render Mermaid source into pure ASCII text (default) or themed SVGs using a local script powered by the `beautiful-mermaid` package.

## Quick Start

Install once per machine (from the skill folder) before first use.

1. Install dependencies once per machine:
   `bun install`
2. Render pure ASCII (default):
   `bun run scripts/render-mermaid.ts < diagram.mmd`
3. Render SVG:
   `bun run scripts/render-mermaid.ts --format svg --theme tokyo-night --out /tmp/diagram.svg < diagram.mmd`

## Workflow

1. Clarify output target if needed (SVG vs ASCII/Unicode, theme, file path).
2. Produce Mermaid source.
3. Run the renderer script (prefer stdin pipe over temp files).
4. Return both the Mermaid source and the rendered output or file path.

**Best Practice**: Use `echo '...' | bun run scripts/render-mermaid.ts` to pass Mermaid source directly via stdin, avoiding temporary file creation.

## CLI Options

**Core**
- `--format svg|ascii` (default: `ascii`)
- `--input <file>` (else read from stdin)
- `--out <file>` (else write to stdout)
- `--theme <name>` (one of the built-in themes, for SVG format)

**SVG styling**
- `--bg <hex>` `--fg <hex>` `--accent <hex>` `--muted <hex>` `--surface <hex>` `--border <hex>` `--line <hex>`
- `--font <family>`
- `--transparent`

**ASCII styling**
- `--use-ascii` (use pure `+---+` ASCII style, enabled by default)
- `--padding-x <n>` `--padding-y <n>` `--box-border-padding <n>`
- `--fullwidth` (convert to fullwidth characters, enabled by default)

## Script

`scripts/render-mermaid.ts` renders Mermaid from stdin or a file into SVG or ASCII/Unicode.
