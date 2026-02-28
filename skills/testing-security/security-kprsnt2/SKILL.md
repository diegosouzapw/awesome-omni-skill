---
name: security
description: Application security best practices including OWASP Top 10, authentication, and data protection.
globs: ["**/*.js", "**/*.ts", "**/*.py", "**/*.java", "**/*.go"]
alwaysApply: ["**/SECURITY.md"]
priority: 100
tags: ["security"]
---

# Application Security Best Practices

## Input Validation
- Validate all user input
- Use allowlisting over blocklisting
- Sanitize HTML to prevent XSS
- Use parameterized queries for SQL
- Validate file uploads (type, size)

## Authentication
- Use strong password hashing (bcrypt, argon2)
- Implement MFA where possible
- Use secure session management
- Implement rate limiting on auth endpoints
- Use JWTs properly (short expiry, refresh tokens)

## Authorization
- Implement least privilege
- Check authorization on every request
- Use role-based access control
- Implement row-level security
- Audit access to sensitive data

## Data Protection
- Encrypt sensitive data at rest
- Use TLS for data in transit
- Don't log sensitive data
- Implement proper key management
- Use secure cookie flags

## Common Vulnerabilities (OWASP Top 10)
- Injection: Use prepared statements
- Broken Auth: Secure session handling
- XSS: Sanitize output, use CSP
- CSRF: Use anti-CSRF tokens
- Security Misconfiguration: Review defaults
- Sensitive Data Exposure: Encrypt everything
- Broken Access Control: Check on server
- SSRF: Validate URLs, use allowlists

## Dependencies
- Keep dependencies updated
- Use npm audit/pip audit
- Pin dependency versions
- Use lockfiles
- Monitor for vulnerabilities
