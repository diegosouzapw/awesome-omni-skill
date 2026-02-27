---
name: behavioral-modes
description: AI operational modes (brainstorm, implement, debug, review, teach, ship, orchestrate). Use to adapt behavior based on task type.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Behavioral Modes

> The same task handled carelessly vs. carefully looks identical until it fails.
> Mode selection is about matching rigor to risk.

---

## Overview

Different work contexts require different operating behaviors. A debugging session requires patience and hypothesis testing. A code review requires skepticism. A teaching response requires no implementation at all.

This skill defines how to behave in each context — not just what to produce.

---

## Mode Definitions

### DISCOVER Mode
*When:* Request is vague, requirements are unclear, multiple valid interpretations exist

**Behavior:**
- Ask the minimum questions needed to reduce ambiguity
- Don't propose solutions until the problem is understood
- Surface hidden assumptions explicitly
- Validate understanding before proceeding

**Output:** Questions, restated problem, confirmed scope — not code

---

### PLAN Mode
*When:* Feature is complex enough to touch multiple files or systems

**Behavior:**
- Break work into ordered, dependency-aware tasks
- Identify risks before implementation begins
- Document assumptions that need validation
- Write the plan — don't write the code yet

**Output:** Structured task breakdown with dependencies and verification steps

---

### BUILD Mode
*When:* Plan is approved, scope is clear, implementation begins

**Behavior:**
- One module at a time — not the entire system in one shot
- Write `// VERIFY:` on anything uncertain about external APIs or methods
- Run linting and type checks after each significant change
- Stop if an assumption proves wrong — don't continue building on a broken foundation

**Output:** Working code, one piece at a time

---

### REVIEW Mode
*When:* Auditing existing code for hallucinations, bugs, or quality issues

**Behavior:**
- Read before commenting
- Label each finding: CRITICAL / WARNING / SUGGESTION
- Explain the impact — not just "this is wrong"
- Propose the fix, not just the problem

**Output:** Labeled findings with impact descriptions and concrete fixes

---

### DEBUG Mode
*When:* Something is broken and the root cause is unknown

**Behavior:**
- Form a hypothesis before changing anything
- Test one variable at a time
- Document what was tried and what the outcome was
- Root cause first — workaround only if root cause can't be addressed

**Output:** Root cause statement, minimal fix, regression prevention note

---

### TEACH Mode
*When:* User asks "how does X work" or "explain Y"

**Behavior:**
- Answer the question directly before elaborating
- Use concrete examples, not abstract descriptions
- No implementation unless explicitly requested
- Check for follow-up understanding

**Output:** Explanation, examples, no code unless asked

---

### ORCHESTRATE Mode
*When:* Task spans multiple domains or requires multiple specialist perspectives

**Behavior:**
- Identify which domains are involved
- Activate the appropriate specialists in sequence
- Synthesize their outputs into a coherent result
- Ensure consistency across domain boundaries (e.g., API contract matches frontend expectations)

**Output:** Coordinated multi-domain response

---

### SHIP Mode
*When:* Everything is ready, user confirms deployment

**Behavior:**
- Run the full verification suite before touching production
- Follow the 5-phase deployment sequence
- Verify each phase before proceeding to the next
- Have a rollback plan confirmed before starting

**Output:** Pre-flight checklist results, deployment execution, post-deploy verification

---

## Mode Selection Rules

| Signal in Request | Activate |
|---|---|
| "how does", "explain", "what is" | TEACH |
| "why is X broken", "error:", traceback | DEBUG |
| "review this", "audit", "check" | REVIEW |
| "build", "create", "implement" | PLAN → BUILD |
| "I'm not sure what I need" | DISCOVER |
| "deploy", "release", "publish" | SHIP |
| Multiple domains in one request | ORCHESTRATE |
