---
name: create-skill
description: Creates new Claude Code skills, optimizes existing skills, or audits all skills/rules/references following Anthropic best practices. Use when adding workflows, improving skill quality, or checking configuration consistency.
argument-hint: '[new <skill-name> | optimize <skill-name> | audit]'
user-invocable: true
---

# Create Skill

Creates, optimizes, or audits Claude Code configuration for the helix portfolio.

## Modes

**Create new skill**: `/create-skill new <skill-name>`
**Optimize existing skill**: `/create-skill optimize <skill-name>`
**Audit all configuration**: `/create-skill audit`

## Helix Structure

```
.claude/
├── rules/                    # Code conventions (auto-loaded)
│   ├── vue-components.md
│   ├── typescript.md
│   ├── styling.md
│   ├── content-system.md
│   └── git-workflow.md
├── skills/
│   └── <skill-name>/
│       ├── SKILL.md          # Required: Main instructions
│       ├── references/       # Optional: Supporting documentation
│       │   └── *.md
│       └── scripts/          # Optional: Python utilities
└── settings.local.json       # Permissions
```

## YAML Frontmatter

Required fields:

```yaml
---
name: skill-name
description: Third-person description of what and when.
allowed-tools: Read, Glob, Bash
---
```

Optional fields:

| Field            | Purpose                               |
| ---------------- | ------------------------------------- |
| `argument-hint`  | Autocomplete hint, e.g. `[filename]`  |
| `user-invocable` | Set `true` for `/skill-name` commands |
| `context`        | Set `fork` for subagent execution     |

## Naming Conventions

Use `verb-noun` pattern:

- `create-*` for generation workflows
- `review-*` for analysis workflows
- `resolve-*` for fix workflows

Rules:

- Lowercase, hyphens, numbers only
- Maximum 64 characters
- No reserved words: `anthropic`, `claude`

## Description Best Practices

**Third person** (injected into system prompt):

- Good: "Generates minimalist images for the portfolio"
- Bad: "I generate images for you"

**Include what + when**:

```yaml
description: Audits SEO compliance. Use before deployment or when optimizing search visibility.
```

**Under 200 characters** for clean display.

## Instruction Quality

Skills are prompts. Apply prompt engineering:

### Be Specific

```markdown
# Bad
"create an image"

# Good
"generate a 1080x1080 black and white abstract image representing [concept]"
```

### State Constraints

- What NOT to do: "don't use color"
- What to reference: "follow visual-identity.md guidelines"
- Expected output: "return CDN URLs for all variants"

### Emphasize Critical Rules

```markdown
IMPORTANT: Never upload without --name flag.
```

### Provide Examples

For output-sensitive skills, show concrete input/output pairs.

## Content Guidelines

### Conciseness

- Challenge each paragraph: "Does Claude need this?"
- Assume Claude knows standard tools/patterns
- No verbose explanations

### Progressive Disclosure

- SKILL.md: Essential workflow (under 200 lines)
- references/: Detailed guidelines, examples

### Checklists for Complex Tasks

```markdown
- [ ] Step 1: Gather requirements
- [ ] Step 2: Generate output
- [ ] Step 3: Verify quality
- [ ] Step 4: Update content files
```

## Create New Skill Workflow

1. **Determine category**: create-_, review-_, resolve-\*

2. **Create structure**:

   ```bash
   mkdir -p .claude/skills/{skill-name}
   mkdir -p .claude/skills/{skill-name}/references  # if needed
   ```

3. **Write SKILL.md**:
   - Frontmatter (name, description, allowed-tools)
   - When to Use section
   - Commands section
   - Workflow steps

4. **Add references/** if detailed guidelines needed

5. **Update CLAUDE.md** skills table

6. **Test** with `/skill-name`

## Optimize Existing Skill Workflow

1. **Read current skill** files

2. **Audit against checklist**:
   - [ ] Description third person, under 200 chars
   - [ ] Description includes what + when
   - [ ] SKILL.md under 200 lines
   - [ ] No verbose explanations
   - [ ] Critical rules use IMPORTANT
   - [ ] References in references/ (not inline)
   - [ ] Examples for output-sensitive tasks

3. **Apply fixes** to SKILL.md

4. **Update CLAUDE.md** if description changed

## Audit All Configuration Workflow

Run `/create-skill audit` to check all configuration:

### 1. Audit Skills

For each skill in `.claude/skills/`:

| Check       | Requirement                                   |
| ----------- | --------------------------------------------- |
| Frontmatter | name, description, allowed-tools present      |
| Description | Third person, what + when, < 200 chars        |
| Length      | SKILL.md < 200 lines                          |
| References  | Detailed content in references/, not SKILL.md |
| Emphasis    | Critical rules marked IMPORTANT               |

### 2. Audit Rules

For each rule in `.claude/rules/`:

| Check      | Requirement                           |
| ---------- | ------------------------------------- |
| Focused    | Single topic per file                 |
| Concise    | < 50 lines                            |
| Actionable | Concrete guidelines, not explanations |
| No overlap | Each rule covers unique content       |

### 3. Audit References

For each `references/*.md` in skills:

| Check     | Requirement                      |
| --------- | -------------------------------- |
| Necessary | Contains info Claude can't infer |
| Current   | No outdated information          |
| Linked    | Referenced from SKILL.md         |

### 4. Check CLAUDE.md Alignment

- All user-invocable skills listed in table
- Descriptions match skill files
- No stale entries

### 5. Output Report

```markdown
## Configuration Audit Report

### Skills
| Skill | Status | Issues |
|-------|--------|--------|
| create-image | PASS | - |
| review-seo | WARN | Description too long |

### Rules
| Rule | Status | Issues |
|------|--------|--------|
| vue-components.md | PASS | - |
| typescript.md | PASS | - |

### References
| File | Status | Issues |
|------|--------|--------|
| visual-identity.md | PASS | - |
| seo-guidelines.md | PASS | - |

### CLAUDE.md
- [ ] All skills listed: YES/NO
- [ ] Descriptions match: YES/NO

### Summary
- Items audited: X
- Passing: Y
- Warnings: Z
- Failures: W
```

## Anti-Patterns

- First-person descriptions ("I help you...")
- Verbose explanations of common concepts
- SKILL.md over 200 lines without references/
- Inconsistent naming patterns
- Missing IMPORTANT on critical rules
- Deeply nested references (SKILL.md → ref.md → detail.md)

## Related

- **After creating skill**: Update CLAUDE.md skills table
- **For code changes**: Follow `.claude/rules/` conventions
