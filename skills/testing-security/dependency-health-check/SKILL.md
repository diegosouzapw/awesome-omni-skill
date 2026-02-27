---
name: Dependency Health Check
description: Assess dependency health across CVE exposure, freshness, maintainer risk, and license compatibility
category: security
version: 1.0.0
triggers:
  - weekly-cadence
  - new-dependency-added
  - security-audit
globs: "**/requirements*.txt,**/package.json,**/go.mod,**/pyproject.toml"
---

# Dependency Health Check Skill

Assess dependency health across CVE exposure, freshness, maintainer risk, and license compatibility.

## Trigger Conditions
- Weekly automated health check cadence
- New dependency added to the project
- User invokes with "dependency health" or "check dependencies"

## Input Contract
- **Required:** Dependency manifest (go.mod, package.json, requirements.txt)
- **Optional:** CVE database reference, license policy

## Output Contract
- Health report per dependency (CVE count, freshness, maintainer score, license)
- Risk-scored upgrade priority matrix
- SBOM (Software Bill of Materials)

## Tool Permissions
- **Read:** Lock files, dependency tree, CVE databases
- **Write:** Health reports, SBOM
- **Execute:** Dependency scanning tools

## Execution Steps
1. Parse dependency manifest and resolve full tree (direct + transitive)
2. Check each dependency against CVE databases
3. Score freshness (versions behind latest)
4. Assess maintainer health (bus factor, commit frequency, funding)
5. Check license compatibility with project distribution model
6. Generate SBOM
7. Produce risk-scored priority matrix

## Success Criteria
- Full dependency tree analyzed (direct + transitive)
- CVE status current (within 24 hours)
- License compatibility verified
- SBOM generated

## Escalation Rules
- Escalate Critical CVEs immediately
- Escalate if a direct dependency has bus factor of 1
- Escalate if GPL dependency found in proprietary project

## Example Invocations

**Input:** "Run dependency health check on our Go service"

**Output:** 142 dependencies analyzed (23 direct, 119 transitive). CVEs: 0 Critical, 2 High (golang.org/x/net, google.golang.org/grpc), 5 Medium. Freshness: 4 dependencies >2 major versions behind. License: all compatible (MIT, Apache-2.0, BSD). SBOM generated. Priority: upgrade x/net immediately (CVE-2025-1234, CVSS 8.1).
