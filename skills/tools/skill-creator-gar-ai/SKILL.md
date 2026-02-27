---
name: skill-creator
description: Create new Claude Code skills with proper structure and conventions. Use when the user wants to create a new skill, scaffold skill templates, or understand skill anatomy.
---

# Skill Creator

Create and manage Claude Code skills with proper structure and conventions.

## Skill Anatomy

Skills live in `.claude/skills/<skill-name>/` and have two forms:

### Simple Skills (Documentation Only)

```
my-skill/
└── SKILL.md
```

### Complex Skills (With Scripts)

```
my-automation/
├── SKILL.md
├── run.py
├── list.py
└── cleanup.py
```

## SKILL.md Format

All skills require a `SKILL.md` with YAML frontmatter:

```yaml
---
name: skill-name
description: Brief description. Use when [trigger conditions]...
---

# Skill Title

[Instructions and documentation]
```

### Required Frontmatter Fields

**name** (required)

- Maximum 64 characters
- Lowercase letters, numbers, and hyphens only
- Cannot contain XML tags
- Cannot contain reserved words: "anthropic", "claude"

**description** (required)

- Maximum 1024 characters
- Must be non-empty
- Cannot contain XML tags
- Should describe what the skill does AND when to use it

### Optional Frontmatter Fields

```yaml
---
name: my-skill
description: Does X. Use when Y...
license: MIT
allowed-tools:
  - Bash
  - Read
metadata:
  author: your-name
  version: 1.0.0
---
```

## Progressive Disclosure

Skills load content in stages to minimize context usage:

| Level | When Loaded | Token Cost | Content |
|-------|-------------|------------|---------|
| **Level 1: Metadata** | Always (startup) | ~100 tokens | `name` and `description` from frontmatter |
| **Level 2: Instructions** | When triggered | <5k tokens | SKILL.md body |
| **Level 3+: Resources** | As needed | Unlimited | Bundled files, scripts (via bash) |

**Key principle**: Claude loads SKILL.md only when the skill is triggered, and reads additional files only when referenced.

## Best Practices (from Anthropic docs)

### Naming Conventions

Use **gerund form** (verb + -ing) for skill names:

- `processing-pdfs`
- `analyzing-spreadsheets`
- `managing-databases`

Or domain prefixes:

- `rust-error-handling`
- `git-worktree-manager`

### Writing Descriptions

**Always write in third person** (injected into system prompt):

- Good: "Processes Excel files and generates reports"
- Avoid: "I can help you process Excel files"
- Avoid: "You can use this to process Excel files"

**Be specific and include triggers**:

```yaml
# Good
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.

# Bad
description: Helps with documents
```

### Content Guidelines

- Keep SKILL.md body under **500 lines**
- Split large content into separate files
- Keep file references **one level deep** from SKILL.md
- Add table of contents to files >100 lines
- Use consistent terminology throughout
- Avoid time-sensitive information
- Be concise - Claude is already smart

### Degrees of Freedom

Match specificity to task fragility:

- **High freedom**: Multiple approaches valid, use text instructions
- **Medium freedom**: Preferred pattern exists, use pseudocode/templates
- **Low freedom**: Operations are fragile, provide exact scripts

## Available Scripts

### create_skill.py

Creates a simple documentation skill.

```bash
python3 create_skill.py --name "my-pattern" --description "Use when implementing X..."
python3 create_skill.py --name "rust-caching" --description "Caching patterns for Rust" --category rust
```

### create_complex_skill.py

Creates a skill with script templates.

```bash
python3 create_complex_skill.py --name "my-automation" --description "Automate X tasks" --scripts "run,list,cleanup"
```

### list_skills.py

Lists all existing skills.

```bash
python3 list_skills.py           # Summary view
python3 list_skills.py --verbose # Show scripts for complex skills
```

## Workflow Examples

**User says: "Create a skill for database migrations"**

```bash
python3 create_skill.py --name "db-migrations" --description "Database migration patterns. Use when creating or managing database schema changes."
```

**User says: "I need a skill that automates deployment"**

```bash
python3 create_complex_skill.py --name "deploy-manager" --description "Automate deployment workflows. Use when deploying, rolling back, or checking deployment status." --scripts "deploy,rollback,status"
```

**User says: "What skills do we have?"**

```bash
python3 list_skills.py
```

## Checklist for Effective Skills

Before sharing a skill, verify:

- [ ] Description is specific and includes trigger conditions
- [ ] Description written in third person
- [ ] SKILL.md body under 500 lines
- [ ] No time-sensitive information
- [ ] Consistent terminology throughout
- [ ] File references one level deep
- [ ] Scripts have clear documentation (for complex skills)

## References

- [Agent Skills Overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [Best Practices Guide](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Skills Specification](https://github.com/anthropics/skills/tree/main/spec)
- [Skills Cookbook](https://github.com/anthropics/claude-cookbooks/tree/main/skills)
