---
name: u06250-community-feedback-harvesting-for-entrepreneurship-operations
description: Build and operate the "Community Feedback Harvesting for entrepreneurship operations" capability for entrepreneurship operations. Use when this exact capability is required by autonomous or human-guided missions.
---
# Community Feedback Harvesting for entrepreneurship operations

## Metadata
- skill_id: `u06250-community-feedback-harvesting-for-entrepreneurship-operations`
- capability: `Community Feedback Harvesting for entrepreneurship operations`
- domain: `entrepreneurship operations`
- operating_mode: `production`
- delivery_contract: `deterministic, auditable, handoff-ready`

## Allowed Tools
Use only tools enabled by the active runtime policy.
Preferred local-first toolset:
- `read`, `write`, `edit`
- `exec`, `process`
- `web_search`, `web_fetch` (only when external verification is required)

## Inputs (formatted)
| Field | Type | Required | Format | Purpose |
|---|---|---:|---|---|
| `mission_request` | string | yes | concise objective + constraints | Defines the requested outcome. |
| `capability_trigger` | string | yes | exact capability name | Confirms this skill is the correct lane. |
| `source_signals` | array<object> | yes | source-tagged records | Provides normalized working inputs. |
| `evidence_refs` | array<string> | yes | IDs/URLs/file paths | Supports factual traceability. |
| `acceptance_criteria` | array<string> | yes | measurable checks | Defines pass/fail conditions. |
| `downstream_consumer` | string | yes | team/agent/system name | Identifies handoff target. |
| `risk_tier` | enum | yes | `low`/`medium`/`high` | Sets approval and routing strictness. |
| `approval_token` | string | conditional | signed token or ticket ID | Required when `risk_tier=high`. |

## Outputs (formatted)
| Output | Type | Format | Consumer | Required |
|---|---|---|---|---:|
| `primary_artifact_bundle` | object | versioned JSON/Markdown bundle | orchestrator | yes |
| `execution_scorecard` | object | gate-by-gate status + metrics | operator | yes |
| `handoff_packet` | object | machine-readable envelope | downstream skill/system | yes |
| `exceptions_log` | array<object> | structured error list | operator/audit | no |

## Guidelines
- Keep execution deterministic: same input should yield materially identical decisions.
- Preserve provenance on every claim, score, and recommendation.
- Prefer minimal viable output that passes gates over verbose narrative.
- Escalate quickly when inputs are incomplete, contradictory, or policy-sensitive.
- Optimize for day-to-day operability: clear status, clear blockers, clear next action.

## Musts
- [ ] Confirm the request explicitly matches `Community Feedback Harvesting for entrepreneurship operations` in `entrepreneurship operations`.
- [ ] Validate schema and required fields before scoring or transformation.
- [ ] Run policy/risk checks before publication or downstream routing.
- [ ] Fail closed on missing evidence, failed validation, or unmet approval.
- [ ] Include a complete handoff packet with owner, status, and next step.

## Targets (day/week/month operating cadence)
- **Day:** Triage incoming work, execute validated runs, and hand off only gate-passing outputs.
- **Week:** Review failures/retries, tune thresholds and rules, and close recurring quality gaps.
- **Month:** Re-baseline acceptance criteria, refresh playbooks, and archive audit-ready evidence.

## Common Actions
1. Intake and classify request against capability trigger.
2. Normalize inputs and attach evidence references.
3. Execute core transformation/scoring workflow.
4. Run validation gates (schema, determinism, policy-risk, approval).
5. Build output artifacts and issue handoff packet.
6. Record outcomes for weekly and monthly review.

## External Tool Calls Needed
None required by default.
Use external calls only when fresh outside evidence is necessary, and log each call in `evidence_refs`.

## Validation & Handoff
Validation sequence:
1. `schema-gate` — required fields present and well-typed.
2. `determinism-gate` — repeat run is stable within configured tolerance.
3. `policy-risk-gate` — legal/policy/risk checks pass.
4. `approval-gate` — high-risk runs include explicit human approval token.

Handoff minimum contract:
```json
{
  "skill_id": "u06250-community-feedback-harvesting-for-entrepreneurship-operations",
  "capability": "Community Feedback Harvesting for entrepreneurship operations",
  "domain": "entrepreneurship operations",
  "status": "pass|fail|blocked",
  "consumer": "<downstream_consumer>",
  "artifacts": ["primary_artifact_bundle", "execution_scorecard"],
  "next_action": "<clear owner action>",
  "evidence_refs": []
}
```

