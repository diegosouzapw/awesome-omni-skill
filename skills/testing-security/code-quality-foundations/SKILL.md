---
name: code-quality-foundations
description: Code quality pillars, goals, abstraction layers, and tradeoff thinking. Use when evaluating code quality, setting quality goals, choosing abstraction levels, making design tradeoffs, or auditing code against quality pillars. Covers readability, modularity, testability, reusability, and the principle of least astonishment.
---

# Code Quality Foundations

## The Four Goals of High-Quality Code

| Goal | Question to Ask | Failure Mode |
|------|----------------|--------------|
| **It works** | Does it solve the problem correctly? | Bugs, unhandled edge cases, unmet requirements |
| **It keeps working** | Will it survive changes around it? | Brittle dependencies, no tests, hidden assumptions |
| **It's adaptable** | Can requirements change without a rewrite? | Rigid coupling, over-engineering, hardcoded assumptions |
| **It doesn't reinvent the wheel** | Are we reusing proven solutions? | Custom code for solved problems, duplicated effort |

## The Six Pillars of Code Quality

| Pillar | What It Means | Key Technique |
|--------|--------------|---------------|
| **Readable** | Other engineers can understand it quickly | Descriptive names, clean layers, consistent style |
| **No surprises** | Behavior matches expectations (POLA) | Explicit contracts, no magic values, no hidden side effects |
| **Hard to misuse** | Difficult to use incorrectly | Type safety, immutability, validated construction |
| **Modular** | Components can be swapped independently | Interfaces, DI, separation of concerns, cohesion |
| **Reusable & generalizable** | Solves problems broadly, not just one case | Focused parameters, generics, avoiding assumptions |
| **Testable** | Can be verified in isolation | Modularity enables testability; design for it from the start |

**POLA** = Principle of Least Astonishment. If a function does something a reasonable caller wouldn't expect, it's a bug in the design, even if it "works."

### Modern Additions (2024-2026 Industry Consensus)

| Concern | Why It's Now Explicit | Original Coverage |
|---------|----------------------|-------------------|
| **Secure** | Shift-left security; threat model at design time | Implicit in "works" |
| **Efficient** | Resource optimization is a design choice, not afterthought | Subsumed under "works" |
| **Reliable** | Explicit error recovery and graceful degradation | Subsumed under "keeps working" |

## Decision Tables

### "Should I Abstract This?"

| Signal | Action |
|--------|--------|
| Same logic appears in 2+ places | Extract to shared function/class |
| Function can't be described in one sentence | Split into smaller functions |
| Class has method groups that use different fields | Consider splitting into separate classes |
| Changing one feature requires touching many files | Improve modularity and layer boundaries |
| Only one use case exists | Wait — don't abstract prematurely |

### "Where Should I Invest Quality Effort?"

| Situation | Focus On | Rationale |
|-----------|----------|-----------|
| Greenfield project | Modularity, clean interfaces | Foundation decisions compound; structure early |
| Legacy codebase | Testing, readability | Understand before changing safely |
| High-traffic service | Reliability, efficiency, testing | Production failures are expensive |
| Prototype / spike | Working code, speed | Validate the idea; plan to rewrite |
| Shared library / API | Hard to misuse, no surprises | You can't predict how consumers will use it |
| Security-sensitive code | Security, testability | Breaches are catastrophic and trust-destroying |

## Checklists

### Before Writing Code

- [ ] Problem clearly understood (requirements, constraints, edge cases)
- [ ] Existing solutions checked (libraries, shared code, prior art)
- [ ] Testing strategy considered (how will this be verified?)
- [ ] Key tradeoffs identified (what are we choosing, and what are we giving up?)

### Code Quality Self-Review

- [ ] Functions translate to single sentences
- [ ] No surprises: behavior matches names and types
- [ ] Hard to misuse: invalid states are unrepresentable where possible
- [ ] Modular: changes are localized, not scattered
- [ ] Testable: can be unit tested without complex setup
- [ ] Clean abstractions: API doesn't leak implementation details
