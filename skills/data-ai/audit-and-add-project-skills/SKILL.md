---
name: audit-and-add-project-skills
description: Audits project skills in .agent/skills/ and Codex skills for Cursor compatibility, then helps add compatible skills to .cursor/skills/. Use when the user wants to migrate project skills to Cursor, check if skills work with Cursor, or add existing skills to Cursor.
---

# Audit and Add Project Skills to Cursor

This skill scans project and Codex skills, checks Cursor compatibility, and guides adding them to Cursor.

## Skill Sources

| Source | Path | Notes |
|--------|------|-------|
| Project skills | `{workspaceFolder}/.agent/skills/*/SKILL.md` | Often Codex/Claude format |
| Codex skills | `~/.codex/skills/**/SKILL.md` | May need adaptation |

## Cursor Skill Format (Required)

Cursor expects:

```markdown
---
name: skill-name
description: What the skill does and when to use it (third person, specific)
---
# Title
Body...
```

### Compatibility Checklist

- [ ] **name**: Lowercase, hyphens only, max 64 chars
- [ ] **description**: Third person, includes WHAT and WHEN (trigger terms)
- [ ] **Body**: Markdown instructions (procedures, examples, workflows)

### Fields Cursor Ignores (OK to Keep)

- `license`, `metadata`, `disable-model-invocation`, `globs`, `alwaysApply`
- Codex-specific: `metadata.short-description`, `agents/openai.yaml` references

### Adaptation Notes

| Project/Codex Pattern | Cursor Equivalent |
|-----------------------|-------------------|
| `${PROJECT_ROOT}` | `{workspaceFolder}` or omit |
| `${BRAIN}`, `${HANDS}` | Omit or replace with "the agent" |
| References to "Codex" or "Claude" | Replace with "the agent" or "Cursor" |
| `~/.codex/skills/` paths | Use project `.cursor/skills/` or `~/.cursor/skills/` |

## Audit Workflow

1. **Discover skills**: List SKILL.md files in `.agent/skills/` and optionally `~/.codex/skills/`
2. **Read each SKILL.md**: Extract frontmatter (name, description)
3. **Validate**: Check name format, description quality, required structure
4. **Report**:
   - ✅ Compatible as-is
   - ⚠️ Compatible with minor edits (update description, fix name)
   - ❌ Needs adaptation (Codex-specific content, missing fields)

## Add-to-Cursor Workflow

**Target**: Project skills → `.cursor/skills/{skill-name}/`

1. Create `.cursor/skills/` if it doesn't exist
2. For each compatible skill:
   - Create `.cursor/skills/{skill-name}/`
   - Copy `SKILL.md` (and optional `reference.md`, `scripts/` if present)
   - Apply adaptations from audit (description, path refs)
3. Preserve original `.agent/skills/`—do not delete

**Do NOT** add skills to `~/.cursor/skills-cursor/`. That directory is reserved for Cursor built-in skills.

## Quick Commands

**List project skills:**
```
Get-ChildItem -Path ".agent/skills" -Directory | ForEach-Object { $_.Name }
```

**Check if SKILL.md exists:**
```
Test-Path ".agent/skills/{skill-name}/SKILL.md"
```

**Copy a skill to Cursor:**
```
Copy-Item -Path ".agent/skills/{skill-name}" -Destination ".cursor/skills/{skill-name}" -Recurse
```

## Decision: Project vs Personal

- **Project** (`.cursor/skills/`): Skills shared with the repo; available only in this workspace
- **Personal** (`~/.cursor/skills/`): Skills for all your projects

Use project scope for workspace-specific skills (NinjaTrader, UniversalOR, etc.). Use personal for generic skills (pdf, docx, code-formatter).
