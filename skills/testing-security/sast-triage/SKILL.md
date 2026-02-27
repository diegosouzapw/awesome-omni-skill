---
name: SAST Triage
description: Triage static analysis findings from gosec, golangci-lint, and govulncheck — classify severity, filter false positives, and prioritize remediation
category: security
version: 1.0.0
triggers:
  - security-scan-results
  - ci-security-failure
  - dependency-update
globs: "**/*.go,**/go.mod"
---

# SAST Triage Skill

Systematically triage static application security testing (SAST) results to separate real vulnerabilities from false positives and prioritize remediation.

## Trigger Conditions
- CI security scan produces findings
- `gosec`, `golangci-lint`, or `govulncheck` run completes
- Dependency update introduces new vulnerabilities
- User invokes with "triage security findings" or "sast-triage"

## Input Contract
- **Required:** SAST tool output (gosec JSON, golangci-lint output, govulncheck results)
- **Optional:** Previous triage results for delta comparison

## Output Contract
- Classified findings: Critical/High/Medium/Low/FalsePositive
- CWE/CVE mapping for each finding
- Remediation priority with estimated effort
- False positive justifications

## Tool Permissions
- **Read:** All Go source files, SAST output, go.mod, go.sum
- **Write:** Triage report
- **Search:** Grep for vulnerable patterns, dependency versions
- **Shell:** Run `gosec`, `govulncheck`, `golangci-lint`

## Execution Steps

1. **Collect findings**: Run or parse SAST tool outputs
2. **Deduplicate**: Merge findings across tools that point to the same issue
4. **Filter false positives**: Identify findings that are false positives due to context (e.g., test files, disabled code)
5. **Map to CWE/CVE**: Link each finding to its CWE or CVE identifier
7. **Report**: Produce triage report with actions for each finding
