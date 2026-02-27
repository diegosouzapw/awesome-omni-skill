---
name: skill-authoring
description: Create or update Claude Agent Skills. Use this when users want to build a new skill, scaffold a skill folder, improve an existing skill, or validate structure.
allowed-tools: "Read,Write,Edit,Bash,Glob,Grep"
---

## About Skills

Skills are modular, self-contained packages that extend Claude's capabilities with specialized knowledge, workflows, and tools. They transform Claude from a general-purpose agent into a specialized agent equipped with procedural knowledge.

### What Skills Provide

1. Specialized workflows - Multi-step procedures for specific domains
2. Tool integrations - Instructions for working with specific file formats or APIs
3. Domain expertise - Company-specific knowledge, schemas, business logic
4. Bundled resources - Scripts, references, and assets for complex tasks

## When to use

- "Build me a new skill for X"
- "Create a skill that does Y"
- "Validate my skill structure"
- "Improve this existing skill"
- "What sections should a skill have?"

## When NOT to use

- General coding tasks (use other skills or direct Claude)
- Questions about Claude Code itself (use claude-code-guide)
- Simple file operations that don't need a skill workflow

## Core Principles

### Concise is Key

The context window is a public good. Challenge each piece of information: "Does Claude really need this?" Prefer concise examples over verbose explanations.

### Degrees of Freedom

Match specificity to task fragility:

| Level | Use When |
|-------|----------|
| **High freedom** | Multiple valid approaches, context-dependent decisions |
| **Medium freedom** | Preferred pattern exists, some variation acceptable |
| **Low freedom** | Fragile operations, consistency critical |

Think of Claude exploring a path: a narrow bridge needs guardrails (low freedom), an open field allows many routes (high freedom).

## Anatomy of a Skill

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name + description)
│   └── Markdown instructions
├── scripts/     - Executable code (Python/Bash)
├── references/  - Documentation loaded as needed
└── assets/      - Templates, binaries for output
```

### SKILL.md Frontmatter

Only two required fields:

- `name`: The skill identifier (lowercase, hyphens)
- `description**: Triggers + purpose. Include "Use this when..." with specific conditions.

Do not add `license` or other fields.

### When to Use Each Resource Type

- **scripts/**: Deterministic operations, repeatedly rewritten code
- **references/**: Long docs, schemas, APIs, patterns Claude references
- **assets/**: Templates, icons, boilerplate copied to output

## Progressive Disclosure

Keep SKILL.md under 500 lines. Move details to references:

```
# Quick start
[essential workflow]

# Advanced
- Feature A: See [REFERENCES/A.md](REFERENCES/A.md)
- Feature B: See [REFERENCES/B.md](REFERENCES/B.md)
```

## Skill Creation Workflow

### Step 1 — Capture Requirements

See [SKILLSPEC.md](references/skill_spec.md) for the format.

### Step 2 — Plan Resources

For each example, determine:

- Does it need a script? (deterministic, repeated)
- Does it need a reference? (long docs, schemas)
- Does it need assets? (templates, boilerplate)

### Step 3 — Initialize

```bash
python "{baseDir}/scripts/init_skill.py" <skill-name> --path <output-dir>
```

### Step 4 — Fill In

1. Replace generated SKILL.md with final version
2. Add scripts to `scripts/`
3. Add docs to `references/`
4. Add assets to `assets/`
5. Delete unused sample files

### Step 5 — Gap Analysis

Verify all sections exist:

| Section | Required |
|---------|----------|
| Description | States when to use |
| When to use | 3-8 triggers |
| When NOT to use | Recommended |
| Workflow | Numbered steps |
| Tool usage | Scripts + references documented |
| Failure handling | Explicit table/list |
| Examples | Concrete walkthroughs |
| Acceptance tests | 5+ test cases |

Use [CHECKLIST.md](references/checklist.md) for full validation.

### Step 6 — Precision Analysis

Check trigger quality:

- **Overlap**: Do triggers match other common skills?
- **Specificity**: Are triggers specific enough?
- **Differentiation**: Document distinctions if overlap exists

### Step 7 — Validate

```bash
python "{baseDir}/scripts/quick_validate.py" <skill-dir>
```

## Updating Existing Skills

1. Run gap analysis against [CHECKLIST.md](references/checklist.md)
2. Run precision analysis on triggers
3. Apply fixes:
   - Improve description/triggers
   - Add missing failure handling
   - Add "When NOT to use"
   - Move long content to references/
   - Add scripts for repeated steps
4. Re-validate

## Failure Modes

| Failure | Response |
|---------|----------|
| Missing `name` or `description` | Validation fails; add required fields |
| Invalid skill name format | Use lowercase, 2-64 chars, `a-z0-9_-` |
| Placeholders (TODO) in SKILL.md | Replace with actual content before validating |
| {baseDir} reference not found | Verify path exists relative to skill folder |
| Validation errors | Fix all ERRORs before validating |

## Examples

### Creating a New Skill

User: "Build me a skill to rotate PDFs"

Assistant uses this workflow:
1. Captures requirements (rotate PDF, 3 triggers)
2. Plans resources (scripts/rotate_pdf.py)
3. Runs init_skill.py
4. Fills in SKILL.md with PDF-specific workflow
5. Adds rotate_pdf.py script
6. Validates

### Updating an Existing Skill

User: "This skill is missing failure handling"

Assistant:
1. Runs gap analysis → finds missing "Failure handling"
2. Adds failure modes table
3. Re-validates

## Acceptance Tests

| Prompt | Expected Behavior |
|--------|-------------------|
| "Build a new skill called pdf-editor" | Creates skill scaffold, runs init script |
| "Create a skill for image processing" | Follows full workflow, validates skill |
| "Validate my skill at /path/to/skill" | Runs quick_validate.py, reports issues |
| "What sections should a skill have?" | References checklist.md |
| "Help me improve my existing skill" | Runs gap analysis, suggests fixes |
| "Show me the skill spec format" | References skill_spec.md |
| "Why isn't my skill triggering?" | Checks trigger specificity, suggests improvements |
