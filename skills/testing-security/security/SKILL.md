---
name: security
description: Use this skill when designing or reviewing systems where security is a concern - authentication, authorization, data protection, input handling, or any system processing untrusted input. Applies adversarial thinking to specifications, designs, and implementations.
version: 0.1.0
---

# Security Engineering

## When to Apply

Use this skill when the system involves:
- Authentication or authorization
- Processing untrusted input (user input, external APIs, file uploads)
- Storing or transmitting sensitive data
- Cryptographic operations
- Multi-tenant isolation
- Compliance requirements (PCI, HIPAA, SOC2, GDPR)

## Mindset

Security engineers think like attackers and design for the adversarial case.

**Questions to always ask:**
- What's the trust boundary here? Who is trusted, who isn't?
- What happens if this input is malicious?
- What's the blast radius if this component is compromised?
- Who can access this? Who shouldn't be able to?
- What sensitive data flows through here? Where does it rest?
- How would an attacker abuse this feature?
- What gets logged? What shouldn't be logged?

**Assumptions to challenge:**
- "Users won't do that" - Attackers will. Design for malicious input.
- "It's internal only" - Internal networks get compromised. Defense in depth.
- "We'll add security later" - Retrofitting security is expensive and error-prone.
- "HTTPS is enough" - Transport security doesn't protect data at rest or in logs.
- "We hash passwords" - With what? Salted? Using bcrypt/argon2 or MD5?
- "Only admins can access this" - How is that enforced? Can it be bypassed?

## Practices

### Input Validation
Validate all input at trust boundaries. Use allowlists, not blocklists. Validate type, length, format, and range. Reject invalid input; don't sanitize it into validity. **Don't** trust client-side validation, use blocklists for security, or assume input is well-formed.

### Authentication
Use established protocols (OAuth2, OIDC). Implement rate limiting and account lockout. Use secure session management with proper expiration. **Don't** roll your own auth, store passwords in plaintext or weak hashes, or use predictable session tokens.

### Authorization
Check permissions on every request, not just UI. Use principle of least privilege. Centralize authorization logic. **Don't** rely on UI hiding for access control, check permissions only at the edge, or use role checks when resource checks are needed.

### Secrets Management
Never hardcode secrets. Use secret managers or environment injection. Rotate secrets regularly. **Don't** commit secrets to version control, log secrets, or pass secrets in URLs.

### Cryptography
Use standard libraries and algorithms. Never invent your own crypto. Use appropriate algorithms (argon2/bcrypt for passwords, AES-GCM for encryption). **Don't** use deprecated algorithms (MD5, SHA1, DES), ECB mode, or predictable IVs.

### Data Protection
Encrypt sensitive data at rest and in transit. Classify data by sensitivity. Minimize data collection and retention. **Don't** store more than needed, keep data longer than required, or expose sensitive data in logs/URLs/errors.

### Error Handling
Return generic errors to users. Log detailed errors internally with correlation IDs. Never expose stack traces, SQL errors, or internal paths. **Don't** leak information through error messages, timing differences, or response sizes.

### Audit Logging
Log security-relevant events (auth, access, changes). Include who, what, when, from where. Protect logs from tampering. **Don't** log sensitive data, skip logging security events, or allow log deletion without audit trail.

## Vocabulary

Use precise terminology:

| Instead of | Say |
|------------|-----|
| "secure" | "encrypted with AES-256-GCM" / "authenticated via OAuth2" |
| "hashed" | "bcrypt with cost 12" / "argon2id" |
| "validated" | "allowlist validated" / "schema validated" / "bounds checked" |
| "safe" | "parameterized query" / "escaped for HTML context" |
| "admin only" | "requires ADMIN role" / "enforced by policy X" |
| "internal" | "within trust boundary X" / "requires mTLS" |

## SDD Integration

**During Specification:**
- Identify trust boundaries and threat actors
- Classify data by sensitivity
- Specify compliance requirements
- Define authentication and authorization requirements explicitly

**During Design:**
- Document trust boundaries per component
- Specify input validation rules at each boundary
- Design authorization model (RBAC, ABAC, etc.)
- Plan for secrets management and key rotation
- Identify what gets logged and what must not be logged

**During Review:**
- Verify input validation at all trust boundaries
- Check authorization is enforced server-side, not just UI
- Confirm secrets aren't hardcoded or logged
- Validate error messages don't leak information
- Check for OWASP Top 10 vulnerabilities
