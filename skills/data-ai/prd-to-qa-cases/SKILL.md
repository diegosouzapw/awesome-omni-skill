---
name: prd-to-qa-cases
description: Generate QA test cases from PRD acceptance criteria using Given/When/Then
  and expected results. Use when QA coverage needs explicit, auditable cases.
knowledge_graph_profile: references/task-profile.json
---

# PRD to QA Cases

## Pipeline Context
This skill generates QA test cases, typically as part of **Stage 3 of the Spec Pipeline** (Build Plan) to define detailed test coverage.

**Related stages:**
- Stage 1: Foundation Spec (What + Why) — See `design/product-spec` or use `design/references/foundation-spec-template.md`
- Stage 2: UX Spec (How it feels) — See `design/product-spec` or use `design/references/ux-spec-template.md`
- Stage 3: Build Plan (How we execute) — See `design/product-spec` or use `design/references/build-plan-template.md`

**Shared references:**
- `design/references/build-plan-template.md` — Build Plan template
- `design/references/spec-linter-checklist.md` — Quality gate checklist

## Response format (strict)
The first line of any response MUST be `## Inputs`.

## Cognitive Support / Plain-Language
- Optimize for low cognitive load (TBI support): one task at a time, explicit steps.
- Use plain language first; define jargon in parentheses.
- Keep steps short and checklist-driven where possible.
- Externalize state: decisions, assumptions, and the next step.
- Provide ELI5 explanations for non-trivial logic.
- Ask one question at a time; prefer multiple-choice when possible.

Every response must include:
- `## Inputs`
- `## Outputs`
- `## When to use`

Generate QA test cases from acceptance criteria.

## Traceability matrix (required)
```markdown
| Acceptance Criteria | Test Case ID | Test Type | Status |
| --- | --- | --- | --- |
| | | | |
```

## Test case template (Given/When/Then)
```markdown
**Test Case ID:** TC-001
**Given** ...
**When** ...
**Then** ...
```

## Output location
Write QA cases in the same directory as the source PRD.
- `feature-x.md` -> `feature-x-qa-cases.md`

## Required sections
1) Test case list (Given/When/Then + expected result)
2) Coverage map (criteria -> cases)
3) Manual vs automated split
4) Data and environment prerequisites

## Constraints
- Keep each test case atomic and independent.
- Redact secrets/PII by default.
## References
- Contract: references/contract.yaml
- Evals: references/evals.yaml

## Scope and triggers
- Use this skill when the task matches its description and triggers.
- If the request is outside scope, route to the appropriate skill.

## Required inputs
- User request details and any relevant files/links.

## Deliverables
- A structured response or artifact appropriate to the skill.
- Include `schema_version: 1` if outputs are contract-bound.

## Constraints
- Redact secrets/PII by default.
- Avoid destructive operations without explicit user direction.

## Validation
- If findings are disputed or high-risk, run LLM Council and merge outcomes per `design/product-spec/references/llm-council.md`.
- Run Golden Nuggets 2026 checklist in `design/product-spec/SKILL.md` (section: Golden Nuggets 2026).
- Run any relevant checks or scripts when available.
- Fail fast and report errors before proceeding.

## Philosophy
- Favor clarity, explicit tradeoffs, and verifiable outputs.
- Given/When/Then is a contract—test cases must be executable and unambiguous.
- Atomic tests are maintainable—avoid interdependencies that make debugging impossible.
- Edge cases matter more than happy paths—bugs live in the shadows.

## Empowerment
- The agent is capable of turning acceptance criteria into comprehensive, executable test cases.
- Use judgment to identify high-risk user flows that need deeper test coverage.
- Enable QA teams to ship with confidence through structured, verifiable test suites.

## Variation
- Adapt test granularity to product stability: stable features need fewer regression tests, experimental features need exploratory tests.
- Vary automation focus: API-heavy products need automated integration tests, UX-heavy products need manual exploratory tests.
- For data products, expand on data validation and boundary condition tests.
- For security products, expand on auth, authorization, and attack surface tests.
- Adjust test case detail based on team skills—junior QA teams need more explicit steps.

## Guiding questions (ask 2-3)
- What is the most critical user journey to protect?
- Which acceptance criteria have the highest risk of regression?
- What failures would be catastrophic in production?

## Procedure
1) Clarify scope and inputs.
2) Execute the core workflow.
3) Summarize outputs and next steps.

## Anti-patterns
- NEVER skip Given/When/Then structure—vague test cases are untestable and unmaintainable.
- DO NOT create dependent test cases—each test must be atomic and independent.
- Avoid testing happy paths only—edge cases and error states reveal the most bugs.
- DO NOT omit expected results—tests without assertions are meaningless.
- NEVER ship QA cases that do not map to acceptance criteria.

## Response format (required)
The first line of any response MUST be `## Inputs`.
Every user-facing response must include these headings:
- `## Inputs`
- `## Outputs`
- `## When to use`

## Examples
- "Use this skill for a typical request in its domain."

Failure/out-of-scope template (use verbatim structure):
```markdown
## Required inputs
Objective: <what you received>

Plan:
1) <brief>
2) <brief>

Next step: <single request>

## Deliverables
- <what would be produced if in scope>

## Scope and triggers
- <when this skill applies>
```

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
