---
name: ambiguity-detection
description: Detects critical product, scope, data, risk, and success ambiguities in requirements or PRDs and expresses them as structured, decision-forcing clarification questions without proposing solutions or workflow actions.
---

# Ambiguity Detection & Clarification Generation

## Purpose

This skill identifies **critical decision gaps** in product requirements or PRDs that, if left unresolved, would lead to misalignment, rework, or irreversible downstream mistakes.

It does **not** resolve ambiguity.  
It **surfaces it precisely and neutrally** as structured clarification questions.

Use this skill as a validation pass before roadmap definition, feature decomposition, or execution planning.

---

## When to Use This Skill

Use this skill when you need to:

- Validate whether a PRD or requirement set is **decision-complete**
- Detect hidden assumptions that affect scope, data ownership, or risk
- Prepare structured clarification questions for stakeholders
- Ensure irreversible or high-impact decisions are made explicitly

Do **not** use this skill to:

- answer questions
- define defaults
- decide priority or severity
- pause or resume workflows
- rewrite PRDs
- plan implementation or UX

---

## Core Principle

**If a missing decision could change the shape of the product, it must be surfaced.**

This skill favors:

- precision over completeness
- decision-forcing questions over open-ended discussion
- minimal, high-signal outputs

---

## What Counts as Ambiguity

Ambiguity is **not** missing detail.

Ambiguity **is** unresolved uncertainty that affects:

- product boundaries
- user trust or responsibility
- data authority or mutability
- irreversible workflows
- compliance or risk posture
- success or failure interpretation

If different answers would lead to materially different designs, it is ambiguity.

---

## Ambiguity Detection Categories

Evaluate the input strictly across the following categories.

### 1. User & Actor Ambiguity

Detect uncertainty about:

- primary vs secondary users
- conflicting incentives between actors
- explicitly out-of-scope users or roles

---

### 2. Scope Boundary Ambiguity

Detect uncertainty about:

- where the product’s responsibility starts and ends
- delegated vs owned behavior
- edge cases at integration boundaries

---

### 3. Data & State Ambiguity

Detect uncertainty about:

- authoritative data sources
- mutable vs immutable state
- derived vs stored data
- ownership across systems

---

### 4. Workflow & Control Ambiguity

Detect uncertainty about:

- irreversible actions
- retry or rollback expectations
- partial failure handling
- required vs optional steps

(This is conceptual, not orchestration logic.)

---

### 5. Risk, Trust & Compliance Ambiguity

Detect uncertainty about:

- regulatory or legal assumptions
- auditability requirements
- security or privacy expectations
- user consent or disclosure boundaries

---

### 6. Success & Failure Ambiguity

Detect uncertainty about:

- how success is evaluated
- acceptable failure modes
- trade-offs between competing outcomes

---

## Question Generation Guidelines

When ambiguity is detected:

- Ask **decision-forcing** questions
- Avoid leading language
- Avoid implied defaults
- Provide structured options only when they clarify the decision space
- Prefer fewer, higher-impact questions

### Bad Question
>
> “Should we handle errors better?”

### Good Question
>
> “If an external dependency fails mid-operation, should the system automatically roll back, allow partial completion, or require manual intervention?”

---

## Output Format

The output should be **Markdown content only**, suitable for direct inclusion in a clarification document.

Use the following structure:

```markdown
# Project Clarifications

> Please review and select options or provide input for each question.

## Q1: [Decision Area]

- [ ] Option A: [Description]
- [ ] Option B: [Description]
- [ ] Other: [Please specify]

## Q2: [Decision Area]
...
```

Only include options when they meaningfully bound the decision space.

---

## Important Boundaries

This skill **must not**:

- ask the user questions directly
- decide whether execution should pause
- infer or assume answers
- modify or rewrite PRD content
- propose implementation approaches
- create files or trigger tools
- prioritize or rank ambiguities

All orchestration and decision flow belongs to the calling agent.

---

## Output Expectations

The output of this skill should be:

- concise and high-signal
- free of speculation
- neutral in tone
- deterministic for the same input
- focused on decisions that materially affect product shape

Assume the output will be reviewed by senior product, engineering, and compliance stakeholders.
