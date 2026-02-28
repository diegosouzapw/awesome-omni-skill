---
name: governance
version: "2.1.0"
description: "Engineering standards and code governance enforcement with parallel agent execution. Triggers during code reviews, commits, and standards discussions. 70-90% token savings via delegated execution."
category: development
tags:
  - governance
  - code-review
  - standards
  - compliance
  - quality
  - automation
author: wrsmith108
repository: https://github.com/wrsmith108/governance-claude-skill
license: MIT
triggers:
  keywords:
    - code review
    - review this
    - commit
    - before I merge
    - standards
    - compliance
    - code quality
    - best practices
    - retro
    - retrospective
  explicit:
    - /governance
    - /review
    - /retro
    - /audit
delegates_to: reviewer
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
---

# Governance Skill

Enforces engineering standards and code quality policies with parallel agent execution for context efficiency.

> **Standards Document**: [standards.md](standards.md) (starter default — replace with your own)
> **Audit Script**: `npm run audit:standards`
> **Full Protocol**: [agent-prompt.md](agent-prompt.md)

---

## Execution Model (v2.1)

**Type**: Delegated Execution — delegates to the built-in `reviewer` subagent type.

This skill uses the `reviewer` subagent (available in all Claude Code installations — no custom agent required):

1. **Spawn** the reviewer agent
2. **Delegate** the governance task with instructions from `agent-prompt.md`
3. **Await** completion
4. **Report** the structured summary to the user

### Token Efficiency

| Execution Mode | Token Usage | Benefit |
|----------------|-------------|---------|
| Direct (v1.x) | ~3,000–8,000 | Full context in main thread |
| Delegated (v2.x) | ~500–800 | 70–90% token savings |

---

## How to Invoke

When this skill is triggered, read `agent-prompt.md` for the full protocol, then delegate:

### Code Review
```javascript
Task({
  description: "Governance code review",
  prompt: `[See agent-prompt.md for full instructions]
Review the following files for standards compliance:
- [list files or PR description]

Execute full code review workflow:
1. Run standards audit
2. Check each file against standards.md
3. Fix ALL issues immediately (zero deferral)
4. Return structured summary`,
  subagent_type: "reviewer"
})
```

### Pre-Commit Check
```javascript
Task({
  description: "Pre-commit governance",
  prompt: `[See agent-prompt.md for full instructions]
Run pre-commit governance checks:
1. typecheck, lint, format:check, test
2. Check for untracked source files
3. Return pass/fail with specific failures`,
  subagent_type: "reviewer"
})
```

### Retrospective
```javascript
Task({
  description: "Run retrospective",
  prompt: `[See agent-prompt.md for full instructions]
Run retrospective for [topic/sprint]:
1. Analyze completed issues
2. Gather metrics
3. Write report
4. Return summary`,
  subagent_type: "reviewer"
})
```

---

## Quick Reference

### Pre-Commit Checklist

```
[ ] npm run typecheck
[ ] npm run lint
[ ] npm run test
[ ] npm run audit:standards
[ ] No console.log statements
[ ] No hardcoded secrets
```

### Compliance Check

```bash
npm run audit:standards
node scripts/governance-check.mjs
```

---

*Full agent protocol, anti-pattern tables, and output format in [agent-prompt.md](agent-prompt.md)*
*Version 2.1.0 — Structural cleanup: flat layout, built-in reviewer agent, standards.md default added*
