---
name: skill-authoring
description: Create and validate Claude Code Skills. Use when building new skills, improving existing skills, or discussing skill architecture, frontmatter, progressive disclosure, or best practices.
---

# Skill Authoring

Build effective Skills that Claude can discover and use.

## Quick Start

```bash
# Initialize a new skill
python .claude/skills/skill-authoring/scripts/init_skill.py my-skill-name

# Validate a skill
python .claude/skills/skill-authoring/scripts/quick_validate.py .claude/skills/my-skill/

# Package for distribution
python .claude/skills/skill-authoring/scripts/package_skill.py .claude/skills/my-skill/
```

## Skill Structure

```
my-skill/
├── SKILL.md              # Required: instructions + YAML frontmatter
├── scripts/              # Optional: executable code (not loaded into context)
├── references/           # Optional: documentation (loaded when needed)
└── assets/               # Optional: output templates, images
```

## YAML Frontmatter

```yaml
---
name: my-skill-name
description: Does X for Y. Use when working with Y or when user mentions X.
---
```

**Rules:**
- `name`: lowercase, hyphens, max 64 chars, no "anthropic"/"claude"
- `description`: max 1024 chars, third person, include WHAT + WHEN triggers

## Three-Level Loading

| Level | When | Cost | Content |
|-------|------|------|---------|
| L1: Metadata | Startup | ~100 tokens | name + description |
| L2: Instructions | Triggered | <5k tokens | SKILL.md body |
| L3: Resources | As needed | Unlimited | scripts/, references/ |

## Core Principles

### 1. Be Concise

Claude is smart. Only add what Claude doesn't know.

```python
# BAD: 150 tokens explaining what PDFs are
# GOOD: 50 tokens with just the code
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

### 2. Match Freedom to Fragility

| Task Type | Freedom | Approach |
|-----------|---------|----------|
| Multiple valid ways | High | Text guidelines |
| Preferred pattern | Medium | Pseudocode/templates |
| Fragile operations | Low | Exact scripts |

### 3. Progressive Disclosure

Keep SKILL.md < 500 lines. Split details into references/:

```markdown
## Quick start
[Core instructions]

## Advanced
See [references/advanced.md](references/advanced.md)
```

### 4. One-Level Deep References

```markdown
# GOOD: flat
SKILL.md -> references/api.md
SKILL.md -> references/examples.md

# BAD: nested
SKILL.md -> advanced.md -> details.md -> info
```

## Output Patterns

See [references/output-patterns.md](references/output-patterns.md) for:
- Template Pattern (strict vs flexible)
- Examples Pattern (input/output pairs)

## Workflow Patterns

See [references/workflows.md](references/workflows.md) for:
- Sequential workflows
- Conditional workflows
- Feedback loops

## Scripts

**Execute, don't load:**
```markdown
Run: `python scripts/analyze.py input.pdf`
```

**Handle errors explicitly:**
```python
def process(path):
    try:
        return open(path).read()
    except FileNotFoundError:
        print(f"Creating {path}")
        return ""
```

## Anti-Patterns

| Avoid | Fix |
|-------|-----|
| Windows paths `\` | Use `/` always |
| Too many options | Provide sensible default |
| Magic numbers | Document constants |
| Deep nesting | Keep refs 1 level deep |
| First person description | Use third person |

## Validation Checklist

- [ ] name: lowercase, hyphens, max 64 chars
- [ ] description: WHAT + WHEN, third person
- [ ] SKILL.md < 500 lines
- [ ] References 1 level deep
- [ ] Scripts handle errors
- [ ] No Windows paths
