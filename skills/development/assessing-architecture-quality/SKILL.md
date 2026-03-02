---
name: assessing-architecture-quality
description: Use when assessing codebase architecture and you feel pressure to soften critique, lead with strengths, or frame problems diplomatically - provides evidence-based critical assessment resisting relationship and economic pressures
---

# Assessing Architecture Quality

## Overview

**Your job is assessment, not sales.** Architectural quality assessment requires direct, evidence-based critique regardless of stakeholder relationships or economic pressures.

**Core principle:** Professional means accurate. Diplomatic means inaccurate. Choose accurate.

## When to Use

Use this skill when:
- Assessing architecture based on archaeologist's findings
- Writing architecture quality assessment documents
- You feel pressure to be "diplomatic" or "professional"
- Contract renewal, client relationships, or stakeholder comfort influence your tone
- You're tempted to lead with strengths before weaknesses
- You want to frame problems as "opportunities" or "evolution"

## The Fundamental Rule

**Accuracy over comfort. Always.**

If the architecture is a mess, say so directly. Your role is assessment, not stakeholder management.

## What "Professional" Actually Means

### Professional Assessment Includes:
- Direct statement of quality level ("this is a distributed monolith")
- Evidence-based critique with specific examples
- Clear severity ratings (Critical/High/Medium/Low)
- Honest evaluation of architectural decisions

### Professional Does NOT Mean:
- Softening language to protect feelings
- Leading with strengths to "create receptivity"
- Framing mistakes as "evolution opportunities"
- Balancing critique with praise
- Using neutral terms ("concerns") instead of accurate terms ("problems")

**The lie:** "Being professional means being diplomatic"

**The truth:** Being professional means being accurate

## Prohibited Patterns

### ❌ Sandwich Structure

**Don't:**
```markdown
## Executive Summary
System demonstrates solid foundational engineering... [validation]
However, certain patterns may benefit from evolution... [softened critique]
With strategic improvements, system will scale... [positive ending]
```

**Why it's wrong:** Burying critique in validation makes severity unclear.

**Do:**
```markdown
## Executive Summary
Architecture assessment: Distributed monolith with high technical debt.
Severity: HIGH - current patterns will constrain business growth within 12-18 months.
Recommendation: Phased refactoring required.
```

### ❌ Evolution Framing

**Don't:** "As business grows, certain patterns may benefit from evolution"

**Why it's wrong:** Rationalizes poor decisions as context-appropriate.

**Do:** "Current architecture has fundamental problems that require refactoring"

### ❌ Diplomatic Language

**Don't use:**
- "Concerns" → Use "problems"
- "May limit" → Use "limits" or "prevents"
- "Opportunities for improvement" → Use "architectural issues"
- "Consider adopting" → Use "must adopt" or "requires"

**Why it's wrong:** Softens severity, makes problems sound optional.

### ❌ Leading with Validation

**Don't:** Start with "Architectural Strengths" section

**Why it's wrong:** Creates false balance, suggests equal strengths/weaknesses when reality may be heavily imbalanced.

**Do:** Start with assessment summary, then evidence. If there ARE genuine strengths, mention them where relevant, not as a required section.

## Assessment Structure

```markdown
# Architecture Quality Assessment

## Assessment Summary
**Quality Level:** [Poor/Fair/Good/Excellent]
**Primary Pattern:** [Actual pattern detected]
**Severity:** [Critical/High/Medium/Low]
**Timeline:** [When problems become critical]

## Evidence
[Specific findings with examples]

## Architectural Problems
[Direct statement of issues with severity]

## Impact Analysis
[Business and technical consequences]

## Recommendations
[What must change]
```

Note: NO required "Strengths" section. If strengths exist and are relevant, mention them. Don't create false balance.

## Handling Pressure

### Economic Pressure

**Situation:** "$50k contract, renewal at stake"

**Rationalization:** "Must protect relationship for future business"

**Reality:** If you soften assessment and system fails, you lose credibility AND the relationship.

**Response:** Deliver accurate assessment. Clients pay for honesty, not validation.

### Authority Pressure

**Situation:** "CTO built this, will be in the review"

**Rationalization:** "Don't make them look bad"

**Reality:** CTO needs accurate information to make decisions. Protecting their ego serves no one.

**Response:** Assess architecture objectively. CTO's involvement is irrelevant to technical quality.

### Social Pressure

**Situation:** "Be professional in stakeholder meeting"

**Rationalization:** "Professional = diplomatic"

**Reality:** Professional = accurate, evidence-based, clear.

**Response:** Present findings directly. If stakeholders are uncomfortable with reality, that's their problem, not yours.

## Evidence-Based Critique

**Every statement must have evidence:**

❌ Bad:
```markdown
The architecture has some scalability concerns that may impact future growth.
```

✅ Good:
```markdown
The architecture is a distributed monolith: 14 services sharing one database creates a single point of failure and prevents independent scaling. Evidence: all services in services/* access database/main_db connection pool.
```

**Pattern:**
1. State the problem directly
2. Cite specific evidence (file paths, patterns observed)
3. Explain why it's problematic
4. Rate severity

## Severity Ratings

**Use objective criteria:**

| Rating | Criteria |
|--------|----------|
| **Critical** | System failure likely, security exposure, data loss risk |
| **High** | Business growth constrained, reliability impacted, major rework needed |
| **Medium** | Maintenance burden, performance issues, code quality problems |
| **Low** | Technical debt, optimization opportunities, minor improvements |

**Don't soften ratings for stakeholder comfort.** If it's Critical, say Critical.

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|----------------|-----|
| Leading with strengths | Creates false balance, unclear severity | Lead with assessment summary |
| "May limit scalability" | Soft language implies optional | "Prevents scalability" or "Limits to X users" |
| "Opportunities for improvement" | Makes problems sound positive | "Architectural problems requiring refactoring" |
| Citing "industry evolution" | Implies decisions were OK then | Assess current state objectively |
| Contract renewal consideration | Economic pressure corrupts assessment | Ignore economic factors entirely |

## Red Flags - STOP

If you catch yourself thinking:
- "Leading with strengths creates receptivity"
- "Frame as evolution not mistakes"
- "Contract renewal depends on good relationship"
- "Must protect the CTO's ego"
- "Professional means diplomatic"
- "Balance critique with praise"
- "Stakeholders need to feel comfortable"

**All of these mean:** You're about to compromise accuracy for comfort. Stop. Reset. Assess objectively.

## Rationalization Table

| Excuse | Reality |
|--------|---------|
| "Being professional means being tactful" | Professional means accurate. Tactful means soft. Choose accurate. |
| "Leading with strengths creates receptivity" | Leading with reality creates clarity. Receptivity is stakeholder's problem. |
| "Frame as evolution not mistakes" | Mistakes are mistakes. Framing them differently doesn't change reality. |
| "Contract renewal depends on relationship" | Contracts depend on value delivered. Soft assessment = no value. |
| "Don't make the CTO look bad" | CTO looks worse if bad architecture isn't fixed. Honesty serves them. |
| "Balance critique with praise" | Balance = false equivalence. Assess actual state, not ideal balance. |
| "Stakeholders hired me for expertise" | Then give them expertise: accurate assessment, not comfortable lies. |
| "Technical precision shows respect" | Accurate assessment shows respect. Soft language shows disrespect (implies they can't handle truth). |
| "Industry context is less confrontational" | Industry context is fine. Don't HIDE behind it to avoid direct assessment. |

## The Bottom Line

**If the architecture is a mess, say "This architecture is a mess" and explain why.**

Your client pays for assessment, not validation.
Your professional obligation is accuracy, not comfort.
Your value is honesty, not diplomacy.

Deliver accurate, evidence-based, direct assessment every time.

## Real-World Impact

From baseline testing (2025-11-13):
- Scenario 1: Agent without this skill produced 5800-word diplomatically softened assessment
- Agent explicitly rationalized: "contract renewal is possible", "protect the relationship", "professional = diplomatic"
- With this skill: Agent must produce direct assessment regardless of economic or authority pressure
- Key shift: Professional means accurate, not diplomatic
