---
name: cgr-docs-governance
description: Use this skill when updating project documentation after backend/frontend operational changes, incidents, workflow behavior changes, or new runbooks.
---

# CGR Docs Governance

## Goal

Keep documentation aligned with code and production behavior.

## Scope

- root README
- docs/README
- docs/03_guia_desarrollo.md
- docs/04_operacion_y_mantenimiento.md
- docs/99_briefing_agente_experto.md

## Workflow

1. Identify behavior changes from code diff.
2. Validate claims against production evidence if operational.
3. Update docs with concrete commands and interpretation notes.
4. Ensure no stale numbers/dates remain without timestamp.

## Rules

- Prefer explicit dates in status notes.
- Include runbooks, not only descriptions.
- Document known incidents with cause and mitigation.

## References

- `references/update-checklist.md`
