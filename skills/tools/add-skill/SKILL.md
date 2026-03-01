---
name: add-skill
description: Guide for adding a new skill to an existing plugin. Use when creating or adding skills to plugins in this marketplace.
---

# Add Skill Guide

Guide for adding a new skill to an existing plugin in this marketplace.

## Skill Directory Structure

```
plugins/<plugin-name>/skills/<skill-name>/
├── SKILL.md           # Required: skill definition
├── *.md               # Optional: additional instructions
└── scripts/           # Optional: executable scripts
    └── *.sh
```

## SKILL.md Structure

Every skill requires a `SKILL.md` file with YAML frontmatter:

```markdown
---
name: skill-name
description: Brief description of what this skill does and when to use it
---

# Skill Name

## Instructions

[Clear, step-by-step guidance for Claude to follow]

## Examples

[Concrete examples of using this skill]
```

## Required Fields

### name

- Maximum 64 characters
- Lowercase letters, numbers, and hyphens only
- Cannot contain "anthropic" or "claude"

### description

- Must be non-empty
- Maximum 1024 characters
- Include both what the skill does AND when to use it

## Three Levels of Content

### Level 1: Metadata (always loaded)

YAML frontmatter with `name` and `description`. Loaded at startup (~100 tokens).

### Level 2: Instructions (loaded when triggered)

Main body of SKILL.md. Loaded when skill is triggered (under 5k tokens
recommended).

### Level 3: Resources (loaded as needed)

Additional files and scripts. Only loaded when referenced.

## Using Scripts

For complex logic, create executable scripts:

```
skills/<skill-name>/scripts/
└── do-something.sh
```

Benefits:

- Script code never enters context window
- Only script output consumes tokens
- More reliable than generated code

## Script Path Resolution

**Important**: Skills are loaded from a plugin cache, not the repository.
Hardcoded paths like `plugins/foo/skills/bar/scripts/script.sh` will NOT work.

When referencing scripts in SKILL.md, use this pattern:

```markdown
**Script location**: `scripts/my-script.sh` (relative to this skill's directory)

Before running, locate this skill's directory (where this SKILL.md is located),
then execute: bash <skill-directory>/scripts/my-script.sh
```

This ensures Claude finds the correct script path regardless of where the plugin
is installed.

## Reference

- Agent Skills overview:
  https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview
- Best practices:
  https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices
