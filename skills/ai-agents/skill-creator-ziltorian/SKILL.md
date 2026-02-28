---
name: skill-creator
description: Guide for creating effective, universal Agent Skills that work across multiple AI coding platforms (Claude, GitHub Copilot, Cursor, Windsurf, Aider). Use when creating new skills, updating existing skills, or learning skill creation best practices. Includes specifications, examples, and quality standards.
license: MIT
metadata:
  version: "1.0"
  category: ai-assistant-configuration
  author: ziltorian
---

# Skill Creator Guide

Create professional, reusable Agent Skills following the open standard specification.

## Overview

Learn to create universal Agent Skills that work across multiple AI agent platforms. Skills are structured instruction sets enabling AI agents to perform specialized tasks consistently across projects.

## When to Use This Skill

- Creating new Agent Skill from documentation/SDK
- Updating existing skill to follow best practices
- Converting project-specific code into reusable skills
- Learning proper skill structure and formatting
- Validating skill quality before deployment

## Core Principles

### 1. Universal and Reusable

**DO:**
- Create skills teaching "how to work with X"
- Focus on platform/tool/library workflows
- Make skills work in any project context

**DON'T:**
- Create project-specific skills
- Hardcode paths, credentials, URLs
- Assume specific project structure

### 2. Clear and Discoverable

**Description is critical** - determines when agents activate your skill.

Include:
- What the skill does (primary capabilities)
- When to use it (trigger keywords)
- Specific terms users might mention

**Example:**

```yaml
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF documents, extracting data, filling forms, combining PDFs, or when user mentions PDFs, forms, or document processing.
```

### 3. Progressive Disclosure  

**Level 1 - Metadata (~100 tokens):**
- `name` and `description` in YAML frontmatter
- Loaded for all skills at startup
- Enables skill discovery

**Level 2 - Instructions (<5000 tokens):**
- Main SKILL.md body
- Loaded when skill activates
- Keep under 500 lines
- Core workflows and examples

**Level 3 - Resources (on-demand):**
- Files in `references/`, `scripts/`, `assets/`
- Loaded only when needed
- Unlimited detail allowed

## SKILL.md Structure

### Required YAML Frontmatter

```yaml
---
name: skill-name-here
description: Comprehensive description with what it does and when to use it
---
```

### Optional Frontmatter

```yaml
---
name: pdf-processing
description: Extract text and tables from PDF files...
license: MIT
compatibility: Python 3.8+ with pdfplumber package
metadata:
  version: "1.0"
  category: document-processing
---
```

### Essential Sections

```markdown
# [Skill Title]

Brief intro (1-2 sentences)

## Overview
What this skill teaches (3-5 sentences)

## When to Use This Skill
Bullet list of use cases

## Core Workflows

### Workflow 1: [Name]
Brief description + minimal example + link to details

### Workflow 2: [Name]
Brief description + minimal example + link to details

## Quick Reference
Essential cheat sheet

## Further Reading
Links to detailed documentation
```

**Keep SKILL.md concise:**
- Target: 300-500 lines
- Essential info only
- Link to references for details
- Examples minimal but complete

## Creating a Skill

### Step-by-Step Process

1. **Research:** Gather docs, examples, best practices
2. **Plan:** Define scope, name, description, structure
3. **Create:** Write SKILL.md with core workflows  
4. **Detail:** Add reference files for depth
5. **Test:** Verify all examples work
6. **Review:** Check universality (no project-specific content)

See [step-by-step guide](references/step-by-step-guide.md) for complete walkthrough.

### Naming Convention

**Format:** `lowercase-hyphen-case`

**Rules:**
- 1-64 characters
- Descriptive but concise
- Match directory name exactly

**Examples:**
- ✅ `pdf-processing`
- ✅ `github-copilot-sdk`
- ✅ `react-state-management`
- ❌ `PDFProcessing` (not lowercase)
- ❌ `pdf_processing` (underscores)
- ❌ `pdf` (too vague)

### Writing Description

**Formula:**
```
[What it does] + [When to use] + [Trigger keywords]
```

**Test:**
- Does it clearly state what skill does?
- Does it cover common use cases?
- Does it include trigger keywords?
- Is it under 500 characters?

## File Organization

```
.github/skills/skill-name/
├── SKILL.md                    # Main file (300-500 lines)
├── references/                # Detailed documentation
│   ├── getting-started.md     # Installation, setup
│   ├── examples.md            # Code examples
│   ├── advanced-techniques.md # Advanced features
│   └── troubleshooting.md     # Common issues
├── scripts/                   # Helper scripts (optional)
│   └── setup.sh
└── assets/                    # Images, diagrams (optional)
    └── diagram.png
```

## Quality Checklist

Before deployment:

- [ ] SKILL.md under 500 lines
- [ ] All code examples tested and working
- [ ] No hardcoded secrets or project-specific paths
- [ ] Links to references are correct
- [ ] Description includes trigger keywords
- [ ] Name matches directory and uses hyphens
- [ ] Examples complete and runnable
- [ ] No broken external links

## Best Practices

**DO:**
- Keep SKILL.md focused on essentials
- Link to references for details
- Test all code examples
- Use progressive disclosure
- Write universal, reusable content
- Include trigger keywords in description

**DON'T:**
- Put everything in SKILL.md
- Include project-specific details
- Hardcode paths or credentials
- Skip testing examples
- Forget to update description

## Common Patterns

See [example skills](references/example-skills.md) for real-world examples following these patterns.

See [Agent Skills Specification](references/agent-skills-specification.md) for complete standard.

## Further Reading

- [Step-by-Step Guide](references/step-by-step-guide.md) - Complete skill creation walkthrough  
- [Example Skills](references/example-skills.md) - Real skills with analysis
- [Agent Skills Specification](references/agent-skills-specification.md) - Open standard
- [GitHub Copilot Skills Spec](references/github-copilot-skills-spec.md) - GitHub's specification
- [License Guide](references/license-guide.md) - Choosing appropriate license
- [Skills Quick Comparison](references/skills-quick-comparison.md) - Specification comparison
