---
name: sequentialthinking
description: Use this skill for systematic problem-solving through structured, iterative thinking. Break down complex problems, iterate on understanding, catch edge cases, and validate solutions comprehensively.
---

# Sequential Thinking Skill

This skill guides you through **structured, reflective problem-solving** using iterative thinking. It's essential for tackling complex problems, multi-step tasks, and scenarios where thorough analysis prevents costly mistakes.

## Why Sequential Thinking?

- **Clarity**: Break down ambiguous problems into manageable steps.
- **Completeness**: Catch edge cases and hidden requirements before implementation.
- **Efficiency**: Invest thinking time upfront to avoid rework and debugging.
- **Flexibility**: Adjust your approach mid-course when assumptions change.
- **Confidence**: Validate decisions through structured reflection.

## Mental Model

Sequential thinking is a **deliberate, non-linear problem-solving process**:
- You don't commit to a path immediately; instead, you think through multiple aspects.
- Each thought can refine, question, or revise previous thoughts.
- You track assumptions and adjust them as understanding deepens.
- The process naturally branches when exploring alternatives.
- You stop when confident the solution is correct and complete.

## When to Use Sequential Thinking

✅ **Use for:**
- Complex, multi-step tasks (>3 steps or >2 interdependent components)
- Ambiguous requirements that need clarification
- Problems where edge cases or hidden risks exist
- Design decisions with trade-offs to weigh
- Debugging persistent issues requiring root-cause analysis
- Tasks where implementation complexity is uncertain
- Multi-file edits affecting different parts of a system

❌ **Skip for:**
- Trivial single-step tasks ("What's 2+2?", "List files in a directory")
- Simple, well-defined requests with one obvious answer
- Quick lookups or factual queries
- Purely conversational exchanges

## Core Parameters & Workflow

### Key Parameters

**`thought`** (string)
- Your current thinking step, which can include:
  - Regular analytical steps
  - Revisions of previous thoughts
  - Questions about previous decisions
  - Realizations about needing more analysis
  - Changes in approach
  - Hypothesis generation and verification

**`thoughtNumber`** (integer)
- Current position in your thinking sequence (e.g., 1, 2, 3)
- Increments even when revising or branching

**`totalThoughts`** (integer)
- Your estimate of how many thoughts you'll need
- **Adjustable**: Increase if new complexities emerge; decrease if solved earlier
- Initial estimate: 3–7 for most problems; adjust as you learn more

**`nextThoughtNeeded`** (boolean)
- `true`: Continue thinking; more analysis needed
- `false`: Stop; you've reached a confident conclusion

**`isRevision`** (boolean)
- `true`: This thought revises or questions a previous thought
- `false`: This is a new forward-moving step
- Use when assumptions prove wrong or understanding shifts

**`revisesThought`** (integer)
- If `isRevision: true`, reference the thought number being reconsidered
- Example: "Thought 3 assumed X, but now I see Y is true"

**`branchFromThought`** (integer)
- When exploring an alternative approach, reference the branching point
- Useful for: "What if we approach this differently?"

**`branchId`** (string)
- Identifier for an alternative branch (e.g., "approach-a", "db-design-alt")
- Helps track parallel explorations

**`needsMoreThoughts`** (boolean)
- Set to `true` if you reach the estimated total but realize more thinking is needed
- Allows graceful expansion without breaking flow

## Workflow Patterns

### Pattern 1: Linear Problem-Solving (Simplest)

```
Thought 1: Understand the problem
  ↓
Thought 2: Identify root causes or constraints
  ↓
Thought 3: Outline solution approach
  ↓
Thought 4: Verify solution handles edge cases
  ↓
[nextThoughtNeeded: false]
```

**Example Use Case**: Fixing a bug with a clear stack trace.

### Pattern 2: Iterative Refinement (Common)

```
Thought 1: Initial understanding
  ↓
Thought 2: Question assumption → [isRevision: true, revisesThought: 1]
  ↓
Thought 3: Revised approach based on new insight
  ↓
Thought 4: Check for edge cases
  ↓
[nextThoughtNeeded: false]
```

**Example Use Case**: Designing an API with unclear requirements.

### Pattern 3: Branching Exploration (Advanced)

```
Thought 1: Understand problem
  ↓
Thought 2: Outline Approach A
  ↓
Thought 3: Branch to Approach B [branchFromThought: 1, branchId: "alt-approach"]
  ↓
Thought 4: Compare approaches
  ↓
Thought 5: Choose best approach
  ↓
[nextThoughtNeeded: false]
```

**Example Use Case**: Choosing between architectural patterns.

### Pattern 4: Discovery & Expansion

```
Thought 1: Initial scope assessment (totalThoughts: 4)
  ↓
Thought 2: First insight reveals complexity
  ↓
Thought 3: Realize more analysis needed [needsMoreThoughts: true]
  ↓
Thought 4: Additional analysis
  ↓
Thought 5: More thinking...
  ↓
[nextThoughtNeeded: false]
```

**Example Use Case**: Uncovering hidden dependencies in a large refactor.

## Best Practices

### 1. **Think Before Acting**
   - Use sequential thinking to plan before making code changes.
   - Each thought should inform the next, building toward a complete understanding.

### 2. **Question Your Assumptions**
   - Mark thoughts with `isRevision: true` when an assumption breaks.
   - Example: "I thought this was a performance issue, but it's actually a concurrency bug."

### 3. **Adjust Total Thoughts Dynamically**
   - Don't overthink trivial problems.
   - Expand `totalThoughts` when discovering new complexities.
   - Example: Started with 3, now at 5, but see I need 7 → adjust and continue.

### 4. **Branch to Explore Alternatives**
   - Use branching when you want to compare approaches without committing.
   - Merge insights: "Approach A is simpler, but Approach B scales better."

### 5. **Validate Edge Cases Late (But Before Coding)**
   - In final thoughts, ask: "What could break this solution?"
   - Consider: boundary conditions, null values, concurrency, performance, security.

### 6. **Stop When Confident**
   - Set `nextThoughtNeeded: false` only when:
     - You fully understand the problem
     - You've identified a viable solution
     - You've considered major edge cases
     - You're ready to implement with confidence

### 7. **Use Thought Tracking for Readability**
   - Be concise in each thought; avoid rambling.
   - Use clear language that would make sense in a document review.

## Common Patterns & Examples

### Example 1: Debugging a Complex Issue

```
Thought 1: Symptom is slow queries. Possible causes: missing index, inefficient join, N+1 query.

Thought 2: Let me think through the data model. The join involves 3 tables...
  The query runs in O(n²) because we're not indexed on the foreign key.

Thought 3: Solution: Add an index. But wait—are there writes on this table?
  If so, indexing might slow writes. Trade-off check needed.

Thought 4: [isRevision: true, revisesThought: 3] Actually, the table is read-heavy.
  Adding an index is safe. Also consider query caching for frequently-fetched data.

Thought 5: Verify edge case: What if index creation locks the table in production?
  Use `CONCURRENTLY` option in PostgreSQL. Plan for maintenance window.

[nextThoughtNeeded: false]
```

### Example 2: Designing a Feature with Unknowns

```
Thought 1: Feature request: user profiles. Questions: Should profiles be public or private?
  No clear spec. I'll assume private by default with optional sharing.

Thought 2: Database schema: users → profiles (1:1 relation). Privacy stored as boolean or enum?
  Enum is more scalable (public, private, friends-only). Revised approach.

Thought 3: [isRevision: true, revisesThought: 1] Reconsidering sharing model.
  If users can share with friends, we need to store relationships. This adds complexity.
  Keep it simple for v1: public or private only.

Thought 4: API design. GET /users/:id/profile should check permissions. POST with auth.
  Edge case: What if user has no profile yet? Return 404 or empty object?
  Return empty object with a 200 for consistency.

Thought 5: Edge cases to handle:
  - Deleted user: cascade delete profile or soft-delete?
  - Concurrent updates: Use optimistic locking with version field.
  - Performance: Profile fetches are common. Cache with 1-hour TTL.

[nextThoughtNeeded: false]
```

### Example 3: Exploring Multiple Approaches

```
Thought 1: Need to refactor module A. Three possible approaches:
  A) Incremental: Refactor one function at a time.
  B) Big-bang: Rewrite the entire module.
  C) Extract-and-replace: Extract logic into new module, then replace gradually.

Thought 2: [branchFromThought: 1, branchId: "approach-A"] Approach A is low-risk
  but could take weeks. Good for small teams.

Thought 3: [branchFromThought: 1, branchId: "approach-B"] Approach B is fast but risky.
  High chance of bugs. Needs extensive testing.

Thought 4: [branchFromThought: 1, branchId: "approach-C"] Approach C balances risk and speed.
  Requires careful API design between old and new modules.

Thought 5: Comparing branches:
  - A: Safe, slow, low effort
  - B: Fast, risky, high effort
  - C: Balanced, moderate effort
  Our codebase is large and in production. Risk mitigation is critical.
  Choose Approach C. Invest in API clarity.

[nextThoughtNeeded: false]
```

## Integration with Other Tools & Workflows

### Sequential Thinking + Code Implementation
1. **Plan**: Use sequential thinking to design the solution.
2. **Code**: Implement based on the plan.
3. **Test**: Run tests and validate edge cases discovered in thinking.
4. **Iterate**: If tests fail, return to sequential thinking to revise assumptions.

### Sequential Thinking + Debugging
1. **Observe**: Describe the bug symptoms.
2. **Think**: Use sequential thinking to hypothesize root causes.
3. **Investigate**: Gather evidence (logs, traces, etc.).
4. **Revise**: Update hypotheses based on evidence.
5. **Fix**: Implement fix with confidence.

### Sequential Thinking + Code Review
- Reviewers can use sequential thinking to structure feedback.
- "Thought 1: Code changes X. Thought 2: But this breaks case Y. Thought 3: Suggest fix..."

## Common Pitfalls & How to Avoid Them

### 1. **Over-Thinking Trivial Problems**
   - **Problem**: Using sequential thinking for "What's the current date?"
   - **Solution**: Skip sequential thinking for simple, single-answer queries.

### 2. **Not Revising When Assumptions Break**
   - **Problem**: Continuing with flawed assumptions instead of marking `isRevision: true`.
   - **Solution**: Immediately flag revisions when evidence contradicts your thinking.

### 3. **Stopping Too Early**
   - **Problem**: Setting `nextThoughtNeeded: false` before considering edge cases.
   - **Solution**: Always reserve 1–2 final thoughts for "What could go wrong?"

### 4. **Ignoring Edge Cases**
   - **Problem**: Solution works for the happy path but breaks on boundaries.
   - **Solution**: In final thoughts, explicitly list boundary conditions, error cases, concurrency, performance.

### 5. **Too Many Branches**
   - **Problem**: Branching into too many alternatives and losing track.
   - **Solution**: Limit to 2–3 main branches; merge insights by comparing them systematically.

### 6. **Vague Thoughts**
   - **Problem**: Thoughts are rambling or unclear.
   - **Solution**: Write thoughts as if explaining to a colleague; be concise and specific.

## Real-World Integration

### For AI Agents in Beastmode
Sequential thinking is **essential** for autonomous agents tackling complex tasks:
- **Before Implementation**: Think through the entire solution first.
- **During Implementation**: Reference the thinking plan to avoid scope creep.
- **During Testing**: Use thinking to verify edge cases are covered.
- **On Failure**: Revise thinking and iterate.

### For Developers
- Use sequential thinking when starting complex features or debugging.
- Share your thinking with the team in code review comments.
- Adjust thinking approach based on team feedback.

## Survival Kit

- **Day 0**: Read this skill and review the patterns.
- **Week 1**: Use sequential thinking on your next complex task. Start with Pattern 1 (linear).
- **Week 2**: Try Pattern 2 (revision) when you encounter assumption changes.
- **Week 3**: Experiment with branching (Pattern 3) for design decisions.
- **Ongoing**: Refine your thinking process based on what works for you.

## Key Takeaways

1. **Think before acting**: Invest in analysis to avoid rework.
2. **Iterate flexibly**: Adjust your understanding as you learn.
3. **Catch edge cases**: Spend final thoughts on validation.
4. **Branch thoughtfully**: Explore alternatives without losing focus.
5. **Stop with confidence**: Set `nextThoughtNeeded: false` only when ready.

Sequential thinking transforms scattered problem-solving into a **methodical, documented process** that leads to better designs, fewer bugs, and faster implementations.
