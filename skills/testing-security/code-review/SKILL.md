---
name: code-review
description: Code reviewer focusing on correctness, regressions, security, and test coverage - P0/P1/P2 severity
---

# Code Review Skill

Reviews code for correctness, security, bugs, and best practices with severity ratings.

---

## Purpose

The Code Review skill provides thorough code analysis:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      CODE REVIEW FRAMEWORK                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐        │
│  │ CORRECTNESS│  │  SECURITY  │  │   QUALITY  │  │  PATTERNS  │        │
│  └────────────┘  └────────────┘  └────────────┘  └────────────┘        │
│       │               │               │               │                 │
│       ▼               ▼               ▼               ▼                 │
│   • Logic bugs    • Injection     • Readability   • AGENTS.md          │
│   • Edge cases    • Auth/Authz    • Maintainable  • Conventions        │
│   • Regressions   • Data exposure • Performance   • Consistency        │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────┐      │
│  │                    SEVERITY RATINGS                           │      │
│  │  P0: Critical  |  P1: Important  |  P2: Nice-to-have         │      │
│  └──────────────────────────────────────────────────────────────┘      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Severity Levels

### P0 - Critical (Must Fix)
Issues that MUST be fixed before merge:
- **Security vulnerabilities** (injection, XSS, auth bypass)
- **Data corruption/loss** risks
- **Crashes** or critical runtime errors
- **Breaking changes** without migration
- **Business logic errors** that cause wrong behavior

### P1 - Important (Should Fix)
Issues that SHOULD be fixed:
- **Logic errors** in edge cases
- **Performance issues** with significant impact
- **Error handling** gaps
- **Test coverage** gaps for critical paths
- **Pattern violations** causing maintenance burden

### P2 - Nice-to-have (Consider Fixing)
Issues that would improve code:
- **Code style** inconsistencies
- **Minor performance** improvements
- **Documentation** gaps
- **Refactoring** opportunities
- **Minor pattern** deviations

---

## Review Categories

### 1. Correctness Review

Check for logic errors and bugs:

```
□ Business Logic
  - Does code implement requirements correctly?
  - Are calculations accurate?
  - Are conditions/branching correct?

□ Edge Cases
  - Null/empty handling
  - Boundary conditions
  - Error states

□ State Management
  - State transitions correct?
  - No stale state issues?
  - Proper initialization?

□ Async Operations
  - Race conditions?
  - Proper await usage?
  - Error propagation?
```

### 2. Security Review

Check for security vulnerabilities:

```
□ Input Validation
  - All user inputs validated?
  - No injection vulnerabilities?
  - Proper sanitization?

□ Authentication & Authorization
  - Auth checks in place?
  - Proper permission checks?
  - Session handling?

□ Data Protection
  - Sensitive data not exposed?
  - Proper encryption?
  - No hardcoded secrets?

□ API Security
  - Proper error messages (no info leak)?
  - Rate limiting consideration?
  - HTTPS enforced?
```

### 3. Quality Review

Check code quality:

```
□ Readability
  - Clear naming?
  - Reasonable function length?
  - Comments where needed?

□ Maintainability
  - Single responsibility?
  - No code duplication?
  - Testable design?

□ Performance
  - No unnecessary operations?
  - Proper list handling?
  - Widget rebuild optimization?

□ Error Handling
  - All errors caught?
  - Meaningful error messages?
  - Proper recovery?
```

### 4. Pattern Compliance Review (AGENTS.md)

Check adherence to project patterns:

```
□ State Management
  - Using StateNotifier correctly?
  - State class has copyWith?
  - Provider properly defined?

□ Models
  - Using Equatable?
  - Using ReturnValue for JSON?
  - Has props override?

□ Styling
  - Using TypographyTheme?
  - Using ColorApp?
  - Using Gap/SizeApp?

□ Widget Structure
  - Separate widget classes?
  - No _buildX methods?
  - ConsumerWidget where needed?

□ File Organization
  - Correct folder structure?
  - Proper file naming?
  - Correct layer separation?
```

### 5. Test Coverage Review

Check test adequacy:

```
□ Unit Tests
  - Critical logic tested?
  - Edge cases covered?
  - Error paths tested?

□ Widget Tests
  - UI states tested?
  - User interactions tested?

□ Mock Usage
  - Proper mocking?
  - No real API calls in tests?
```

---

## Review Process

### Step 1: Gather Context

```
1. Identify files to review
   - New files created
   - Modified files
   - Related test files

2. Load context
   - AGENTS.md patterns
   - Original requirements (if available)
   - Related existing code
```

### Step 2: Systematic Review

```
For each file:
  1. Read through completely
  2. Check correctness
  3. Check security
  4. Check quality
  5. Check patterns
  6. Note all findings
```

### Step 3: Categorize Findings

```
For each finding:
  1. Assign severity (P0/P1/P2)
  2. Identify category
  3. Provide specific location
  4. Explain the issue
  5. Suggest fix
```

### Step 4: Generate Report

```
Compile findings into structured report:
  - Executive summary
  - Findings by severity
  - Findings by category
  - Recommendations
```

---

## Common Issues Checklist

### Flutter/Dart Specific

```
□ Widget Rebuilds
  - Const constructors where possible?
  - Keys used appropriately?
  - No expensive operations in build()?

□ Async/Await
  - Proper Future handling?
  - No fire-and-forget without intent?
  - Cancellation handled?

□ Null Safety
  - Proper null checks?
  - No unnecessary null assertions (!)?
  - Late variables justified?

□ Memory Leaks
  - Listeners disposed?
  - Controllers disposed?
  - Streams closed?
```

### Riverpod Specific

```
□ Provider Definition
  - Correct provider type?
  - Proper scoping?
  - No circular dependencies?

□ State Updates
  - Using copyWith correctly?
  - No direct state mutation?
  - Proper AsyncValue handling?

□ Ref Usage
  - Using read vs watch correctly?
  - No ref in async callbacks?
```

### API/Data Specific

```
□ Response Handling
  - All fields mapped?
  - ReturnValue used correctly?
  - Error responses handled?

□ Request Building
  - Proper parameters?
  - Headers correct?
  - Body formatted correctly?
```

---

## Output Template

```markdown
# Code Review: {Feature/PR Name}

## Metadata
- **Date**: {YYYY-MM-DD}
- **Files Reviewed**: {count}
- **Reviewer**: Claude Code / Codex CLI

---

## Executive Summary

| Severity | Count | Status |
|----------|-------|--------|
| P0 (Critical) | {X} | {BLOCKING/CLEAR} |
| P1 (Important) | {X} | |
| P2 (Nice-to-have) | {X} | |

**Verdict**: {APPROVE / REQUEST CHANGES / NEEDS DISCUSSION}

---

## P0 - Critical Issues

{If none: "No critical issues found."}

### P0-1: {Issue Title}

**File**: `path/to/file.dart`
**Line**: {line number}
**Category**: {Security/Correctness/etc.}

**Issue**:
{Description of the problem}

**Code**:
```dart
// Current code
{problematic code}
```

**Impact**:
{What could go wrong}

**Suggested Fix**:
```dart
// Fixed code
{corrected code}
```

---

## P1 - Important Issues

### P1-1: {Issue Title}

**File**: `path/to/file.dart`
**Line**: {line number}
**Category**: {category}

**Issue**: {description}

**Suggested Fix**: {fix}

---

## P2 - Nice-to-have

### P2-1: {Issue Title}

**File**: `path/to/file.dart`
**Line**: {line number}

**Suggestion**: {improvement suggestion}

---

## Pattern Compliance

### AGENTS.md Adherence

| Pattern | Status | Notes |
|---------|--------|-------|
| State Management | ✓/✗ | {notes} |
| Model Pattern | ✓/✗ | {notes} |
| Styling | ✓/✗ | {notes} |
| Widget Structure | ✓/✗ | {notes} |
| File Organization | ✓/✗ | {notes} |

---

## Test Coverage Assessment

| Area | Coverage | Recommendation |
|------|----------|----------------|
| {area} | {level} | {recommendation} |

---

## Files Reviewed

| File | Status | Issues |
|------|--------|--------|
| `path/to/file.dart` | {OK/ISSUES} | P0: {X}, P1: {X}, P2: {X} |

---

## Recommendations

### Must Do (Blocking)
1. {P0 issue fix}

### Should Do
1. {P1 issue fix}

### Consider
1. {P2 improvement}

---

## Approval Status

{APPROVED / APPROVED WITH COMMENTS / CHANGES REQUESTED}

{Final notes}
```

---

## Prompt

When user invokes `/code-review`, execute:

```
I will now conduct a thorough code review.

## Gathering Context

1. Identifying files to review...
   - New files: {list}
   - Modified files: {list}

2. Loading project patterns from AGENTS.md...

## Reviewing Code

### File: {path/to/file.dart}

**Correctness Check**:
- Logic: {findings}
- Edge cases: {findings}
- State management: {findings}

**Security Check**:
- Input validation: {findings}
- Auth: {findings}
- Data protection: {findings}

**Quality Check**:
- Readability: {findings}
- Performance: {findings}
- Error handling: {findings}

**Pattern Compliance**:
- AGENTS.md adherence: {findings}

[Repeat for each file]

## Summary

### P0 Issues: {count}
{List critical issues}

### P1 Issues: {count}
{List important issues}

### P2 Issues: {count}
{List suggestions}

## Verdict

{APPROVE / REQUEST CHANGES}

{Reasoning and required actions}
```

---

## Quick Commands

```
/code-review                  - Review recent changes
/code-review path/to/file     - Review specific file
/code-review --staged         - Review staged changes
/code-review --security       - Security-focused review
/code-review --patterns       - Pattern compliance only
```
