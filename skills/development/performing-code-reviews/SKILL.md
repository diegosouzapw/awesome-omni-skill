---
name: performing-code-reviews
description: Use when reviewing code for quality, security, and maintainability. Enforces verification tooling as table stakes, loads skill-based review lenses, and produces structured actionable output.
---

# Performing Code Reviews

## Overview

Code review ensures quality gates are met before integration. This skill enforces verification tooling (tests, linter, type checker) as non-negotiable infrastructure, loads domain skills as review lenses, and produces structured output with categorized issues.

## The Review Standard

The primary purpose of code review is improving overall code health. The standard:

**Approve when the code improves overall health of the codebase, even if it isn't perfect.**

Seek continuous improvement, not perfection. Codebases degrade through small decrements in quality over time — your job is to prevent degradation while enabling forward progress.

### Guiding Principles

1. **Technical facts and data override opinions.** Cite specific principles, not preferences.
2. **Style guides are authoritative on style matters.** Don't debate style that's covered by a guide.
3. **When multiple approaches are equally valid, accept the author's choice.** Don't impose your preference without technical justification.
4. **Reviews serve an educational role.** Teaching developers about patterns and principles has value.

## When to Use

- After implementing a feature or fix, before merge
- When reviewing pull requests
- When auditing existing code for quality
- When a numbered step from a plan is complete

## Core Pattern

### Step 1: Load Skills

Load skills in this order to create review lenses:

1. `designing-software` — architecture, separation of concerns
2. `defensive-coding` — boundary validation, error handling
3. `writing-useful-tests` — test quality, coverage analysis
4. Language-specific skill based on file types:
   - Python: `howto-code-in-python`
5. `technical-writing` — clear review output

### Step 2: Run Verification Commands

Run and document results for each:

| Tool | Required | If Missing |
|------|----------|------------|
| Test suite | Yes | Critical issue |
| Linter (ruff/eslint) | Yes | Critical issue |
| Type checker (mypy/tsc) | Yes | Critical issue |
| Build | If applicable | Report result |

**Missing verification tooling blocks the review.** These tools are table stakes for maintainable code.

### Step 3: Review Implementation

Review **every line** you're assigned. Consider broader context and system impact. Apply each loaded skill as a lens:

#### Design (from `designing-software`)
- Does the overall architecture make sense?
- Do interactions between components make sense?
- Are concerns properly separated?
- Are abstractions at the right level?
- Does this change belong in the codebase?

#### Functionality
- Does the code accomplish what the author intended?
- Is the intended behavior good for users?
- Consider edge cases, concurrency issues, and error conditions.

#### Complexity
- Is the code more complex than necessary?
- Can code readers understand it quickly?
- Is there over-engineering for hypothetical future problems?

#### Defensive Coding (from `defensive-coding`)
- Inputs validated at boundaries?
- Error handling appropriate?
- Resources properly managed?

#### Style and Naming (from language skill)
- Style conventions followed?
- Idioms used correctly?
- Language-specific gotchas avoided?
- Variable, function, and class names communicate purpose?
- Names appropriately sized (not too short or unwieldy)?

#### Comments
- Comments explain *why*, not *what*?
- Is the code self-explanatory without excessive comments?
- Any misleading or outdated comments?

#### Cross-file Patterns
- Duplication across similar files?
- Inconsistent patterns in related components?
- Consistency with existing codebase?

### Step 4: Review Tests

For each source file:
1. Verify corresponding test file exists
2. List public methods/functions
3. Check each has test coverage
4. Verify tests test real behavior (not mocks)
5. Check error paths and edge cases
6. Tests actually fail when code breaks?

### Step 5: Categorize Issues

| Category | Criteria |
|----------|----------|
| **Critical** | Failing tests, security issues, missing verification tooling, data corruption risk |
| **Important** | Code organization, performance, missing tests, documentation gaps |
| **Minor** | Style preferences, naming, refactoring opportunities |

**Any Critical issue = review blocked.**

### Step 6: Produce Structured Output

Use this exact template:

```markdown
# Code Review: [Component/Feature Name]

## Status
**[APPROVED / BLOCKED - CHANGES REQUIRED]**

## Issue Summary
**Critical: [count] | Important: [count] | Minor: [count]**

## Verification Evidence
- Tests: [command] → [results]
- Linter: [command] → [results]
- Type checker: [command] → [results]

## Skills Applied
- [skill name]: [what it checked]

## Strengths
- [What the author did well — clean algorithms, thorough testing, good patterns]

## Critical Issues (count: N)
### N. [Issue title]
**Location**: `file:line`
**Issue**: [What's wrong]
**Impact**: [Why it matters]
**Fix**: [How to resolve]

## Important Issues (count: N)
[Same format]

## Minor Issues (count: N)
[Same format]

## Decision
**[APPROVED / BLOCKED - CHANGES REQUIRED]**
```

## Writing Review Comments

### Be Kind

Focus comments on the code, not the developer. The code is the subject, not the person.

| Instead of | Write |
|------------|-------|
| "Why did you use threads here?" | "The concurrency model adds complexity without clear performance benefit." |
| "You forgot to validate input" | "Input validation is missing at the entry point." |

### Explain Your Reasoning

Developers benefit when you explain *why*, not just *what*. Help them understand how the suggestion improves code health.

```markdown
**Issue**: Exception handling swallows the error context.
**Impact**: When this fails in production, debugging will be difficult because
the original exception is lost. Chained exceptions preserve the stack trace.
**Fix**: Use `raise NewException(...) from e` to chain exceptions.
```

### Balance Direction with Discovery

Point out problems and let the developer decide how to fix them when reasonable. They often know the context better and may produce a better solution. Provide direct fixes when:
- The solution is non-obvious
- Time is constrained
- The issue is straightforward

### Acknowledge Strengths

Note what the developer did well. This provides positive reinforcement and helps you notice good patterns.

## Quick Reference

```
1. Load skills: designing-software → defensive-coding → writing-useful-tests → language skill
2. Run verification: tests, linter, type checker (missing = Critical)
3. Review: design, functionality, complexity, defensive coding, style, comments, cross-file
4. Review test coverage (file-to-test mapping)
5. Categorize: Critical (blocks) / Important / Minor
6. Output structured report with strengths and issues
```

## Common Mistakes

| Mistake | Why It Fails | Correct Approach |
|---------|--------------|------------------|
| Skipping verification tools | Misses entire categories of issues | Run tests, linter, type checker first |
| Reviewing without skill lenses | Ad-hoc, inconsistent coverage | Load skills systematically |
| Generic "looks good" | No actionable feedback | Use structured template with Location/Issue/Impact/Fix |
| Only reviewing within files | Miss duplication, inconsistency | Compare similar files across module |
| Approving with Critical issues | Quality gates not enforced | Any Critical = blocked, no exceptions |
| Missing verification = "note it" | Tooling is table stakes | Missing tooling is Critical, blocks review |
| Focusing only on problems | Misses teaching opportunity | Acknowledge strengths too |
| Opinions without reasoning | Author can't learn or evaluate | Always explain the *why* |
| Blocking on personal preference | Slows progress without benefit | Accept valid alternatives |

## Anti-Rationalizations

- "The code works, tooling can come later" — Tooling is infrastructure. Without it, you can't verify the code keeps working. Block until configured.
- "It's a small change, full review is overkill" — Small changes still need verification evidence. Use the template.
- "I'll note the test gaps as follow-up" — Missing tests for implemented code is Important. Track it in the review, don't defer.
- "The linter/type errors are false positives" — Fix the configuration or suppress with comments. Passing tools is the baseline.
- "I just prefer it the other way" — That's not a valid reason to block. Cite principles, not preferences.
- "They should have known better" — Review is for teaching, not judging. Be kind.

## Summary

1. **Approve when code improves health.** Seek continuous improvement, not perfection.
2. **Verification tooling is non-negotiable.** Missing linter, type checker, or tests = Critical issue, review blocked.
3. **Load skills as review lenses.** Each skill provides specific questions to ask.
4. **Review every line.** Design, functionality, complexity, tests, naming, comments, cross-file patterns.
5. **Be kind and explain why.** Focus on code, not person. Teaching has value.
6. **Acknowledge strengths.** Note what was done well.
7. **Any Critical issue blocks the review.** No exceptions.
