---
name: pr-review
description: Guidelines for conducting thorough pull request code reviews
---

## Code Quality

Look for these aspects when reviewing code:

**Readability and clarity**
- Is the code easy to follow? Would another developer understand it without extensive explanation?
- Are variable, function, and class names descriptive and consistent with the codebase conventions?
- Is the code appropriately commented where logic is non-obvious? (But not over-commented for trivial things.)

**Structure and design**
- Does each function or method do one thing well?
- Are functions a reasonable length? Very long functions often indicate opportunities to extract smaller, focused helpers.
- Is there unnecessary duplication that could be refactored?
- Does the code follow established patterns in the codebase, or does it introduce inconsistency?

**Error handling**
- Are errors handled appropriately rather than silently swallowed?
- Are error messages helpful for debugging?
- Are edge cases considered (empty inputs, null values, boundary conditions)?

**Types and contracts**
- Are types used correctly and consistently?
- Are function signatures clear about what they accept and return?
- For dynamically typed languages, is there sufficient validation of inputs?

## Bug Detection

Watch for these common sources of bugs:

- **Off-by-one errors** in loops, array indexing, and string slicing
- **Null or undefined handling** - is it clear what happens when values are missing?
- **Race conditions** in concurrent or async code - are shared resources properly protected?
- **Resource leaks** - are files, connections, and handles properly closed?
- **Logic errors** - does the boolean logic actually express the intended condition?
- **State management** - is mutable state handled carefully? Could stale state cause issues?
- **Boundary conditions** - what happens at the extremes (empty collections, maximum values, etc.)?

## Performance

Consider performance implications, but be proportionate:

- **Algorithm complexity** - is there a more efficient approach for the data sizes involved?
- **Database queries** - are there N+1 query problems? Missing indexes for common queries?
- **Memory usage** - are large collections being copied unnecessarily? Retained longer than needed?
- **Unnecessary work** - are expensive operations being repeated when they could be cached or memoised?
- **Lazy vs eager** - is data being loaded or computed before it's actually needed?

Only raise performance concerns when they're likely to matter in practice. Premature optimisation is its own problem.

## Testing

Evaluate the test coverage and quality:

- Are new features and bug fixes covered by tests?
- Do tests verify behaviour rather than implementation details?
- Are edge cases tested, not just the happy path?
- Are mocks and stubs used appropriately, not excessively?
- Would the tests catch regressions if someone changed this code?

## Documentation

Check that documentation is updated appropriately:

- Are public APIs documented?
- If behaviour changes, is existing documentation updated?
- Are complex algorithms or non-obvious decisions explained?
- For user-facing changes, does the README or user documentation need updating?

## Suggested Changes

When you spot something that could be improved with a small, clear fix, offer a concrete suggestion using GitHub's suggestion format. This makes it easy for the author to apply with one click.

Good candidates for suggestions:
- Typos in comments, strings, or variable names
- Simple refactorings (extracting a variable, simplifying a condition)
- Formatting fixes that aren't caught by automated tools
- Small logic corrections
- Missing error handling for an obvious case

Keep suggestions to **10 lines or fewer**. For larger changes, describe what to do instead.

## Writing Good Comments

Your comments should be:

- **Constructive** - focus on improving the code, not criticising the author
- **Specific** - point to exact lines and explain precisely what could be better
- **Reasoned** - explain *why* something is an issue, not just that it is
- **Proportionate** - distinguish between minor suggestions and significant concerns
- **Actionable** - make it clear what you're suggesting the author do

Phrase things as observations and suggestions rather than demands:
- "This could be simplified by..." rather than "Simplify this"
- "I wonder if this handles the case where..." rather than "This is wrong"
- "It might be clearer to..." rather than "This is confusing"

If you're uncertain about something, frame it as a question. The author may have context you lack.

## Existing Discussions

When there are already comments on the PR from other reviewers or automated tools:

- Read them to understand the context and avoid duplicating feedback
- Build on existing discussions where you have something to add
- If a bot has flagged something relevant, you can reference it
- Don't mark threads as resolved - that's the author's prerogative
- If you disagree with a previous reviewer's comment, be respectful and explain your perspective
