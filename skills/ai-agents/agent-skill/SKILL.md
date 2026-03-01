---
name: agent-skill
description: Create and structure Claude Code agent skills. Use when the user wants to create a new skill, write an agent skill, make a claude skill, or asks about SKILL.md files.
---

# Agent Skill Creator

Help users create well-structured Claude Code skills through an interactive process.

## Workflow

### Step 1: Ask about location first

Before anything else, ask where the skill should be created:

- **Personal** (`~/.claude/skills/`): Available across all projects for this user
- **Project** (`./.claude/skills/`): Shared with team via version control

Use the AskUserQuestion tool for this.

### Step 2: Gather requirements through questions

Ask about each of these in sequence:

1. **Skill name**: What should the skill be called?
   - Must be lowercase letters, numbers, and hyphens only
   - Maximum 64 characters
   - Examples: `code-review`, `api-docs`, `test-generator`

2. **Purpose**: What does this skill help with?
   - What specific task or capability does it provide?
   - What problem does it solve?

3. **Trigger phrases**: When should this skill activate?
   - What words would users say when they need this skill?
   - Be specific about scenarios (not vague like "helps with code")

4. **Key instructions**: What should Claude do when this skill is active?
   - Step-by-step process
   - Required outputs or formats
   - Constraints or rules to follow

5. **Tool restrictions**: Should the skill limit available tools?
   - Full access (default): No restrictions
   - Read-only: `Read, Grep, Glob`
   - Write-focused: `Read, Write, Edit, Glob, Grep`
   - Custom: Specify exact tools

6. **Complexity**: Will this skill need supporting files?
   - Simple: Single SKILL.md under 500 lines
   - Complex: Multiple files with progressive disclosure

### Step 3: Generate the skill

Create the skill file at the chosen location:
- Personal: `~/.claude/skills/{skill-name}/SKILL.md`
- Project: `./.claude/skills/{skill-name}/SKILL.md`

## Skill File Structure

### Required YAML frontmatter

```yaml
---
name: skill-name-here
description: Clear description of what it does and when to use it. Include trigger keywords users would say. Max 1024 chars.
allowed-tools: Tool1, Tool2  # Optional - omit for full access
model: claude-sonnet-4-20250514  # Optional - only if specific model needed
---
```

### Instruction sections

After frontmatter, include:

1. **Title**: `# Skill Name`
2. **Overview**: Brief explanation of what the skill does
3. **Instructions**: Clear, numbered steps for Claude to follow
4. **Examples** (optional): Concrete usage examples
5. **References** (if multi-file): Links to supporting files

## Writing effective descriptions

The description determines when Claude activates the skill. It uses semantic matching.

Bad: "Helps with documents"
Good: "Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDFs or when the user mentions document extraction."

Include:
- Specific capabilities (verbs: extract, generate, review, format)
- Trigger keywords users would naturally say
- Clear "use when" guidance

## Multi-file structure

For complex skills, keep SKILL.md under 500 lines and link to supporting files:

```
skill-name/
├── SKILL.md         # Overview and navigation
├── reference.md     # Detailed documentation
├── examples.md      # Usage examples
└── scripts/         # Utility scripts (executed, not loaded)
    └── helper.py
```

Key rules:
- Keep references one level deep (no A -> B -> C chains)
- Scripts execute without loading into context
- Claude reads supporting files only when needed

## Example skills

See [examples.md](examples.md) for reference examples of well-structured skills.

## Quality checklist

Before finalizing, verify:

- [ ] Name is lowercase with hyphens only
- [ ] Description includes trigger keywords
- [ ] Description explains when to use it
- [ ] Instructions are clear and actionable
- [ ] Tool restrictions match the skill's purpose
- [ ] SKILL.md stays under 500 lines
- [ ] Supporting files are one level deep only
