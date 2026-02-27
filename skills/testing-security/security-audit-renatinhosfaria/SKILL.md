---
name: security-audit
description: Use when reviewing code for security vulnerabilities and OWASP top 10 issues
phases: [R, V]
---

# Security Audit - Vulnerability Assessment

## The Iron Law

**ASSUME ALL INPUT IS MALICIOUS.** Every system boundary is an attack surface. Every data flow from an untrusted source must be validated, sanitized, and escaped before use.

## Process

1. **Identify Attack Surface** -- Map all entry points (HTTP params, file uploads, DB reads, env vars, third-party APIs, WebSocket messages)
2. **Injection Audit** -- Check for SQL, command, XSS, NoSQL, and template injection (see references for full table)
3. **Auth & Session Audit** -- Verify password hashing, session invalidation, JWT validation, rate limiting
4. **Authorization Audit** -- Confirm AuthZ on every endpoint, tenant isolation, server-side role checks
5. **Data Exposure Audit** -- Check for hardcoded secrets, sensitive data in logs, missing encryption
6. **Dependency Audit** -- Run `pnpm audit` and check for outdated packages with known CVEs
7. **Categorize Findings** -- Rate each finding by severity (Critical/High/Medium/Low/Info)

## Checklist

- [ ] Attack surface mapped (all entry points identified)
- [ ] Injection audit completed (SQL, command, XSS, NoSQL, template)
- [ ] Authentication and session audit completed
- [ ] Authorization audit completed (every endpoint checked)
- [ ] Sensitive data exposure audit completed
- [ ] Dependency audit run (pnpm audit)
- [ ] Findings categorized by severity
- [ ] Report generated in structured format

## Red Flags

**STOP the audit and escalate immediately if you find:**
- Hardcoded credentials (API keys, passwords, database URIs) in source code
- Disabled authentication or authorization checks
- SQL queries built with string concatenation from user input
- Sensitive data (passwords, tokens, PII) written to logs
- CORS set to `*` in production configuration
- Default or weak credentials in production environment

## References

- [OWASP Checklist & Detailed Audit Steps](references/owasp-checklist.md)
- [Report Template](references/report-template.md)
