---
name: api-docs
description: Generate, validate, and maintain API documentation (REST/OpenAPI and GraphQL) with reproducible structure and evidence-backed confidence.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Task, TodoWrite
model: sonnet
x-version: 3.2.0
x-category: delivery
x-vcl-compliance: v3.1.1
x-cognitive-frames:
  - HON
  - MOR
  - COM
  - CLS
  - EVD
  - ASP
  - SPC
---


## STANDARD OPERATING PROCEDURE

### Purpose
Ship accurate, consumable API docs with validated schemas, runnable examples, and clear auth/error guidance.

### Trigger Conditions
- **Positive:** REST/GraphQL doc creation or updates, versioned API rollouts, deprecation notices, interactive explorer setup.
- **Negative:** Narrative/internal docs (route to `documentation`) or prompt-focused tasks (route to `prompt-architect`).

### Guardrails
- **Structure-first:** maintain `examples/`, `tests/`, `resources/`, `references/` alongside `SKILL.md`.
- **Constraint extraction:** capture HARD/SOFT/INFERRED constraints on audience, security posture, and versioning; confirm inferred.
- **Validation:** run spec validators (OpenAPI/GraphQL), exercise examples, and note unrun commands.
- **Confidence ceilings:** apply `{inference/report:0.70, research:0.85, observation/definition:0.95}` to all claims.
- **Safety:** no secret exposure; redact internal endpoints when producing public docs.

### Execution Phases
1. **Scope & Sources**
   - Identify surfaces (REST/GraphQL), auth schemes, target audience, and supported versions.
   - Gather schemas, code annotations, and existing references; log constraints.
2. **Outline & Contracts**
   - Define required sections: overview, auth, endpoints/operations, errors, rate limits, changelog.
   - Confirm INFERRED needs (SDK snippets, language coverage).
3. **Author & Validate**
   - Draft OpenAPI/GraphQL artifacts; generate runnable examples in `examples/`.
   - Validate specs (swagger-cli, graphql-schema-linter) and sample requests.
4. **Publish & Harden**
   - Wire interactive explorer (Swagger UI/GraphQL Playground) if in scope.
   - Add tests in `tests/` (lint, link check, example execution) and store outputs in `resources/`.
5. **Review & Deliver**
   - Summarize deltas, risks, and next steps; cite sources in `references/`.
   - Provide confidence statement with ceiling.

### Output Format
- Scope summary + constraints (HARD/SOFT/INFERRED, confirmed).
- Spec status and validation results.
- Example calls and changelog notes.
- Evidence with **Confidence: X.XX (ceiling: TYPE Y.YY)**.

### Validation Checklist
- [ ] Audience and version coverage confirmed; inferred asks resolved.
- [ ] Specs validated; examples executed or annotated as not run.
- [ ] Auth, errors, and rate limits documented.
- [ ] Tests present in `tests/`; artifacts stored in `resources/`; references recorded.
- [ ] Confidence ceilings applied to claims and guidance.

### MCP / Memory Tags
- Namespace: `skills/delivery/api-docs/{api}/{version}`
- Tags: `WHO=api-docs-{session}`, `WHY=skill-execution`, `WHAT=spec+docs`

Confidence: 0.70 (ceiling: inference 0.70) - SOP integrates skill-forge structure-first and prompt-architect constraint/ceiling rules.
