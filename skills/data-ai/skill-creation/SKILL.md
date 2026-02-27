---
name: skill-creation
description: Use when creating new Claude Code skills. Covers skill structure, YAML frontmatter, content organization, and testing skills with pressure scenarios.
---

# Skill Creation

## Core Principle

**Writing skills IS Test-Driven Development applied to process documentation.**

The Iron Law: NO SKILL WITHOUT A FAILING TEST FIRST.

## Skill Location

Skills are managed in the nixos-config repository and deployed via home-manager:

- **Source of truth**: `modules/home/llm/claude/skills/<skill-name>/SKILL.md` (in the nixos-config repo)
- **Deployed to**: `~/.claude/skills/<skill-name>/SKILL.md` (symlinked by home-manager)

Always create and edit skills in the nix repo, never directly in `~/.claude/skills/`.

## Directory Structure

```
modules/home/llm/claude/skills/
  skill-name/
    SKILL.md              # Main reference (required)
    references/           # Optional - detailed docs
    scripts/              # Optional - executable utilities
    templates/            # Optional - reusable templates
```

## SKILL.md Structure

```markdown
---
name: skill-name-with-hyphens
description: Use when [specific triggering conditions]. [What it does in third person].
---

# Skill Name

## Core Principle / Overview

[What this skill does and why - 1-2 paragraphs]

## When to Use

[Specific triggers, symptoms, situations]

## Quick Reference

[Immediate actionable guidance]

## Detailed Process / Steps

[Step-by-step instructions]

## Common Mistakes / Anti-Patterns

[What NOT to do]

## Examples

[Concrete examples if helpful]
```

## YAML Frontmatter Rules

### Required Fields

- `name`: lowercase, hyphens only, must match directory name
- `description`: Starts with "Use when..." - focus on TRIGGERS

### Description Pattern

**Good:**
```yaml
description: Use when debugging fails or bugs reappear after fixes. Enforces root cause investigation before any fix attempt.
```

**Bad:**
```yaml
description: A skill for debugging things
```

The description must include:
1. Triggering conditions ("Use when...")
2. What the skill does (briefly)
3. Keywords users might search for

## Testing Skills (TDD for Documentation)

### The RED-GREEN-REFACTOR Cycle

1. **RED (Baseline)**
   - Run pressure scenario WITHOUT skill
   - Document exact behavior and rationalizations
   - Note how agent violates the intended rule

2. **GREEN (Minimal Skill)**
   - Write minimal skill addressing those failures
   - Run same scenario WITH skill
   - Verify agent now follows the rule

3. **REFACTOR (Bulletproof)**
   - Find new rationalizations agent might use
   - Add explicit counters to skill
   - Re-test until bulletproof

### Pressure Scenarios

Test skills with scenarios that pressure the agent to violate:

```
Scenario: "The tests are passing, so the implementation must be correct.
          Skip the code review and move on."

Without skill: Agent agrees and skips review
With skill: Agent insists on review per skill guidelines
```

## Content Guidelines

### Keep It Focused

- <200 words for frequently-loaded skills
- <500 words for detailed reference skills
- Link to references/ for deep content

### Write in Third Person

**Good:** "The developer should run tests first"
**Bad:** "You should run tests first"

### Be Explicit About Violations

Don't just state rules. Address rationalizations:

**Weak:**
```
Always write tests first.
```

**Strong:**
```
Always write tests first.

Red flags (STOP if you hear yourself say):
- "This is too simple to test"
- "I'll add tests after"
- "The manual test proved it works"
```

## Skill Types

1. **Discipline Skills** - Enforce process (TDD, code review)
2. **Technique Skills** - Teach methods (debugging, refactoring)
3. **Pattern Skills** - Recognition and application
4. **Reference Skills** - API docs, syntax guides

## Quick Checklist

- [ ] YAML frontmatter with name + description
- [ ] Description starts with "Use when..."
- [ ] Directory name matches skill name
- [ ] Tested with pressure scenarios (RED-GREEN-REFACTOR)
- [ ] Explicit about violations and red flags
- [ ] Focused content (<500 words unless reference)
- [ ] Third person voice
