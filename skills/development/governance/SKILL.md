---
name: governance
version: "2.0.0"
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
delegates_to: governance-specialist
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

> **Standards Document**: [docs/architecture/standards.md](../../../docs/architecture/standards.md)
> **Audit Script**: `npm run audit:standards`

---

## Execution Model (v2.0)

**Type**: Delegated Execution

This skill operates through the `governance-specialist` subagent for context efficiency:

1. **Spawn** the specialist agent in parallel
2. **Delegate** the governance task with clear instructions
3. **Await** completion without intervention
4. **Report** the structured summary to the user

### Token Efficiency

| Execution Mode | Token Usage | Benefit |
|----------------|-------------|---------|
| Direct (v1.x) | ~3,000-8,000 | Full context in main thread |
| Delegated (v2.0) | ~500-800 | 70-90% token savings |

---

## How to Invoke (Orchestrator Instructions)

When this skill is triggered, delegate to a parallel agent:

### Code Review
```javascript
Task({
  description: "Governance code review",
  prompt: `Review the following files for standards compliance:
- [list files or PR description]

Execute full code review workflow:
1. Run standards audit
2. Check each file against standards
3. Fix ALL issues immediately (zero deferral)
4. Create code review report
5. Return structured summary`,
  subagent_type: "reviewer"
})
```

### Pre-Commit Check
```javascript
Task({
  description: "Pre-commit governance",
  prompt: `Run pre-commit governance checks:
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
  prompt: `Run retrospective for [topic/sprint]:
1. Analyze completed issues
2. Gather metrics
3. Write report
4. Return summary`,
  subagent_type: "reviewer"
})
```

---

## Behavioral Classification

**Zero Deferral Policy**: The specialist fixes ALL issues immediately - no tickets, no "later".

| Pattern | Status |
|---------|--------|
| "Would you like me to fix these issues?" | Anti-pattern |
| "Created ticket to track this for later." | Anti-pattern |
| "Found 5 issues. All fixed. Commits: abc123, def456." | Correct |

---

## Quick Start

### 1. Run Setup Check

```bash
node .claude/skills/governance/scripts/governance-check.mjs
```

### 2. Two-Document Model

| Document | Purpose | Update Frequency |
|----------|---------|------------------|
| **CLAUDE.md** | AI operational context | Frequent |
| **standards.md** | Engineering policy (authoritative) | Infrequent |

### 3. Common Operations

```bash
# Check standards compliance
npm run audit:standards

# Pre-commit suite
npm run typecheck && npm run lint && npm run test
```

---

## Core Standards Quick Reference

### Code Quality (§1)

| Standard | Requirement | Check |
|----------|-------------|-------|
| TypeScript | Strict mode enabled | `npm run typecheck` |
| No `any` types | Use `unknown` for external data | Audit script |
| Max file length | 500 lines | Audit script |
| JSDoc | Required for public APIs | Audit script |

### Naming Conventions (§1.2)

| Element | Convention | Example |
|---------|------------|---------|
| Components | PascalCase | `Button.tsx` |
| Utilities | camelCase | `userService.ts` |
| Constants | SCREAMING_SNAKE | `MAX_RETRIES` |
| DB columns | snake_case | `user_id` |

### Testing (§2)

| Layer | Target |
|-------|--------|
| Unit tests | 90-100% |
| API routes | 90-100% |
| E2E critical | 100% |

---

## Compliance Check

```bash
npm run audit:standards
```

Checks:
- TypeScript strict mode
- No `any` types in source
- File length limits (500 lines)
- JSDoc on exported functions
- Test files exist

---

## Anti-Patterns vs Correct Patterns

### Code Quality

| Anti-Pattern | Correct Pattern |
|--------------|-----------------|
| ❌ `any` type | ✅ `unknown` for external data |
| ❌ 600+ line files | ✅ Split at 500 lines |
| ❌ Hardcoded secrets | ✅ Environment variables |

### Workflow

| Anti-Pattern | Correct Pattern |
|--------------|-----------------|
| ❌ Commit to main | ✅ Feature branches with PR |
| ❌ Skip audit | ✅ Run `audit:standards` before commit |
| ❌ Defer issues | ✅ Fix immediately |

---

## Pre-Commit Checklist

```
[ ] npm run typecheck
[ ] npm run lint
[ ] npm run test
[ ] npm run audit:standards
[ ] No console.log statements
[ ] No hardcoded secrets
```

---

## PR Review Checklist

| Category | Criteria |
|----------|----------|
| Correctness | Does it work? Edge cases? |
| Security | No secrets, injection, auth bypasses |
| Performance | No N+1, memory leaks |
| Testing | Tests exist for new code |

---

## Output Format (Specialist Response)

The governance-specialist returns structured summaries:

```
- **Task:** Code review for feature X
- **Checks Run:** typecheck, lint, audit:standards
- **Results:**
  - Found 3 issues: 1 missing test, 1 type issue, 1 style
  - All issues fixed in-place
- **Issues Fixed:** 3 (commits: abc1234, def5678, ghi9012)
- **Report:** docs/code_review/2026-01-28-feature-x.md
- **Status:** PASS
```

---

*Updated: January 2026*
*Version 2.0.0 - Delegated Execution Model*
