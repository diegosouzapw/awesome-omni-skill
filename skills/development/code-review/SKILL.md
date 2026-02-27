---
name: code-review
description: Best practices for performing thorough, constructive code reviews. Use when reviewing PRs to ensure quality feedback that improves code and helps developers grow.
---

# Code Review Skill

Guidelines for performing effective code reviews that catch issues, improve code quality, and maintain a positive team dynamic.

## Review Mindset

### Bias Towards Shipping

**The primary goal is to get working code merged, not to achieve perfection.**

Before requesting changes, always ask yourself:
> "Is this feedback worth an additional review cycle?"

Most feedback should be:
- Approved with suggestions for the author to consider
- Comments for future reference, not blocking
- Follow-up issues for later improvement

**Reserve "Request Changes" for actual problems:**
- Security vulnerabilities
- Bugs that will cause runtime errors
- Breaking changes to existing functionality
- Missing critical requirements

**Do NOT block for:**
- Style preferences (unless egregiously inconsistent)
- "I would have done it differently"
- Minor optimizations that don't matter at current scale
- Missing tests for edge cases that are unlikely
- Refactoring opportunities (create a follow-up issue instead)

### Goals of Code Review
1. **Catch real bugs** before they reach production
2. **Prevent security issues** - the things that actually matter
3. **Share knowledge** across the team
4. **Ship working software** - don't let perfect be the enemy of good

### The Right Attitude
- You're reviewing the **code**, not the person
- Assume good intent - the author tried their best
- Be a collaborator, not a gatekeeper
- Your job is to help ship good code, not to find fault
- **Velocity matters** - every review cycle has a cost

## Review Process

### 1. Understand the Context First

Before looking at code:
```
1. Read the PR description
2. Read the linked issue
3. Understand WHAT is being done and WHY
4. Consider: Is this the right approach?
```

### 2. First Pass: High-Level Review

Ask yourself:
- Does this solve the problem described in the issue?
- Is the approach reasonable?
- Are there obvious architectural concerns?
- Is anything missing?

### 3. Second Pass: Detailed Review

Look at each file for:
- Correctness (does it work?)
- Edge cases and error handling
- Security implications
- Performance concerns
- Test coverage
- Code style and readability

### 4. Summarize Your Review

End with an overall assessment:
```markdown
**[Architect]** Overall this looks good. Clean implementation of the user registration flow.

A few suggestions:
1. Consider adding rate limiting (security)
2. The validation error messages could be more user-friendly
3. Minor: prefer `const` over `let` where possible

Approving with minor suggestions - none are blocking.
```

## What to Look For

### Correctness
- Does the code do what it's supposed to do?
- Are there logic errors?
- Are edge cases handled?
- What happens with null/undefined/empty inputs?

```markdown
**Concern:** This will throw if `user` is null:
\`\`\`javascript
const name = user.profile.name;
\`\`\`

Consider:
\`\`\`javascript
const name = user?.profile?.name ?? 'Unknown';
\`\`\`
```

### Error Handling
- Are errors caught appropriately?
- Are error messages helpful?
- Does the error handling match the project patterns?

```markdown
**Suggestion:** The error is swallowed here - the user won't know what went wrong:
\`\`\`javascript
try {
  await saveUser(data);
} catch (e) {
  console.log(e);
}
\`\`\`

Consider surfacing this to the user or rethrowing.
```

### Security (OWASP Top 10)
- SQL injection (parameterized queries?)
- XSS (output encoding?)
- Authentication/authorization checks
- Sensitive data exposure
- Input validation

```markdown
**Security Issue:** User input is interpolated directly into the query:
\`\`\`javascript
const query = `SELECT * FROM users WHERE email = '${email}'`;
\`\`\`

Use parameterized queries to prevent SQL injection.
```

### Performance
- Unnecessary loops or iterations
- N+1 query problems
- Missing indexes
- Large payloads
- Memory leaks

```markdown
**Performance:** This creates an N+1 query problem - one query per user:
\`\`\`javascript
for (const user of users) {
  user.posts = await getPosts(user.id);
}
\`\`\`

Consider using a batch query or join.
```

### Maintainability
- Is the code readable?
- Are names descriptive?
- Is the logic easy to follow?
- Will this be easy to modify later?

```markdown
**Readability:** This function does a lot - consider splitting:
- Validation logic → `validateUserInput()`
- Transformation → `transformUserData()`
- Persistence → `saveUser()`

This would make each piece easier to test and understand.
```

### Tests
- Are tests included?
- Do tests cover the main paths?
- Do tests cover edge cases?
- Are tests readable and maintainable?

```markdown
**Testing:** Good coverage of the happy path! Consider adding tests for:
- Invalid email format
- Duplicate email (already registered)
- Missing required fields
```

### Consistency
- Does it follow project conventions?
- Does it match existing patterns?
- Is it consistent with itself?

```markdown
**Consistency:** The rest of the codebase uses `async/await`, but this uses `.then()`:
\`\`\`javascript
fetchUser(id).then(user => { ... });
\`\`\`

Consider using async/await for consistency.
```

## How to Give Feedback

### Be Specific and Actionable

**Bad:**
> This code is confusing.

**Good:**
> The nested callbacks make this hard to follow. Consider using async/await or extracting the inner logic into named functions.

### Explain the Why

**Bad:**
> Use `const` here.

**Good:**
> Use `const` here since `user` is never reassigned. This signals intent to readers and catches accidental reassignment.

### Offer Solutions

**Bad:**
> This won't scale.

**Good:**
> This loads all users into memory, which won't scale. Consider:
> 1. Pagination with limit/offset
> 2. Streaming with cursor-based pagination
> 3. If the use case allows, a count query instead

### Distinguish Severity

Use prefixes to indicate importance:

| Prefix | Meaning | Frequency |
|--------|---------|-----------|
| **Blocker:** | Must fix before merge - security/bugs only | Rare |
| **Suggestion:** | Would improve the code, not blocking | Common |
| **Nit:** | Minor style/preference, definitely not blocking | Occasional |
| **Question:** | Seeking to understand | As needed |
| **Praise:** | Something done well | Often! |

**Important:** Most comments should be Suggestions or Nits, not Blockers. If you find yourself writing many Blockers, reconsider whether they truly block shipping.

```markdown
**Blocker:** This SQL query is vulnerable to injection. (Security - must fix)

**Suggestion:** Consider extracting this into a helper function for reuse. (Not blocking - can be done later)

**Nit:** Extra blank line here. (Definitely not blocking)

**Question:** Why did you choose this approach over X? (Curious, not blocking)

**Praise:** Really clean error handling here! (Always welcome)
```

### Ask Questions Instead of Demanding

**Demanding:**
> Change this to use a Map instead of an object.

**Collaborative:**
> Have you considered using a Map here? It might give better performance for frequent lookups. What do you think?

## Common Review Scenarios

### When You'd Do It Differently

Don't block for preference. Ask yourself:
- Is their way wrong, or just different?
- Does it work correctly?
- Is it maintainable?

If it's just different:
```markdown
**Note:** I might have used X approach here, but this works well too. Not blocking.
```

### When Something Is Missing

```markdown
**Missing:** I don't see error handling for the case where the API returns 404. What should happen?
```

### When You Don't Understand

```markdown
**Question:** I'm not following the logic in this section. Could you add a comment explaining the business rule, or walk me through it?
```

### When Praising Good Work

Don't just point out problems:
```markdown
**Praise:** Nice job extracting this into a reusable hook! This will help with the other forms too.

**Praise:** The test coverage here is excellent.
```

## Review Response Templates

**Default to Approve.** Most reviews should approve, possibly with suggestions.

### Approve (Most Common)
```markdown
**[Architect]** LGTM! Clean implementation, good test coverage.

Minor suggestions (not blocking):
- Line 42: prefer const
- Consider adding a comment explaining the retry logic

Merging as-is is fine. Address these if you agree, or not - your call.
```

### Approve with Suggestions
```markdown
**[Architect]** Approving - this is solid work.

A few things to consider (can address now or in follow-up):
1. The validation could be more specific about what's wrong
2. Consider adding logging for debugging

Not blocking merge. Ship it!
```

### Request Changes (Rare - Use Sparingly)

**Only use for genuine blockers: security issues, bugs, or missing critical functionality.**

```markdown
**[Architect]** Good progress! Found one issue that needs fixing before merge:

**Blocker:**
1. SQL injection vulnerability in the search query - this is a security risk

Everything else looks good. Happy to re-review once the security fix is in.
```

**Ask yourself before requesting changes:**
- Will this cause a production incident if shipped?
- Is this a security vulnerability?
- Does it break existing functionality?

If the answer to all three is "no", consider approving with suggestions instead.

### Decline (N/A)
```markdown
**[UI/UX]** N/A - No UI changes in this PR.

Reviewed: Backend/infrastructure changes only, no user-facing impact.
```

## Review Etiquette

### Do
- Review promptly (within 24 hours ideally)
- Be thorough but not pedantic
- Acknowledge good work
- Offer to discuss complex issues in person/call
- Re-review quickly after changes

### Don't
- Nitpick excessively
- Bike-shed on minor style issues
- Block for preferences
- Be condescending
- Leave reviews hanging

## Self-Review Checklist

Before requesting review, authors should self-review:

- [ ] I've re-read my own diff
- [ ] Tests pass locally
- [ ] No console.logs or debug code left
- [ ] No commented-out code
- [ ] Variable names are descriptive
- [ ] Complex logic has comments
- [ ] Error cases are handled
- [ ] No obvious security issues
