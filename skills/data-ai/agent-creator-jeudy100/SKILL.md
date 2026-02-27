---
name: agent-creator
description: Guide for creating effective agents. This skill should be used when users want to create a new agent (or update an existing agent) that handles complex, multi-step tasks autonomously.
---

# Agent Creator

This skill provides guidance for creating effective agents.

## About Agents

Agents are autonomous workers that handle complex, multi-step tasks. They run in isolated contexts via the Task tool, can use multiple skills, and operate with minimal user intervention. Think of them as specialized team members—they transform Claude from a general-purpose assistant into a focused expert for specific workflows.

### What Agents Provide

1. **Autonomous workflows** - Multi-step procedures that run without constant user input
2. **Tool orchestration** - Coordinated use of multiple tools to achieve goals
3. **Decision making** - Logic to handle branches, errors, and edge cases
4. **Structured output** - Consistent, predictable result formats

### Agents vs Skills

| Aspect | Agent | Skill |
|--------|-------|-------|
| Invocation | Task tool | Slash command or inline |
| Context | Isolated subprocess | Main conversation |
| Autonomy | High (runs independently) | Low (guided execution) |
| Complexity | Multi-step workflows | Single-purpose operations |
| State | Own context window | Shared context |

## Core Principles

### Concise is Key

The context window is a public good. Only include what Claude needs to execute the workflow. Challenge each section: "Does this instruction justify its token cost?"

Prefer concise examples over verbose explanations.

### Autonomy with Guardrails

Agents should work independently but know when to ask users for decisions:
- Make routine decisions automatically
- Escalate ambiguous situations with clear options
- Never make destructive changes without confirmation

### Composability

Agents should leverage existing skills rather than reimplementing functionality. Reference skills in "Skills to Use" and call them during execution.

## Anatomy of an Agent

Every agent is a single markdown file in `.claude/agents/`:

```
.claude/agents/
└── agent-name.md
```

### Required Structure

```markdown
---
name: [agent-name]
description: [1-2 sentence description of what this agent does]
tools: [Tool1, Tool2, ...]
skills:
  - [skill-name-1]
  - [skill-name-2]
---

# [Agent Name] Agent

[1-2 sentence description of what this agent does]

## Tools Available
- [Tool]: [What it's used for]

## Skills to Use
- [skill-name]: [When to use it]
- pattern-evaluator (agent): Evaluate and persist reusable patterns discovered during execution

## Instructions

### Step 0: Discover Project Context
[Context discovery using Explore agent]

### Step 1: [First Action]
[Clear instructions]

### Step N: [Final Action]
[Clear instructions]

### Step N+1 (Final): Evaluate Reusable Patterns

Run the **pattern-evaluator** agent to assess whether any reusable patterns (rules, skills, or agents) were discovered during this session and should be persisted.

## Output Format
[Template showing expected output structure]

## Error Handling
[User-facing questions with options for each error scenario]

## Important Notes
- [Key behaviors and constraints]
```

### Section Guidelines

**YAML Frontmatter** (required): Must include `name`, `description`, and `tools` fields. The `name` should be lowercase with hyphens. The `description` is used by Claude to determine when to delegate to this agent. The `tools` field lists the tools the agent can use. Optional fields:
- `skills`: List of skill names to preload into the agent's context (e.g., `[run-tests, lint-code]`). Use this to give the agent access to skills referenced in "Skills to Use".
- `model`: Override the default model (e.g., `sonnet`, `opus`, `haiku`).
- `maxTurns`: Limit the number of agentic turns.
- `permissionMode`: Set tool permission behavior.

**Header**: 1-2 sentences describing the agent's purpose and expertise.

**Tools Available**: List each tool with its purpose. Common tools:
- Bash: Execute commands
- Read: Read files
- Write: Create/overwrite files
- Edit: Modify files
- Grep: Search file contents
- Glob: Find files by pattern

**Skills to Use**: Reference existing skills. Check `.claude/skills/*/SKILL.md` for available skills.

**Instructions**: Numbered steps starting with Step 0. Each step should be atomic and specific.

**Output Format**: Markdown template showing the structure of results.

**Error Handling**: Questions with 3-4 concrete options (never open-ended).

**Important Notes**: Constraints, behaviors, and edge cases.

### Step 0: Context Discovery

Every agent starts with context discovery:

```markdown
### Step 0: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for [task]. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: [keywords]
> 3. **Project Type** - Detect from package.json, pyproject.toml, go.mod, etc.
>
> Return: [What to extract]

From the Explore results, extract:
- [Item 1]
- [Item 2]
```

### Error Handling Pattern

Always offer concrete options, never open-ended questions:

```markdown
### [Scenario Name]

\`\`\`
Question: "[Clear description of what happened]"
Options:
  - [Action 1]
  - [Action 2]
  - [Alternative approach]
  - Cancel
\`\`\`
```

## Agent Creation Process

### Step 1: Understand the Agent's Purpose

Ask clarifying questions:
- "What task should this agent automate?"
- "What tools will it need?"
- "What skills should it leverage?"
- "What are the expected inputs and outputs?"
- "What can go wrong?"

### Step 2: Plan the Structure

Outline:
1. **Workflow steps** - Context discovery (Step 0) + main task steps
2. **Tools needed** - Read, Write, Bash, Grep, Glob, etc.
3. **Skills to leverage** - Search `.claude/skills/*/SKILL.md`
4. **Error scenarios** - What can fail at each step

### Step 3: Write the Agent

Create `.claude/agents/[agent-name].md`:

1. YAML frontmatter with name, description, tools, and skills
2. Header with concise description
3. Tools Available with purposes
4. Skills to Use with triggers
5. Instructions starting with Step 0
6. Output Format with template
7. Error Handling for each failure
8. Important Notes for constraints

**Writing Guidelines:**
- Use imperative form ("Run tests", not "Running tests")
- Be specific ("use Grep to search for X" not "check the files")
- Keep steps atomic and focused
- Include examples where helpful

### Step 4: Validate

Check for:
- [ ] YAML frontmatter with name, description, tools, and skills
- [ ] Header with clear description
- [ ] Tools Available section
- [ ] Skills to Use section
- [ ] Step 0 for context discovery
- [ ] Numbered, atomic steps
- [ ] Output Format with template
- [ ] Error Handling with options (not open-ended)
- [ ] Important Notes

### Step 5: Iterate

1. Test the agent on real tasks
2. Notice struggles or inefficiencies
3. Update instructions or error handling
4. Test again

## Example Agent

```markdown
---
name: database-migration
description: Creates, validates, and runs database migrations safely
tools: Bash, Read, Write, Grep, Glob, Task
skills:
  - lint-code
  - review-changes
---

# Database Migration Agent

You are a database migration specialist. Your job is to create, validate, and run database migrations safely.

## Tools Available
- Bash: Execute migration commands
- Read: Read migration files and schemas
- Write: Create new migration files
- Grep: Search for schema references
- Glob: Find migration files

## Skills to Use
- lint-code: Validate migration SQL syntax
- review-changes: Review migration before execution

## Instructions

### Step 0: Discover Project Context

Use the **Explore** agent:

**Explore Prompt:**
> Discover database migration context. Find:
> 1. Root CLAUDE.md for migration guidelines
> 2. CLAUDE.md files with keywords: database, migration, schema, sql
> 3. Migration tool from package.json, requirements.txt, etc.
>
> Return: Migration tool, naming conventions, database type, migration directory

### Step 1: Analyze Current State

1. Use Glob to find existing migrations: `**/migrations/*.sql`
2. Read the most recent migration to understand naming
3. Identify the database type and schema

### Step 2: Create Migration

1. Generate migration filename following conventions
2. Write the migration file with up/down sections
3. Use lint-code skill to validate syntax

### Step 3: Validate Migration

1. Check for destructive operations (DROP, TRUNCATE)
2. Verify foreign key references exist
3. Use review-changes skill for final review

### Step 4: Execute (if requested)

1. Confirm with user before running
2. Execute migration command
3. Report success or failure with details

## Output Format

## Migration Report

**Migration**: [filename]
**Type**: [create table / alter table / etc.]
**Status**: [Created / Validated / Executed]

### Changes
- [change 1]
- [change 2]

### Warnings (if any)
- [warning]

## Error Handling

### Migration Tool Not Detected

Question: "Could not detect the migration tool. Which one should I use?"
Options:
  - Prisma (Node.js)
  - Alembic (Python)
  - golang-migrate (Go)
  - Specify custom tool

### Destructive Operation Detected

Question: "This migration contains destructive operations (DROP/TRUNCATE). How should I proceed?"
Options:
  - Add backup step before migration
  - Continue without backup
  - Modify migration to be non-destructive
  - Cancel

### Step 5 (Final): Evaluate Reusable Patterns

Run the **pattern-evaluator** agent to assess whether any reusable patterns (rules, skills, or agents) were discovered during this session and should be persisted.

## Important Notes

- Never run migrations without user confirmation
- Always include rollback (down) migration
- Check for data loss before destructive operations
- Follow project naming conventions
```

## What NOT to Include

- README.md or other documentation files
- Excessive explanations (Claude is smart)
- Duplicate information from CLAUDE.md
- Overly verbose output formats
