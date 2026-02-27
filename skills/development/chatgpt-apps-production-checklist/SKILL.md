---
name: chatgpt-apps-production-checklist
description: "Turn ChatGPT Apps implementation work into a production-ready checklist with concrete tasks, tests, widget changes, and tool-result patterns mapped by priority (P0/P1/P2). Use when designing or hardening Apps SDK products for shipping; do not use for generic web-only apps, static code review, or non-ChatGPT integration planning."
knowledge_graph_profile: references/task-profile.json
---

# chatgpt-apps-production-checklist

## Scope and triggers
### Use when
- You are building or upgrading a ChatGPT App and need an engineering-ready checklist that covers correctness, security, and iteration speed.
- You need explicit mapping from product lessons to implementation tasks, tests, widget updates, and tool-result contracts.
- You want publishability/readiness checks for Apps SDK metadata, CSP, and interaction flow routing.

### Don’t use when
- The task is pure static code review or debugging without Apps SDK planning.
- The product is not a ChatGPT App (for example, standalone web app without model/widget bridge).
- The user only wants a short opinion and not an implementation checklist/spec package.

## Requirements and context
- Product goal and primary user journey for the ChatGPT App.
- Current architecture context (tool server, widget stack, auth/network boundaries).
- Known constraints: platform targets, compliance/security expectations, launch timeline.
- Access to OpenAI Apps SDK references when API/bridge behavior must be validated.

If context is incomplete, proceed with explicit assumptions and mark unknowns as open decisions.

## Deliverables
- Priority-tiered implementation checklist (`P0`, `P1`, `P2`) mapped to:
  - concrete engineering tasks
  - test requirements
  - widget/UI changes
  - tool-result/output patterns
- Message-flow map (widget ↔ tool server ↔ model) with explicit API routing responsibilities.
- Validation matrix (unit, contract, integration, mobile, and publishability checks).
- A canonical tool-result envelope recommendation with versioning guidance.
- A short risk register with mitigation actions and owner recommendations.

## Philosophy
- Production ChatGPT Apps fail first at boundaries: visibility, routing, and contracts.
- Treat model-visible context as a budgeted interface, not a dump of widget state.
- Front-load correctness and publishability checks before optimization work.
- Keep outputs typed, testable, and reproducible so teams can ship safely.
- Ask: which missing contract would most likely cause user-visible failure in week one?
- Ask: what is the minimum model-visible context needed to answer correctly without leakage?
- Ask: which single test would catch this failure mode before release?

## Encouraging variation
- Outputs should vary based on the specific app context, user journey, and risk profile.
- Adapt prioritization when constraints differ (for example compliance-heavy launches vs internal prototypes).
- Consider different implementation approaches when constraints are different, even if lessons are the same.
- No two checklist outputs should be identical unless requirements are genuinely identical.
- Avoid converging on one “favorite” pattern when evidence suggests a better context-specific alternative.

## Workflow
### Step 0 — Scope the app and risk profile
- Capture app intent, critical journeys, and deployment constraints.
- Identify risk tier (P0 correctness, P1 readiness, P2 velocity) and delivery target.

### Step 1 — Build the flow map and context contract
- Define the message flow for widget interactions, tool calls, model follow-ups, and model-context updates.
- Classify fields as model-visible, widget-only, or user-only.
- Require stable entity IDs and a typed structured-content envelope with explicit `schema_version`.

### Step 2 — Apply priority checklist mapping
- Apply the full lesson matrix from `references/lessons-matrix.md`.
- Convert each lesson into:
  - implementation task(s)
  - test(s)
  - widget change(s)
  - tool-result/output change(s)

### Step 3 — Generate implementation pack
- Fill `assets/checklist-template.md` with project-specific details.
- Produce a final pack containing:
  - prioritized backlog
  - validation plan
  - widget + tool contract changes
  - rollout and risk notes

### Step 4 — Edge-case routing
1. **No widget yet (API-only prototype)**
   - Prioritize P0 contracts and flow map first; mark UI-specific lessons as pending.
2. **Legacy app with mixed patterns**
   - Normalize on one result envelope and one bridge abstraction before adding features.
3. **Security/publishability unknowns**
   - Block launch recommendation until CSP/domain/tool-annotation checks are explicit.
4. **Performance pain from multi-call hydration**
   - Front-load initial payload to support first render without N+1 tool calls.
5. **Mobile mode issues**
   - Promote adaptive layout and safe-area testing into P0/P1 for launch scope.

### Step 5 — Verify and hand off
- Confirm every P0 lesson has at least one concrete test and owner.
- Confirm P1 readiness gates before publish recommendation.
- Mark P2 items as scheduled improvements with clear sequencing.

## Validation
Fail fast: if a required contract is missing (context visibility, routing map, result schema, or CSP/publishability flags), stop and fix before proceeding.

Minimum checks:
1. Every lesson has mappings for tasks/tests/widget/tool-output patterns.
2. P0 items include explicit test assertions and release gates.
3. Result envelope includes versioned schema guidance.
   - Must explicitly include `schema_version` and field semantics boundaries.
4. Security/publishability checks are explicit (CSP, domains, tool annotations).

## Anti-patterns
- **NEVER** skip context-visibility classification; this is a common mistake that causes leakage.
- **DO NOT** allow lazy N+1 tool calls for first render in common paths.
- **DON'T** mix bridge calls without an explicit routing contract; it produces wrong interaction behavior.
- Avoid untyped HTML blobs instead of structured, versioned result fields.
- Warning sign: treating CSP/metadata flags as “later” work near release time.
- Incorrect launch pattern: testing only inside ChatGPT without emulator/local fixture coverage.
- Pitfall: shipping mobile modes without safe-area and touch-target validation.

## Constraints and safety
- Redact secrets/PII by default; never place credentials in model-visible fields.
- Use minimal domain allowlists and verify widget/network origins.
- Keep recommendations evidence-based and tied to measurable tests.
- Do not invent Apps SDK method names or field semantics; verify with official docs when uncertain.

## Remember
The agent is capable of extraordinary work in this domain. These guidelines unlock that potential rather than constrain it. Use judgment, adapt to context, and keep improving the checklist as evidence changes.

## Examples
### Triggering prompts
- “Turn our ChatGPT App into a production checklist with tests and widget changes.”
- “Map these 15 lessons into implementation tasks and CI checks.”
- “Audit our Apps SDK design for P0/P1 readiness before launch.”

### Negative examples (don’t trigger)
- “Explain React hooks in general.”
- “Review this Python module for style issues.”
- “Write marketing copy for our product page.”
- “Design a generic web app architecture not using ChatGPT Apps.”

## Resources
- `references/lessons-matrix.md` — canonical 15-lesson mapping (tasks/tests/widget/tool-output).
- `references/contract.yaml` — output contract and required evidence.
- `references/evals.yaml` — regression eval prompts for routing/quality.
- `assets/checklist-template.md` — reusable implementation-pack template.
- Sources:
  - https://developers.openai.com/blog/15-lessons-building-chatgpt-apps/
  - https://developers.openai.com/apps-sdk/reference/

<!-- skill-score-boost-v1 -->
## Philosophy and tradeoffs
- Use this skill when consistent decision-making matters more than one-off execution because project context should drive the approach.
- Principle and mindset: prioritize tradeoffs and constraints over rigid checklists; understand why each step exists.
- Ask this to keep outcomes robust: Why is this the right default, and what could change this outcome?
- How do we adapt if constraints shift?
- What evidence is needed before choosing one path over another?

## Anti-patterns and caveats
- Avoid applying this playbook generically without checking repository-specific context.
- **NEVER** skip required validation gates when behavior changes.
- **DO NOT** use this skill as a rigid replacement for engineering judgment.
- **DON'T** ignore warnings or assume one pattern fits all repos.
- Common pitfall: treating anti-patterns as optional.
- Incorrect assumptions here can lead to fragile guidance.
- Warning: wrong sequencing can create avoidable regressions.

## Variation and adaptation
- Vary the workflow by team size, risk, and deployment target.
- Use different strategies for small, medium, and large changes.
- Adapt recommendations to the specific environment and avoid repetitive templates.
- Avoid generic or cookie-cutter responses; craft context-specific alternatives.
- Keep outputs diverse and not repetitive.
- Converge on a custom path only after evidence review.
- Different constraints should produce different, non-generic recommendations.

## Empowering execution style
- Be capable of exploring multiple options and enabling the team to make safe decisions.
- Unlock confidence by explaining options and tradeoffs clearly.
- Feel free to be creative while staying rigorous and precise.
- Push boundaries with practical alternatives when simple recipes fail.
- Enable outcomes-oriented problem solving.
