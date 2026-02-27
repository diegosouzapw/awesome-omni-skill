---
name: Threat Model
description: Conduct STRIDE-based threat modeling for system components and data flows
category: security
version: 1.0.0
triggers:
  - ingress-point-added
  - auth-changes
  - security-assessment
globs: "**/api/**,**/auth/**,**/security/**"
---

# Threat Model Skill

Conduct STRIDE-based threat modeling for system components and data flows.

## Trigger Conditions
- New ingress points are added (APIs, webhooks, queue consumers)
- Authentication or authorization changes are made
- User invokes with "threat model" or "security assessment"

## Input Contract
- **Required:** System or component to model
- **Required:** Data flow diagram or description
- **Optional:** Trust boundaries, existing security controls

## Output Contract
- STRIDE analysis per component
- Risk matrix (likelihood × impact)
- Prioritized mitigation recommendations
- Attack tree for highest-risk threats

## Tool Permissions
- **Read:** All source code, config files, API specs, network policies
- **Write:** `docs/security/` directory
- **Search:** Codebase for security-relevant patterns

## Execution Steps
1. Map the system's data flow diagram
2. Identify trust boundaries
3. Apply STRIDE to each component crossing a trust boundary
4. Assess likelihood and impact for each threat
5. Prioritize threats by risk score
6. Recommend mitigations for top threats
7. Document in threat model report

## Success Criteria
- Every ingress point has been analyzed
- STRIDE applied to all trust boundary crossings
- Risk matrix populated with scores
- Top 5 threats have concrete mitigation recommendations

## Escalation Rules
- Escalate if a Critical-severity threat is identified
- Escalate if existing controls are insufficient for a High-risk threat
- Escalate if the threat requires infrastructure-level changes

## Example Invocations

**Input:** "Threat model the new webhook ingestion endpoint"

**Output:** STRIDE analysis: Spoofing (no HMAC verification — HIGH), Tampering (no request body signing — MEDIUM), Repudiation (no audit log — MEDIUM), Information Disclosure (error messages leak internals — LOW), DoS (no rate limiting — HIGH), EoP (webhook can trigger admin actions — CRITICAL). Mitigation plan prioritized.
