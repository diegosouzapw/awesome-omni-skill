---
name: security-review-audit
description: Full codebase security audit with OWASP Top 10 guidance, language-specific patterns, checklists, and fix examples. Use for comprehensive audits split by module/area.
license: MIT
parallel_friendly: true
---

# Security Code Review Guide

## Overview

Perform thorough security reviews of code to identify vulnerabilities, misconfigurations, and security anti-patterns. This skill helps you think like an attacker while providing actionable fixes.

---

# Process

## Phase 1: Reconnaissance

Before diving into code, understand the attack surface:

### 1.1 Identify Entry Points
- HTTP endpoints (routes, controllers, handlers)
- API endpoints (REST, GraphQL, gRPC)
- WebSocket handlers
- File upload handlers
- Authentication endpoints
- Admin/privileged endpoints

### 1.2 Identify Data Flows
- User input sources (forms, query params, headers, cookies)
- Database queries and ORM usage
- External API calls
- File system operations
- Command execution
- Serialization/deserialization

### 1.3 Identify Trust Boundaries
- Authentication checks
- Authorization/permission checks
- Input validation layers
- Output encoding layers

---

## Phase 2: Vulnerability Hunting

Systematically check for each vulnerability class:

### 2.1 Injection Vulnerabilities

**SQL Injection**
- Look for string concatenation in queries
- Check ORM usage for raw queries
- Verify parameterized queries are used
- Check stored procedures for dynamic SQL

**Command Injection**
- Find all `exec`, `system`, `popen`, `subprocess` calls
- Check for user input in command arguments
- Verify proper escaping or allowlisting

**XSS (Cross-Site Scripting)**
- Find all places user input is rendered in HTML
- Check for proper output encoding
- Look for `innerHTML`, `dangerouslySetInnerHTML`, `v-html`
- Check CSP headers

**Template Injection**
- Find template rendering with user input
- Check for SSTI in Jinja2, Twig, ERB, etc.

### 2.2 Authentication & Session

**Authentication Flaws**
- Password hashing (bcrypt/argon2 vs MD5/SHA1)
- Timing-safe comparison for secrets
- Account enumeration via error messages
- Brute force protection
- Password reset flow security

**Session Management**
- Session token entropy
- Secure cookie flags (HttpOnly, Secure, SameSite)
- Session fixation protection
- Session timeout/invalidation

### 2.3 Authorization

**Broken Access Control**
- IDOR (Insecure Direct Object References)
- Missing function-level access control
- Privilege escalation paths
- JWT validation issues

### 2.4 Cryptography

**Crypto Weaknesses**
- Hardcoded secrets/keys
- Weak algorithms (MD5, SHA1, DES, RC4)
- ECB mode usage
- Missing or weak random number generation
- Certificate validation disabled

### 2.5 Data Exposure

**Sensitive Data**
- Secrets in logs
- PII in error messages
- Sensitive data in URLs
- Missing encryption at rest
- Verbose error messages in production

---

## Phase 3: Reporting

For each finding, document:

1. **Vulnerability Type**: CWE ID and name
2. **Severity**: Critical/High/Medium/Low
3. **Location**: File, line number, function
4. **Description**: What the vulnerability is
5. **Impact**: What an attacker could do
6. **Proof of Concept**: How to exploit (if safe)
7. **Remediation**: Specific fix with code example

---

## Phase 4: Fix Verification

After fixes are applied:
- Verify the fix addresses the root cause
- Check for regression in related code
- Ensure fix doesn't introduce new issues
- Add tests to prevent regression

---

# Reference Files

Load these as needed during review:

- [OWASP Top 10](./reference/owasp_top_10.md) - Most critical web vulnerabilities
- [Language Patterns](./reference/language_patterns.md) - Language-specific vulnerability patterns
- [Secure Coding Checklist](./reference/checklist.md) - Quick reference checklist
- [Common Fixes](./reference/common_fixes.md) - Code examples for common fixes

---

# Quick Reference: OWASP Top 10 (2021)

| # | Vulnerability | What to Look For |
|---|--------------|------------------|
| A01 | Broken Access Control | Missing auth checks, IDOR, privilege escalation |
| A02 | Cryptographic Failures | Weak hashing, hardcoded secrets, missing encryption |
| A03 | Injection | SQL, command, XSS, template injection |
| A04 | Insecure Design | Missing threat modeling, insecure patterns |
| A05 | Security Misconfiguration | Default creds, verbose errors, missing headers |
| A06 | Vulnerable Components | Outdated dependencies with known CVEs |
| A07 | Auth Failures | Weak passwords, missing MFA, session issues |
| A08 | Data Integrity Failures | Insecure deserialization, missing integrity checks |
| A09 | Logging Failures | Missing audit logs, sensitive data in logs |
| A10 | SSRF | Unvalidated URLs, internal network access |
