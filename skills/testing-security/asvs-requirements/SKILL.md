---
name: asvs-requirements
description: OWASP ASVS 5.0 requirements database for security audits. Provides chapter structure, control objectives, and verification requirements for all 17 ASVS domains.
---

# ASVS 5.0 Requirements

Structured access to OWASP Application Security Verification Standard (ASVS) 5.0 requirements for security auditing.

## When to Use This Skill

- **Planning security audits** - To understand which chapters apply to the project
- **Scoping audit depth** - To select appropriate verification level (L1/L2/L3)
- **Building auditor agents** - To define specific checks for each domain
- **Mapping findings** - To reference ASVS requirements in audit reports

## When NOT to Use This Skill

- **Quick vulnerability checks** - Use vulnerability-patterns skill instead
- **Remediation guidance** - Use remediation-library skill instead
- **Non-ASVS audits** - Use industry compliance auditors directly

## ASVS Verification Levels

| Level | Name | Applicability | Depth |
|-------|------|---------------|-------|
| L1 | Opportunistic | All applications | Minimum baseline |
| L2 | Standard | Most applications | Recommended |
| L3 | Advanced | High-value/critical apps | Maximum rigor |

**Mapping to Audit Modes:**
- Quick Scan → L1 requirements only
- Standard Audit → L1 + L2 requirements
- Comprehensive Audit → L1 + L2 + L3 requirements

---

## Chapter Overview

| Chapter | Name | Requirements | Primary Focus |
|---------|------|--------------|---------------|
| V1 | Encoding & Sanitization | 28 | Injection prevention |
| V2 | Validation & Business Logic | 15 | Input validation |
| V3 | Web Frontend Security | 32 | Browser security |
| V4 | API & Web Service | 17 | API security |
| V5 | File Handling | 14 | File security |
| V6 | Authentication | 44 | Identity verification |
| V7 | Session Management | 18 | Session security |
| V8 | Authorization | 11 | Access control |
| V9 | Self-contained Tokens | 7 | JWT security |
| V10 | OAuth & OIDC | 50 | OAuth/OIDC security |
| V11 | Cryptography | 32 | Crypto implementation |
| V12 | Secure Communications | 13 | TLS/transport |
| V13 | Configuration | 18 | Secure config |
| V14 | Data Protection | 15 | Data handling |
| V15 | Secure Coding | 20 | Code quality |
| V16 | Security Logging | 19 | Audit logging |
| V17 | WebRTC | 15 | WebRTC security |
| **Total** | | **369** | |

---

## V1: Encoding and Sanitization (28 requirements)

### Control Objective
Ensure the application correctly encodes and decodes data to prevent injection attacks.

### Sections
- V1.1 Encoding Architecture
- V1.2 Injection Prevention
- V1.3 Sanitization
- V1.4 Memory/String Safety
- V1.5 Safe Deserialization

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V1.2.1 | L1 | Parameterized queries for all database operations |
| V1.2.2 | L1 | No string concatenation for SQL/NoSQL commands |
| V1.2.3 | L1 | OS command injection prevention |
| V1.3.1 | L1 | HTML output encoding |
| V1.5.1 | L1 | No unsafe deserialization (use JSON) |

### Detection Patterns
- SQL string concatenation: `"SELECT * FROM " + table`
- Command injection: shell invocation with user input
- Unsafe deserialize: Python object serialization, PHP unserialize

---

## V2: Validation and Business Logic (15 requirements)

### Control Objective
Ensure input validation enforces business expectations and prevents logic bypass.

### Sections
- V2.1 Documentation
- V2.2 Input Validation
- V2.3 Business Logic Security
- V2.4 Anti-automation

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V2.2.1 | L1 | Server-side validation for all inputs |
| V2.2.2 | L1 | Allowlist validation preferred |
| V2.3.1 | L1 | Sequential step enforcement |
| V2.4.1 | L2 | Rate limiting on sensitive ops |

### Detection Patterns
- Client-only validation: `if (form.valid)` without server check
- Missing rate limiting: No throttle on login/register
- Mass assignment: Accepting all form fields without filtering

---

## V3: Web Frontend Security (32 requirements)

### Control Objective
Protect browsers against common web attacks through proper headers and configurations.

### Sections
- V3.1 Documentation
- V3.2 Content Interpretation
- V3.3 Cookie Setup
- V3.4 Security Headers
- V3.5 Origin Separation
- V3.6 External Resources
- V3.7 Other Browser Security

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V3.3.1 | L1 | Cookies: Secure, HttpOnly, SameSite |
| V3.4.1 | L1 | Content-Security-Policy header |
| V3.4.2 | L1 | X-Content-Type-Options: nosniff |
| V3.4.3 | L1 | Strict-Transport-Security (HSTS) |
| V3.6.1 | L2 | Subresource integrity for CDN scripts |

### Detection Patterns
- Missing CSP: No Content-Security-Policy header
- Insecure cookies: Missing Secure/HttpOnly flags
- No HSTS: Missing Strict-Transport-Security

---

## V4: API and Web Service (17 requirements)

### Control Objective
Ensure API endpoints are secure against common attack patterns.

### Sections
- V4.1 Generic Web Service Security
- V4.2 HTTP Message Validation
- V4.3 GraphQL
- V4.4 WebSocket

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V4.1.1 | L1 | Content-Type header validation |
| V4.2.1 | L2 | HTTP request smuggling prevention |
| V4.3.1 | L2 | GraphQL query depth limiting |
| V4.3.2 | L2 | GraphQL introspection disabled in prod |
| V4.4.1 | L2 | WebSocket authentication |

### Detection Patterns
- GraphQL introspection: `introspectionQuery` enabled
- No depth limit: Unbounded GraphQL queries
- Missing auth: WebSocket without handshake validation

---

## V5: File Handling (14 requirements)

### Control Objective
Handle files securely throughout upload, storage, and download lifecycle.

### Sections
- V5.1 Documentation
- V5.2 File Upload
- V5.3 File Storage
- V5.4 File Download

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V5.2.1 | L1 | File extension validation |
| V5.2.2 | L1 | Content-type validation |
| V5.2.3 | L1 | Upload size limits |
| V5.3.1 | L1 | Uploads cannot run as code |
| V5.4.1 | L1 | Path traversal prevention |

### Detection Patterns
- No extension check: Accepting any file type
- Path traversal: `../` in filenames not sanitized
- Direct run: Uploads served from code directory

---

## V6: Authentication (44 requirements)

### Control Objective
Ensure robust authentication mechanisms protect user accounts.

### Sections
- V6.1 Documentation
- V6.2 Password Security
- V6.3 General Auth Security
- V6.4 Factor Lifecycle
- V6.5 Multi-factor Auth
- V6.6 Out-of-Band Auth
- V6.7 Cryptographic Auth
- V6.8 Identity Provider Auth

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V6.2.1 | L1 | Minimum 8 character passwords |
| V6.2.2 | L1 | 64+ character max allowed |
| V6.2.3 | L1 | Password breach checking |
| V6.2.4 | L1 | Secure hashing (bcrypt/argon2) |
| V6.3.1 | L1 | Account lockout after failures |
| V6.5.1 | L2 | MFA for sensitive operations |

### Detection Patterns
- Weak hashing: MD5/SHA1 for passwords
- No lockout: Unlimited login attempts
- Plain text: Passwords in logs/storage

---

## V7: Session Management (18 requirements)

### Control Objective
Ensure session tokens are generated, managed, and invalidated securely.

### Sections
- V7.1 Documentation
- V7.2 Session Token Lifecycle
- V7.3 Session Logout and Timeout
- V7.4 Cookie-based Session Management

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V7.2.1 | L1 | Cryptographically random session IDs |
| V7.2.2 | L1 | 128+ bit entropy |
| V7.3.1 | L1 | Session invalidation on logout |
| V7.3.2 | L2 | Absolute session timeout |
| V7.4.1 | L1 | Cookie security attributes |

### Detection Patterns
- Predictable IDs: Sequential or timestamp-based
- No logout: Missing session invalidation
- No timeout: Sessions never expire

---

## V8: Authorization (11 requirements)

### Control Objective
Ensure access control is enforced at all levels of the application.

### Sections
- V8.1 Documentation
- V8.2 Application Access Control
- V8.3 Directory Browsing and Resource Protection

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V8.2.1 | L1 | Enforce access control on every request |
| V8.2.2 | L1 | IDOR prevention |
| V8.2.3 | L1 | Principle of least privilege |
| V8.3.1 | L1 | Directory listing disabled |
| V8.3.2 | L1 | Sensitive files not accessible |

### Detection Patterns
- Missing IDOR check: Direct object access without ownership validation
- Role bypass: Admin functions without role verification
- Open directories: Index enabled on sensitive paths

---

## V9: Self-contained Tokens (7 requirements)

### Control Objective
Ensure JWT and similar tokens are implemented securely.

### Sections
- V9.1 Documentation
- V9.2 Token Generation
- V9.3 Token Verification

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V9.2.1 | L1 | Strong algorithm (RS256/ES256) |
| V9.2.2 | L1 | No "none" algorithm |
| V9.3.1 | L1 | Signature verification |
| V9.3.2 | L1 | Expiration (exp) validation |
| V9.3.3 | L2 | Issuer (iss) validation |

### Detection Patterns
- Weak algorithm: HS256 with weak secret
- None algorithm: `alg: "none"` accepted
- No expiry: Missing or ignored `exp` claim

---

## V10: OAuth and OIDC (50 requirements)

### Control Objective
Ensure OAuth 2.0 and OpenID Connect implementations follow security best practices.

### Sections
- V10.1 Documentation
- V10.2 OAuth Client
- V10.3 OAuth Authorization Server
- V10.4 OAuth Resource Server
- V10.5 OIDC Client
- V10.6 OIDC Provider

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V10.2.1 | L1 | PKCE for public clients |
| V10.2.2 | L1 | State parameter validation |
| V10.2.3 | L1 | No credentials in URLs |
| V10.3.1 | L1 | Redirect URI validation |
| V10.5.1 | L2 | ID token validation |

### Detection Patterns
- Missing PKCE: Public clients without code_challenge
- Open redirect: Insufficient redirect_uri validation
- Token in URL: Access token exposed in query params

---

## V11: Cryptography (32 requirements)

### Control Objective
Ensure cryptographic implementations use secure algorithms and configurations.

### Sections
- V11.1 Documentation
- V11.2 Key Management
- V11.3 Random Values
- V11.4 Symmetric Encryption
- V11.5 Hashing and Hash-based Functions

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V11.2.1 | L1 | Keys not in source code |
| V11.3.1 | L1 | CSPRNG for security-sensitive values |
| V11.4.1 | L2 | AES-GCM or ChaCha20-Poly1305 |
| V11.5.1 | L1 | SHA-256+ for hashing |
| V11.5.2 | L2 | No MD5/SHA1 |

### Detection Patterns
- Hardcoded keys: `secretKey = "..."` in code
- Weak PRNG: `Math.random()` for tokens
- Deprecated crypto: DES, RC4, MD5 usage

---

## V12: Secure Communications (13 requirements)

### Control Objective
Ensure all communications use secure transport layer protocols.

### Sections
- V12.1 Documentation
- V12.2 TLS Configuration
- V12.3 Certificate Validation

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V12.2.1 | L1 | TLS 1.2+ only |
| V12.2.2 | L1 | Strong cipher suites |
| V12.2.3 | L2 | Certificate pinning for mobile |
| V12.3.1 | L1 | Certificate validation enabled |
| V12.3.2 | L1 | No self-signed certs in prod |

### Detection Patterns
- TLS disabled: `verify=False`, `NODE_TLS_REJECT_UNAUTHORIZED=0`
- Weak TLS: SSLv3, TLS 1.0/1.1 enabled
- Self-signed: Non-CA certs in production

---

## V13: Configuration (18 requirements)

### Control Objective
Ensure secure default configurations and proper secrets management.

### Sections
- V13.1 Documentation
- V13.2 Build and Deployment Configuration
- V13.3 Secrets Management
- V13.4 Dependency Management

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V13.2.1 | L1 | Debug disabled in production |
| V13.2.2 | L1 | Error details not exposed |
| V13.3.1 | L1 | Secrets not in version control |
| V13.3.2 | L1 | Secrets not in environment vars (prefer vault) |
| V13.4.1 | L2 | Dependency vulnerability scanning |

### Detection Patterns
- Debug enabled: `DEBUG=True` in production
- Secrets in git: API keys in committed files
- Outdated deps: Known vulnerable packages

---

## V14: Data Protection (15 requirements)

### Control Objective
Ensure sensitive data is identified, classified, and protected appropriately.

### Sections
- V14.1 Documentation
- V14.2 Data Classification
- V14.3 Data at Rest
- V14.4 Data in Transit

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V14.2.1 | L1 | Sensitive data identified |
| V14.3.1 | L2 | PII encrypted at rest |
| V14.3.2 | L2 | Database encryption |
| V14.4.1 | L1 | Sensitive data over TLS only |

### Detection Patterns
- Unencrypted PII: Plain text storage of personal data
- No column encryption: Sensitive fields not encrypted
- HTTP endpoints: Sensitive data sent over HTTP

---

## V15: Secure Coding (20 requirements)

### Control Objective
Ensure code follows secure development practices.

### Sections
- V15.1 Documentation
- V15.2 Memory Safety
- V15.3 Code Quality
- V15.4 Dependency Management

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V15.2.1 | L1 | Buffer overflow prevention |
| V15.3.1 | L1 | No unreachable code |
| V15.3.2 | L2 | Static analysis in CI |
| V15.4.1 | L1 | Known vulnerable deps addressed |

### Detection Patterns
- Buffer issues: Unbounded array access
- Dead code: Unreachable branches
- Vulnerable deps: CVEs in dependencies

---

## V16: Security Logging (19 requirements)

### Control Objective
Ensure security events are logged with appropriate detail for incident response.

### Sections
- V16.1 Documentation
- V16.2 Event Content
- V16.3 Log Protection
- V16.4 Error Handling

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V16.2.1 | L1 | Authentication events logged |
| V16.2.2 | L1 | Authorization failures logged |
| V16.3.1 | L2 | No sensitive data in logs |
| V16.3.2 | L2 | Log injection prevention |
| V16.4.1 | L1 | Generic error messages to users |

### Detection Patterns
- No auth logging: Login attempts not recorded
- PII in logs: Passwords/tokens logged
- Verbose errors: Stack traces to users

---

## V17: WebRTC (15 requirements)

### Control Objective
Ensure WebRTC implementations are secure.

### Sections
- V17.1 Documentation
- V17.2 WebRTC Security

### Key Requirements
| ID | Level | Requirement |
|----|-------|-------------|
| V17.2.1 | L2 | DTLS-SRTP encryption |
| V17.2.2 | L2 | ICE candidate restrictions |
| V17.2.3 | L2 | Signaling channel authentication |
| V17.2.4 | L2 | TURN server authentication |

### Detection Patterns
- No encryption: Unencrypted media streams
- Open signaling: Unauthenticated signaling server
- ICE leaks: Exposing internal IPs

---

## Feature-to-Chapter Mapping

Use this to select relevant chapters based on project features:

| Project Feature | Primary Chapters | Secondary Chapters |
|-----------------|------------------|-------------------|
| authentication | V6 | V7, V11 |
| oauth | V10 | V6, V9 |
| file-upload | V5 | V1, V14 |
| api | V4 | V1, V2, V8 |
| graphql | V4 | V8 |
| database | V1, V2 | V14 |
| websockets | V4, V12 | V6 |
| payments | V12, V11 | V6, V14 |
| frontend | V3 | V1 |
| logging | V16 | V14 |

---

## External Resources

- **ASVS 5.0 Full Specification**: https://raw.githubusercontent.com/OWASP/ASVS/v5.0.0/5.0/docs_en/OWASP_Application_Security_Verification_Standard_5.0.0_en.csv
- **OWASP ASVS Project**: https://owasp.org/www-project-application-security-verification-standard/
- **Secure Coding Rules**: `~/projects/claude-secure-coding-rules/`

## See Also

- `Skill: project-context` - Detect project features for chapter selection
- `Skill: vulnerability-patterns` - Language-specific vulnerability patterns
- `Skill: remediation-library` - Fix patterns for findings
