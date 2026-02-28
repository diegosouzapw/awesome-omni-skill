---
name: skill-creator
description: Create new Agent Skills with proper structure, templates, and best practices. Use when building custom skills for Claude Code, GitHub Copilot, or other Agent Skills-compatible tools.
license: MIT
metadata:
  author: agent-kit
  version: "1.1.0"
---

# Skill Creator

Create well-structured Agent Skills that work across Claude Code, GitHub Copilot, VS Code, and OpenAI Codex.

## What is a Skill?

A skill is a directory containing:
- `SKILL.md` - Instructions with YAML frontmatter
- `assets/` - Templates, data files (optional)
- `references/` - Additional documentation (optional)
- `scripts/` - Executable code (optional)

Skills extend AI agent capabilities with domain-specific knowledge and workflows.

## Creation Workflow

### Step 1: Gather Requirements

Ask the user:
1. **What should the skill do?** (capability)
2. **When should it be used?** (trigger conditions)
3. **What resources does it need?** (templates, scripts, references)
4. **Where should it be installed?** (project or personal)

### Step 2: Choose Skill Location

```
# Project skill (shared with team via git)
.claude/skills/{skill-name}/SKILL.md

# Personal skill (only for you)
~/.claude/skills/{skill-name}/SKILL.md

# agent-kit content (for distribution)
content/skills/{skill-name}/SKILL.md
```

### Step 3: Create Directory Structure

```bash
mkdir -p {location}/{skill-name}/{assets,references,scripts}
```

### Step 4: Write SKILL.md

Use the template from [assets/skill-template.md](assets/skill-template.md).

Key sections:
1. **Frontmatter** - name, description, metadata
2. **Purpose** - What the skill does
3. **When to Use** - Trigger conditions
4. **Instructions** - Step-by-step guidance
5. **Examples** - Concrete usage examples
6. **Related Skills** - Links to other skills

### Step 5: Add Resources (if needed)

- **assets/**: Templates the skill uses
- **references/**: Additional docs for complex topics
- **scripts/**: Automation scripts

### Step 6: Create Slash Command

Every skill should have a corresponding slash command for easy invocation.

**Command location:**
```
# Project command (matches skill location)
.claude/commands/{skill-name}.md

# agent-kit content (for distribution) - MUST use ak- prefix
content/commands/ak-{skill-name}.md
```

**IMPORTANT: Naming Rules**

1. **agent-kit commands MUST use `ak-` prefix**: All commands in `content/commands/` must be prefixed with `ak-` to avoid conflicts with user-created commands.
   - ✅ `content/commands/ak-create-plan.md` → `/ak-create-plan`
   - ❌ `content/commands/create-plan.md` → conflicts with user commands

2. **No command/skill name conflicts**: Command names must not conflict with existing skill names in the same scope.

**Command template:**
```markdown
---
description: {One-line description of what command does}
arguments:
  - name: {arg-name}
    description: {What the argument is for}
    required: false
---

Use the {skill-name} skill to help the user {accomplish task}.

Follow the {skill-name} skill instructions in @skills/{skill-name}/SKILL.md exactly.

Key steps:
1. {Step from skill}
2. {Step from skill}
...
```

See [assets/command-template.md](assets/command-template.md) for the full template.

### Step 7: Validate with skill-validator

Before finalizing, run the skill-validator to ensure the skill meets all requirements:

```
User: Validate my new skill at {path}

Claude: [Runs skill-validator checks]
[Reports any errors, warnings, or suggestions]
```

The validator checks:
- Frontmatter requirements (name, description)
- SKILL.md structure and token limits
- All referenced files exist
- Corresponding slash command exists
- **agent-kit commands use `ak-` prefix** (for `content/commands/`)
- **No command/skill name conflicts**

**Only proceed if validation passes with no errors.**

### Step 8: Test the Skill

1. Ask Claude something that should trigger it
2. Verify it activates correctly
3. Check the output quality
4. Iterate on instructions

## Frontmatter Requirements

```yaml
---
name: skill-name              # Required: lowercase, hyphens, max 64 chars
description: What and when    # Required: max 1024 chars, include trigger words
license: MIT                  # Optional: license type
metadata:                     # Optional: additional info
  author: your-name
  version: "1.0.0"
---
```

### Name Rules
- Max 64 characters
- Lowercase letters, numbers, hyphens only
- No spaces or underscores
- Cannot contain "anthropic" or "claude"

### Description Best Practices

The description is critical for discovery. Include:
1. **What** the skill does
2. **When** to use it (trigger words)

❌ Bad: `Helps with documentation`
✅ Good: `Generate API documentation from code. Use when creating docs, OpenAPI specs, or README files for libraries.`

## Progressive Loading

Skills use 3-level loading to stay efficient:

| Level | Loads | When | Size Limit |
|-------|-------|------|------------|
| 1 | Frontmatter | Startup | ~100 tokens |
| 2 | SKILL.md body | Triggered | < 5000 tokens |
| 3 | Bundled files | Referenced | Unlimited |

**Keep SKILL.md under 5000 tokens.** Move detailed content to references/.

## Bundled Resources

### assets/ - Templates and Data

```markdown
# In SKILL.md
See [assets/template.md](assets/template.md) for the format.
```

Use for:
- Document templates
- Configuration examples
- Data schemas

### references/ - Extended Documentation

```markdown
# In SKILL.md
For advanced usage, see [references/advanced.md](references/advanced.md).
```

Use for:
- Detailed examples
- Troubleshooting guides
- API references

### scripts/ - Executable Code

```markdown
# In SKILL.md
Run the validation script:
\`\`\`bash
./scripts/validate.sh
\`\`\`
```

Use for:
- Validation utilities
- Code generation
- Data processing

Scripts execute without loading into context - only output is captured.

## Common Patterns

### Workflow Skill
For multi-step processes:
```markdown
## Workflow

1. **Phase 1: Discovery**
   - Scan project structure
   - Identify relevant files
   
2. **Phase 2: Analysis**
   - Parse content
   - Extract patterns
   
3. **Phase 3: Generation**
   - Apply template
   - Write output
```

### Reference Skill
For domain knowledge:
```markdown
## Quick Reference

| Term | Definition |
|------|------------|
| ... | ... |

## Detailed Reference

See [references/full-guide.md](references/full-guide.md)
```

### Tool Skill
For specific capabilities:
```markdown
## Usage

\`\`\`bash
{command} [options]
\`\`\`

## Options

| Flag | Description |
|------|-------------|
| --flag | What it does |
```

## Output

After creation, provide:
1. Path to the new skill
2. Path to the slash command
3. Validation results
4. How to test it
5. How to iterate

```
Created:
  Skill:   .claude/skills/my-skill/SKILL.md
  Command: .claude/commands/my-skill.md

Validation: ✅ PASSED (0 errors, 0 warnings)

To test:
1. Start a new conversation
2. Use the slash command: /my-skill
3. Or ask: "Help me with {trigger phrase}"
4. Verify the skill activates

To iterate:
- Edit SKILL.md and test again
- No restart needed - changes take effect immediately
```

## Examples

### Example: Create a code review skill

```
User: Create a skill for reviewing React components

Claude: [Asks clarifying questions]
- What aspects to review? (performance, accessibility, patterns)
- Any specific rules or style guide?
- Should it suggest fixes or just identify issues?

[Creates skill structure]
.claude/skills/react-review/
├── SKILL.md
├── assets/
│   └── checklist.md
└── references/
    └── patterns.md
```

### Example: Create a documentation skill

```
User: Create a skill for writing JSDoc comments

Claude: [Creates skill with template]
.claude/skills/jsdoc-writer/
├── SKILL.md
└── assets/
    └── jsdoc-template.md
```

## Template Reference

See [assets/skill-template.md](assets/skill-template.md) for the starter template.

## Validation

Before finalizing, run `skill-validator` to verify:
- [ ] Frontmatter has required fields (name, description)
- [ ] Name follows rules (lowercase, hyphens, no reserved words)
- [ ] Description includes what AND when
- [ ] SKILL.md is under 5000 tokens
- [ ] All referenced files exist
- [ ] Examples are concrete and testable
- [ ] Corresponding slash command exists
- [ ] Command references the skill correctly
- [ ] agent-kit commands use `ak-` prefix (for `content/commands/`)
- [ ] No command/skill name conflicts

## Related Skills

- `skill-validator` - Validates skills after creation (invoked automatically)
- `doc-contents` - For documentation generation
- `create-plan` - For implementation planning
- `brainstorm` - For exploring skill ideas before creating
