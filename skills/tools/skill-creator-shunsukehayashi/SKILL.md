---
name: skill-creator
description: Create new Claude Code skills with proper structure. Use when building custom skills or automating workflows.
allowed-tools: Bash, Read, Write
---

# Skill Creator

**Version**: 1.0.0
**Purpose**: Generate properly structured Claude Code skills

---

## Workflow

### Step 1: Gather Requirements

Ask user:
1. What does this skill do?
2. When should it activate (triggers)?
3. What tools does it need?

### Step 2: Create Structure

```bash
mkdir -p ~/.claude/skills/[skill-name]
```

### Step 3: Generate SKILL.md

Template:
```markdown
---
name: [kebab-case-name]
description: [What it does]. Use when [triggers].
allowed-tools: [tools]
---

# [Title]

## Workflow

### Step 1: [First step]
...
```

### Step 4: Validate

```bash
cat ~/.claude/skills/[name]/SKILL.md
```

---

## Rules

- Name: kebab-case only
- No reserved words: claude, anthropic, mcp
- Description: action verb + trigger condition
