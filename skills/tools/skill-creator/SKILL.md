---
name: Skill Creator
description: Scaffolds new Claude Code skills with proper structure, templates, and best practices. Use when creating custom skills for workflows, automation, or domain-specific tasks.
version: 1.0.0
dependencies: none
---

# Skill Creator

A meta-skill for scaffolding new Claude Code skills. Creates properly structured skills with YAML frontmatter, activation triggers, templates, examples, and guidelines following Anthropic's official skill format.

## When This Skill Activates

This skill automatically activates when you:
- Want to create a new skill for Claude Code
- Need to scaffold a skill template
- Ask about skill structure or format
- Want to add automation or workflows as skills
- Need to document domain-specific knowledge as a skill

**Keywords**: create skill, new skill, scaffold skill, skill template, add skill, make skill, build skill

## Skill Structure Overview

Every skill follows this directory structure:

```
.claude/skills/
└── skill-name/
    └── SKILL.md          # Required: Main skill definition
    ├── templates/        # Optional: Reusable templates
    ├── examples/         # Optional: Usage examples
    └── checklists/       # Optional: Workflow checklists
```

### Required: SKILL.md Format

```markdown
---
name: Skill Name
description: Clear description of what this skill does and when to use it
version: 1.0.0
dependencies: optional comma-separated list
---

# Skill Name

[Introduction paragraph explaining the skill's purpose]

## When This Skill Activates

This skill automatically activates when you:
- [Trigger 1]
- [Trigger 2]
- [Trigger 3]

**Keywords**: keyword1, keyword2, keyword3

## Core Content

[Main skill instructions, workflows, patterns]

## Templates

[Reusable prompt templates or code templates]

## Examples

[Concrete usage examples]

## Best Practices

[Guidelines and recommendations]

---

**Version**: 1.0.0
**Last Updated**: YYYY-MM-DD
**Maintained By**: [Team/Person]
```

## Creating a New Skill: Step-by-Step

### Step 1: Define the Skill Purpose

Before creating, answer these questions:

```
1. What problem does this skill solve?
2. When should Claude activate this skill?
3. What are the key workflows or patterns?
4. What templates would be reusable?
5. What examples would help illustrate usage?
```

### Step 2: Create the Directory

```bash
mkdir -p .claude/skills/your-skill-name
```

### Step 3: Create SKILL.md

Use the template generator below, then customize:

```markdown
---
name: Your Skill Name
description: What this skill does and when to use it (shown in skill list)
version: 1.0.0
dependencies: none
---

# Your Skill Name

A comprehensive skill for [purpose]. Helps with [key workflows] in the CircleTel codebase.

## When This Skill Activates

This skill automatically activates when you:
- [Describe trigger scenario 1]
- [Describe trigger scenario 2]
- [Describe trigger scenario 3]

**Keywords**: [comma, separated, activation, keywords]

## Core Workflows

### Workflow 1: [Name]

**Purpose**: [What this workflow accomplishes]

**Steps**:
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Example**:
[Show concrete example of this workflow]

### Workflow 2: [Name]

[Continue pattern for additional workflows]

## Templates

### Template 1: [Name]

**Use When**: [Scenario for this template]

```
[Template content with placeholders]
```

## CircleTel-Specific Patterns

[Document any project-specific patterns, files, or conventions relevant to this skill]

## Validation Checklist

After using this skill, verify:
- [ ] [Verification item 1]
- [ ] [Verification item 2]
- [ ] [Verification item 3]

## Best Practices

1. **[Practice 1]**: [Explanation]
2. **[Practice 2]**: [Explanation]
3. **[Practice 3]**: [Explanation]

---

**Version**: 1.0.0
**Last Updated**: YYYY-MM-DD
**Maintained By**: CircleTel Development Team
```

### Step 4: Add to CLAUDE.md (if important)

For frequently-used skills, add an entry to the Skills System section in CLAUDE.md:

```markdown
| **your-skill-name** | Brief description | Activation command |
```

## Skill Category Templates

### Category A: Debugging/Troubleshooting Skill

Use when creating skills for debugging specific systems or error types.

```markdown
---
name: [System] Debugger
description: Systematic debugging for [system] issues - identifies root causes and proposes fixes
version: 1.0.0
dependencies: none
---

# [System] Debugger

Helps debug and fix issues related to [system].

## When This Skill Activates

This skill automatically activates when you:
- Encounter errors related to [system]
- Debug [system] failures
- Troubleshoot [system] performance issues

**Keywords**: [system] error, [system] debug, [system] fix, [system] issue

## Common Error Patterns

### Error: [Error Name/Code]

**Symptom**: [What the user sees]

**Root Cause**: [Why this happens]

**Fix**:
```[language]
[Code fix]
```

**Prevention**: [How to avoid this error]

[Repeat for each common error]

## Debugging Workflow

### Phase 1: Reproduce
[Steps to reproduce the issue]

### Phase 2: Investigate
[How to gather information]

### Phase 3: Fix
[How to implement solutions]

### Phase 4: Validate
[How to verify the fix works]
```

### Category B: Code Generation Skill

Use when creating skills that generate specific types of code.

```markdown
---
name: [Component Type] Generator
description: Generates [component type] following CircleTel patterns and conventions
version: 1.0.0
dependencies: none
---

# [Component Type] Generator

Generates production-ready [component type] code following project conventions.

## When This Skill Activates

This skill automatically activates when you:
- Need to create a new [component type]
- Ask for [component type] boilerplate
- Want to scaffold [component type] structure

**Keywords**: create [type], new [type], generate [type], scaffold [type]

## Generated Structure

```
[directory structure showing what gets generated]
```

## Code Templates

### Template: [Variant 1]

**Use When**: [Scenario]

```[language]
[Full code template]
```

### Template: [Variant 2]

[Continue for variants]

## Customization Options

| Option | Description | Default |
|--------|-------------|---------|
| [option1] | [description] | [default] |
| [option2] | [description] | [default] |

## Post-Generation Checklist

- [ ] Update imports in parent component
- [ ] Add route to navigation (if applicable)
- [ ] Run type check: `npm run type-check`
- [ ] Test component renders correctly
```

### Category C: Workflow/Process Skill

Use when creating skills for multi-step processes.

```markdown
---
name: [Process Name] Workflow
description: Guides through [process] with step-by-step instructions and validation
version: 1.0.0
dependencies: none
---

# [Process Name] Workflow

A structured workflow for completing [process] correctly and completely.

## When This Skill Activates

This skill automatically activates when you:
- Start [process]
- Need to complete [process]
- Ask about [process] steps

**Keywords**: [process], how to [process], start [process], complete [process]

## Workflow Overview

```
[ASCII diagram or numbered list showing workflow stages]
```

## Step-by-Step Guide

### Step 1: [Step Name]

**Purpose**: [Why this step exists]

**Actions**:
1. [Action 1]
2. [Action 2]

**Verification**: [How to verify step is complete]

**Common Issues**:
- [Issue 1]: [Solution]
- [Issue 2]: [Solution]

[Repeat for each step]

## Workflow Checklist

Pre-Workflow:
- [ ] [Prerequisite 1]
- [ ] [Prerequisite 2]

During Workflow:
- [ ] Step 1 completed
- [ ] Step 2 completed
[Continue]

Post-Workflow:
- [ ] [Verification 1]
- [ ] [Verification 2]
```

### Category D: Integration Skill

Use when creating skills for external API/service integrations.

```markdown
---
name: [Service] Integration
description: Integrates with [Service] API - handles auth, requests, and error handling
version: 1.0.0
dependencies: [required packages]
---

# [Service] Integration

Helps integrate with [Service] following best practices and handling common scenarios.

## When This Skill Activates

This skill automatically activates when you:
- Need to call [Service] API
- Integrate [Service] functionality
- Debug [Service] connection issues

**Keywords**: [service], [service] api, [service] integration, connect [service]

## Configuration

### Environment Variables

```env
[SERVICE]_API_KEY=your_api_key
[SERVICE]_API_URL=https://api.service.com
```

### Client Setup

```typescript
// lib/[service]/client.ts
[Client setup code]
```

## Common Operations

### Operation 1: [Name]

```typescript
[Code example]
```

### Operation 2: [Name]

```typescript
[Code example]
```

## Error Handling

| Error Code | Meaning | Solution |
|------------|---------|----------|
| [code] | [meaning] | [solution] |

## Rate Limiting

[Document rate limits and how to handle them]
```

## Best Practices for Skill Creation

### 1. Clear Activation Triggers
- List specific scenarios when the skill activates
- Use precise keywords that match user intent
- Avoid overly broad triggers that cause false activations

### 2. Structured Content
- Use consistent heading hierarchy
- Include code examples with proper syntax highlighting
- Provide templates that are copy-paste ready

### 3. CircleTel Context
- Reference project-specific files and patterns
- Include relevant CLAUDE.md sections
- Document project-specific conventions

### 4. Validation & Verification
- Include checklists for completion verification
- Document common errors and solutions
- Provide testing commands

### 5. Maintainability
- Include version and last updated date
- Document dependencies
- Keep skills focused (one purpose per skill)

## Examples of Well-Structured Skills

### Example 1: Simple Utility Skill

```markdown
---
name: API Route Generator
description: Generates Next.js 15 API routes with proper async params handling
version: 1.0.0
dependencies: none
---

# API Route Generator

Generates API routes following Next.js 15 patterns.

## When This Skill Activates

**Keywords**: create api route, new endpoint, api endpoint

## Template

```typescript
import { NextRequest, NextResponse } from 'next/server'
import { createClient } from '@/lib/supabase/server'

export async function GET(
  request: NextRequest,
  context: { params: Promise<{ id: string }> }
) {
  const { id } = await context.params
  const supabase = await createClient()

  // Implementation

  return NextResponse.json({ data })
}
```

## Checklist
- [ ] Async params handled correctly
- [ ] Service role client used
- [ ] Error handling included
- [ ] Type check passes
```

### Example 2: Complex Workflow Skill

See `.claude/skills/bug-fixing/SKILL.md` for a comprehensive example of a debugging workflow skill.

## Skill Validation Checklist

After creating a new skill, verify:

- [ ] SKILL.md exists in `.claude/skills/[skill-name]/`
- [ ] YAML frontmatter includes name, description, version
- [ ] "When This Skill Activates" section is clear
- [ ] Keywords are specific and relevant
- [ ] Templates are complete and copy-paste ready
- [ ] Examples demonstrate real usage
- [ ] CircleTel-specific patterns are documented
- [ ] Version and last updated date included
- [ ] Skill activates on expected triggers (test it)

## Quick Reference

### Create New Skill (Commands)

```bash
# Create skill directory
mkdir -p .claude/skills/my-skill-name

# Create SKILL.md
# Use template from this skill

# Test skill activates
# Mention keywords in conversation
```

### Skill File Locations

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill definition (required) |
| `templates/` | Reusable templates |
| `examples/` | Usage examples |
| `checklists/` | Workflow checklists |

### Skill Naming Conventions

- Use lowercase with hyphens: `my-skill-name`
- Be descriptive but concise
- Avoid generic names like "helper" or "utility"

---

**Version**: 1.0.0
**Last Updated**: 2025-12-15
**Maintained By**: CircleTel Development Team
**Reference**: https://github.com/anthropics/skills
