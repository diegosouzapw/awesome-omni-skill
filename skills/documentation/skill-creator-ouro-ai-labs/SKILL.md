---
name: skill-creator
description: Guide for creating effective ouro skills. Use when users want to create a new skill (or update an existing skill) that extends ouro's capabilities with specialized knowledge, workflows, or tool integrations.
---

# Skill Creator

This skill guides you through creating effective ouro skills that extend the agent's capabilities.

## Core Principles

### Concise is Key

Every line in a skill costs tokens. Write tightly:

- Remove filler words ("please", "you should", "it's important to")
- Use imperative/infinitive form ("Run script" not "You should run the script")
- Prefer bullet lists over prose
- Skip obvious context the LLM already knows

### Set Appropriate Degrees of Freedom

- **Too specific** = rigid, breaks on edge cases
- **Too broad** = vague, inconsistent outputs
- **Just right** = clear guardrails with room for judgment

Example: Instead of "Always use pytest", say "Use pytest for Python tests; adapt for other languages."

## Anatomy of a Skill

Every skill consists of a required SKILL.md file and optional bundled resources:

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter metadata (required)
│   │   ├── name: (required)
│   │   └── description: (required)
│   └── Markdown instructions (required)
└── Bundled Resources (optional)
    ├── scripts/          - Executable code (Python/Bash/etc.)
    ├── references/       - Documentation to load into context as needed
    └── assets/           - Files used in output (templates, icons, fonts, etc.)
```

### SKILL.md (required)

Every SKILL.md consists of:

- **Frontmatter** (YAML): Contains `name` and `description` fields. These are the only fields that ouro reads to determine when the skill gets used, thus it is very important to be clear and comprehensive in describing what the skill is, and when it should be used.
- **Body** (Markdown): Instructions and guidance for using the skill. Only loaded AFTER the skill triggers.

### Bundled Resources (optional)

Skills can include three types of resources:

| Type | Purpose | When to use |
|------|---------|-------------|
| `scripts/` | Executable code (Python, Bash, etc.) | Reusable automation, data processing, API calls |
| `references/` | Documentation files (.md) | Schemas, API docs, style guides |
| `assets/` | Output files (templates, images) | Boilerplate projects, icon sets, fonts |

### What NOT to Include

- README.md, CHANGELOG.md (skill consumers don't need repo docs)
- Test files (unless the skill is about testing)
- CI/CD configs
- Package manifests for the skill itself

## Progressive Disclosure Patterns

### Pattern 1: Single entry point

For simpler skills, keep everything in SKILL.md:

```markdown
# My Skill

## Quick Start
...

## Advanced Options
...
```

### Pattern 2: Domain-based organization

For skills with multiple domains, organize content by domain:

```
bigquery-skill/
├── SKILL.md (overview and navigation)
└── references/
    ├── finance.md (revenue, billing metrics)
    ├── sales.md (opportunities, pipeline)
    └── product.md (API usage, features)
```

When a user asks about sales metrics, ouro only reads sales.md.

### Pattern 3: Conditional details

Show basic content, link to advanced content:

```markdown
# DOCX Processing

## Creating documents
Use docx-js for new documents. See [DOCX-JS.md](references/DOCX-JS.md).

## Editing documents
For simple edits, modify the XML directly.

**For tracked changes**: See [REDLINING.md](references/REDLINING.md)
```

## Skill Creation Process

Skill creation involves these steps:

1. Understand the skill with concrete examples
2. Plan reusable skill contents (scripts, references, assets)
3. Initialize the skill (run init_skill.py)
4. Edit the skill (implement resources and write SKILL.md)
5. Validate and test
6. Iterate based on real usage

### Step 1: Understanding with Concrete Examples

To create an effective skill, understand concrete examples of how the skill will be used:

- "What functionality should this skill support?"
- "Can you give some examples of how this skill would be used?"
- "What would a user say that should trigger this skill?"

Conclude this step when there is a clear sense of the functionality the skill should support.

### Step 2: Planning Reusable Contents

Analyze each example by:

1. Considering how to execute on the example from scratch
2. Identifying what scripts, references, and assets would be helpful

Examples:
- PDF rotation → `scripts/rotate_pdf.py`
- BigQuery queries → `references/schema.md` with table schemas
- Frontend boilerplate → `assets/hello-world/` template

### Step 3: Initialize the Skill

Run the initialization script:

```bash
python scripts/init_skill.py <skill-name> --path <output-directory> [--resources scripts,references,assets]
```

Examples:

```bash
python scripts/init_skill.py my-skill --path ~/.ouro/skills
python scripts/init_skill.py my-skill --path ~/.ouro/skills --resources scripts,references
```

### Step 4: Edit the Skill

#### Write the Frontmatter

```yaml
---
name: skill-name
description: What the skill does and when to use it. Include specific triggers/contexts. This is the primary triggering mechanism.
---
```

**Important**: Include all "when to use" information in the description, not in the body. The body is only loaded after triggering.

#### Write the Body

- Use imperative/infinitive form
- Include workflow steps
- Reference bundled resources with relative paths
- Keep it concise

### Step 5: Validate

Check that:
- SKILL.md has valid YAML frontmatter
- `name` and `description` are present
- Referenced files exist
- Scripts are executable

### Step 6: Iterate

After testing the skill:

1. Use the skill on real tasks
2. Notice struggles or inefficiencies
3. Update SKILL.md or bundled resources
4. Test again

## Skill Naming Conventions

- Use lowercase letters, digits, and hyphens only
- Prefer short, verb-led phrases that describe the action
- Namespace by tool when it improves clarity (e.g., `gh-address-comments`, `linear-address-issue`)
- Keep names under 64 characters

Examples:
- `pdf-editor` (what it edits)
- `gh-pr-review` (namespaced by tool)
- `frontend-webapp-builder` (descriptive)
