---
name: aoc-solution-review
description: Reviews Advent of Code TypeScript solutions for code quality, TypeScript idioms, and performance. Use when asked to review a day's solution, critique AoC code, or get feedback on puzzle implementations.
---

# AoC Solution Review

Reviews Advent of Code solutions with focus on learning TypeScript through practical feedback.

## When to Use

- User asks to "review day N"
- User wants feedback on their AoC solution
- User asks for code quality or performance analysis

## Review Process

1. **Run checks**: Execute automated validation first
   ```bash
   bun test src/days/dayNN.test.ts   # Verify tests pass
   bunx eslint src/days/dayNN.ts     # Check for lint errors (no --fix)
   bunx prettier --check src/days/dayNN.ts  # Check formatting
   ```
2. **Read the solution**: Load `src/days/dayNN.ts` and `src/days/dayNN.test.ts`
3. **Consult the oracle**: Ask for a thorough review covering all sections below
4. **Research algorithms**: If the solution uses a notable algorithm (BFS, dynamic programming, etc.), do a brief web search for relevant Wikipedia or reference articles to mention
5. **Present findings**: Organize feedback into the standard sections

## Review Sections

### 1. TypeScript Quality
- **Type safety**: Identify `any`, unnecessary `!` assertions, weak unions
- **Idiomatic patterns**: Suggest modern TS/JS idioms (destructuring, optional chaining, nullish coalescing)
- **Naming**: Flag unclear variable names, suggest improvements
- **Organization**: Comment on function size, separation of concerns

### 2. Code Readability
- **Parsing vs logic**: Is input parsing cleanly separated from solution logic?
- **Comments**: Are complex algorithms explained? (But don't over-comment obvious code)
- **Magic numbers**: Flag unexplained constants
- **Control flow**: Identify overly nested or confusing logic

### 3. Performance Analysis
- **Complexity**: State time/space complexity (e.g., O(n²))
- **Hot loops**: Identify operations in tight loops that could be optimized
- **Data structures**: Suggest better structures if applicable (Map vs object, Set vs array)
- **Practical balance**: Note when optimization isn't worth the complexity

### 4. Potential Bugs & Edge Cases
- **Off-by-one errors**: Check loop bounds
- **Undefined access**: Identify risky array/object accesses
- **Empty input**: Would the solution handle empty or malformed input?

### 5. Automated Checks
Report results from the automated checks run in step 1:
- **Tests**: Pass/fail status
- **ESLint**: Any errors or warnings
- **Prettier**: Formatting issues (if any)

### 6. Algorithm Notes
If a notable algorithm is used, briefly mention:
- What algorithm/technique is being used
- Link to Wikipedia or other reference for further reading
- Whether there's a more efficient approach worth knowing about

### 7. Suggestions (Prioritized)
List 3-5 concrete improvements, ordered by impact:
1. **Must fix**: Bugs or serious issues
2. **Should improve**: Clear wins for readability or performance
3. **Consider**: Nice-to-haves or learning opportunities

## Output Format

```markdown
## Day N Review

### Automated Checks
- Tests: ✓ passed / ✗ failed
- ESLint: ✓ clean / [errors]
- Prettier: ✓ formatted / [issues]

### TypeScript Quality
[findings]

### Readability
[findings]

### Performance
[findings]

### Edge Cases
[findings]

### Algorithm Notes
[technique used, reference link, alternatives]

### Suggestions
1. [highest priority]
2. [next priority]
...
```

## Guidelines

- Be direct and matter-of-fact — skip praise and filler
- Explain *why* something is better, not just *what* to change
- Reference specific line numbers when pointing out issues
- Never offer to apply fixes — this skill is pedagogic only, not for writing code
- Keep suggestions practical for AoC context (not production-grade over-engineering)

## Examples

See `reference/examples.md` for calibration examples showing:
- Good vs bad patterns for common issues
- Appropriate feedback tone and specificity
- How to phrase constructive criticism
