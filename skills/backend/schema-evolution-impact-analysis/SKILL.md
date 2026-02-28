---
name: Schema Evolution Impact Analysis
description: Analyze the impact of model/schema changes on downstream code — affected repositories, services, handlers, tests, and migration requirements
category: database
version: 1.0.0
triggers:
  - model-change
  - migration-added
  - schema-change
globs: "**/models/**,**/migrations/**,**/database/**"
---

# Schema Evolution Impact Analysis Skill

When a model or database schema changes, trace the full downstream impact to identify all code that needs updating.

## Trigger Conditions
- Model struct fields are added, removed, or modified
- New migration files are created
- Database schema configuration changes
- User invokes with "schema impact" or "schema-evolution-impact"

## Input Contract
- **Required:** The changed model file and specific field changes
- **Optional:** Migration file for the change

## Output Contract
- List of all files that reference the changed model/field
- Impact classification per file (breaking / needs-update / unaffected)
- Migration requirements (new migration needed, data backfill, etc.)
- Test files that need updating
- DTO/response files that need updating

## Tool Permissions
- **Read:** All Go source files
- **Write:** None (read-only analysis)
- **Search:** Grep for model type names, field names, GORM tags

## Execution Steps

1. **Identify changes**: Parse the diff to find added/removed/modified fields and their types
2. **Trace references**: Search for all files importing or referencing the changed model
3. **Classify impact**: For each reference, determine if it's breaking (uses removed field), needs-update (uses modified field), or unaffected
4. **Check migration**: Verify a migration exists for the schema change; check if data backfill is needed
5. **Check DTOs**: Verify request/response DTOs reflect the model changes
6. **Check tests**: Identify test files that create or assert on the changed fields
7. **Report**: Produce impact report with file:line references and required actions

## Success Criteria
- All downstream impacts identified
- Migration exists for every schema change
- No orphaned references to removed fields

## References
- `.cursor/rules/122-gorm-conventions.mdc`
- `.cursor/skills/schema-migration/SKILL.md`
