---
name: API Versioning
description: Design and manage API versioning strategies for backward-compatible evolution
category: api
version: 1.0.0
triggers:
  - api-breaking-change
  - deprecation-needed
  - version-bump
globs: "**/api/**,**/openapi/**,**/routes/**"
---

# API Versioning Skill

Design and manage API versioning strategies for backward-compatible evolution.

## Trigger Conditions
- Breaking API change is proposed
- New API endpoint is created
- User invokes with "version API" or "API deprecation plan"

## Input Contract
- **Required:** Current API spec and proposed change
- **Required:** Consumer list (who calls this endpoint)
- **Optional:** Sunset timeline, migration support level

## Output Contract
- Versioning recommendation (strategy + implementation)
- Breaking change analysis with consumer impact
- Migration guide for consumers
- Deprecation timeline with Sunset headers

## Tool Permissions
- **Read:** API specs, route definitions, consumer registry
- **Write:** API specs, migration guides, deprecation notices
- **Search:** Codebase for endpoint usage

## Execution Steps
1. Analyze proposed change for breaking vs. non-breaking
2. If breaking: determine versioning strategy (URL path, header, content negotiation)
3. Assess consumer impact using consumer registry
4. Draft migration guide for affected consumers
5. Configure Sunset and Warning headers on deprecated endpoints
6. Set deprecation timeline (minimum 3 months)
7. Create monitoring for deprecated endpoint usage

## Success Criteria
- Breaking changes identified and versioned correctly
- All affected consumers notified with migration guide
- Sunset headers configured on deprecated endpoints
- Monitoring in place to track deprecated endpoint usage

## Escalation Rules
- Escalate if >10 consumers are affected by breaking change
- Escalate if deprecation timeline conflicts with consumer contracts
- Escalate if no versioning strategy exists yet

## Example Invocations

**Input:** "We need to change the /users response from {name: string} to {first_name: string, last_name: string}"

**Output:** Breaking change: field removal (name) + field addition (first_name, last_name). Strategy: v2 endpoint at /api/v2/users. v1 maintained with computed 'name' field for 6 months. Migration guide: update client SDK, map first_name+last_name. Sunset header on v1: 2026-08-01. Monitor v1 usage weekly.
