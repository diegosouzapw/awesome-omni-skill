---
name: beautiful-mermaid-renderer
description: Render Mermaid diagrams using beautiful-mermaid as Unicode/ASCII (terminal/chat) or SVG (rich UI). Use when you need a quick, good-looking diagram preview in plain text or want to generate an SVG from Mermaid source (Bun preferred).
---

# Beautiful Mermaid Renderer

## Overview

Render Mermaid source into Unicode/ASCII box-drawing diagrams for terminal/chat output, or into SVG for embedding in docs/UIs.

## Quick start (Bun)

1) Install dependencies (one-time per machine):

`cd ~/.codex/skills/beautiful-mermaid-renderer && bun install`

2) Render Unicode (default) to stdout:

`cat diagram.mmd | ./scripts/render.sh --format unicode`

3) Render ASCII (maximum compatibility):

`cat diagram.mmd | ./scripts/render.sh --format ascii`

4) Render SVG to a file:

`cat diagram.mmd | ./scripts/render.sh --format svg --out diagram.svg`

## Workflow

1) Draft or collect Mermaid source
- Prefer small diagrams that fit in a chat/terminal viewport.
- When replying, include the Mermaid source in a fenced `mermaid` block when it helps the user edit or reuse it.

2) Render for user viewing (default)
- Prefer `--format unicode` and paste the output in a fenced code block.
- If box-drawing characters display poorly, rerender with `--format ascii`.

3) Render SVG when needed
- Use `--format svg` and write to a file (`--out`).
- Optionally apply a built-in theme (`--theme`) or custom colors (`--bg` / `--fg`).

## CLI reference (`scripts/render.sh`)

- `--format <unicode|ascii|svg>` (default: `unicode`)
- `--in <file>` (optional; otherwise read from stdin)
- `--out <file>` (optional; otherwise write to stdout)
- `--theme <name>` (SVG only) and `--list-themes`
- `--bg <color>` / `--fg <color>` (SVG only; overrides theme)
- `--transparent` (SVG only)
- `--paddingX <n>` / `--paddingY <n>` / `--boxBorderPadding <n>` (text only)
