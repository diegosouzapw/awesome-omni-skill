---
name: security-review
description: Performs security reviews of Hone code using OWASP guidelines. Use when reviewing database queries, CSV import logic, API endpoints, authentication, encryption, or when the user asks about security.
allowed-tools: Read, Grep, Glob
---

# Security Review for Hone

Based on OWASP Top 10 (2021) and modern web security practices.

## OWASP Top 10 Relevance to Hone

### A01:2021 - Broken Access Control
- **Risk**: Unauthorized access to other users' financial data
- **Hone Context**: Currently single-user, but if multi-user is added:
  - Implement proper session management
  - Validate user owns requested resources
  - Use principle of least privilege

### A02:2021 - Cryptographic Failures
- **Data Classification**: Transaction data is sensitive PII
- **At Rest**: SQLCipher for database encryption (per DESIGN.md)
- **In Transit**:
  - All external connections MUST use TLS/HTTPS
  - Ollama connection should use HTTPS if remote
- **Hashing**: Using SHA-256 for deduplication (appropriate choice)

**Database Encryption (Required)**:
```rust
// SQLCipher - open encrypted database
let conn = Connection::open("hone.db")?;
conn.pragma_update(None, "key", &passphrase)?;

// Key derivation - use Argon2 to derive key from user passphrase
use argon2::{Argon2, PasswordHasher};
let salt = SaltString::generate(&mut OsRng);
let argon2 = Argon2::default();
let key = argon2.hash_password(passphrase.as_bytes(), &salt)?;
```

**Implementation Requirements**:
- Use `rusqlite` with `bundled-sqlcipher` feature
- Derive encryption key from passphrase using Argon2
- Passphrase provided at startup (env var or prompt)
- Never log or expose the passphrase

**Backup Encryption**:
- Backups encrypted with `age` before upload to Cloudflare R2
- Consider post-quantum algorithms for future-proofing (harvest now, decrypt later threat)

### A03:2021 - Injection
- **SQL Injection** (Critical for hone-core/src/db.rs)
  - All queries MUST use parameterized statements
  - Never interpolate user input into SQL

```rust
// SECURE - parameterized query
conn.execute(
    "INSERT INTO transactions (account_id, amount) VALUES (?, ?)",
    params![account_id, amount]
)?;

// VULNERABLE - string interpolation
conn.execute(
    &format!("SELECT * FROM transactions WHERE merchant = '{}'", merchant),
    []
)?;
```

- **CSV Injection** (hone-core/src/import.rs)
  - Sanitize fields starting with `=`, `+`, `-`, `@` (Excel formula injection)
  - Validate numeric fields are actually numeric

### A04:2021 - Insecure Design
- **Threat Modeling**: Financial data attracts attackers
- **Defense in Depth**: Multiple layers of validation
- **Secure Defaults**: Restrictive CORS, minimal permissions

### A05:2021 - Security Misconfiguration
- **CORS**: Restrict to specific origins in production

```rust
// Development (permissive)
CorsLayer::permissive()

// Production (restrictive)
CorsLayer::new()
    .allow_origin("https://your-domain.com".parse::<HeaderValue>().unwrap())
    .allow_methods([Method::GET, Method::POST])
```

- **Error Messages**: Never expose stack traces or internal paths
- **Headers**: Set security headers (via tower-http)
  - `X-Content-Type-Options: nosniff`
  - `X-Frame-Options: DENY`
  - `Content-Security-Policy`

### A06:2021 - Vulnerable Components
- Run `cargo audit` regularly (already set up)
- Keep dependencies updated
- Review transitive dependencies

### A07:2021 - Authentication Failures
- Currently N/A (single-user, local)
- If adding auth:
  - Use established libraries (not custom)
  - Implement rate limiting
  - Secure session management

### A08:2021 - Data Integrity Failures
- **CSV Import**: Validate file integrity
  - Check file size limits
  - Validate expected columns exist
  - Reject malformed data gracefully

### A09:2021 - Security Logging
- Log security-relevant events:
  - Failed authentication attempts (if added)
  - Access to sensitive endpoints
  - Import operations
- Don't log sensitive data (transaction details, amounts)

### A10:2021 - SSRF
- **Ollama Integration**: Validate URL is localhost/trusted
- Don't allow user-controlled URLs for HTTP requests

## Frontend Security (React/TypeScript)

### XSS Prevention
- React escapes by default (good)
- Never use `dangerouslySetInnerHTML` with user data
- Sanitize data before rendering if from external source

### Sensitive Data
- Don't store financial data in localStorage/sessionStorage
- Clear sensitive state on logout
- Use httpOnly cookies for auth tokens (if added)

### Content Security Policy
```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'">
```

## Secrets Management

### Never Commit
- API keys, tokens, passwords
- Database credentials
- Private keys

### Secure Storage
- Use `.env` files (gitignored)
- Environment variables at runtime
- Consider `dotenv` crate for Rust

### Detection Patterns
```
# Patterns that indicate hardcoded secrets
api_key\s*[:=]
password\s*[:=]
secret\s*[:=]
token\s*[:=]
-----BEGIN.*PRIVATE KEY-----
```

## Security Review Checklist

### Database Layer (db.rs)
- [ ] SQLCipher encryption enabled (bundled-sqlcipher feature)
- [ ] Key derived via Argon2 from passphrase
- [ ] All queries use parameterized statements (`params![]`)
- [ ] No string interpolation in SQL
- [ ] Input validation before queries
- [ ] Errors don't expose SQL details

### Import Layer (import.rs)
- [ ] File size limits enforced
- [ ] Path traversal prevented
- [ ] CSV fields sanitized
- [ ] Malformed input handled gracefully

### API Layer (hone-server)
- [ ] Request validation on all endpoints
- [ ] Generic error responses
- [ ] CORS configured appropriately
- [ ] Security headers set

### Frontend (ui/)
- [ ] No sensitive data in localStorage
- [ ] XSS vectors checked
- [ ] CSP headers configured
- [ ] API errors handled gracefully

## Resources

- [OWASP Top 10](https://owasp.org/Top10/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [Rust Security Guidelines](https://rustsec.org/)
- [React Security Best Practices](https://snyk.io/blog/10-react-security-best-practices/)
