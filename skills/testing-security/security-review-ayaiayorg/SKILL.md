---
name: security-review
description: Comprehensive security analysis and vulnerability assessment. Use this when reviewing code for security issues, analyzing potential vulnerabilities, or conducting security audits.
license: MIT
---

# Security Review Skill

This skill provides a systematic approach to security analysis, vulnerability assessment, and secure coding practices.

## When to Use This Skill

- When conducting security-focused code reviews
- When analyzing potential vulnerabilities
- When reviewing authentication/authorization code
- When assessing data handling and privacy concerns
- When evaluating dependency security
- When responding to security incidents

## Security Analysis Framework

### 1. Input Validation

**Check for:**
- Unvalidated user input
- Missing input sanitization
- Improper type checking
- Missing length/range validation

**Common vulnerabilities:**
- SQL Injection
- Cross-Site Scripting (XSS)
- Command Injection
- Path Traversal
- XML External Entity (XXE)

**Secure practices:**
```
✓ Validate all input server-side
✓ Use parameterized queries
✓ Sanitize output for HTML context
✓ Implement whitelist validation
✓ Enforce strict type checking
```

### 2. Authentication & Authorization

**Review areas:**
- Password handling and storage
- Session management
- Token generation and validation
- Permission checks
- Multi-factor authentication

**Vulnerabilities to catch:**
- Weak password policies
- Hardcoded credentials
- Insecure session handling
- Missing authorization checks
- Broken access control

**Best practices:**
```
✓ Hash passwords with bcrypt/argon2
✓ Use secure session tokens
✓ Implement proper RBAC
✓ Check permissions on every request
✓ Use HTTPS for all authentication
```

### 3. Data Protection

**Assess:**
- Sensitive data handling
- Encryption at rest and in transit
- Data exposure in logs/errors
- PII (Personally Identifiable Information) protection
- Secret management

**Common issues:**
- Plaintext storage of sensitive data
- Weak encryption algorithms
- Logging sensitive information
- Exposing secrets in code
- Insufficient data anonymization

**Secure approach:**
```
✓ Encrypt sensitive data at rest
✓ Use TLS 1.2+ for data in transit
✓ Never log passwords or tokens
✓ Use key management services
✓ Implement data classification
```

### 4. Dependency Security

**Check for:**
- Outdated dependencies with known CVEs
- Unused dependencies
- Dependencies from untrusted sources
- Transitive dependency vulnerabilities

**Tools to use:**
```bash
# Node.js
npm audit
npm audit fix

# Python
pip-audit
safety check

# Go
go list -m all | nancy sleuth

# General
dependabot alerts
snyk test
```

### 5. API Security

**Review:**
- Rate limiting
- API authentication
- CORS configuration
- Request validation
- Error message information disclosure

**Vulnerabilities:**
- Missing rate limits (DoS risk)
- Weak API keys
- Overly permissive CORS
- Verbose error messages
- Missing request validation

**Implementation:**
```
✓ Implement rate limiting
✓ Use API keys or OAuth tokens
✓ Configure strict CORS policies
✓ Validate all API inputs
✓ Return generic error messages
```

## Vulnerability Categories (OWASP Top 10)

### A01: Broken Access Control
- Missing authentication checks
- Insecure direct object references
- Privilege escalation

### A02: Cryptographic Failures
- Using weak encryption (MD5, SHA1)
- Hardcoded secrets
- Insecure random number generation

### A03: Injection
- SQL Injection
- NoSQL Injection
- Command Injection
- LDAP Injection

### A04: Insecure Design
- Missing security controls
- Insufficient threat modeling
- Lack of security requirements

### A05: Security Misconfiguration
- Default credentials
- Unnecessary features enabled
- Missing security headers
- Verbose error messages

### A06: Vulnerable Components
- Outdated libraries
- Known CVEs in dependencies
- Unpatched systems

### A07: Authentication Failures
- Weak password policies
- Credential stuffing vulnerability
- Missing multi-factor authentication

### A08: Data Integrity Failures
- Insecure deserialization
- Missing integrity checks
- Unsigned code execution

### A09: Logging & Monitoring Failures
- Insufficient logging
- No alerting mechanism
- Logs not reviewed

### A10: Server-Side Request Forgery (SSRF)
- Unvalidated URLs
- Internal resource access
- Cloud metadata exposure

## Security Review Process

### 1. Threat Modeling

Identify:
- Assets to protect
- Potential attackers
- Attack vectors
- Impact of compromise

### 2. Code Analysis

**Manual review:**
- Authentication/authorization flows
- Data handling paths
- Input validation points
- Error handling logic
- Cryptographic operations

**Automated scanning:**
```bash
# Static analysis
semgrep --config=auto .
bandit -r python_code/
gosec ./...

# Dependency check
safety check
npm audit
```

### 3. Testing for Vulnerabilities

**Security testing:**
- Fuzzing input fields
- Testing authentication bypass
- Attempting privilege escalation
- Checking for injection points
- Testing file upload security

### 4. Verification

- Reproduce identified issues
- Assess severity and impact
- Prioritize remediation
- Verify fixes don't break functionality

## Security Checklist

### Authentication
- [ ] Passwords hashed with strong algorithm (bcrypt, argon2)
- [ ] No hardcoded credentials
- [ ] Secure session management
- [ ] Account lockout after failed attempts
- [ ] Multi-factor authentication where appropriate

### Authorization
- [ ] Permission checks on all endpoints
- [ ] Proper role-based access control
- [ ] No insecure direct object references
- [ ] Validated user context in all operations

### Input/Output
- [ ] All input validated and sanitized
- [ ] Parameterized queries used
- [ ] Output encoded for context
- [ ] File uploads properly validated
- [ ] No user input in system commands

### Cryptography
- [ ] Strong encryption algorithms (AES-256)
- [ ] Secure random number generation
- [ ] Proper key management
- [ ] TLS 1.2+ for all connections
- [ ] No deprecated algorithms (MD5, SHA1)

### Data Protection
- [ ] Sensitive data encrypted at rest
- [ ] PII properly handled
- [ ] Secrets not in code or logs
- [ ] Secure backup procedures
- [ ] Data retention policies enforced

### Dependencies
- [ ] No known vulnerabilities (run npm audit / pip-audit)
- [ ] Dependencies from trusted sources
- [ ] Minimal dependency footprint
- [ ] Regular update schedule

### Error Handling
- [ ] Generic error messages to users
- [ ] Detailed logs for debugging (without secrets)
- [ ] No stack traces in production
- [ ] Proper exception handling

### Security Headers
- [ ] Content-Security-Policy
- [ ] X-Frame-Options
- [ ] X-Content-Type-Options
- [ ] Strict-Transport-Security
- [ ] X-XSS-Protection

## Common Secure Coding Patterns

### Parameterized Queries (SQL Injection Prevention)
```python
# ✗ Vulnerable
query = f"SELECT * FROM users WHERE id = {user_id}"

# ✓ Secure
query = "SELECT * FROM users WHERE id = %s"
cursor.execute(query, (user_id,))
```

### Output Encoding (XSS Prevention)
```javascript
// ✗ Vulnerable
element.innerHTML = userInput;

// ✓ Secure
element.textContent = userInput;
// Or use a library like DOMPurify
```

### Path Traversal Prevention
```python
# ✗ Vulnerable
file_path = os.path.join(base_dir, user_filename)

# ✓ Secure
safe_path = os.path.normpath(os.path.join(base_dir, user_filename))
if not safe_path.startswith(base_dir):
    raise SecurityError("Invalid file path")
```

### Secure Password Hashing
```javascript
// ✗ Vulnerable
const hash = crypto.createHash('md5').update(password).digest('hex');

// ✓ Secure
const hash = await bcrypt.hash(password, 12);
```

## Reporting Security Issues

When you find a security vulnerability:

1. **Assess severity** using CVSS or similar framework
2. **Document the issue**:
   - Vulnerability description
   - Affected code/components
   - Reproduction steps
   - Potential impact
   - Suggested remediation

3. **Prioritize remediation**:
   - Critical: Immediate fix required
   - High: Fix in current sprint
   - Medium: Schedule for next release
   - Low: Add to backlog

4. **Verify the fix**:
   - Test that vulnerability is resolved
   - Ensure no regression
   - Update security documentation

## Resources

- OWASP Top 10: https://owasp.org/www-project-top-ten/
- CWE (Common Weakness Enumeration): https://cwe.mitre.org/
- NIST Guidelines: https://csrc.nist.gov/
- Security Headers: https://securityheaders.com/
- OWASP Cheat Sheets: https://cheatsheetseries.owasp.org/

Remember: Security is not a one-time check but an ongoing process. Regular security reviews and staying updated on new vulnerabilities are essential for maintaining secure applications.
