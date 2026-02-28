---
name: dotfiles
description: Dotfiles project guidelines (English, minimal config, Makefile for installs)
metadata:
  globs: ""
---

# Dotfiles Guidelines

- All comments and documentation must be written in **English**
- Keep configurations **simple and minimal** — avoid over-engineering
- **Avoid unnecessary comments** — prefer self-documenting code. Only add comments when they explain *why* something is done, not *what* it does. Configuration option names are usually self-explanatory.
- Primary platform: **macOS** (Darwin), but keep configs **portable to Linux** (EC2, servers)
- Avoid macOS-specific features when a cross-platform alternative exists
- Use `uname` or similar checks when platform-specific code is unavoidable

## Installation & Package Management

**CRITICAL**: Always use the Makefile for installing tools and applications. Never install packages directly with `brew install`, `npm install -g`, or other package managers. See `makefile.md` for details.

## Symlink Conventions

Configuration files are symlinked from this repo:

- `~/.config/fish` → `~/.dotfiles/fish`
- `~/.config/nvim` → `~/.dotfiles/nvim`
- `~/.config/Cursor/User/*` → `~/.dotfiles/cursor/*`

Always update the Makefile when adding new symlinked configurations.

## Code Style

### Comments
- **Minimal comments**: Only comment when explaining non-obvious behavior or important context
- **No obvious comments**: Don't comment what the code already clearly shows (e.g., `scrollback_lines = 200000` doesn't need a comment)
- **Self-documenting**: Prefer clear variable/option names over comments
- **When to comment**: Only when explaining *why* something is configured a certain way, not *what* it does

## Primary Work Context

Main technologies used daily:

- **TypeScript** — Use strict mode, prefer explicit types
- **PostgreSQL** — Use UPPER CASE for SQL keywords
- **Node.js** — Managed via Volta
