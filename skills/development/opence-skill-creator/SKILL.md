---
name: opence-skill-creator
description: Learn how to create effective project skills following opence conventions.
allowed-tools:
  - Read
  - Write
user-invocable: true
---

<!-- OPENCE:START -->
# opence-skill-creator

Learn how to create effective project skills following opence conventions.

## When to Create a Skill

Create a skill during the **compound phase** when you identify:
- Repeatable workflows that can be encoded as instructions
- Recurring pitfalls or manual checks that need documentation
- Domain-specific knowledge that will be useful in future changes

Don't create a skill for one-off solutions or highly specific edge cases.

## Quick Start

Use the CLI to create skills (recommended):

```bash
# Create a new skill
opence skill add api-testing --description "Guidelines for testing API endpoints"

# List all skills
opence skill list

# Show skill details
opence skill show api-testing
```

The CLI automatically:
- Creates proper directory structure
- Generates correct frontmatter for each tool
- Validates naming conventions
- Creates references/ and scripts/ directories

## Skill Structure

```
.claude/skills/my-skill/
├── SKILL.md           # Concise entry point (< 200 lines)
├── references/        # Detailed documentation
│   └── *.md          # Extended guides, examples
└── scripts/          # Reusable code
    └── *.sh, *.py    # Helper scripts
```

**SKILL.md**: Keep concise. Include "When to use", key steps, and pointers to references/.

**references/**: Put detailed docs, long examples, API references here.

**scripts/**: Add reusable code that users can copy or execute.

## Naming Rules

- **Format**: kebab-case (e.g., `api-testing`, `deploy-prod`)
- **Reserved**: Don't use `opence-` prefix (reserved for native skills)
- **Unique**: Check with `opence skill list` to avoid duplicates

**Valid**: `api-testing`, `error-recovery`, `db-migration`
**Invalid**: `api testing` (spaces), `api_testing` (underscores), `opence-custom` (reserved)

## Writing Effective Descriptions

The `description` field determines when Claude automatically loads your skill. Write it carefully.

**Good description patterns:**
- Start with an action verb: Explains, Creates, Reviews, Validates, Deploys, Fixes
- Include keywords users naturally say
- List specific trigger scenarios
- Be specific but not overly narrow

**Examples of good descriptions:**
```
"Explains code with visual diagrams and analogies. Use when explaining how code works, teaching about a codebase, or when the user asks 'how does this work?'"

"Fix a GitHub issue following our coding standards. Use when working on issues or bugs."

"Research a topic thoroughly using multiple sources. Use for deep dives or comprehensive analysis."
```

**Anti-patterns to avoid:**
- Too abstract: `"Helps with development tasks"` — Claude can't decide when to use it
- Too specific: `"Fix the login button CSS on mobile Safari 15.4"` — Won't match future requests
- Missing scenarios: `"API testing utility"` — No trigger context

**Trigger phrase checklist:**
- [ ] Does it include words users would naturally say?
- [ ] Would Claude know when to load it automatically?
- [ ] Is it distinct from other skills' triggers?

## Next Steps

After creating a skill:
1. Edit SKILL.md to add instructions and examples
2. Add detailed docs to references/ if needed
3. Add helper scripts to scripts/ if applicable
4. Test the skill in actual usage
5. Commit to version control

See references/ for detailed guidance on:
- [Skill evaluation checklist](references/skill-evaluation-checklist.md) — Four-question framework and "is it worth creating" criteria
- Directory structure and sizing
- Frontmatter formats for different tools
- Best practices and anti-patterns
<!-- OPENCE:END -->
