---
name: Compound Learnings
description: Capture insights and corrections to make future work easier. Implements RSI (Recursive Self-Improvement) through automated correction detection and pattern extraction.
version: 2.0.0
author: CircleTel Engineering
dependencies: error-registry
---

# Compound Learnings

> "Each unit of engineering work should make subsequent units easier—not harder."

This skill implements the compound engineering philosophy AND **Recursive Self-Improvement (RSI)**: systematically capturing what you learn during development to accelerate future work. Now includes automatic correction detection to learn from mistakes.

## When This Skill Activates

This skill should be invoked:
- After completing a significant task or feature
- When you discover a pattern that will be reused
- After debugging a tricky issue
- When you find a better way to do something
- At the end of a productive session
- **NEW**: When Claude is corrected during a task (auto-detected)

**Keywords**: compound, learnings, document what I learned, capture pattern, session summary, retrospective, correction, mistake, wrong, actually, instead

---

## RSI: Self-Updating Through Corrections

**NEW in v2.0**: This skill now detects and learns from corrections automatically.

### Correction Detection Triggers

The skill watches for these patterns indicating a correction:
- "No, actually..." / "That's not right" / "That's wrong"
- "Instead, do..." / "The correct way is..."
- User provides alternative code immediately after Claude's output
- User references documentation contradicting Claude's suggestion
- "Don't do that" / "Never do X"

### Correction Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                    RSI CORRECTION LOOP                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    CLAUDE OUTPUT ──► USER CORRECTION ──► DETECT ──► EXTRACT     │
│                                                │        │        │
│                                                │        ▼        │
│                                                │    STORE IN     │
│                                                │   corrections/  │
│                                                │        │        │
│                                                │        ▼        │
│    FUTURE OUTPUTS ◄──── APPLY RULE ◄──── SYNTHESIZE RULE        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Command: /correction

Use `/correction` to manually capture a correction.

```bash
# After being corrected
/correction

# With specific context
/correction "Use admin_notes not notes column"
```

### Correction Entry Format

```markdown
# Correction: [Brief Description]

**Date**: YYYY-MM-DD
**Session**: [session-name]
**Skill Affected**: [which skill was wrong]

## Original Approach
[What Claude did/suggested]

## Correction Received
[How user corrected Claude]

## Learning Extracted
[Rule derived from correction]

## Application
- **Update to**: [Which skill/pattern to update]
- **Rule**: [Concrete rule to add]

## Validation
- [ ] Rule added to skill
- [ ] No regression in related areas
```

## The Compound Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                    COMPOUND ENGINEERING CYCLE                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    PLAN ──────► WORK ──────► REVIEW ──────► COMPOUND            │
│      │                                          │                │
│      │                                          │                │
│      └──────────── Future work is easier ◄──────┘                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Command: /compound

Use `/compound` to trigger the learning capture workflow.

### Usage

```bash
# After completing work
/compound

# With specific focus
/compound "console migration patterns"

# Quick pattern capture
/compound --pattern "API error handling"
```

## What to Capture

### 1. Friction Points (What Slowed You Down)

```markdown
## Friction: [Brief Title]

**Task**: What were you trying to do?
**Obstacle**: What slowed you down?
**Root Cause**: Why did this happen?
**Solution**: How did you resolve it?
**Prevention**: How can future tasks avoid this?
```

### 2. Patterns (Reusable Solutions)

```markdown
## Pattern: [Pattern Name]

**Context**: When to use this pattern
**Problem**: What problem does it solve?
**Solution**: The pattern itself (with code)
**Trade-offs**: Pros and cons
**Examples**: Where it's used in CircleTel
```

### 3. Discoveries (Things You Learned)

```markdown
## Discovery: [Brief Title]

**Context**: What were you working on?
**Insight**: What did you learn?
**Impact**: How does this change future work?
**References**: Related files, docs, or code
```

### 4. Shortcuts (Time Savers)

```markdown
## Shortcut: [Brief Title]

**Before**: How it was done previously
**After**: The faster/better way
**Savings**: Estimated time saved
**Command/Code**: The actual shortcut
```

## File Organization

```
.claude/skills/compound-learnings/
├── SKILL.md                          # This file
├── templates/
│   ├── learning-template.md          # Template for learnings
│   ├── pattern-template.md           # Template for patterns
│   └── correction-template.md        # NEW: Template for corrections
├── corrections/                       # NEW: RSI corrections
│   └── YYYY-MM-DD_topic.md           # Detected corrections
├── extracted-rules/                   # NEW: Synthesized rules
│   └── rule-name.md                  # Rules derived from corrections
└── learnings/
    ├── YYYY-MM-DD_topic.md           # Date-prefixed learnings
    └── patterns/
        ├── api-routes.md             # API route patterns
        ├── error-handling.md         # Error handling patterns
        ├── testing.md                # Testing patterns
        └── migrations.md             # Migration patterns
```

## Extracted Rules (RSI Output)

Rules synthesized from corrections are stored in `extracted-rules/`:

```markdown
# Rule: [Rule Name]

**Derived From**: corrections/YYYY-MM-DD_topic.md
**Confidence**: high | medium | low
**Applications**: N times

## The Rule
[Clear, actionable rule]

## Example
[Code or workflow example]

## Counter-Example
[What NOT to do - the original mistake]
```

### Example Extracted Rules

| Rule | Derived From | Applications |
|------|--------------|--------------|
| Use `admin_notes` not `notes` | Correction 2026-02-06 | 3 |
| Always `await context.params` | Correction 2026-02-08 | 5 |
| Add service_role policy | Correction 2026-02-09 | 4 |

## Integration with MEMORY.md

After capturing learnings, the most important insights should be summarized in MEMORY.md for cross-session persistence:

```markdown
# In MEMORY.md

## Key Patterns
- API routes: Use apiLogger, not console.log (see compound-learnings/patterns/api-routes.md)
- Error handling: Use lib/errors/app-error.ts classes (see patterns/error-handling.md)

## Recent Learnings
- 2026-02-09: Console migration - batch by domain, use parallel agents
```

## Best Practices

### DO Capture

- Solutions to problems that took >30 minutes to solve
- Patterns you'll use more than twice
- Gotchas specific to CircleTel's stack
- Commands or workflows that saved significant time
- Architectural decisions and their rationale

### DON'T Capture

- Obvious things documented elsewhere
- One-off fixes unlikely to recur
- External library documentation (just link to it)
- Temporary workarounds (capture the proper fix instead)

## Example Compound Session

```
You: /compound

Claude: Let me analyze this session for compoundable learnings...

## Session Summary: Console Migration (2026-02-09)

### Patterns Extracted
1. **Batch Migration Pattern**: Group files by domain (cron, payments, admin)
   for focused, reviewable commits.

2. **Logger Selection Pattern**: Match logger to domain:
   - `apiLogger` - API routes
   - `cronLogger` - Scheduled tasks
   - `webhookLogger` - External webhooks
   - `paymentLogger` - Payment flows

### Friction Points
1. **Duplicate catch blocks**: Automated migration introduced syntax errors.
   Prevention: Always run type-check before committing bulk changes.

### Time Savings
- Parallel subagents reduced migration time by ~70%
- Domain-based batching made PRs reviewable

Shall I save these to learnings/2026-02-09_console-migration.md?
```

## Metrics

Track the compound effect over time:

| Metric | Description |
|--------|-------------|
| Patterns Created | Number of reusable patterns documented |
| Friction Resolved | Issues prevented by prior learnings |
| Time Saved | Estimated hours saved by patterns |
| MEMORY.md Updates | Cross-session insights captured |

## Related Skills

- `session-manager` - Name and resume sessions
- `refactor` - Identify improvement opportunities
- `tech-debt-analyzer` - Track technical debt
- `filesystem-context` - Persist context across sessions

---

**Philosophy**: Don't just ship code—ship knowledge. Every task completed is an investment in your future velocity.
