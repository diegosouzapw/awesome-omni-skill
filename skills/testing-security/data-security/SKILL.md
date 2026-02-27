---
name: data-security
description: "Assess data security controls: classification, access, encryption, retention, and exposure risk."
version: 1.0.0
category: review
tags: [security, data, encryption, access-control]
metadata:
  ai-engineering:
    scope: read-only
    token_estimate: 740
---

# Data Security Review

## Purpose

Evaluate confidentiality and integrity controls for data across storage, transit, and access paths.

## Trigger

- Handling sensitive data in new/changed features.
- Security assessments and release gates.

## Procedure

1. Classify data types and sensitivity levels.
2. Verify at-rest and in-transit encryption controls.
3. Evaluate authentication/authorization boundaries.
4. Check retention, deletion, and auditability requirements.
5. Report exposure risks and remediation priorities.

## Output Contract

- Data security findings with severity and remediation plan.

## Governance Notes

- High/critical findings are merge-blocking unless risk-accepted.

## References

- `agents/security-reviewer.md`
- `skills/review/security/SKILL.md`
