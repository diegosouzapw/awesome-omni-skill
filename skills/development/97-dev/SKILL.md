---
name: 97-dev
description: Apply timeless programming wisdom from "97 Things Every Programmer Should Know" when writing, reviewing, or refactoring code. Use for design decisions, code quality checks, professional development guidance, testing strategies, and workflow optimization.
---

# 97-dev: Programmer's Wisdom

Distilled principles from 97 Things Every Programmer Should Know. Apply when writing, reviewing, or making design decisions.

## Core Philosophy

**Code is design.** Software development is a creative discipline requiring craftsmanship, not mechanical construction.

**The code tells the truth.** Documentation lies, comments decay - only executable code reveals actual behavior. Make code self-explanatory.

**Care about your code.** Excellence stems from attitude, not just knowledge. Craft elegant code that is clearly correct.

## Quick Principles

| Principle | One-liner |
|-----------|-----------|
| Simplicity | Remove everything unnecessary; less is more |
| Boy Scout | Leave code cleaner than you found it |
| DRY | Single authoritative representation for each piece of knowledge |
| SRP | One reason to change per class/module/function |
| Comments | Comment only what code cannot say - explain *why*, not *what* |
| Tech debt | Pay immediately or track the compounding interest |
| Testing | Non-negotiable professional obligation |
| Errors | Always check, always handle, every time |
| Next commit | Know exactly what you're committing before you start |
| Users | You are not the user - observe, don't assume |

## Detailed References

Load these when you need deeper guidance on specific topics:

### [references/simplicity.md](references/simplicity.md)
**When:** Refactoring bloated code, making architectural decisions, deciding what to remove, questioning if features are needed. Covers: Beauty in simplicity, reduction over addition, improving by removing, code as design.

### [references/quality.md](references/quality.md)
**When:** Code review, enforcing standards, improving maintainability, designing APIs and interfaces. Covers: Boy Scout Rule, DRY principle, Single Responsibility, interface design, code as truth.

### [references/professionalism.md](references/professionalism.md)
**When:** Career decisions, team dynamics, handling pressure, technical debt discussions, attitude check. Covers: Professional responsibility, caring about code, long-term thinking, prudent debt management.

### [references/testing.md](references/testing.md)
**When:** Writing tests, handling errors, debugging issues, arguing for test coverage, writing comments. Covers: Testing as engineering rigor, error handling discipline, debugging strategy, comment guidelines.

### [references/learning.md](references/learning.md)
**When:** Professional development, skill building, code reading sessions, understanding complexity limits. Covers: Continuous learning strategies, deliberate practice, reading code, knowing your limits.

### [references/workflow.md](references/workflow.md)
**When:** Planning work, commit strategy, user research, daily development practices. Covers: Know your next commit, you are not the user, version control practices, breaking things safely.

## Checklist

**Writing code:**
- [ ] Single responsibility per function/class?
- [ ] Any duplication to extract?
- [ ] Anything removable without losing functionality?
- [ ] Names descriptive enough to skip comments?
- [ ] Would I maintain this for years?

**Reviewing code:**
- [ ] Leaves codebase cleaner?
- [ ] Error cases handled?
- [ ] Interface easy to use correctly?
- [ ] Matches existing patterns?

**Debugging:**
- [ ] Ruled out my own code first?
- [ ] Isolated problem systematically?
- [ ] Testing assumptions, not seeking confirmation?

## Source

[97 Things Every Programmer Should Know](https://github.com/97-things/97-things-every-programmer-should-know) - O'Reilly, Creative Commons.
