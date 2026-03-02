---
name: evaluative-research
description:
  Methodology for evaluating options, comparing technologies, and making
  evidence-based decisions between alternatives. Use when the user needs to
  choose between competing approaches, libraries, or architectures with a
  structured comparison. Triggers when user says "compare these options", "which
  approach should we use", "evaluate alternatives", "help me decide between X
  and Y", "technology comparison", or wants a structured
  pros/cons/recommendation analysis.
---

# Evaluative Research Methodology

A structured approach to evaluating options and making evidence-based decisions
between alternatives. Use this methodology when the core question is "which
option should we choose?" rather than "what happened?" or "how does this work?"

## When to Use

Use this methodology when you need to:

- Compare libraries, frameworks, or services for a specific use case
- Evaluate migration paths between technologies
- Assess build vs. buy decisions
- Choose between architectural approaches
- Analyze trade-offs between competing solutions
- Conduct technology landscape analysis

**Key indicator**: There are multiple viable options and you need a structured
way to decide between them.

**Not for**: Debugging or root cause analysis (use investigation-methodology),
finding what's missing from a spec (use gap-analysis), or understanding how
something works (use general research).

## Evaluative Research Framework

### Phase 1: Define the Decision

Before comparing options, articulate what you're deciding:

- **Decision statement**: Frame as a clear question ("Which editor library
  should we use for rich text editing?")
- **Decision drivers**: What matters most? (performance, DX, community,
  maintenance, cost, time-to-implement)
- **Constraints**: Non-negotiable requirements that eliminate options early
- **Context**: Current architecture, team expertise, existing patterns
- **Stakeholders**: Who is affected by this decision?

**Stopping point**: You can state the decision, its drivers, and constraints in
2-3 sentences.

### Phase 2: Landscape Scan

Cast a wide net to identify all viable options:

- **Identify candidates**: Research the space broadly before narrowing
- **Quick-filter**: Eliminate options that violate hard constraints
- **Categorize**: Group options by approach (e.g., "full framework" vs.
  "lightweight library" vs. "build custom")
- **Select finalists**: 2-4 options for deep evaluation (more than 4 becomes
  unwieldy)

**Sources to check:**

- Official documentation and getting-started guides
- GitHub: stars, issues, commit frequency, release cadence
- npm/package registry: download trends, bundle size, dependency count
- Community: Stack Overflow activity, Discord/forum presence
- Existing codebase: similar patterns already implemented

**Stopping point**: You have 2-4 finalists and can explain why others were
eliminated.

### Phase 3: Deep Evaluation

Evaluate each finalist against your decision drivers:

- **Hands-on assessment**: Read docs, review API surface, check examples
- **Codebase fit**: How does each option integrate with existing architecture?
- **Effort estimate**: What does adoption look like? (learning curve, migration
  effort, boilerplate)
- **Risk assessment**: What could go wrong? (abandonment, breaking changes,
  scaling limits)
- **Long-term trajectory**: Is the project growing, stable, or declining?

**For each option, document:**

- Description (what it is, approach it takes)
- Strengths (evidence-backed, not just marketing)
- Weaknesses (real limitations, not hypothetical)
- Effort estimate (Low/Medium/High with rationale)
- Risk level (Low/Medium/High with rationale)

**Stopping point**: You can articulate the genuine trade-offs between finalists.

### Phase 4: Comparative Analysis

Synthesize findings into a decision:

- **Build comparison matrix**: Rate each option against each decision driver
- **Identify key trade-offs**: What do you give up with each choice?
- **Consider second-order effects**: How does each option affect adjacent
  systems?
- **Weight by priority**: Not all criteria are equal — weight by what matters
  most
- **Formulate recommendation**: Clear primary recommendation with confidence
  level

**Comparison Matrix Format:**

| Criteria        | Weight | Option A | Option B | Option C |
| --------------- | ------ | -------- | -------- | -------- |
| [Driver 1]      | High   | [Rating] | [Rating] | [Rating] |
| [Driver 2]      | Medium | [Rating] | [Rating] | [Rating] |
| [Driver 3]      | Low    | [Rating] | [Rating] | [Rating] |
| Effort          | —      | [L/M/H]  | [L/M/H]  | [L/M/H]  |
| Risk            | —      | [L/M/H]  | [L/M/H]  | [L/M/H]  |
| **Overall Fit** | —      | [Rating] | [Rating] | [Rating] |

**Stopping point**: You have a clear recommendation with evidence-backed
rationale.

## Output Format

Structure evaluative research outputs as:

```markdown
# Investigation: [Decision Topic]

## Summary

[2-3 sentences: what decision was needed, what was evaluated, what's
recommended]

## Decision Context

- **Decision**: [Clear question being answered]
- **Drivers**: [What matters most, in priority order]
- **Constraints**: [Hard requirements]
- **Current state**: [Relevant existing architecture/tools]

## Options Evaluated

### Option 1: [Name]

**What it is**: [Brief description] **Strengths**:

- [Evidence-backed strength]

**Weaknesses**:

- [Evidence-backed weakness]

**Effort**: [Low/Medium/High] — [rationale] **Risk**: [Low/Medium/High] —
[rationale]

[Repeat for each option]

### Options Eliminated Early

- [Option X]: Eliminated because [reason]

## Comparison

| Criteria | Option 1 | Option 2 | Option 3 |
| -------- | -------- | -------- | -------- |
| ...      | ...      | ...      | ...      |

### Key Trade-offs

[The 2-3 most significant trade-offs between the top options]

## Recommendation

**Primary**: [Option name] — [Confidence: High/Medium/Low]

**Rationale**: [Why this option, tied to decision drivers and evidence]

**When to reconsider**: [Under what circumstances would you choose differently?]

## Next Steps

1. [Immediate action]
2. [Validation step]
3. [Follow-up if needed]

## References

- [Links to docs, benchmarks, repos evaluated]
```

## Methodological Principles

**Evidence over opinion**: Every rating in the comparison matrix should trace
back to something concrete — documentation, benchmarks, code review, or
real-world usage data.

**Honest about uncertainty**: If you can't properly evaluate a criterion, say
so. "Unable to assess without prototyping" is better than a guess.

**Decision drivers first**: Start from what matters, not from what's easy to
compare. Don't let available data drive the criteria.

**Disconfirm actively**: For your recommended option, try hardest to find its
weaknesses. For the runner-up, try hardest to find its strengths.

**State the trade-off clearly**: The best decisions acknowledge what you're
giving up, not just what you're gaining.

## Quality Checklist

Before concluding evaluative research:

- [ ] Decision statement is clear and specific
- [ ] Decision drivers reflect actual priorities (not generic criteria)
- [ ] At least 2-4 options were seriously evaluated
- [ ] Eliminated options have documented rationale
- [ ] Strengths and weaknesses are evidence-backed
- [ ] Comparison matrix uses weighted criteria
- [ ] Key trade-offs are explicitly stated
- [ ] Recommendation includes confidence level
- [ ] "When to reconsider" conditions are specified
- [ ] References are included for verification

## Scaling to Complexity

**Quick evaluation** (clear winner, limited scope):

- 2-3 options, 3-4 criteria
- Focus on the one criterion that differentiates
- Stop when one option clearly dominates

**Standard evaluation** (multiple good options):

- 3-4 options, 5-7 criteria
- Full comparison matrix with weights
- Stop when trade-offs are clear and recommendation is defensible

**Strategic evaluation** (high-stakes, long-term impact):

- 3-5 options, 7+ criteria
- Prototype or proof-of-concept for top candidates
- Stop when you have high confidence or documented why certainty isn't possible
