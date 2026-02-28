---
name: code-reviewer
description: Perform systematic code reviews following best practices and team standards
version: 1.0.0
author: AgentSkills.NET Examples
tags:
  - code-review
  - quality
  - workflow
allowed-tools:
  - filesystem
  - git
---

# Code Reviewer

Conduct thorough, systematic code reviews following industry best practices.

## Purpose

This skill guides you through performing comprehensive code reviews that improve code quality, catch bugs, maintain standards, and facilitate knowledge sharing.

## Instructions

### Phase 1: Initial Assessment (2-3 minutes)

Before diving into code:

1. **Read the PR/MR description**
   - Understand the goal and context
   - Note any special considerations
   - Check linked issues or tickets

2. **Review the change overview**
   - Check files changed count (>20 files may need splitting)
   - Look at additions/deletions ratio
   - Identify areas of concern

3. **Verify CI/Build Status**
   - Ensure all tests pass
   - Check build succeeds
   - Review any warnings
   - Note failing checks to address

### Phase 2: Code Quality Review (10-15 minutes)

Examine code for quality issues:

1. **Readability**
   - Clear variable and function names
   - Appropriate comments for complex logic
   - Consistent formatting
   - Reasonable function/method length

2. **Code Structure**
   - Single Responsibility Principle
   - DRY (Don't Repeat Yourself)
   - Appropriate abstraction levels
   - Logical organization

3. **Naming Conventions**
   - Follows project standards
   - Descriptive and meaningful
   - Consistent across codebase
   - No abbreviations unless standard

4. **Code Smells**
   - Magic numbers or strings
   - Excessive complexity
   - Large classes or methods
   - Tight coupling

### Phase 3: Logic and Correctness (10-15 minutes)

Verify the implementation:

1. **Logic Review**
   - Implementation matches requirements
   - Edge cases handled
   - Error conditions addressed
   - Boundary conditions checked

2. **Potential Bugs**
   - Off-by-one errors
   - Null/undefined handling
   - Resource leaks
   - Race conditions
   - Integer overflow

3. **Algorithm Efficiency**
   - Reasonable time complexity
   - Appropriate data structures
   - No unnecessary iterations
   - Optimizable sections noted

### Phase 4: Testing (5-10 minutes)

Evaluate test coverage:

1. **Test Presence**
   - New code has tests
   - Modified code has updated tests
   - Tests are meaningful

2. **Test Quality**
   - Tests actually verify behavior
   - Edge cases covered
   - Error cases tested
   - Tests are maintainable

3. **Test Organization**
   - Clear test names
   - Arrange-Act-Assert structure
   - No test interdependencies

### Phase 5: Security Review (5-10 minutes)

Check for security issues:

1. **Input Validation**
   - All user input validated
   - Sanitization applied
   - Type checking performed

2. **Common Vulnerabilities**
   - SQL injection risks
   - XSS vulnerabilities
   - CSRF protection
   - Path traversal issues

3. **Authentication & Authorization**
   - Proper access controls
   - Session management
   - Secure credential handling

4. **Data Protection**
   - Sensitive data encrypted
   - No secrets in code
   - PII handled properly

### Phase 6: Documentation (3-5 minutes)

Verify documentation:

1. **Code Documentation**
   - Public APIs documented
   - Complex logic explained
   - Non-obvious decisions noted

2. **README Updates**
   - New features documented
   - Breaking changes noted
   - Examples provided

3. **API Documentation**
   - Parameters documented
   - Return values described
   - Error cases listed

## Providing Feedback

Structure your review comments effectively:

### Severity Levels

Use these severity indicators:

- 🔴 **CRITICAL**: Must fix before merge (blocks merge)
- 🟠 **MAJOR**: Should fix before merge (strong suggestion)
- 🟡 **MINOR**: Could improve (optional)
- 💡 **SUGGESTION**: Alternative approach (consider)
- ❓ **QUESTION**: Need clarification
- ✅ **PRAISE**: Good work (acknowledge)

### Comment Template

```
[SEVERITY] [Category]: Brief title
Line XX: Specific issue
Reason: Why this matters
Suggestion: How to fix

Example:
[code example if helpful]
```

## Examples

### Example 1: Security Issue

```
🔴 CRITICAL: SQL Injection Risk
Line 45: `query = "SELECT * FROM users WHERE id = " + userId`

Reason: Direct string concatenation allows SQL injection attacks
Suggestion: Use parameterized queries

Example:
query = "SELECT * FROM users WHERE id = @userId"
command.Parameters.AddWithValue("@userId", userId)
```

### Example 2: Logic Bug

```
🟠 MAJOR: Off-by-One Error
Line 67: `for (int i = 0; i <= array.Length; i++)`

Reason: Loop will throw IndexOutOfRangeException on last iteration
Suggestion: Change to `i < array.Length`
```

### Example 3: Code Quality

```
🟡 MINOR: Magic Number
Line 89: `if (count > 100) { ... }`

Reason: Unclear what 100 represents
Suggestion: Extract to named constant

Example:
private const int MaxItemsPerPage = 100;
if (count > MaxItemsPerPage) { ... }
```

### Example 4: Positive Feedback

```
✅ PRAISE: Error Handling
Lines 120-135: Excellent error handling with specific messages

This makes debugging much easier. Great work!
```

## Best Practices

1. **Be constructive** - Focus on the code, not the person
2. **Explain reasoning** - Help others learn
3. **Suggest solutions** - Don't just point out problems
4. **Acknowledge good work** - Positive feedback matters
5. **Ask questions** - Understand intent before criticizing
6. **Be timely** - Review promptly to unblock others
7. **Be thorough** - But know when to stop
8. **Focus on important issues** - Don't nitpick style if tests fail

## Common Issues Checklist

- [ ] All tests pass
- [ ] No console.log / debug statements left
- [ ] No commented-out code
- [ ] No TODO comments without tickets
- [ ] Error messages are user-friendly
- [ ] Logging is appropriate (not too much/little)
- [ ] No hardcoded secrets or credentials
- [ ] Database migrations included if schema changed
- [ ] Dependencies updated in package files
- [ ] Breaking changes documented

## When to Request Changes

Request changes when:
- Critical security vulnerabilities exist
- Tests fail or are missing for new features
- Code violates established team standards
- Logic bugs will cause incorrect behavior
- Performance issues will impact users

## When to Approve

Approve when:
- All critical issues resolved
- Tests pass and provide adequate coverage
- Code follows team standards
- Documentation is sufficient
- No security concerns remain
- Minor issues can be addressed in follow-up

## Timeouts

If a review is taking too long:
- **Large PRs**: Request splitting into smaller chunks
- **Complex logic**: Schedule pairing session
- **Many comments**: Discuss in real-time (call/meeting)
- **Disagreement**: Escalate to team lead or architect

## Notes

- Customize this workflow for your team's standards
- Focus on teaching, not gatekeeping
- Balance thoroughness with timely feedback
- Remember: code review is a conversation, not a judgment
