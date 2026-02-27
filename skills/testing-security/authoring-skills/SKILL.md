---
name: authoring-skills
description: Guides creation of effective Claude skills with proper structure, naming, and progressive disclosure. Use when creating new skills, improving existing SKILL.md files, reviewing skill quality, or when the user mentions writing skills, skill authoring, or SKILL.md.
---

# Skill Authoring

Create skills that Claude can discover and use effectively.

## Core Principles

### Conciseness is critical

The context window is shared. Only add what Claude doesn't already know.

Challenge each piece of content:
- "Does Claude really need this explanation?"
- "Can I assume Claude knows this?"
- "Does this justify its token cost?"

**Good** (~50 tokens):
```markdown
## Extract PDF text
Use pdfplumber:
\`\`\`python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
\`\`\`
```

**Bad** (~150 tokens): Explaining what PDFs are and why pdfplumber is recommended.

### Degrees of freedom

Match specificity to task fragility:

| Freedom | When to use | Example |
|---------|-------------|---------|
| **High** (text instructions) | Multiple valid approaches, context-dependent | Code review guidelines |
| **Medium** (pseudocode/parameters) | Preferred pattern exists, some variation OK | Report templates |
| **Low** (exact scripts) | Fragile operations, consistency critical | Database migrations |

### Test with target models

- **Haiku**: Does the skill provide enough guidance?
- **Sonnet**: Is it clear and efficient?
- **Opus**: Does it avoid over-explaining?

## Skill Structure

### Frontmatter requirements

```yaml
---
name: processing-pdfs
description: Extracts text and tables from PDF files. Use when working with PDFs or document extraction.
---
```

**name** (required):
- Max 64 characters
- Lowercase letters, numbers, hyphens only
- No reserved words: "anthropic", "claude"

**description** (required):
- Max 1024 characters
- Third person ("Processes files" not "I can process files")
- Include BOTH what it does AND when to use it
- Include key trigger terms for discovery

### Naming conventions

Use gerund form (verb + -ing):
- `processing-pdfs`
- `analyzing-spreadsheets`
- `managing-databases`

Avoid: `helper`, `utils`, `tools`, `documents`

### Progressive disclosure

Keep SKILL.md under 500 lines. Split content into reference files:

```
my-skill/
├── SKILL.md              # Overview (loaded when triggered)
├── FORMS.md              # Detailed guide (loaded as needed)
├── reference.md          # API reference (loaded as needed)
└── scripts/
    └── validate.py       # Utility script (executed, not loaded)
```

**In SKILL.md**: Quick start, navigation to details
**In reference files**: Complete guides, API docs, examples

Keep references one level deep. Avoid: SKILL.md → advanced.md → details.md

See [reference/structure.md](reference/structure.md) for detailed organization patterns.

## Writing Descriptions

Descriptions enable skill discovery. Be specific:

**Good**:
```yaml
description: Analyze Excel spreadsheets, create pivot tables, generate charts. Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.
```

**Bad**:
```yaml
description: Helps with documents
```

## Common Patterns

### Template pattern

Provide output format templates. Match strictness to requirements:

**Strict** (API responses): "ALWAYS use this exact structure"
**Flexible** (reports): "Sensible default, adapt as needed"

### Examples pattern

For quality-dependent output, provide input/output pairs:

```markdown
**Example 1:**
Input: Added user authentication with JWT tokens
Output: feat(auth): implement JWT-based authentication
```

### Workflow pattern

Break complex tasks into clear steps with checklists:

```markdown
Task Progress:
- [ ] Step 1: Analyze input
- [ ] Step 2: Validate mapping
- [ ] Step 3: Execute changes
- [ ] Step 4: Verify output
```

See [reference/patterns.md](reference/patterns.md) for complete pattern examples.

## Feedback Loops

**Pattern**: Run validator → fix errors → repeat

```markdown
1. Make edits
2. Validate: `python scripts/validate.py`
3. If validation fails: fix and re-validate
4. Only proceed when validation passes
```

See [reference/patterns.md](reference/patterns.md) for workflow details.

## Anti-Patterns

- **Windows paths**: Use `scripts/helper.py` not `scripts\helper.py`
- **Too many options**: Provide one default, mention alternatives only when needed
- **Time-sensitive info**: Use "old patterns" sections instead of dates
- **Inconsistent terminology**: Pick one term, use it throughout
- **Deeply nested references**: Keep one level deep from SKILL.md

## Skills with Executable Code

For skills with scripts:

- **Handle errors explicitly** in scripts (don't punt to Claude)
- **Document constants** (no magic numbers)
- **List dependencies** and verify availability
- **Make execution intent clear**: "Run `script.py`" vs "See `script.py` for algorithm"
- **Create verifiable intermediate outputs** for complex operations

See [reference/advanced.md](reference/advanced.md) for executable code patterns.

## Evaluation

Build evaluations BEFORE writing documentation:

1. Run Claude on tasks without a skill - document failures
2. Create 3+ evaluation scenarios testing these gaps
3. Measure baseline performance
4. Write minimal instructions to pass evaluations
5. Iterate based on results

### Iterative development

Work with Claude to develop skills:
1. Complete a task with Claude, noting context you provide
2. Ask Claude to create a skill capturing the pattern
3. Review for conciseness
4. Test with fresh Claude instance
5. Iterate based on observed behavior

See [reference/checklist.md](reference/checklist.md) for the complete validation checklist.
