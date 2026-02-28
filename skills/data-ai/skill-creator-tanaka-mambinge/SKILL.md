---
name: skill-creator
description: Create and maintain reusable skills for multiple agent platforms (OpenCode, Claude, Antigravity, Copilot). Use when defining or updating SKILL.md files that extend agent capabilities with workflows, tool integrations, or domain expertise.
compatibility:
  - opencode
  - claude
  - antigravity
  - copilot
---

# OpenCode Skill Creator

## Overview

Skills are self-contained SKILL.md files that extend agent platforms (OpenCode, Claude, Antigravity, Copilot) with specialized knowledge and reusable workflows. This skill guides creating effective skills by managing scope, structuring content, and organizing bundled resources.

## File Locations

Skills are discovered from these paths (in order):

**Project-local:** `.opencode/skill/<name>/SKILL.md`
**Global:** `~/.config/opencode/skill/<name>/SKILL.md`

## Skill Structure

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter
│   │   ├── name (required)
│   │   ├── description (required)
│   │   ├── license (optional)
│   │   ├── compatibility (optional)
│   │   └── metadata (optional)
│   └── Markdown body (instructions)
```

## SKILL.md Frontmatter

```yaml
---
name: skill-name
description: Brief, specific description of what the skill does and when to use it.
license: MIT
compatibility: opencode
metadata:
  audience: developers
---
```

**Naming rules:**

- 1–64 characters
- Lowercase alphanumeric with single hyphens
- Cannot start/end with hyphen, no consecutive hyphens
- Regex: `^[a-z0-9]+(-[a-z0-9]+)*$`

**Description rules:**

- 1–1024 characters
- Include what the skill does AND specific triggers/when to use it
- This is OpenCode's primary mechanism for deciding when to load the skill
- Example: "Handle FastAPI route organization with service layers, Pydantic DTOs, and Marshmallow validation. Use when building or refactoring FastAPI endpoints."

## Body Structure

Write concise, procedural instructions. Use sections for:

- **Overview/Architecture** - High-level patterns and principles
- **File Structure** - Directory layout and organization
- **Implementation Steps** - Sequential workflows with code examples
- **References** - Link to bundled documentation files if needed

Keep body content focused on core procedures. Move detailed reference material into `references/` files.

## Design Principles

### Concise First

Assume OpenCode is already intelligent. Only include knowledge it doesn't have. Challenge each piece: "Does OpenCode really need this?"

### Progressive Disclosure

1. **Metadata** (always loaded) - Skill name and trigger description
2. **Body** (loaded when triggered) - Core procedures and guidance
3. **Resources** (loaded as-needed) - Scripts, references, detailed docs

### Appropriate Freedom

- **High freedom** (text-only): Multiple valid approaches, context-dependent decisions
- **Medium freedom** (pseudocode/templates): Preferred patterns exist, some variation allowed
- **Low freedom** (specific scripts): Operations are fragile, consistency critical

## Creation Workflow

1. **Understand the skill** - Gather concrete usage examples and requirements
2. **Plan contents** - Identify scripts, references, and assets needed
3. **Initialize** - Create directory structure at chosen location
4. **Implement** - Write SKILL.md and bundled resources
5. **Test** - Verify the skill works with real OpenCode tasks
6. **Iterate** - Refine based on actual usage patterns

## Example Skill Format

```
---
name: some-skill
description: Short statement describing the capability and when to invoke it.
license: MIT
compatibility: opencode
metadata:
  audience: developers
---

## Overview
Explain the problem space, core architectural patterns, and when to trust this skill.

## File Structure
List key directories and files (or fragments) the skill touches.

## Implementation Steps
Provide ordered steps, ideally with code snippets, that demonstrate how to execute the workflow.

## Examples
Include runnable code samples that echo the patterns shared earlier.

## References
- https://example.com/related-docs
- https://example.com/api-guidelines
```

Use this layout as inspiration for future skills so contributors have a shared rhythm for explanations, samples, and reference links. Keep the links actionable rather than pointing to local files.
