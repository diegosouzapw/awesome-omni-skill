---
name: skill-creator
description: Tools for creating and managing Claude Code Skills. Use when users need to create new skills, validate skill structure, or package skills for distribution. Triggers include creating custom workflows, encapsulating domain knowledge, building document generation templates, or integrating specific tools and APIs.
---

# Skill Creator

Create high-quality custom Skills for Claude Code.

## Core Principles

1. **Concise is Key** — Only add information Claude doesn't already know; every piece must justify its token cost
2. **Appropriate Freedom** — High freedom uses text guidance, low freedom uses specific scripts
3. **Progressive Loading** — Metadata always loaded, body loaded on trigger, resources loaded on demand

## Creation Workflow

```
1. Define Use Cases → Gather concrete scenarios and trigger conditions
2. Plan Contents    → Identify needed scripts, references, and assets
3. Initialize       → Run scripts/init_skill.py
4. Write Content    → Complete SKILL.md and bundled resources
5. Validate & Pack  → Run scripts/package_skill.py
```

## Skill Structure

```
my-skill/
├── SKILL.md           # Required: Main file
├── scripts/           # Optional: Executable scripts
├── references/        # Optional: Reference docs (loaded into context on demand)
└── assets/            # Optional: Templates, images, etc. (not loaded into context)
```

## SKILL.md Specification

### Frontmatter (Required)

```yaml
---
name: my-skill              # kebab-case, max 64 characters
description: Concise description of functionality and trigger scenarios. Must include WHEN to use.
---
```

### Body Structure Patterns

| Pattern | Best For | Example |
|---------|----------|---------|
| **Workflow-based** | Sequential processes | PDF form filling, document approval |
| **Task-based** | Multiple independent operations | Image processing (crop/rotate/compress) |
| **Reference-based** | Standards or specifications | Brand guidelines, coding standards |
| **Capability-based** | Integrated systems | Product management, data analysis platform |

## Scripts

### Initialize New Skill

```bash
python scripts/init_skill.py <skill-name> --path <output-dir>

# Example
python scripts/init_skill.py my-analyzer --path ~/skills
```

### Validate and Package

```bash
python scripts/package_skill.py <skill-folder> [output-dir]

# Example
python scripts/package_skill.py ~/skills/my-analyzer ./dist
```

## Quality Checklist

- [ ] Description includes trigger conditions (WHEN to use)
- [ ] SKILL.md body < 500 lines
- [ ] No redundant docs (README, CHANGELOG, etc.)
- [ ] Scripts tested and working
- [ ] Large reference docs split out with guidance on when to read them

## Common Patterns Reference

See `references/patterns.md` for:
- Sequential workflow pattern
- Conditional branching pattern
- Output template pattern
- Example-driven pattern
