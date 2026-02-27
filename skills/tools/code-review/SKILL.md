---
name: code-review
description:
  Perform code reviews following engineering practices. Use when
  reviewing pull requests, examining code changes, or providing feedback on code
  quality. Covers security, performance, testing, and design review.
---

# Code Review

Follow these guidelines when reviewing code changes.

## Change Discipline

- Write absolute minimum code required
- No sweeping changes
- No unrelated edits
- Stay focused on the specific task
- Don't break existing functionality without asking

## Investigation Approach

When reviewing code:

1. List 5-7 potential issues or concerns for each file
2. Gather evidence (check similar patterns, run tests, trace data flow)
3. Narrow to 1-2 most critical issues per file
4. Verify issues are real (not false positives or already handled)
5. Only report confirmed, actionable feedback

This ensures thorough but focused reviews without noise.

## Review Workflow

### 1. Gather Context

**Understanding the Changes:**
- Use `gh pr view <number>` or `gh pr view <url>` to get PR details (title, description, status)
- Use `git diff <base-branch>...HEAD` to see all changes in the current branch
- Use `git log <base-branch>..HEAD` to see commit history and messages
- Read the PR description carefully to understand the intended purpose
- Identify which files are modified, added, or deleted

**Understanding the Codebase:**
- Use the Task tool with subagent_type=Explore to understand related code patterns
- Read files that interact with the changed code
- Look for similar patterns in the codebase to ensure consistency
- Check for existing tests that might be affected

### 2. Perform Review

Review the changes systematically, focusing on the areas below. Use the TodoWrite tool to track your review progress through different aspects.

### 3. Provide Feedback

Structure your feedback clearly with:
- **Critical Issues**: Must be fixed before merging (blocking)
- **Important Issues**: Should be addressed but not blocking
- **Suggestions**: Nice-to-have improvements
- **Positive Feedback**: Call out good practices

Always explain WHY something is an issue and HOW to fix it.

## Review Checklist

### Correctness & Logic

- **Runtime errors**: Check for potential exceptions, null/undefined access, array out-of-bounds
- **Edge cases**: Empty arrays, null values, boundary conditions, concurrent access
- **Logic errors**: Off-by-one errors, incorrect conditionals, race conditions
- **Type safety**: Proper type annotations, avoiding `any` in TypeScript
- **Error handling**: Appropriate try-catch blocks, error propagation, user-friendly messages

### Performance

- **Algorithm complexity**: Avoid O(n²) or worse where O(n) or O(log n) is possible
- **Database queries**:
  - N+1 query problems (missing prefetch/select_related in Django)
  - Missing indexes for new query patterns
  - Unbounded queries without pagination
  - Inefficient joins or subqueries
- **Memory usage**: Unnecessary data copying, memory leaks, large object allocations
- **Caching**: Opportunities for caching expensive operations
- **Network calls**: Batching, unnecessary requests, missing timeouts

### Security

- **Injection vulnerabilities**: SQL injection, command injection, XSS, path traversal
- **Authentication & Authorization**: Proper permission checks, role validation
- **Data exposure**: Sensitive data in logs, error messages, or API responses
- **Input validation**: Sanitize and validate all user inputs
- **Secrets management**: No hardcoded credentials, API keys, or tokens
- **Dependency vulnerabilities**: Check for known CVEs in new dependencies
- **CORS & CSP**: Proper configuration for web applications

### Design & Architecture

- **Consistency**: Follows existing patterns and conventions in the codebase
- **Separation of concerns**: Clear boundaries between components/modules
- **DRY principle**: Avoid duplicating logic (but don't over-abstract)
- **SOLID principles**: Appropriate use of abstraction and interfaces
- **API design**: Clear contracts, versioning strategy, backward compatibility
- **Configuration**: Externalize environment-specific values
- **Error boundaries**: Proper error handling at system boundaries

### Testing

**Test Coverage Requirements:**
- New features MUST have tests
- Bug fixes SHOULD include regression tests
- Modified code should maintain or improve test coverage

**Test Quality:**
- Tests cover happy path AND edge cases
- Integration tests for component interactions
- Tests are readable and well-named
- No flaky tests (avoid timing dependencies, randomness)
- Mock external dependencies appropriately
- Tests actually verify the requirements (not just code coverage)

**How to Verify:**
- Look for corresponding test files (e.g., `test_*.py`, `*.test.ts`, `*_spec.rb`)
- Use `pytest --cov` (Python), `npm test -- --coverage` (JS), or similar
- Check if critical paths have test coverage

### Code Quality

- **Readability**: Clear variable/function names, appropriate comments for complex logic
- **Formatting**: Follows project style guide (use linters/formatters)
- **Complexity**: Functions are focused and not too long
- **Documentation**: Public APIs have docstrings/JSDoc comments
- **Dead code**: Remove commented-out code, unused imports, unreachable code
- **Magic numbers**: Use named constants for unclear literal values

### Side Effects & Impact

- **Breaking changes**: API changes without deprecation path
- **Backward compatibility**: Will this break existing clients/users?
- **Database migrations**: Safe migrations that won't cause downtime
- **Feature flags**: New features behind toggles when appropriate
- **Deployment considerations**: Requires configuration changes, new services, etc.
- **Monitoring**: Add logging/metrics for critical operations

### Long-Term Considerations

**Flag for senior engineer review when changes involve:**
- Database schema modifications or migrations
- API contract changes (REST, GraphQL, gRPC)
- Authentication/authorization logic changes
- New framework or library adoption
- Performance-critical code paths (hot paths)
- Security-sensitive functionality
- Infrastructure or deployment changes
- Significant architectural decisions

## Feedback Guidelines

### Tone & Communication

- **Be respectful and constructive**: Assume good intent
- **Be specific**: Point to exact lines and explain the issue clearly
- **Provide context**: Explain WHY something is a problem
- **Offer solutions**: Suggest concrete fixes or alternatives
- **Ask questions**: Use "Have you considered...?" when uncertain
- **Praise good work**: Call out clever solutions or good practices
- **Reference documentation**: Link to style guides, best practices, or examples

### Prioritization

**Critical (Blocking):**
- Security vulnerabilities
- Data loss or corruption risks
- Breaking production functionality
- Major performance degradation

**Important (Should Fix):**
- Bugs in new functionality
- Missing test coverage
- Poor error handling
- Design issues that will cause maintenance burden

**Nice to Have (Suggestions):**
- Code style improvements
- Minor refactoring opportunities
- Additional edge case handling
- Documentation enhancements

### Approval Criteria

**Approve when:**
- No critical issues remain
- Important issues are addressed or have clear plan
- Test coverage is adequate
- Code meets quality standards

**Don't block for:**
- Stylistic preferences (if code follows project conventions)
- Perfect code (good enough is good enough)
- Theoretical future requirements
- Personal preference on implementation approach (if both work)

**Remember:** The goal is to reduce risk and maintain quality, not achieve perfection.

## Common Patterns to Flag

Look for: N+1 queries, unbounded queries, SQL injection, missing null checks, unhandled promises, resource leaks, race conditions. Use linters and security scanners to catch common issues.

## Tool Usage for Reviews

- **Use `gh` commands** for GitHub PR operations
- **Use `git` commands** to view diffs and history
- **Use Read tool** to examine specific files
- **Use Grep tool** to search for patterns across the codebase
- **Use Task tool with Explore agent** for understanding code architecture
- **Use Bash tool** to run tests, linters, or build commands when needed
- **Use LSP tool** to trace definitions and references when available

## Output Format

Structure your review as follows:

```
## Code Review Summary

**Overall Assessment:** [Approve/Request Changes/Comment]

**Key Changes:** [Brief summary of what the PR does]

## Critical Issues

[List blocking issues if any]

## Important Issues

[List significant issues that should be addressed]

## Suggestions

[List nice-to-have improvements]

## Positive Feedback

[Call out good practices, clever solutions, or improvements]

## Testing

[Comment on test coverage and quality]

## Recommendation

[Final recommendation with any conditions]
```
