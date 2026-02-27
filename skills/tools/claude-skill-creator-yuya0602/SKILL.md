---
name: claude-skill-creator
description: Guide for creating effective Claude Code skills with proper YAML frontmatter, directory structure, and best practices. Use when creating new skills, updating existing skills, or learning about skill development.
---

# Claude Skill Creator Guide

This guide helps you create well-structured, effective skills for Claude Code that extend capabilities with specialized knowledge, workflows, or tool integrations.

## When to Use This Skill

Use this skill when:
- Creating a new skill from scratch
- Updating an existing skill
- Learning about skill structure and best practices
- Troubleshooting why a skill isn't being activated
- Converting documentation into a skill format

## What Are Skills?

Skills extend Claude's capabilities through organized folders containing instructions and resources. They are **model-invoked**—Claude autonomously decides when to use them based on request context, unlike slash commands which require explicit user activation.

## Directory Structure

Skills can be stored in three locations:

1. **Personal Skills**: `~/.claude/skills/skill-name/`
   - Available across all projects for the user
   - Perfect for personal workflows and preferences

2. **Project Skills**: `.claude/skills/skill-name/`
   - Specific to a project, committed to git
   - Shared with team members automatically

3. **Plugin Skills**: Bundled with installed plugins
   - Distributed as packages

## Creating a New Skill

### Step 1: Create the Directory Structure

```bash
# For personal skills
mkdir -p ~/.claude/skills/my-skill-name

# For project skills
mkdir -p .claude/skills/my-skill-name
```

### Step 2: Create SKILL.md with Required Frontmatter

Every skill MUST have a `SKILL.md` file with YAML frontmatter:

```yaml
---
name: skill-identifier
description: Brief description of what this skill does and when to use it
---

# Skill Name

[Your skill content here]
```

**Critical Requirements:**

- **name**:
  - Lowercase letters, numbers, hyphens only
  - Maximum 64 characters
  - Example: `comprehensive-testing`, `api-docs-writer`, `db-migration-helper`

- **description**:
  - Must describe BOTH what the skill does AND when to use it
  - Maximum 1024 characters
  - Include trigger keywords users would mention
  - Be specific, not generic

**Good Description Examples:**

```yaml
# ✅ GOOD: Specific with clear triggers
description: Implement comprehensive tests with test design tables, equivalence partitioning, boundary value analysis, and 100% branch coverage. Use when writing tests, adding test cases, or improving test coverage for React Native/Expo TypeScript code with Jest.

# ✅ GOOD: Clear functionality and use case
description: Analyze Excel spreadsheets, generate pivot tables, create charts from CSV data. Use when working with Excel files, spreadsheet analysis, or data visualization tasks.

# ❌ BAD: Too vague
description: Helps with data processing

# ❌ BAD: Missing when to use
description: This skill handles database migrations and schema changes
```

### Step 3: Write the Skill Content

Follow this recommended structure:

```markdown
---
name: my-skill-name
description: [What it does and when to use it]
---

# Skill Title

Brief introduction explaining the skill's purpose.

## When to Use This Skill

Explicitly list scenarios:
- Use case 1
- Use case 2
- Use case 3

## Core Concepts / Philosophy

Explain the underlying principles or approach.

## Instructions

Provide clear, step-by-step guidance:

1. **Step 1**: Do this
2. **Step 2**: Then do this
3. **Step 3**: Finally do this

## Examples

Show concrete, practical examples:

### Example 1: [Scenario]

```[language]
[code or content example]
```

### Example 2: [Another Scenario]

```[language]
[code or content example]
```

## Best Practices

- Practice 1
- Practice 2
- Practice 3

## Common Patterns

[Common use cases with templates]

## Troubleshooting

Common issues and solutions:

**Issue**: [Problem]
**Solution**: [How to fix]

## AI Assistant Instructions

Specific guidance for Claude on how to use this skill:

When invoked, you should:
1. [Instruction 1]
2. [Instruction 2]
3. [Instruction 3]

Always/Never:
- Always do X
- Never do Y

## Additional Resources

- [Link to documentation]
- [Link to related tools]
```

## Optional: Tool Restrictions

Use `allowed-tools` to limit Claude's capabilities when the skill is active:

```yaml
---
name: safe-file-reader
description: Safely read and analyze files without making modifications
allowed-tools: Read, Grep, Glob
---
```

This restricts Claude to only specified tools, useful for:
- Read-only operations
- Safety-critical workflows
- Preventing accidental modifications

## Optional: Supporting Files

Organize additional resources alongside SKILL.md:

```
my-skill-name/
├── SKILL.md              # Main skill file (required)
├── reference.md          # Additional reference documentation
├── templates/
│   ├── template1.txt
│   └── template2.txt
└── examples/
    ├── example1.ts
    └── example2.ts
```

Reference these files from SKILL.md:

```markdown
See [reference documentation](reference.md) for more details.

Use this [template](templates/template1.txt) as a starting point.
```

## Best Practices for Skill Creation

### 1. Keep Skills Focused

**✅ DO**: One skill = one capability
- `api-docs-writer`: Generate API documentation
- `test-strategy`: Implement comprehensive tests
- `db-migration`: Handle database schema changes

**❌ DON'T**: Create broad, multi-purpose skills
- `developer-helper`: Does everything (too vague)
- `backend-tools`: Mixed unrelated capabilities

### 2. Write Trigger-Rich Descriptions

Include keywords users would naturally use:

```yaml
# ✅ GOOD: Rich with triggers
description: Generate OpenAPI/Swagger documentation from Express routes, FastAPI endpoints, or GraphQL schemas. Use when documenting APIs, creating API specs, or working with OpenAPI, Swagger, REST, or GraphQL.

# ❌ BAD: Missing triggers
description: Helps with API documentation
```

### 3. Provide Concrete Examples

Users and Claude learn best from examples:

```markdown
## Example: Creating a REST API Endpoint

```typescript
// Given this Express route
app.get('/users/:id', async (req, res) => {
  const user = await db.getUser(req.params.id);
  res.json(user);
});

// Generate this OpenAPI spec
paths:
  /users/{id}:
    get:
      summary: Get user by ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
```
```

### 4. Be Explicit About Workflow

Tell Claude exactly what to do:

```markdown
## AI Assistant Instructions

When this skill is activated:

1. **First**: Analyze the codebase structure
2. **Then**: Ask clarifying questions if needed
3. **Next**: Generate the initial version
4. **Finally**: Validate and test the output

Always:
- Use TypeScript for type safety
- Include error handling
- Add JSDoc comments

Never:
- Skip validation steps
- Generate code without examples
- Assume user preferences
```

### 5. Test Your Skills

After creating a skill, test it:

1. **Test Activation**: Does it trigger with expected keywords?
2. **Test Instructions**: Does Claude follow the workflow correctly?
3. **Test Examples**: Are they clear and helpful?
4. **Test Edge Cases**: Does it handle unusual scenarios?

### 6. Context Window is a Shared Resource

Skills consume context tokens when activated. Write efficiently:

**✅ DO**:
- Assume Claude's base knowledge (don't explain programming basics)
- Include only task-specific context
- Keep SKILL.md under 500 lines
- Use reference files for extensive documentation

**❌ DON'T**:
- Explain concepts Claude already knows
- Include unnecessary background information
- Create monolithic skill files

### 7. Test Across Different Models

Skills that work well with Claude Opus may need more detail for Haiku:

- **Opus**: Can handle more implicit instructions
- **Sonnet**: Good balance of capability and speed
- **Haiku**: Needs more explicit, step-by-step guidance

Test your skill with all models you plan to support.

### 8. Use Progressive Disclosure for Large Skills

For complex skills with extensive documentation:

**Structure**:
```
my-skill-name/
├── SKILL.md              # Core instructions (< 500 lines)
├── reference.md          # Detailed reference (loaded on-demand)
├── examples.md           # Extended examples (loaded on-demand)
└── templates/            # Reusable templates
```

**In SKILL.md, reference other files**:
```markdown
For detailed API reference, see [reference.md](reference.md).
More examples available in [examples.md](examples.md).
```

Claude will only load these files when needed, saving context tokens.

### 9. Provide Utility Scripts for Complex Tasks

**Checklisted Workflows**: Include ordered steps Claude can track progress through

**Pre-written Scripts**: For deterministic operations, provide ready-to-use scripts rather than having Claude generate them
- More reliable
- Saves tokens
- Faster execution

**Choose the Right Language**:

Use **Shell Scripts (.sh)** for:
- Simple, single-purpose operations
- File handling and command chaining
- Logging, notifications, quick validation
- Operations with minimal logic (5 lines or less)

Use **Python Scripts (.py)** for:
- Complex logic and conditional operations
- Data processing and transformation
- Cross-platform compatibility (especially if Windows users exist)
- Reusable modules and libraries

**Cross-Platform Considerations**:
- Shell scripts require WSL/Git Bash on Windows
- Python offers better cross-platform support for team-shared skills
- Use POSIX-compliant commands if you choose shell scripts

**Example**:
```bash
scripts/
├── setup.sh          # Shell: Simple environment setup
├── validate.py       # Python: Complex validation logic
├── format.py         # Python: Data transformation
└── notify.sh         # Shell: Simple notifications
```

Reference from SKILL.md:
```markdown
Run validation: `python scripts/validate.py`
Run setup: `bash scripts/setup.sh`
```

## Common Skill Patterns

### Pattern 1: Code Generation Skill

```yaml
---
name: component-generator
description: Generate React/Vue/Angular components with TypeScript, tests, and stories. Use when creating new components, scaffolding UI elements, or building component libraries.
---

# Component Generator

## Instructions

1. Ask user for component type (React/Vue/Angular)
2. Request component name and props
3. Generate:
   - Component file with TypeScript
   - Test file with comprehensive tests
   - Storybook story file
4. Follow project conventions from existing components
```

### Pattern 2: Analysis Skill

```yaml
---
name: code-complexity-analyzer
description: Analyze code complexity, identify refactoring opportunities, calculate cyclomatic complexity, and suggest improvements. Use when reviewing code, planning refactoring, or improving code quality.
---

# Code Complexity Analyzer

## Instructions

1. Scan provided code files
2. Calculate metrics:
   - Cyclomatic complexity
   - Function length
   - Nesting depth
3. Identify issues:
   - Functions > 50 lines
   - Complexity > 10
   - Deep nesting > 3 levels
4. Suggest specific refactorings
```

### Pattern 3: Documentation Skill

```yaml
---
name: readme-generator
description: Generate comprehensive README files with installation, usage, API docs, and examples. Use when creating new projects, improving documentation, or standardizing README format.
---

# README Generator

## Instructions

1. Analyze project structure and package.json
2. Generate sections:
   - Project description
   - Installation steps
   - Usage examples
   - API documentation
   - Contributing guidelines
3. Include badges for CI, coverage, license
4. Add table of contents for long READMEs
```

## Troubleshooting

### Issue: Claude Doesn't Use the Skill

**Possible Causes:**

1. **Description lacks trigger keywords**
   - ✅ Fix: Add specific terms users would mention
   - Example: Add "Jest", "testing", "test coverage" to description

2. **Skill name has invalid characters**
   - ✅ Fix: Use only lowercase, numbers, hyphens
   - Example: Change `My_Skill_Name` to `my-skill-name`

3. **YAML frontmatter is malformed**
   - ✅ Fix: Validate YAML syntax
   - Check for proper `---` delimiters
   - Ensure no tabs (use spaces)

4. **File is not named SKILL.md**
   - ✅ Fix: Rename to `SKILL.md` (exact case)

5. **Directory structure is wrong**
   - ✅ Fix: Ensure path is `~/.claude/skills/skill-name/SKILL.md`

### Issue: Skill Activates at Wrong Times

**Possible Causes:**

1. **Description is too broad**
   - ✅ Fix: Make description more specific
   - Example: Instead of "helps with files", use "analyze CSV files and generate reports"

2. **Trigger keywords overlap with other skills**
   - ✅ Fix: Use more specific, unique keywords

### Issue: Skill Doesn't Follow Instructions

**Possible Causes:**

1. **Instructions are unclear or ambiguous**
   - ✅ Fix: Use numbered steps, be explicit

2. **Examples don't match instructions**
   - ✅ Fix: Ensure examples demonstrate the workflow

3. **Missing AI Assistant Instructions section**
   - ✅ Fix: Add explicit guidance for Claude

## Sharing Skills with Teams

### For Project Skills

1. Create skill in `.claude/skills/skill-name/`
2. Commit to git:
   ```bash
   git add .claude/skills/skill-name/
   git commit -m "feat: add [skill-name] skill"
   git push
   ```
3. Team members get the skill automatically on `git pull`

### For Plugin Distribution

1. Package skill as npm module
2. Include installation instructions
3. Document required dependencies
4. Provide usage examples

## Quick Reference: Skill Checklist

When creating a skill, ensure:

- [ ] Directory created: `~/.claude/skills/skill-name/` or `.claude/skills/skill-name/`
- [ ] File named exactly `SKILL.md`
- [ ] YAML frontmatter present with `---` delimiters
- [ ] `name` field: lowercase, hyphens, <64 chars
- [ ] `description` field: describes what + when, <1024 chars
- [ ] Description includes trigger keywords
- [ ] Clear "When to Use This Skill" section
- [ ] Step-by-step instructions provided
- [ ] Concrete examples included
- [ ] Best practices documented
- [ ] AI Assistant Instructions added
- [ ] Tested with realistic scenarios
- [ ] No typos or formatting issues

## Example: Complete Skill Template

```markdown
---
name: example-skill
description: [What this does] and [specific use case]. Use when [trigger scenario 1], [trigger scenario 2], or [working with keyword1, keyword2].
---

# Example Skill

Brief introduction to the skill's purpose and value.

## When to Use This Skill

- Scenario 1
- Scenario 2
- Scenario 3

## Core Concepts

Explain the underlying approach or methodology.

## Instructions

1. **Step 1**: First action
2. **Step 2**: Second action
3. **Step 3**: Final action

## Examples

### Example 1: Common Use Case

```typescript
// Code example here
```

### Example 2: Advanced Use Case

```typescript
// Another example
```

## Best Practices

- Practice 1
- Practice 2

## Common Patterns

Pattern templates or reusable snippets.

## Troubleshooting

**Issue**: Problem description
**Solution**: How to resolve

## AI Assistant Instructions

When this skill is activated:

1. Always do X
2. Never do Y
3. Follow this workflow: [steps]

## Additional Resources

- [Documentation link]
- [Tool link]
```

---

## Additional Tips for Effective Skills

1. **Start Simple**: Begin with basic functionality, iterate based on usage
2. **Use Real Examples**: Draw from actual use cases, not hypothetical ones
3. **Be Specific**: Avoid vague language like "helps with" or "handles"
4. **Document Assumptions**: Clarify prerequisites, dependencies, or required knowledge
5. **Iterate Based on Feedback**: Skills improve with real-world usage—update accordingly
6. **Consider Performance**: Every token in your skill uses context window space
7. **Write for Autonomous Use**: Claude should be able to use the skill without asking clarifying questions