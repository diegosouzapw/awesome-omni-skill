---
name: agent-config-maintenance
description: Refactor Codex configuration files and Agent Skills by splitting concerns, deduplicating instructions, and reorganizing guidance across AGENTS.md, project docs, and skills. Use when asked to clean up AGENTS.md, move instructions into skill bundles, or standardize agent setup rules.
---

# Agent Config Maintenance

## Scope

- Target files: `AGENTS.md`, project-doc sections, `skills/*/SKILL.md`, and any references under `skills/*/references/`.
- Focus on separation of concerns: global agent rules vs task-specific workflows.

## Workflow

1. Inventory instructions
   - Use `rg` to find duplicate or conflicting rules across AGENTS, project docs, and skills.
   - Note which rules are global constraints vs task-specific procedures.
2. Decide the split
   - Keep only global, always-on constraints in `AGENTS.md`.
   - Move task-specific procedures into the relevant skill `SKILL.md` or `references/`.
   - Remove duplicated bullets after moving.
3. Update skill metadata
   - Fill `SKILL.md` frontmatter with `name` and a trigger-ready `description`.
   - Keep the description explicit about when to use the skill.
4. Write the skill body
   - Use imperative form.
   - Keep core workflow in `SKILL.md`.
   - Move large checklists or detailed examples into `references/` and link them.
5. Validate consistency
   - Ensure AGENTS and skills do not contradict each other.
   - Verify naming conventions (lowercase + hyphen).
   - Confirm only necessary files exist; avoid extra docs.

## Quality gates

- Keep `SKILL.md` under 500 lines.
- Avoid duplication between `SKILL.md` and any reference files.
- If adding scripts, run them once to validate behavior.
