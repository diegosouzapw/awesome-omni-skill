---
name: code-review
description: Code review patterns and checklist for quality assurance. Use when reviewing code changes, PRs, or validating implementations.
---

# Code Review Standards

## Review Lenses by Domain

### For Code Changes
- **Correctness**: Does it do what it claims?
- **Security**: Are there vulnerabilities?
- **Performance**: Are there bottlenecks?
- **Maintainability**: Can others understand and modify this?
- **Testing**: Is it adequately tested?
- **Error handling**: What happens when things go wrong?

### For Architecture Decisions
- **Scalability**: Will this work at 10x or 100x load?
- **Flexibility**: Can this adapt to changing requirements?
- **Simplicity**: Is this the simplest solution that works?
- **Coupling**: Are components appropriately independent?
- **Standards**: Does this follow established patterns?

### For CI/CD Pipeline Changes
- **Reliability**: Will this make the pipeline more or less stable?
- **Speed**: Impact on build and deploy times?
- **Cost**: Resource usage and runner costs?
- **Debugging**: Can failures be easily diagnosed?
- **Recovery**: What happens when jobs fail?

## Constructive Critique Patterns

### Do This
- Frame critiques to further discussion, not tear down
- Offer specific, actionable feedback with examples
- Acknowledge what is good before pointing out issues
- Suggest alternatives when identifying problems
- Use Socratic questioning to help others discover issues
- Stay focused on the work, never the person

### Effective Challenge Techniques
- Ask "What would make this fail?" before "Will this work?"
- Request evidence for claims and assumptions
- Look for what is missing, not just what is present
- Consider second-order effects
- Test ideas against edge cases
- Question whether the problem itself is correctly defined

## Red Team Thinking

- Assume the role of an adversary trying to break the solution
- Look for the weakest link in any chain of logic
- Consider how requirements might change in the future
- Identify single points of failure
- Question whether success metrics actually measure success

## Antipatterns (Avoid)

### Destructive Criticism
- Criticizing without offering alternatives
- Focusing on style preferences rather than substance
- Making personal attacks
- Being negative for the sake of being negative
- Blocking progress indefinitely

### Analysis Paralysis
- Demanding perfection when good enough will do
- Finding problems without assessing impact
- Treating all issues as equally important
- Continuously moving goalposts

### Poor Communication
- Using vague criticism like "this feels wrong"
- Failing to explain why something is problematic
- Not prioritizing feedback by importance
- Overwhelming with too many issues at once

## When to Escalate Concerns

Raise strong objections when:
- User safety or data privacy is at risk
- Legal or compliance issues are present
- Irreversible actions are proposed without safeguards
- Core architecture is being changed without sufficient analysis
- Technical debt is being added without acknowledgment
