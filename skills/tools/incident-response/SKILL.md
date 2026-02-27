---
name: Incident Response
description: Guide structured incident response following severity-based protocols
category: operations
version: 1.0.0
triggers:
  - incident-detected
  - alert-fires
  - severity-s1-s2
globs: "**/runbooks/**,**/incidents/**"
---

# Incident Response Skill

Guide structured incident response following severity-based protocols.

## Trigger Conditions
- An incident is detected or reported
- An alert fires with severity S1 or S2
- User invokes with "incident response" or "start incident"

## Input Contract
- **Required:** Incident description or alert payload
- **Required:** Severity level (S1-S4)
- **Optional:** Affected services, customer impact estimate

## Output Contract
- Incident timeline with structured updates
- Mitigation steps and status
- Communication templates for stakeholders
- Evidence preservation checklist

## Tool Permissions
- **Read:** Logs, metrics, alerts, runbooks, deployment history
- **Write:** Incident reports, status page updates
- **Execute:** Diagnostic commands, rollback procedures

## Execution Steps
1. Classify severity using defined criteria
2. Identify incident commander and assign roles
3. Establish communication channels (war room)
4. Preserve evidence (logs, metrics, config state, recent deploys)
5. Focus on mitigation first, root cause second
6. Provide structured updates at cadence defined by severity
7. Assess regulatory notification requirements
8. Document resolution and schedule postmortem

## Success Criteria
- Severity correctly classified within 5 minutes
- Communication cadence maintained per severity SLA
- Service restored within MTTR target
- Evidence preserved before mitigation changes state

## Escalation Rules
- Auto-escalate if MTTR exceeds threshold per severity
- Escalate if regulatory notification may be required
- Escalate if customer impact exceeds threshold

## Example Invocations

**Input:** "Payment processing is returning 500 errors for 30% of requests"

**Output:** S1 incident declared. Mitigation: rollback last deploy (payment-service v2.3.1 → v2.3.0). Comms: internal Slack update every 15min, status page updated, customer notification drafted. Evidence: deploy log, error rate graph, and config diff preserved.
