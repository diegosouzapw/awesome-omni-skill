---
name: code-review
description: Systematic code review checklist covering correctness, security, performance, and maintainability. Use when reviewing pull requests, auditing changes, or checking code quality across any language.
metadata:
  version: "1.0"
---

# Code Review

## Decision Tree

```
Review request → What scope?
    ├─ Full PR review → Follow all 6 steps below
    ├─ Security-focused → Steps 1, 3 + security-audit skill
    ├─ Quick sanity check → Steps 1, 2, 6 only
    └─ Performance review → Steps 1, 4 + performance-audit skill
```

## Process

1. **Context** - Read changed files, understand intent from PR description or commit messages
2. **Correctness** - Logic errors, edge cases, off-by-ones, null/undefined handling
3. **Security** - Injection, auth bypass, data exposure, OWASP top 10
4. **Performance** - N+1 queries, unnecessary allocations, algorithm complexity
5. **Maintainability** - Naming, structure, DRY, single responsibility
6. **Tests** - Coverage of changed code, edge cases, assertion quality

## Severity Levels

| Level | Meaning | Action |
|-------|---------|--------|
| **CRITICAL** | Security vulnerability, data loss, crash | Must fix before merge |
| **MAJOR** | Logic error, missing error handling, perf issue | Should fix before merge |
| **MINOR** | Style, naming, documentation gap | Fix or acknowledge |
| **NITPICK** | Preference, alternative approach | Optional |

## Output Format

For each finding:
```
[SEVERITY] file:line - Description
  Why: Impact explanation
  Fix: Suggested code change
```

## Checklist

- [ ] No hardcoded secrets, tokens, or credentials
- [ ] Error paths handled (not just happy path)
- [ ] No unvalidated user input reaches DB/shell/HTML
- [ ] New dependencies justified and audited
- [ ] Breaking changes documented
- [ ] Tests cover the changed behavior
- [ ] No commented-out code committed
- [ ] No debug logging left in

## Anti-Patterns

- **Rubber-stamping** - Approving without reading. Every file deserves attention.
- **Style wars** - Debating formatting that a linter should handle. Automate it.
- **Rewrite requests** - Asking for a total rewrite in review. That's a design discussion, not a review.
- **Scope creep** - Requesting unrelated improvements. File separate issues instead.
