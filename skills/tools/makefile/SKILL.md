---
name: makefile
description: Makefile structure and install workflow
metadata:
  globs: "Makefile"
---

# Makefile Conventions

**Installation constraint:** Always use the Makefile for installing tools and applications. Never run `brew install`, `npm install -g`, or other package manager commands directly — the Makefile is the single source of truth for what is installed and how it is configured.

Structure targets to be self-documenting: the target name should clearly indicate its purpose, avoiding the need for comments.

## Sections (respect this organization)

- `terminal`: Shell and terminal tools (fish, tmux, nvim, wezterm...)
- `work`: Development tools (cursor, docker, gh, terraform...)
- `personal`: Personal apps (obsidian, rust, spotify...)
- `utils`: Utility applications (cleanshot, keycast...)

## Target Patterns

- Tool targets depend on `brew` and the binary path
- Config targets create symlinks from `~/.dotfiles/` to `~/.config/` or `~/`
- Use `${BREW_BIN}` and `${APP_BIN}` variables for paths
- GUI apps (`.app`) are macOS-only; CLI tools should work on both macOS and Linux

## Example

```makefile
newtool: brew ${BREW_BIN}/newtool
${BREW_BIN}/newtool:
	brew install newtool
```

Note: Homebrew works on both macOS and Linux. For Linux-only alternatives, consider adding conditional targets.

## Installation Workflow

When adding a new tool: (1) add it to the Makefile in the right section, (2) run `make <tool-name>`. Do not install via `brew install` or `npm install -g` outside the Makefile.
