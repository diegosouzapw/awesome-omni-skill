---
name: security-audit
description: "Invoke when reviewing security-sensitive code: authentication, authorization, payment processing, data protection, or any code handling sensitive information."
---

# Security Audit Skill

Specialized review for red-light scenarios involving security-critical code.

## When to Use

Automatically invoke when code involves:

- Authentication or authorization
- Payment processing or billing
- Personal identifiable information (PII)
- Cryptography or encryption
- Session management
- API keys or secrets
- Data validation or sanitization
- File uploads or downloads
- Database queries with user input

## Red-Light Scenario Checklist

### Authentication

- [ ] **No hardcoded credentials** - Secrets in environment variables
- [ ] **Strong password hashing** - bcrypt, argon2 (not MD5, SHA1)
- [ ] **Secure token storage** - httpOnly cookies (not localStorage)
- [ ] **Session expiry** - Proper timeout and refresh
- [ ] **Rate limiting** - Prevent brute force attacks
- [ ] **HTTPS/TLS** - All auth traffic encrypted
- [ ] **No custom crypto** - Use established libraries
- [ ] **Secure password reset** - Token-based, time-limited

### Authorization

- [ ] **Server-side checks** - Never trust client-side validation
- [ ] **Principle of least privilege** - Minimal permissions
- [ ] **Permission checks on every request** - No caching of permissions
- [ ] **No exposed role structures** - Internal roles not leaked to client
- [ ] **Audit logging** - Authorization failures logged
- [ ] **No user ID trust** - Validate user identity on server

### Payment Processing

- [ ] **PCI compliance** - Using approved payment processor
- [ ] **No raw card storage** - Never store credit card numbers
- [ ] **No CVV storage** - CVV never persisted
- [ ] **Idempotency** - Duplicate payment prevention
- [ ] **Transaction logging** - All payments logged
- [ ] **Webhook verification** - Signature validation
- [ ] **SSL/TLS required** - Encrypted payment traffic
- [ ] **Client amount distrust** - Server validates amounts

### Data Protection

- [ ] **Encryption at rest** - Sensitive data encrypted in database
- [ ] **Encryption in transit** - HTTPS/TLS for all sensitive data
- [ ] **Input sanitization** - All user input validated and sanitized
- [ ] **Parameterized queries** - No SQL injection vulnerabilities
- [ ] **CSRF protection** - Token-based CSRF prevention
- [ ] **XSS prevention** - Output encoding, CSP headers
- [ ] **No secrets in logs** - Sensitive data not logged
- [ ] **No secrets in version control** - .env files gitignored

### Common Vulnerabilities

Check for these specific issues:

SQL Injection

```javascript
// BAD
db.query(`SELECT * FROM users WHERE id = ${userId}`)

// GOOD
db.query('SELECT * FROM users WHERE id = ?', [userId])
```

XSS

```javascript
// BAD
element.innerHTML = userInput

// GOOD
element.textContent = userInput
```

Hardcoded Secrets

```javascript
// BAD
const apiKey = "sk_live_abc123"

// GOOD
const apiKey = process.env.API_KEY
```

Insecure Token Storage

```javascript
// BAD
localStorage.setItem('token', jwt)

// GOOD
// Set httpOnly cookie on server
res.cookie('token', jwt, { httpOnly: true, secure: true })
```

## Threat Modeling

For each security-sensitive change, document:

### 1. Attack Surface

- What new endpoints or functionality is exposed?
- What user input is accepted?
- What external services are called?

### 2. Potential Threats

- What could an attacker do with this?
- What's the worst-case scenario?
- What data could be compromised?

### 3. Mitigations

- What protections are in place?
- What validation is performed?
- What monitoring/logging exists?

### 4. Blast Radius

- How many users affected if compromised?
- What data is at risk?
- What's the recovery plan?

## Output Format

```markdown
## Security Audit Results

### Critical Issues (FAIL - Must Fix)
- [Issue 1]: [description]
  - Location: [file:line]
  - Threat: [what could go wrong]
  - Fix: [how to address]

### Warnings (CONCERN - Should Fix)
- [Issue 1]: [description]
  - Location: [file:line]
  - Risk: [potential problem]
  - Recommendation: [suggested improvement]

### Passed Checks
- [Check 1]: ✓ Verified
- [Check 2]: ✓ Verified

### Threat Model
- Attack Surface: [description]
- Key Threats: [list]
- Mitigations: [what's in place]
- Blast Radius: [impact if compromised]

### Recommendations
1. [Action item 1]
2. [Action item 2]
```

## Integration with Workflow

This skill is invoked:

1. When security-related globs match (auth/**, payment/**)
2. During `/review` for security-sensitive code
3. Before committing changes to red-light zones
4. When preToolUse hook detects security scenario

## Required Before Approval

For any security-critical code:

- [ ] Threat model documented
- [ ] All critical issues resolved
- [ ] Security review completed
- [ ] Rollback plan exists
- [ ] Monitoring/logging in place
- [ ] No secrets in code or logs

## Common Security Anti-Patterns

Flag these immediately:

1. **OAuth.js + OAuth2.js pattern** - Multiple auth implementations create gaps
2. **Mixed auth and business logic** - Makes security audits impossible
3. **Client-side permission checks only** - Trivially bypassed
4. **Trusting user input** - Always validate and sanitize
5. **Preserving legacy auth patterns** - Old insecure code shouldn't be replicated

## Why This Matters

Security vulnerabilities can:

- Expose user data
- Enable unauthorized access
- Process fraudulent payments
- Damage reputation
- Create legal liability

Red-light scenarios require validated plans and thorough review before implementation.
