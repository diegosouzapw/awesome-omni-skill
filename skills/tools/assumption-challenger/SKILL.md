---
name: assumption-challenger
description: Identify and challenge implicit assumptions in plans, proposals, and technical decisions. Use when strategic-cto-mentor needs to surface hidden assumptions and wishful thinking before they become costly mistakes.
---

# Assumption Challenger

Systematically identifies and stress-tests assumptions that are treated as facts but may not be validated.

## When to Use

- Validating roadmaps and project plans
- Reviewing architecture proposals
- Assessing build vs buy decisions
- Evaluating timelines and budgets
- Challenging strategic initiatives

## Why Assumptions Matter

Most project failures trace back to invalid assumptions:
- **Timeline assumptions**: "We can ship in 6 weeks" (based on nothing)
- **Resource assumptions**: "We'll hire 3 engineers" (in a tight market)
- **Technical assumptions**: "The API can handle 10K requests/sec" (never tested)
- **Business assumptions**: "Users will adopt this feature" (never validated)

**The cost of invalid assumptions compounds over time.** A bad assumption in week 1 can waste months of work.

---

## Assumption Categories

### 1. Timeline Assumptions
Assumptions about how long things will take.

**Common patterns**:
- "This should only take 2 weeks"
- "We'll have the integration done by launch"
- "The team can absorb this additional scope"

**Challenge questions**:
- What's this estimate based on? Past experience or hope?
- What similar work have we done? How long did it actually take?
- What's not included in this estimate? (Testing, documentation, deployment)
- What happens if this takes 2x longer?

### 2. Resource Assumptions
Assumptions about team capacity and availability.

**Common patterns**:
- "We'll hire 2 senior engineers by Q2"
- "The DevOps team can support this"
- "Sarah can lead this while maintaining her other work"

**Challenge questions**:
- What's the hiring timeline? What if we can't find the right people?
- What's the team's current utilization? Where does time come from?
- Who's the backup if the key person is unavailable?
- What happens if the team is 50% of expected capacity?

### 3. Technical Assumptions
Assumptions about system capabilities and constraints.

**Common patterns**:
- "The database can handle the load"
- "We can integrate with their API easily"
- "Our architecture supports this use case"

**Challenge questions**:
- Has this been tested at the required scale?
- What are the documented limits? What happens at those limits?
- What's the failure mode? How do we recover?
- Have we talked to the API provider about our usage patterns?

### 4. Business Assumptions
Assumptions about market, users, and business outcomes.

**Common patterns**:
- "Users want this feature"
- "This will reduce churn by 20%"
- "The market will wait for our solution"

**Challenge questions**:
- What evidence supports this? User research? Data?
- What if users don't adopt this? What's the fallback?
- What are competitors doing in this space?
- How will we know if this assumption is wrong?

### 5. External Assumptions
Assumptions about factors outside your control.

**Common patterns**:
- "The vendor will have the feature ready"
- "Regulations won't change"
- "The market will remain stable"

**Challenge questions**:
- What's our contingency if this doesn't happen?
- What's the vendor's track record on commitments?
- What early warning signs would indicate this is wrong?
- What's the cost of being wrong?

---

## Assumption Identification Process

### Step 1: Surface Assumptions

Read the proposal and identify statements that are treated as facts but aren't validated:

**Flag statements containing**:
- "We will..." (without evidence)
- "We can..." (without proof)
- "Users want..." (without data)
- "It should..." (without testing)
- "We expect..." (without basis)
- "We assume..." (at least they're honest)

### Step 2: Categorize Assumptions

For each assumption, categorize:

| Category | Risk if Wrong | Validation Difficulty |
|----------|---------------|----------------------|
| Timeline | Project delay | Medium (compare to past) |
| Resource | Execution failure | Medium (check market) |
| Technical | System failure | High (requires testing) |
| Business | Wasted investment | High (requires market validation) |
| External | Plans disrupted | Variable |

### Step 3: Assess Each Assumption

For each significant assumption:

```markdown
### Assumption: [Statement]

**Category**: [Type]
**Stated or Implicit**: [Was it stated or hidden?]

**Evidence Supporting**:
- [Evidence 1]
- [Evidence 2]

**Evidence Against**:
- [Counter-evidence 1]
- [Counter-evidence 2]

**Risk if Wrong**:
- [Impact on timeline]
- [Impact on cost]
- [Impact on success]

**Validation Method**:
- [How to test this assumption]
- [Cost/time to validate]

**Verdict**:
[ ] Valid - Evidence supports
[ ] Questionable - Needs validation
[ ] Invalid - Evidence contradicts
[ ] Unknown - Cannot assess
```

### Step 4: Prioritize Challenges

Focus on assumptions that are:
1. **High impact** - Project fails if wrong
2. **Low evidence** - Based on hope, not data
3. **Testable** - Can be validated before commitment

---

## Challenge Patterns

### The Reality Check
Compare assumption to external data.

**Template**:
> "You assume [X]. Industry data shows [Y]. What makes you different?"

**Example**:
> "You assume you can hire 3 senior engineers in 2 months. The average time-to-hire for senior engineers in your market is 4-6 months. What's your strategy to beat that?"

### The History Test
Compare to organization's past performance.

**Template**:
> "You assume [X]. Last time you attempted [similar thing], it took [Y]. What's changed?"

**Example**:
> "You assume 8 weeks for the microservices migration. Your last infrastructure migration took 5 months. What's different this time?"

### The Stress Test
Push assumption to failure point.

**Template**:
> "You assume [X]. What happens when [stress scenario]?"

**Example**:
> "You assume the system handles 10K concurrent users. What happens during a flash sale with 50K? What's the failure mode?"

### The Dependency Audit
Trace assumption to its dependencies.

**Template**:
> "For [assumption] to be true, what else must be true?"

**Example**:
> "For your 6-week timeline to work, you need: (1) API specs finalized by week 1, (2) no scope changes, (3) 100% team availability, (4) no production incidents. How realistic is that?"

### The Inverse Test
Consider what happens if assumption is wrong.

**Template**:
> "If [assumption] is wrong, what's the impact? What's your Plan B?"

**Example**:
> "If users don't adopt this feature, what's the fallback? Have you defined failure criteria?"

---

## Wishful Thinking Indicators

Red flags that suggest assumption is based on hope rather than evidence:

### 1. The Optimistic Timeline
- "Should only take..."
- "If everything goes well..."
- "We can do it if we're focused..."

**Reality**: Things rarely go perfectly. Add 30-50% buffer.

### 2. The Magical Hiring
- "We'll just hire..."
- "Once we have the team..."
- "We're planning to bring on..."

**Reality**: Hiring takes 3-6 months for senior roles. Onboarding adds another 2-3 months to productivity.

### 3. The Simple Integration
- "It's just an API call..."
- "Should be straightforward..."
- "They have good documentation..."

**Reality**: Integrations always have edge cases, rate limits, and unexpected behaviors.

### 4. The Obvious Market
- "Everyone needs this..."
- "Users have been asking for..."
- "It's clear that..."

**Reality**: "Everyone" is not a market segment. Validate with actual user research.

### 5. The Linear Scaling
- "If we can do X, we can do 10X..."
- "We'll scale as needed..."
- "Growth shouldn't be a problem..."

**Reality**: Scaling is non-linear. What works at 10K users may fail at 100K.

---

## Output Format

When challenging assumptions, provide:

```markdown
# Assumption Analysis: [Plan/Proposal Name]

## Summary
- **Total Assumptions Identified**: [Count]
- **High-Risk Assumptions**: [Count]
- **Requires Immediate Validation**: [Count]

## Critical Assumptions (Must Validate Before Proceeding)

### Assumption 1: [Statement]

**Category**: Timeline / Resource / Technical / Business / External
**Stated or Implicit**: Stated / Implicit

**The Problem**:
[Why this assumption is questionable]

**Evidence For**:
- [Supporting evidence]

**Evidence Against**:
- [Counter-evidence]

**If Wrong, Impact**:
- Timeline: [Impact]
- Budget: [Impact]
- Success: [Impact]

**How to Validate**:
- [Validation method]
- [Time required]
- [Cost]

**Verdict**: Valid / Questionable / Invalid / Unknown

---

### Assumption 2: [Statement]
[Same format]

---

## Medium-Risk Assumptions (Should Validate)
[List with brief analysis]

## Low-Risk Assumptions (Monitor)
[List]

## Recommendations

### Before Proceeding
1. [Validation action 1]
2. [Validation action 2]

### Risk Mitigation
1. [Mitigation for assumption 1]
2. [Mitigation for assumption 2]

### Contingency Plans Needed
1. [Plan B for assumption 1]
2. [Plan B for assumption 2]
```

---

## Integration with Validation

The assumption-challenger skill feeds into the broader validation workflow:

```
Proposal/Plan
     │
     ▼
[assumption-challenger] → List of assumptions with verdicts
     │
     ▼
[antipattern-detector] → Identified anti-patterns
     │
     ▼
[validation-report-generator] → Final 8-section report
```

---

## References

- [Wishful Thinking Patterns](wishful-thinking-patterns.md) - Common patterns to watch for
- [Challenge Questions](challenge-questions.md) - Ready-to-use challenge questions by category
