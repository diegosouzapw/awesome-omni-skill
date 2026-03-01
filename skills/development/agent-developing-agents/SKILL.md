---
name: agent-developing-agents
description: AI agent development standards including frontmatter structure, naming conventions, tool access patterns, model selection, and Bash-only file operations for .opencode/ folders
---

# Developing AI Agents

Comprehensive guidance for creating AI agents following repository conventions.

## Core Requirements

- Frontmatter: name, description, tools, model, color, skills
- Name must match filename exactly
- Use Bash tools for .opencode/ folder operations
- Non-empty skills field required

## References

[AI Agents Convention](../../../governance/development/agents/ai-agents.md)

## Tool Usage Documentation

Agents should document which tools they use and why, helping users understand capabilities and maintainers understand dependencies.

### Tool Documentation Pattern

Add "Tools Usage" section (optional but recommended) listing each tool with its purpose:

```markdown
## Tools Usage

- **Read**: Read files to validate/create/fix
- **Glob**: Find files by pattern in directories
- **Grep**: Extract content patterns (code blocks, commands, etc.)
- **Write**: Create/update files and reports
- **Bash**: Generate UUIDs, timestamps, file operations
- **Edit**: Apply fixes to existing files
- **WebFetch**: Access official documentation URLs
- **WebSearch**: Find authoritative sources, verify claims
```

### When to Document Tools

**Recommended for**:

- Agents with 4+ tools (helps users understand capabilities)
- Agents where tool selection isn't obvious
- Agents with unusual tool combinations
- Reference documentation for complex agents

**Optional for**:

- Simple agents with 2-3 obvious tools
- Agents following standard patterns

### Tool Documentation Examples

**Checker Agents** (Read, Glob, Grep, Write, Bash, WebFetch, WebSearch):

```markdown
## Tools Usage

- **Read**: Read documentation files to validate
- **Glob**: Find markdown files in directories
- **Grep**: Extract code blocks, commands, version numbers
- **Write**: Generate audit reports to generated-reports/
- **Bash**: Generate UUIDs, timestamps for reports
- **WebFetch**: Access official documentation URLs
- **WebSearch**: Find versions, verify tools, fallback for 403s
```

**Fixer Agents** (Read, Edit, Bash, Write):

```markdown
## Tools Usage

- **Read**: Read audit reports and files to fix
- **Edit**: Apply fixes to docs/ files
- **Bash**: Apply fixes to .opencode/ files (sed, heredoc)
- **Write**: Generate fix reports to generated-reports/
```

**Maker Agents** (Read, Write, Glob, Grep):

```markdown
## Tools Usage

- **Read**: Read existing files for context
- **Write**: Create new documentation files
- **Glob**: Find related files for cross-references
- **Grep**: Extract patterns for consistency
```

### Placement

Add "Tools Usage" section:

- After "Core Responsibility" or main description
- Before detailed workflow sections
- Near top for quick reference

## When to Use This Agent

Agents should include guidance on when to use them vs other agents, improving discoverability and preventing misuse.

### When to Use Pattern

Add "When to Use This Agent" section with two subsections:

```markdown
## When to Use This Agent

**Use when**:

- [Primary use case 1]
- [Primary use case 2]
- [Primary use case 3]
- [Specific scenario that fits]

**Do NOT use for**:

- [Anti-pattern 1] (use [other-agent] instead)
- [Anti-pattern 2] (use [alternative-tool/approach])
- [Edge case that doesn't fit]
- [Common misuse scenario]
```

### When to Include

**Highly Recommended for**:

- Agents with overlapping scopes (e.g., multiple checkers)
- Agents that users might confuse (e.g., maker vs editor)
- Agents with specific prerequisites (e.g., needs audit report)
- Specialized agents with narrow focus

**Examples by Agent Type**:

**Checker Agents**:

```markdown
## When to Use This Agent

**Use when**:

- Validating [domain] content before release
- Checking [domain] after updates
- Reviewing community contributions
- Auditing [domain] for compliance

**Do NOT use for**:

- Link checking (use [link-checker] instead)
- File naming/structure (use [rules-checker])
- Creating new content (use [maker-agent])
- Fixing issues (use [fixer-agent] after review)
```

**Fixer Agents**:

```markdown
## When to Use This Agent

**Use when**:

- After running [checker-agent] - You have an audit report
- Issues found and reviewed - You've reviewed checker's findings
- Automated fixing needed - You want validated issues fixed
- Safety is critical - You need re-validation before changes

**Do NOT use for**:

- Initial validation (use [checker-agent])
- Content creation (use [maker-agent])
- Manual fixes (use Edit tool directly)
- When no audit report exists
```

**Maker Agents**:

```markdown
## When to Use This Agent

**Use when**:

- Creating new [domain] content
- Need standardized structure/format
- Following [domain] conventions
- Building content from templates

**Do NOT use for**:

- Validating existing content (use [checker-agent])
- Fixing issues (use [fixer-agent])
- Bulk updates (use Edit tool for simple changes)
- Content outside [domain] scope
```

### Placement

Add "When to Use This Agent" section:

- After agent description or core responsibility
- Before detailed workflow/process sections
- Early in file for quick reference

### Benefits

✅ Improves agent discoverability
✅ Prevents misuse and confusion
✅ Clarifies agent boundaries
✅ Guides users to appropriate alternatives
✅ Reduces trial-and-error

## Updated References

[AI Agents Convention - Complete specification](../../../governance/development/agents/ai-agents.md)
[Agent Documenting References Skill](./SKILL.md)
[Agent Selecting Models Skill](./SKILL.md)
