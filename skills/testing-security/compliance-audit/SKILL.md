---
name: Compliance Audit
description: Audit technical controls against compliance framework requirements
category: compliance
version: 1.0.0
triggers:
  - audit-preparation
  - control-review
  - compliance-check
globs: "**/compliance/**,**/security/**,**/audit/**"
---

# Compliance Audit Skill

Audit technical controls against compliance framework requirements.

## Trigger Conditions
- Control mapping changes or evidence staleness >30 days
- Quarterly compliance review cycle
- User invokes with "compliance audit" or "control assessment"

## Input Contract
- **Required:** Framework(s) to audit against (SOC 2, GDPR, FedRAMP, etc.)
- **Required:** Control mapping matrix
- **Optional:** Prior audit results, remediation status

## Output Contract
- Control status report (compliant, gap, partial)
- Evidence inventory with freshness scores
- POA&M for open findings
- Compliance posture score

## Tool Permissions
- **Read:** All configs, logs, access policies, encryption settings
- **Write:** Audit reports, evidence documentation
- **Search:** Control implementation evidence

## Execution Steps
1. Load control mapping matrix for target framework
2. For each control, search for implementation evidence
3. Verify evidence is current (not stale)
4. Classify each control: compliant, partial, gap
5. For gaps, create POA&M entries with owners and deadlines
6. Calculate overall compliance posture score
7. Generate audit report

## Success Criteria
- Every control classified with evidence reference
- No evidence older than 30 days for continuous controls
- POA&M entries have assigned owners and deadlines
- Compliance score calculated

## Escalation Rules
- Escalate if critical control gaps are found
- Escalate if evidence cannot be generated automatically
- Escalate if compliance score drops below threshold

## Example Invocations

**Input:** "Audit SOC 2 Type II controls for the payment service"

**Output:** 47 controls assessed. 39 compliant, 5 partial (access reviews overdue, MFA not enforced for service accounts, log retention below 1yr for 2 services, encryption key rotation not automated, backup restore not tested). 3 gaps (no formal change management for DB, missing vendor risk assessment, incident response not tested). POA&M created with 30/60/90 day deadlines.
