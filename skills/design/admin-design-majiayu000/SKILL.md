---
name: admin-design
description: Minimal, high-clarity admin UI design for this repo. Use when redesigning /admin pages (translation manager, dashboards, tables, forms), defining admin design tokens, or improving admin UX/keyboard workflows without changing core functionality.
---

# Admin Design

## Goals

- Minimal, low-distraction UI for internal tools.
- High information density without clutter.
- Clear hierarchy and obvious primary actions.
- Fast scanning and keyboard-first workflows.
- Core 2.0 inspired admin shell (collapsible sidebar + header, soft cards).
- Header stays minimal: workspace label + Sign Out only (no avatar, no user details).
- Preserve existing functionality and data flows.

## Workflow

1. Audit the current page structure and required features before proposing layout changes.
2. Pick a layout pattern from `references/layout-patterns.md`.
3. Apply the admin tokens from `references/design-tokens.md` (mirrors `.admin-root` in `client/src/index.css`).
4. Keep interaction behavior consistent with `references/interaction-guidelines.md`.
5. Only introduce new components if they reduce clicks or improve clarity.
6. Validate that every existing action still works (save, suggest, publish, notes, voice, Manglish).

## References

- `references/design-tokens.md`
- `references/layout-patterns.md`
- `references/interaction-guidelines.md`
- `references/component-patterns.md`
