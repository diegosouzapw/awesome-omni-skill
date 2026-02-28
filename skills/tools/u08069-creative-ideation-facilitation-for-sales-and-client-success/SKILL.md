---
name: u08069-creative-ideation-facilitation-for-sales-and-client-success
description: Operate the "Creative Ideation Facilitation for sales and client success" capability in production for sales and client success workflows. Use when mission execution explicitly requires this capability and outcomes must be reproducible, policy-gated, and handoff-ready.
---

# Creative Ideation Facilitation for sales and client success

## Metadata
- **Skill ID:** `u08069-creative-ideation-facilitation-for-sales-and-client-success`
- **Capability:** Creative Ideation Facilitation for sales and client success
- **Domain:** sales and client success
- **Purpose:** Operate the "Creative Ideation Facilitation for sales and client success" capability in production for sales and client success workflows. Use when mission execution explicitly requires this capability and outcomes must be reproducible, policy-gated, and handoff-ready.
- **Use When:** The request explicitly needs this capability and requires a concrete, handoff-ready result.

## Allowed Tools
- Use only tools allowed by the current runtime policy.
- Preferred local tools: `read`, `write`, `edit`, `exec`, `process`.
- Optional research tools (only when evidence is needed): `web_search`, `web_fetch`, `browser`.
- If no tool is necessary, complete using reasoning plus provided context.

## Inputs (formatted)
```yaml
required:
  objective: string
  context: string
  constraints:
    - string
  success_criteria:
    - string
optional:
  prior_artifacts:
    - path_or_url
  stakeholders:
    - string
  deadline: string
  risk_level: low|medium|high
```

## Outputs (formatted)
```yaml
primary_output:
  type: markdown|json|text
  summary: string
supporting_output:
  assumptions:
    - string
  decisions:
    - string
  next_actions:
    - string
status:
  validation: pass|fail
  blockers:
    - string
```

## Guidelines
1. Clarify goal, audience, and constraints before producing deliverables.
2. Reuse existing artifacts where possible to avoid duplicated work.
3. Keep outputs concise, actionable, and aligned to the stated success criteria.
4. Document assumptions and unresolved questions explicitly.
5. Prefer reversible, low-risk recommendations when uncertainty is high.
6. Escalate for human review when policy, legal, safety, or reputation risk is material.

## Musts
- Must keep all claims tied to provided evidence or clearly labeled assumptions.
- Must provide at least one concrete next action.
- Must state blockers if requirements are incomplete.
- Must fail closed (no publish/send action) when required approvals are missing.

## Targets (day/week/month operating cadence)
- **Day:** Process active requests, triage priority, and publish a clear status outcome.
- **Week:** Review repeated blockers, improve templates/checklists, and tune decision quality.
- **Month:** Audit outcomes, retire low-value steps, and update operating guidance from lessons learned.

## Common Actions
- `triage_request`
- `collect_required_inputs`
- `draft_output`
- `validate_against_success_criteria`
- `prepare_handoff_bundle`
- `escalate_or_close`

## External Tool Calls Needed
- **Default:** None required.
- **When needed:**
  - Use `web_search`/`web_fetch` for external evidence verification.
  - Use `browser` only for UI-dependent retrieval or validation.
  - Use execution tools (`exec`/`process`) only for local, auditable transformations.

## Validation & Handoff
```yaml
validation_checks:
  - inputs_complete
  - constraints_applied
  - output_meets_success_criteria
  - risks_and_assumptions_documented
handoff_required:
  summary: string
  artifact_paths:
    - path
  open_questions:
    - string
  recommended_owner: string
  recommended_next_step: string
```
