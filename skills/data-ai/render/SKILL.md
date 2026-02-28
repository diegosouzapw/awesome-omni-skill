---
name: render
version: 0.3.0
description: Render mermaid diagrams from .mmd files to ASCII and SVG for Craft.do. Use when the user asks to "render a mermaid diagram", "update diagrams in Craft", or "regenerate ASCII from .mmd files".
globs: ["**/*.mmd"]
---

Render mermaid diagrams from `.mmd` source files to ASCII (injected into local markdown) and SVG (uploaded to Craft.do docs).

## Workflow

1. Edit the `.mmd` source file with `%% diagram:NAME` markers between diagrams
2. Run `craft render file.mmd --local-only` to regenerate ASCII in the corresponding `.md` file
3. Run `craft render file.mmd --doc DOC_ID` to also upload SVGs to a Craft document

The `.md` file uses `<!-- mermaid:NAME -->` markers as injection points. ASCII content between a marker and the next marker or heading is replaced on each render.

## Commands

The craft binary is at: `${CLAUDE_PLUGIN_ROOT}/bin/craft`

Always use this full path when running craft via Bash. Do not assume `craft` is on PATH.

- `render <file.mmd> --local-only` -- regenerate ASCII in the .md file only
- `render <file.mmd> --doc DOC_ID` -- also upload SVGs to a Craft document
- `render <file.mmd> --theme NAME` -- color theme (default: catppuccin-mocha). Available: zinc-dark, tokyo-night, catppuccin-mocha, nord, dracula, github-light, github-dark, solarized-light, solarized-dark, one-dark

## Dependencies

Requires node 24+ and npm install in the plugin directory.
