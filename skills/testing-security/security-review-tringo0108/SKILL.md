---
name: security-review
description: Security vulnerability detection based on OWASP Top 10. Use when reviewing code for security issues.
license: MIT
---

# Security Review

Proactive security review based on OWASP Top 10 and security best practices.

## OWASP Top 10 Checklist

### A01: Broken Access Control

- [ ] Verify authorization checks on all endpoints
- [ ] Check for IDOR vulnerabilities
- [ ] Ensure principle of least privilege

### A02: Cryptographic Failures

- [ ] Sensitive data encrypted in transit (HTTPS)
- [ ] Sensitive data encrypted at rest
- [ ] Strong encryption algorithms used

### A03: Injection

- [ ] SQL queries use parameterized statements
- [ ] User input is validated and sanitized
- [ ] Command injection prevented

### A04: Insecure Design

- [ ] Security requirements defined
- [ ] Threat modeling performed
- [ ] Secure design patterns used

### A05: Security Misconfiguration

- [ ] Default credentials changed
- [ ] Unnecessary features disabled
- [ ] Error messages don't leak info

### A06: Vulnerable Components

- [ ] Dependencies are up to date
- [ ] Known vulnerabilities addressed
- [ ] Components from trusted sources

### A07: Authentication Failures

- [ ] Strong password policies
- [ ] Multi-factor authentication available
- [ ] Session management secure

### A08: Data Integrity Failures

- [ ] Input validation on all data
- [ ] Integrity checks on critical data
- [ ] Signed updates and migrations

### A09: Logging Failures

- [ ] Security events logged
- [ ] Logs don't contain sensitive data
- [ ] Log injection prevented

### A10: SSRF

- [ ] URL validation on server requests
- [ ] Allowlist for external resources
- [ ] Network segmentation in place

## Severity Levels

- **Critical**: Immediate exploitation possible
- **High**: Significant risk, fix soon
- **Medium**: Should be addressed
- **Low**: Minor issue, fix when convenient


## Parent Hub
- [_security-compliance-mastery](../_security-compliance-mastery/SKILL.md)


## Part of Workflow
This skill is utilized in the following sequential workflows:
- [_workflow-feature-lifecycle](../_workflow-feature-lifecycle/SKILL.md)
- [_workflow-security-audit](../_workflow-security-audit/SKILL.md)
