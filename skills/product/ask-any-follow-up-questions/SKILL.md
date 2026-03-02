---
name: ask-any-follow-up-questions
description: Ask follow-up questions to clarify requirements, constraints, and success criteria. Do not use automatically, only when invoked explicitly.
---

# Ask Any Follow-Up Questions

## Goal

Ask follow-up questions (must-have first, then nice-to-have) to avoid wrong work and reduce rework. Do not implement until must-have questions are answered, or the user explicitly approves proceeding with stated assumptions.

## When to use

Use this skill only when explicitly invoked. Once invoked:
- You may ask clarifying questions even if the request seems mostly specified.
- Still optimize for speed: ask the smallest set that meaningfully reduces ambiguity.

## Workflow

### 1) Identify what’s unclear or risky

After a quick mental check of how you would do the work, consider gaps in:
- Objective (what should change vs stay the same)
- Definition of done (acceptance criteria, examples, edge cases)
- Scope (files/components/users in/out)
- Constraints (compatibility, performance, style, dependencies, time)
- Environment (language/runtime versions, OS, build/test runner)
- Safety/reversibility (migrations, rollout/rollback, risk level)

If multiple plausible interpretations exist, treat as underspecified.

### 2) Ask must-have questions first (keep it small)

Ask 1–5 must-have questions first. Prefer questions that eliminate whole branches of work.

Make questions easy to answer:
- Numbered questions; short; scannable
- Multiple-choice options when possible
- Include recommended defaults (clearly marked)
- Include a fast-path response (e.g., reply `defaults`)
- Include “Not sure — use default” when helpful
- Provide a compact reply format (e.g., `1b 2a 3c`)

### 3) Ask nice-to-have questions (only if useful)

After must-haves are answered (or defaults accepted), optionally ask up to 1–3 nice-to-have questions that:
- Improve UX/maintainability/perf
- Reduce future rework
- Clarify preferences (naming, style, logging, docs)

If user wants speed, skip nice-to-haves.

### 4) Pause before acting

Until must-have answers arrive:
- Do not run commands, edit files, or produce a plan that depends on unknowns.
- Allowed: low-risk discovery only (read docs/configs, inspect repo structure) if it doesn’t commit to a direction.

If user asks you to proceed without answers:
- State assumptions as a short numbered list
- Ask for confirmation
- Proceed only after they confirm or correct assumptions

### 5) Confirm interpretation, then proceed

Once answers are received:
- Restate requirements in 1–3 sentences (objective + scope + constraints + done)
- Then implement

## Question templates

- “Before I start, I need: (1) … (2) … (3) …. If you don’t care about (2), I’ll assume ….”
- “Which should it be? A) … B) … C) … (pick one)”
- “What does ‘done’ look like? Example: …”
- “Any constraints (versions, perf, style, deps)? If none, I’ll target existing project defaults.”

### Reply format example

```text
1) Scope?
a) Minimal change (recommended/default)
b) Refactor while touching the area
c) Not sure — use default

2) Compatibility target?
a) Current project defaults (recommended/default)
b) Also support older versions: <specify>
c) Not sure — use default

3) Verification?
a) Existing tests only (recommended/default)
b) Add/adjust tests for new behavior
c) Not sure — use default

Reply with: defaults (or 1a 2a 3b)
```

## Anti-patterns

- Don’t ask questions you can answer with quick, low-risk discovery reads (configs, existing patterns, docs).
- Don’t ask open-ended questions when tight options would eliminate ambiguity faster.
- Don’t “question-spam”: if answers won’t change implementation, skip.
