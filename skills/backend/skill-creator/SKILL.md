---
name: skill-creator
description: Create, validate, and package Claude skills automatically. Use PROACTIVELY when users want to create a new skill, validate existing skill files, or package skills for distribution. Works across Claude.ai, API, and Claude Code platforms.
---

# Skill Creator

An interactive guide for creating, validating, and packaging new Claude skills. This skill helps you build high-quality skills that extend Claude's capabilities with specialized knowledge, workflows, or tool integrations.

## What Are Skills?

Skills are modular capabilities that extend Claude's functionality. Each skill packages:
- **Instructions**: Step-by-step guidance for completing specific tasks
- **Metadata**: Name and description for automatic discovery
- **Resources**: Optional scripts, templates, and reference documentation

Skills use progressive disclosure - only loading what's needed when needed - to minimize token usage while maximizing capability.

## When to Use This Skill

Use the skill-creator when you need to:
- **Create a new skill** from scratch
- **Validate an existing skill** against Claude's requirements
- **Package a skill** for distribution
- **Update a skill** to meet current standards
- **Learn** skill best practices and requirements

## Skill Creation Workflow

### Step 1: Understand the Need

Ask clarifying questions to understand:

1. **Purpose**: What task or domain will this skill address?
2. **Scope**: Is it simple (just instructions) or complex (with scripts/references)?
3. **Platform**: Which Claude surfaces should it target? (Claude.ai, API, Claude Code, or all)
4. **Automation**: Should it be automatically invoked or user-triggered?

Example questions:
- "What specific problem does this skill solve?"
- "Will the skill need helper scripts or just provide guidance?"
- "Should Claude use this skill proactively when users mention [topic]?"
- "Does the skill need platform-specific features (like network access)?"

### Step 2: Design the Skill

Based on the requirements, plan:

**Directory Structure Options:**

**Option A: Simple Skill (Instructions only)**
```
skill-name/
└── SKILL.md
```

**Option B: Skill with Scripts**
```
skill-name/
├── SKILL.md
└── scripts/
    ├── __init__.py
    ├── helper1.py
    └── helper2.py
```

**Option C: Complex Skill (Full featured)**
```
skill-name/
├── SKILL.md
├── scripts/
│   ├── __init__.py
│   └── utilities.py
└── references/
    ├── specification.md
    └── examples.md
```

**Naming Convention:**
- Lowercase letters, numbers, and hyphens only
- Maximum 64 characters
- Cannot contain "anthropic" or "claude"
- Descriptive and concise (e.g., `git-helper`, `api-tester`, `doc-writer`)

### Step 3: Write SKILL.md

Every skill must have a SKILL.md file with this structure:

```markdown
---
name: skill-name
description: Brief description of what this skill does and when to use it (max 1024 chars)
---

# Skill Title

## Overview
What the skill does and why it's useful.

## When to Use This Skill
Specific scenarios where this skill should be invoked.

## Instructions

### Step 1: [First Action]
Detailed guidance for the first step...

### Step 2: [Second Action]
Detailed guidance for the second step...

### Step N: [Final Action]
Detailed guidance for completion...

## Best Practices
Tips for optimal results.

## Common Issues
Troubleshooting guidance.

## Examples
Concrete examples of the skill in action.
```

**Critical Requirements:**

✅ **Required YAML Frontmatter**
```yaml
---
name: skill-name          # lowercase, hyphens, max 64 chars
description: What it does and when to use it  # max 1024 chars
---
```

✅ **Description Best Practices**
- Explain WHAT the skill does
- Explain WHEN to use it
- Use action-oriented language (e.g., "Use when...", "MUST be used for...")
- Be specific about capabilities
- Maximum 1024 characters
- No XML tags

✅ **Content Organization**
- Clear hierarchy with headings
- Step-by-step instructions
- Concrete examples
- Platform-specific guidance if needed

### Step 4: Add Supporting Resources (Optional)

**Scripts (Python recommended):**
- Use for deterministic operations
- Validate, transform, or package data
- Generate templates or boilerplate
- Follow security best practices (see references/skill-specification.md)

Example script structure:
```python
#!/usr/bin/env python3
"""
Brief description of what this script does.
"""
import argparse
from pathlib import Path
from typing import Tuple

def main() -> int:
    """Main entry point."""
    parser = argparse.ArgumentParser(description="...")
    # ... argument parsing ...

    try:
        result = do_work(args)
        print(f"Success: {result}")
        return 0
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        return 1

if __name__ == "__main__":
    sys.exit(main())
```

**Reference Documentation:**
- Technical specifications
- API documentation
- Database schemas
- Best practices guides
- Example templates

References are loaded on-demand, so include comprehensive details without worrying about token cost.

### Step 5: Validate the Skill

Use the validation script to ensure compliance:

```bash
python .claude-plugin/scripts/validate_skill.py path/to/skill/
```

The validator checks:
- ✅ YAML frontmatter format
- ✅ Required fields (name, description)
- ✅ Naming conventions
- ✅ Description length and format
- ✅ No XML tags or restricted terms
- ✅ File structure

Fix any errors before proceeding.

### Step 6: Test the Skill

1. **Place the skill** in the appropriate location:
   - Project-level: `.claude/skills/skill-name/`
   - User-level: `~/.claude/agents/skill-name.md` (for single-file skills)
   - Plugin: `.claude-plugin/` (for distribution)

2. **Invoke the skill** and verify:
   - Instructions are clear and actionable
   - Scripts execute correctly
   - References load when needed
   - Platform-specific features work as expected

3. **Iterate** based on results:
   - Clarify ambiguous instructions
   - Fix script bugs
   - Add missing examples
   - Improve error handling

### Step 7: Package for Distribution (Optional)

To share the skill:

```bash
python .claude-plugin/scripts/package_skill.py path/to/skill/ --output skill-name.zip
```

This creates a distributable package including:
- SKILL.md
- All scripts and references
- Metadata/manifest
- Installation instructions

## Platform-Specific Considerations

### Claude.ai
- **Scope**: User-specific (not shared across workspace)
- **Network**: Varies by user/admin settings
- **Distribution**: Upload through Skills UI
- **Best for**: Personal productivity, individual workflows

### Claude API
- **Scope**: Workspace-wide accessible
- **Network**: No access by default
- **Dependencies**: Pre-configured only, no dynamic installation
- **Best for**: Programmatic workflows, enterprise deployments

### Claude Code
- **Scope**: Project-level (.claude/skills/) or user-level (~/.claude/agents/)
- **Network**: Full access available
- **Dependencies**: Can install packages as needed
- **Best for**: Development workflows, technical tasks

**Include platform notes when relevant:**
```markdown
## Platform Notes

### Claude Code
This skill requires network access to fetch API data. It works best in Claude Code where network access is available by default.

### Claude API
Note: This skill requires pre-configured API keys via environment variables.
```

## Best Practices

### Token Efficiency
- Keep SKILL.md focused on essential instructions
- Move detailed specs to reference files
- Use progressive disclosure (load references only when needed)
- Aim for 30-50 tokens when skill is inactive (just frontmatter)

### Discoverability
- Write descriptions that clearly explain when to use the skill
- Use action-oriented language: "Use PROACTIVELY when..."
- Include relevant keywords in the description
- Be specific about capabilities

### Quality
- Test thoroughly before distributing
- Include concrete examples
- Provide troubleshooting guidance
- Handle errors gracefully
- Validate all user inputs

### Security
- Never execute arbitrary code (no eval/exec)
- Validate file paths (prevent traversal attacks)
- Sanitize user inputs
- Use allowlists over denylists
- Follow principle of least privilege

### Maintainability
- Use clear, descriptive names
- Document all functions and scripts
- Keep skills focused on a single purpose
- Version your skills for tracking changes
- Write clear error messages

## Validation Rules Reference

Load the complete specification when needed:
```
Read .claude-plugin/references/skill-specification.md
```

Key rules:
- **Name**: lowercase, hyphens, max 64 chars, no "anthropic"/"claude"
- **Description**: non-empty, max 1024 chars, no XML tags
- **Frontmatter**: Valid YAML between `---` delimiters
- **Files**: UTF-8 encoding, reasonable sizes
- **Scripts**: Executable, proper permissions, type-safe

## Example: Creating a Simple Skill

**User Request**: "Create a skill to help format commit messages"

**Skill Creator Response**:

1. **Clarify requirements**:
   - "Should it validate existing commits or generate new ones?"
   - "Do you want specific commit types enforced (feat, fix, docs)?"
   - "Should it integrate with git commands or just provide guidance?"

2. **After getting answers, create the skill**:

```markdown
---
name: commit-formatter
description: Format git commit messages according to Conventional Commits specification. Use when writing commit messages or validating commit history.
---

# Commit Formatter

Helps create well-formatted git commit messages following Conventional Commits.

## When to Use This Skill
- Writing new commit messages
- Validating existing commits
- Learning commit message best practices

## Instructions

### Step 1: Determine Commit Type
Choose the appropriate type:
- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation only
- **style**: Code style (formatting, missing semicolons)
- **refactor**: Code change that neither fixes a bug nor adds a feature
- **perf**: Performance improvement
- **test**: Adding or updating tests
- **chore**: Maintenance tasks

### Step 2: Write Subject Line
Format: `<type>(<scope>): <subject>`
- Scope is optional but recommended (e.g., api, ui, auth)
- Subject is lowercase, no period at end
- Keep under 50 characters

### Step 3: Add Body (Optional)
- Wrap at 72 characters
- Explain what and why, not how
- Separate from subject with blank line

### Step 4: Add Footer (Optional)
- Breaking changes: `BREAKING CHANGE: <description>`
- Issue references: `Closes #123`

## Examples

**Simple commit**:
```
feat(auth): add OAuth2 login support
```

**With body**:
```
fix(api): handle null response from user service

The user service occasionally returns null when the user
is not found. This commit adds proper null checking and
returns a 404 status code.
```

**Breaking change**:
```
feat(api): redesign authentication endpoints

BREAKING CHANGE: The /auth endpoint now requires POST instead
of GET. Update all clients to use POST with JSON body.

Closes #456
```
```

3. **Validate the skill**:
```bash
python .claude-plugin/scripts/validate_skill.py .claude/skills/commit-formatter/
```

4. **Test it**:
   - Ask Claude to help format a commit
   - Verify the guidance is clear and actionable
   - Check examples are helpful

## Troubleshooting

### Skill Not Being Invoked Automatically

**Problem**: Claude doesn't use the skill even when relevant.

**Solutions**:
- Improve the description with clearer trigger phrases
- Add "Use PROACTIVELY when..." to the description
- Mention the skill explicitly: "Use the [skill-name] skill to..."

### Validation Errors

**Problem**: Validator reports errors.

**Solutions**:
- Check YAML formatting (use `---` delimiters)
- Verify name follows conventions (lowercase, hyphens only)
- Ensure description is under 1024 characters
- Remove any XML tags from text
- Check file encoding is UTF-8

### Scripts Not Executing

**Problem**: Python scripts fail or aren't found.

**Solutions**:
- Verify script has execute permissions: `chmod +x script.py`
- Check shebang line: `#!/usr/bin/env python3`
- Ensure dependencies are installed
- Test script independently before integrating
- Check file paths are correct (use Path from pathlib)

### Cross-Platform Issues

**Problem**: Skill works on one platform but not another.

**Solutions**:
- Check platform constraints (network access, dependencies)
- Use pathlib for file paths (not os.path)
- Avoid platform-specific assumptions
- Test on all target platforms
- Add platform-specific guidance in SKILL.md

## Getting Help

For detailed technical specifications:
```
Read .claude-plugin/references/skill-specification.md
```

For platform-specific guidance:
```
Read .claude-plugin/references/platform-differences.md
```

For more examples:
```
Read .claude-plugin/references/skill-examples.md
```

## Creating the Skill Automatically

When you're ready, I'll:

1. **Create the directory structure** at the specified location
2. **Generate SKILL.md** with proper frontmatter and content
3. **Add scripts** if requested (using the python-dev agent for quality)
4. **Create reference files** if needed
5. **Validate** the entire skill
6. **Package** if for distribution if requested

Just tell me what skill you want to create, and I'll guide you through the process!
