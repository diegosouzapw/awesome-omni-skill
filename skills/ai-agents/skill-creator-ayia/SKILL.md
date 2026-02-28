---
name: skill-creator
description: Create new Claude Agent Skills for NutriProfile. Use this skill when you need to create a new skill, understand skill structure, or modify existing skills. Provides guidance on SKILL.md format and best practices.
allowed-tools: Read,Write,Edit,Glob,Grep
---

# NutriProfile Skill Creator

You are a skill creation expert. This skill helps you create new Claude Agent Skills following the Anthropic specification.

## What Are Skills?

Skills are folders of instructions, scripts, and resources that Claude loads dynamically to improve performance on specialized tasks. They extend Claude's capabilities for specific domains.

## Skill Structure

```
.claude/skills/
└── my-skill-name/
    ├── SKILL.md          # Required: Main skill definition
    ├── references/       # Optional: Additional documentation
    │   ├── api-docs.md
    │   └── examples.md
    ├── scripts/          # Optional: Executable scripts
    │   └── helper.py
    └── assets/           # Optional: Templates, configs
        └── template.json
```

## SKILL.md Format

### Required Structure

```yaml
---
name: skill-name
description: Clear description of what the skill does and when to use it. This is what Claude reads to decide if the skill is relevant.
allowed-tools: Read,Write,Edit,Grep,Glob,Bash
---

# Skill Title

Brief introduction explaining the skill's purpose.

## Context

Background information Claude needs to understand the domain.

## Architecture / Structure

Relevant file structures, components, or systems.

## Key Concepts

Important patterns, rules, or guidelines.

## Examples

Practical examples of how to use the skill.

## Best Practices

Do's and don'ts for the domain.

## Common Tasks

Step-by-step guides for typical operations.
```

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier, lowercase with hyphens |
| `description` | Yes | When/why to use this skill (Claude's trigger) |
| `allowed-tools` | No | Comma-separated tools for skill execution |
| `model` | No | Override default model (e.g., "claude-opus-4-20250514") |
| `license` | No | Skill license terms |

## Creating a New Skill

### Step 1: Identify the Need

Ask yourself:
- What specific task or domain does this skill address?
- What context does Claude need that it doesn't have?
- What tools will Claude need to use?

### Step 2: Create the Structure

```bash
mkdir -p .claude/skills/my-skill-name
```

### Step 3: Write SKILL.md

Start with the template:

```markdown
---
name: my-skill-name
description: [What it does] for NutriProfile. Use this skill when [trigger conditions]. [Key capabilities].
allowed-tools: Read,Write,Edit,Grep,Glob
---

# NutriProfile [Skill Name] Skill

You are a [domain] expert for the NutriProfile application. This skill helps you [main purpose].

## Context

[Background information about the domain]
- Key files: [relevant paths]
- Technologies: [stack info]
- Relationships: [how it connects to other systems]

## Architecture

[File structure or component diagram]

```
path/to/files/
├── file1.ts
├── file2.ts
└── ...
```

## Key Concepts

### [Concept 1]
[Explanation with code examples]

### [Concept 2]
[Explanation with code examples]

## Common Tasks

### Task 1: [Name]
1. Step one
2. Step two
3. Step three

### Task 2: [Name]
[Steps...]

## Best Practices

1. **Do this** - Explanation
2. **Don't do this** - Explanation
3. ...

## Examples

### Example 1: [Scenario]
```typescript
// Code example
```

### Example 2: [Scenario]
[Example...]
```

### Step 4: Add References (Optional)

For complex skills, split content into reference files:

```markdown
# In SKILL.md
For detailed API documentation, see `{baseDir}/references/api-docs.md`
```

### Step 5: Add Scripts (Optional)

```python
# scripts/helper.py
#!/usr/bin/env python3
"""Helper script for the skill."""

def main():
    # Script logic
    pass

if __name__ == "__main__":
    main()
```

Reference in SKILL.md:
```markdown
Run the helper script: `python {baseDir}/scripts/helper.py`
```

## Best Practices for Skills

### Description Field
- **Be specific**: "Manage i18n for 7 languages" not "Handle translations"
- **Include triggers**: "Use when adding translations..."
- **List capabilities**: "Covers FR, EN, DE, ES, PT, ZH, AR"

### Content Organization
- **Keep SKILL.md under 5,000 words**
- **Use clear headers** for parseability
- **Include code examples** for technical skills
- **Reference files** instead of embedding everything

### Tool Permissions
- **Minimal permissions**: Only include needed tools
- **Document why**: Explain tool usage in the skill

### Portability
- **Use `{baseDir}`** for relative paths
- **No hardcoded paths**: `/home/user/project/` won't work
- **No environment assumptions**: Don't assume specific OS

## NutriProfile Skill Categories

### Domain Skills (Application Features)
- `nutrition-analyzer` - Food detection and nutrition
- `recipe-generator` - AI recipe generation
- `ai-coach` - Coaching and gamification

### Development Skills (Code Quality)
- `test-writer` - Testing with Vitest/pytest
- `i18n-manager` - 7-language translations
- `responsive-design` - Mobile-first UI

### Infrastructure Skills (DevOps)
- `api-designer` - FastAPI endpoints
- `database-manager` - PostgreSQL/Alembic
- `deployment-manager` - Fly.io/Cloudflare

### Meta Skills
- `skill-creator` - This skill!

## Skill Discovery

Skills are discovered from:
1. `.claude/skills/` - Project-specific skills
2. `~/.config/claude/skills/` - User-wide skills
3. Plugin-provided skills
4. Built-in Anthropic skills

## Testing Skills

After creating a skill:

1. **Check syntax**: Ensure YAML frontmatter is valid
2. **Test discovery**: Ask Claude about the skill
3. **Test activation**: Give Claude a task matching the description
4. **Verify behavior**: Check Claude uses the skill correctly

## Example: Creating a Security Skill

```markdown
---
name: security-auditor
description: Audit NutriProfile code for security vulnerabilities. Use when reviewing authentication, data validation, API security, or RGPD compliance. Covers OWASP Top 10.
allowed-tools: Read,Grep,Glob
---

# NutriProfile Security Auditor Skill

You are a security expert for NutriProfile...

## OWASP Top 10 Checklist

### 1. Injection
- SQL Injection: Use parameterized queries
- Command Injection: Validate all inputs
...

## Authentication Review
...

## RGPD Compliance
...
```

## Troubleshooting

### Skill Not Activating
- Check `description` field clarity
- Ensure name is unique
- Verify YAML syntax

### Tools Not Available
- Check `allowed-tools` field
- Verify tool names are exact

### Content Too Long
- Split into reference files
- Remove redundant examples
- Use bullet points over prose
