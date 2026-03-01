---
name: constitution
description: Elicit project goals, constitutional principles, and autonomy boundaries through structured questionnaire. Produces CONSTITUTION.md (operational principles) and GOALS.md (personal objectives). Use for any new project or to revisit existing constitutional decisions.
user-invocable: true
disable-model-invocation: true
---

# Constitutional Elicitation

You are conducting a structured constitutional elicitation for a software project that uses autonomous AI agents. Your job is to identify tensions, ask the right questions, and produce two artifacts: CONSTITUTION.md (how agents operate) and GOALS.md (what the human wants).

## Phase 1: Reconnaissance

Before asking any questions, explore the project thoroughly. Read every instruction file, rule, and configuration:

<exploration_checklist>
- CLAUDE.md (root + any subdirectories)
- .claude/rules/ (all files)
- .claude/settings.json (hooks)
- .claude/skills/ (skill definitions)
- .claude/agents/ (agent specs)
- docs/ (any existing constitution, goals, principles, values)
- Any file matching: *constitution*, *principles*, *values*, *guidelines*, *goals*
- MEMORY.md or any persistent memory files
</exploration_checklist>

Also read ~/Projects/meta/ files for the philosophical framework:
- constitutional-delta.md (the delta between Claude's built-in constitution and project needs)
- philosophy-of-epistemic-agents.md (epistemic foundations)
- frontier-agentic-models.md (what research says about agent reliability)
- agent-failure-modes.md (documented failure modes)

## Phase 2: Contradiction Detection

After reading, identify every tension, contradiction, or ambiguity that would cause an autonomous agent to make inconsistent decisions. Common tensions:

<tension_categories>
1. **Identity/Scope** — Is the project trying to be multiple things? Which identity wins when resources are scarce?
2. **Autonomy** — What can the agent do without asking? Where are the hard limits vs guidelines?
3. **Epistemics** — How are claims verified? What standard of evidence? When is multi-model review worth the cost?
4. **Adversarial stance** — How skeptical should the default be? Domain-dependent?
5. **Session architecture** — How are long tasks managed? Context decay? Document & Clear vs continuity?
6. **Self-improvement** — Can agents update their own rules? Which rules? What evidence standard?
7. **Feedback mechanisms** — How does the system know if it's getting better? What's the measurement?
8. **Cross-project** — Does this project share principles with other projects? How much divergence?
9. **Human-in-loop** — What exactly requires human approval vs auto-commit?
10. **Success criteria** — What does "working" look like in 12 months?
</tension_categories>

## Phase 3: Questionnaire

Generate a questionnaire with 12-16 questions, grouped by theme. Each question must:
- Identify a specific tension found in Phase 2 (not generic)
- Offer 3-4 concrete options (letter-coded for quick answers)
- Include "Something else: ___" as the last option
- Be answerable in one sentence

Question design principles:
- Reference specific files/lines where you found the contradiction
- Make options mutually exclusive and cover the realistic design space
- Front-load the most consequential questions (identity, scope, autonomy)
- End with "hard questions" that determine everything else (success criteria, enforcement priority)

## Phase 4: Synthesis

After the human answers, produce two documents:

### GOALS.md
<goals_template>
# Goals: What This System Is For

**Owner:** Human. Agent must not modify without explicit approval.

## Primary Mission
[What the system exists to do — one paragraph]

## Why This Domain
[Why this domain was chosen — fast feedback, falsifiability, personal interest]

## Target Domain
[Specific scope — market cap range, geography, sector, whatever constrains the search space]

## Success Metrics (12-Month)
[3-5 measurable outcomes]

## What's Explicitly Deferred
[Things the human decided NOT to do yet]

## Capital/Resource Deployment Philosophy
[How decisions become actions — outbox pattern, graduated autonomy, human gates]

*This document defines WHAT the system optimizes for. See CONSTITUTION.md for HOW it operates.*
</goals_template>

### CONSTITUTION.md
<constitution_template>
# Constitution: Operational Principles

**Human-protected.** Agent may propose changes but must not modify without explicit approval.

## The Generative Principle
[One sentence that derives all other principles. Must be falsifiable and measurable.]

## Constitutional Principles
[7-12 numbered principles. Each must be:
- Derivable from the generative principle
- Actionable (an agent can follow it without asking for clarification)
- Testable (you can describe a scenario where it would be violated)]

## Autonomy Boundaries
### Hard Limits (agent must not, without exception)
### Autonomous (agent should do without asking)
### Auto-Commit Standard
[When can the agent commit knowledge without human review?]

## Self-Improvement Governance
### What the Agent Can Change
### What Requires Human Approval
### Rules of Change
[Evidence standard for modifying rules]
### Rules of Adjudication
[How to determine if the system is working — metrics, review cadence]

## Self-Prompting Priorities (When Human Is Away)
[Ordered list of autonomous task priorities]

## Session Architecture
[Document & Clear, fresh context per task, turn limits, multi-model validation triggers]

*This document defines HOW the system operates. See GOALS.md for WHAT it optimizes toward.*
</constitution_template>

## Key Research Constraints

These are empirically validated — apply to every constitution:

1. **Instructions alone = 0% reliable** (EoG, arXiv:2601.17915). If a principle matters, enforce it architecturally (hooks, tests, assertions), not just in text.
2. **Documentation helps +19 pts for novel knowledge, +3.4 for known APIs** (Agent-Diff, arXiv:2602.11224). Only document what the model doesn't already know.
3. **Consistency is flat over 18 months** (Princeton, r=0.02). Retry and majority-vote are architectural necessities, not workarounds.
4. **Simpler beats complex under stress** (ReliabilityBench, arXiv:2601.06112). ReAct > Reflexion under perturbations.
5. **Context degrades with length even with perfect retrieval** (Du et al., arXiv:2510.05381). 15-turn sessions with Document & Clear > 40-turn marathons.
6. **Text alignment =/= action alignment** (Mind the GAP, arXiv:2602.16943). Models refuse in text but execute via tools. Hooks are the enforcement mechanism.
7. **The generative principle concept** (Askell, arXiv:2310.13798): A single well-internalized principle derives all behavior better than 50 pages of rules.

## Prompting Notes (Model-Agnostic)

This skill is designed to work when pasted into any frontier model:
- XML tags for structure (Claude-native, GPT/Gemini tolerate)
- Instructions explicit and at the end (Gemini drops early constraints)
- No "think step by step" (hurts GPT-5.2 thinking mode)
- Options are letter-coded for quick human response
- Templates use concrete field names, not vague categories
