---
name: security-review
description: Provides security review guidance including OWASP Top 10, threat modeling, secure coding practices, and security checklists. Triggers on security, threat model, vulnerability, OWASP, authentication, authorization, encryption, secrets, security review, penetration testing.
version: 1.0.0
user-invocable: false
---

# Security Review

## OWASP Top 10 Checklist

Review every application against the current OWASP Top 10. For each category, apply the specific mitigation:

- **A01 Broken Access Control:** Enforce access control on the server side, never rely on client-side checks. Deny by default. Validate that the authenticated user has permission to access the requested resource on every request. Implement CORS restrictively. Disable directory listing. Log and alert on repeated access control failures.
- **A02 Cryptographic Failures:** Classify data by sensitivity. Encrypt sensitive data at rest (AES-256) and in transit (TLS 1.2+). Never use MD5 or SHA-1 for passwords. Use bcrypt, argon2, or scrypt for password hashing. Do not store sensitive data in URLs, logs, or error messages. Rotate encryption keys on a defined schedule.
- **A03 Injection:** Use parameterized queries or prepared statements for all database access. Apply input validation with allowlists, not blocklists. Use ORM methods instead of raw SQL where possible. Escape output contextually (HTML, URL, JavaScript, CSS). Apply Content-Security-Policy headers to mitigate XSS.
- **A04 Insecure Design:** Conduct threat modeling during design phase, not after implementation. Define security requirements alongside functional requirements. Use secure design patterns (e.g., separate privilege levels, fail securely). Create abuse cases alongside use cases.
- **A05 Security Misconfiguration:** Remove default credentials and unused features. Apply hardening guides for all frameworks and servers. Disable detailed error messages in production. Review cloud storage permissions (S3 buckets, GCS). Automate configuration verification in CI.
- **A06 Vulnerable and Outdated Components:** Maintain an inventory of all dependencies and their versions. Run `npm audit`, `pip safety check`, `cargo audit`, or equivalent in CI. Subscribe to security advisories for critical dependencies. Remove unused dependencies. Pin dependency versions and review updates before merging.
- **A07 Identification and Authentication Failures:** Enforce strong password policies (minimum 12 characters, no common passwords). Implement MFA. Use constant-time comparison for credentials. Rate-limit login attempts. Do not expose whether an account exists in error messages.
- **A08 Software and Data Integrity Failures:** Verify the integrity of downloaded dependencies (checksums, lock files). Use signed commits for releases. Validate CI/CD pipeline configuration against tampering. Do not deserialize untrusted data without validation. Use Subresource Integrity (SRI) for CDN assets.
- **A09 Security Logging and Monitoring Failures:** Log authentication events (success and failure), access control failures, input validation failures, and server-side errors. Include timestamp, user ID, IP, and action in every log entry. Do not log sensitive data (passwords, tokens, PII). Send alerts for anomalous patterns. Retain logs for at least 90 days.
- **A10 Server-Side Request Forgery (SSRF):** Validate and sanitize all client-supplied URLs. Use allowlists for permitted domains and IP ranges. Block requests to internal IP ranges (127.0.0.0/8, 10.0.0.0/8, 169.254.169.254). Disable HTTP redirects in server-side requests or validate each redirect target.

## Threat Modeling (STRIDE)

Apply STRIDE analysis during the design phase of every new feature or service:

| Threat | Description | Mitigation |
|--------|-------------|------------|
| **Spoofing** | An attacker pretends to be another user or system | Strong authentication (MFA, certificate-based auth), token validation, mutual TLS for service-to-service |
| **Tampering** | An attacker modifies data in transit or at rest | TLS for transit, integrity checks (HMAC, checksums), database constraints, audit logs for mutations |
| **Repudiation** | An attacker denies performing an action | Immutable audit logs, signed transactions, timestamp verification, non-repudiation through digital signatures |
| **Information Disclosure** | Sensitive data is exposed to unauthorized parties | Encryption at rest and in transit, minimize data exposure in APIs, mask PII in logs, enforce least-privilege access |
| **Denial of Service** | An attacker makes the system unavailable | Rate limiting, input size limits, timeout enforcement, auto-scaling, CDN/WAF for public endpoints |
| **Elevation of Privilege** | An attacker gains higher access than authorized | Principle of least privilege, role-based access control, validate permissions server-side, segregate admin interfaces |

Process:

- Draw a data flow diagram for the feature.
- Identify trust boundaries (user to server, server to database, service to service).
- Apply each STRIDE category to every trust boundary crossing.
- Document identified threats and their mitigations.
- Review the threat model before implementation begins.

## Secure Coding Practices

Apply these rules to every code change:

- **Input validation:** Validate all input on the server side. Use allowlists (accepted characters, ranges, formats) rather than blocklists. Validate type, length, range, and format. Reject invalid input with a generic error; do not echo the invalid input back.
- **Parameterized queries:** Never concatenate user input into SQL, LDAP, or OS commands. Use parameterized queries, prepared statements, or ORM query builders. Example:
  ```sql
  -- WRONG: 'SELECT * FROM users WHERE id = ' + userId
  -- RIGHT: 'SELECT * FROM users WHERE id = $1', [userId]
  ```
- **Password hashing:** Use bcrypt (cost factor 12+) or argon2id. Never use plain hashing algorithms (SHA-256, MD5) for passwords. Never store passwords in plaintext or reversible encryption.
- **Output encoding:** Encode output based on context: HTML entity encoding for HTML body, URL encoding for URL parameters, JavaScript string encoding for inline scripts. Use framework-provided auto-escaping (React JSX, Jinja2 autoescape, Go html/template).
- **CSRF protection:** Include CSRF tokens in all state-changing requests (POST, PUT, DELETE). Use the SameSite cookie attribute (Lax or Strict). Verify the Origin and Referer headers on the server.
- **Error handling:** Never expose stack traces, SQL errors, or internal paths in production error responses. Return generic error messages to clients. Log detailed errors server-side with correlation IDs.

## Authentication Checklist

- Support multi-factor authentication (TOTP, WebAuthn, or SMS as fallback). Enforce MFA for admin accounts.
- Set session timeouts: idle timeout (15-30 minutes for sensitive apps), absolute timeout (8-12 hours).
- Rotate session tokens after login, privilege escalation, and password change. Invalidate old tokens immediately.
- Enforce password policy: minimum 12 characters, check against breached password databases (Have I Been Pwned API), no maximum length below 128 characters.
- Implement account lockout or progressive delays after 5 failed login attempts. Unlock after a time period or manual admin action.
- Use secure cookie attributes: `HttpOnly`, `Secure`, `SameSite=Lax` (or `Strict`), set appropriate `Domain` and `Path`.
- For token-based auth (JWT): use short-lived access tokens (15 minutes), long-lived refresh tokens (7 days) with rotation, and store refresh tokens server-side or in HttpOnly cookies.
- Implement logout that invalidates the session/token server-side, not just client-side.

## Authorization Patterns

- **RBAC (Role-Based Access Control):** Assign permissions to roles, assign roles to users. Use for applications with well-defined user types (admin, editor, viewer). Keep the number of roles manageable (under 10 for most applications).
- **ABAC (Attribute-Based Access Control):** Evaluate policies based on user attributes, resource attributes, and environment conditions. Use for fine-grained access control where RBAC becomes too many roles. Example: "users can edit documents they created in their own department."
- **Principle of least privilege:** Grant the minimum permissions needed for each role and service account. Start with no access and add permissions as needed, not the reverse.
- **Check permissions at every layer:** Validate authorization in the API gateway/middleware, in the service layer, and in database queries (e.g., always include `WHERE tenant_id = ?`). Never rely on a single authorization check point.
- **Resource-level authorization:** Always verify the requesting user has access to the specific resource, not just the resource type. Checking "user can read documents" is not enough; check "user can read this specific document."

## Secret Management

- **Never hardcode secrets** in source code, configuration files, or environment-specific branches. No API keys, passwords, tokens, or certificates in the repository.
- **Use environment variables** for local development. Use a secrets manager (AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager, Doppler) for staging and production.
- **Rotation policy:** Rotate all secrets on a schedule: API keys every 90 days, database passwords every 90 days, signing keys every 180 days. Rotate immediately if a secret is potentially compromised.
- **`.gitignore` patterns:** Ensure these are in `.gitignore`:
  ```
  .env
  .env.*
  *.pem
  *.key
  credentials.json
  service-account.json
  *secret*
  ```
- **Audit access:** Log who accesses secrets and when. Restrict access to production secrets to the minimum number of people and service accounts.
- **Pre-commit hooks:** Install `git-secrets`, `detect-secrets`, or `trufflehog` as a pre-commit hook to prevent accidental secret commits.

## HTTP Security Headers

Set these headers on every HTTP response from your application:

- **Content-Security-Policy (CSP):** `default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; connect-src 'self' https://api.example.com`. Customize per application but start restrictive.
- **Strict-Transport-Security (HSTS):** `max-age=31536000; includeSubDomains; preload`. Enforce HTTPS for all requests. Submit to the HSTS preload list for public applications.
- **X-Frame-Options:** `DENY` or `SAMEORIGIN`. Prevents clickjacking by controlling iframe embedding.
- **X-Content-Type-Options:** `nosniff`. Prevents browsers from MIME-type sniffing responses.
- **Referrer-Policy:** `strict-origin-when-cross-origin` or `no-referrer`. Controls how much referrer information is sent with requests.
- **Permissions-Policy:** Disable unused browser features: `camera=(), microphone=(), geolocation=(), payment=()`.
- **X-XSS-Protection:** `0` (disable the legacy XSS auditor; rely on CSP instead). Setting to `1` can introduce vulnerabilities in older browsers.

Verify headers using [securityheaders.com](https://securityheaders.com) or equivalent scanning tools in CI.

## Dependency Audit

Run dependency audits as part of every CI pipeline:

- **JavaScript/TypeScript:** `npm audit --audit-level=high` or `pnpm audit`. Use `npm audit fix` for automatic patches. Review and merge Dependabot or Renovate PRs within 48 hours for high/critical vulnerabilities.
- **Python:** `pip-audit` or `safety check`. Pin dependencies in `requirements.txt` or `pyproject.toml` with hash verification.
- **Go:** `govulncheck ./...` for known vulnerability scanning. Use `go mod tidy` to remove unused dependencies.
- **Rust:** `cargo audit` checks against the RustSec advisory database. Run as a CI step.
- **Java/Kotlin:** OWASP Dependency-Check plugin for Maven/Gradle. Integrate with build pipeline to fail on CVSS 7+ vulnerabilities.
- **Cross-platform:** Snyk, Trivy, or Grype for container image scanning and multi-language dependency analysis.

Rules:

- Block merges if critical or high vulnerabilities are found in direct dependencies.
- Review medium vulnerabilities within one sprint. Low vulnerabilities in the next scheduled maintenance.
- Remove unused dependencies aggressively. Every dependency is an attack surface.

## Anti-Patterns

- **Hardcoded secrets:** API keys, passwords, or tokens in source code, configuration files, or commit history. Use secret scanning tools to detect and rotate immediately.
- **Client-side-only authentication:** Checking login status or permissions only in the frontend (JavaScript, mobile app). All authorization must be enforced server-side.
- **Security by obscurity:** Relying on hidden URLs, undocumented endpoints, or non-standard ports as a security measure. All endpoints must be authenticated and authorized regardless of discoverability.
- **Disabled CORS for convenience:** Setting `Access-Control-Allow-Origin: *` to avoid CORS issues during development and leaving it in production. Configure CORS with explicit allowed origins.
- **Storing passwords in plaintext or reversible encryption:** Always use one-way hashing with bcrypt or argon2. Reversible encryption is not a substitute for hashing.
- **Trusting client input:** Using client-provided values (user ID, role, price) without server-side validation. Always treat client input as untrusted, even from authenticated users.
- **Logging sensitive data:** Writing passwords, tokens, credit card numbers, or PII to log files. Sanitize all log output.
- **Using outdated cryptographic algorithms:** MD5, SHA-1, DES, RC4, or RSA with key size under 2048 bits. Use current recommended algorithms (AES-256, SHA-256+, RSA-2048+, Ed25519).
- **No rate limiting:** Leaving authentication endpoints, API endpoints, or form submissions without rate limits. Every public endpoint must have rate limiting.
