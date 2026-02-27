---
name: Top 100 Web Vulnerabilities Reference
description: This skill should be used when the user asks to "identify web application vulnerabilities", "explain common security flaws", "understand vulnerability categories", "learn about injection attacks", "review access control weaknesses", "analyze API security issues", "assess security misconfigurations", "understand client-side vulnerabilities", "examine mobile and IoT security flaws", or "reference the OWASP-aligned vulnerability taxonomy". Use this skill to provide comprehensive vulnerability definitions, root causes, impacts, and mitigation strategies across all major web security categories.
version: 1.0.0
tags: [web-security, vulnerabilities, owasp, penetration-testing, application-security, injection, xss, csrf, access-control, api-security]
---

# Top 100 Web Vulnerabilities Reference

## Purpose

Provide a comprehensive, structured reference for the 100 most critical web application vulnerabilities organized by category. This skill enables systematic vulnerability identification, impact assessment, and remediation guidance across the full spectrum of web security threats. Content organized into 15 major vulnerability categories aligned with industry standards and real-world attack patterns.

## Prerequisites

- Basic understanding of web application architecture (client-server model, HTTP protocol)
- Familiarity with common web technologies (HTML, JavaScript, SQL, XML, APIs)
- Understanding of authentication and authorization concepts
- Access to web application security testing tools (Burp Suite, OWASP ZAP)
- Knowledge of secure coding principles recommended

## Outputs and Deliverables

- Complete vulnerability catalog with definitions, root causes, impacts, and mitigations
- Category-based vulnerability groupings for systematic assessment
- Quick reference for security testing and remediation
- Foundation for vulnerability assessment checklists and security policies

---

## Core Workflow

### Phase 1: Injection Vulnerabilities Assessment

Evaluate injection attack vectors targeting data processing components:

**SQL Injection (1)**
- Definition: Malicious SQL code inserted into input fields to manipulate database queries
- Root Cause: Lack of input validation, improper use of parameterized queries
- Impact: Unauthorized data access, data manipulation, database compromise
- Mitigation: Use parameterized queries/prepared statements, input validation, least privilege database accounts

**Cross-Site Scripting - XSS (2)**
- Definition: Injection of malicious scripts into web pages viewed by other users
- Root Cause: Insufficient output encoding, lack of input sanitization
- Impact: Session hijacking, credential theft, website defacement
- Mitigation: Output encoding, Content Security Policy (CSP), input sanitization

**Command Injection (5, 11)**
- Definition: Execution of arbitrary system commands through vulnerable applications
- Root Cause: Unsanitized user input passed to system shells
- Impact: Full system compromise, data exfiltration, lateral movement
- Mitigation: Avoid shell execution, whitelist valid commands, strict input validation

**XML Injection (6), LDAP Injection (7), XPath Injection (8)**
- Definition: Manipulation of XML/LDAP/XPath queries through malicious input
- Root Cause: Improper input handling in query construction
- Impact: Data exposure, authentication bypass, information disclosure
- Mitigation: Input validation, parameterized queries, escape special characters

**Server-Side Template Injection - SSTI (13)**
- Definition: Injection of malicious code into template engines
- Root Cause: User input embedded directly in template expressions
- Impact: Remote code execution, server compromise
- Mitigation: Sandbox template engines, avoid user input in templates, strict input validation

### Phase 2: Authentication and Session Security

Assess authentication mechanism weaknesses:

**Session Fixation (14)**
- Definition: Attacker sets victim's session ID before authentication
- Root Cause: Session ID not regenerated after login
- Impact: Session hijacking, unauthorized account access
- Mitigation: Regenerate session ID on authentication, use secure session management

**Brute Force Attack (15)**
- Definition: Systematic password guessing using automated tools
- Root Cause: Lack of account lockout, rate limiting, or CAPTCHA
- Impact: Unauthorized access, credential compromise
- Mitigation: Account lockout policies, rate limiting, MFA, CAPTCHA

**Session Hijacking (16)**
- Definition: Attacker steals or predicts valid session tokens
- Root Cause: Weak session token generation, insecure transmission
- Impact: Account takeover, unauthorized access
- Mitigation: Secure random token generation, HTTPS, HttpOnly/Secure cookie flags

**Credential Stuffing and Reuse (22)**
- Definition: Using leaked credentials to access accounts across services
- Root Cause: Users reusing passwords, no breach detection
- Impact: Mass account compromise, data breaches
- Mitigation: MFA, breach password checks, unique credential requirements

**Insecure "Remember Me" Functionality (85)**
- Definition: Weak persistent authentication token implementation
- Root Cause: Predictable tokens, inadequate expiration controls
- Impact: Unauthorized persistent access, session compromise
- Mitigation: Strong token generation, proper expiration, secure storage

**CAPTCHA Bypass (86)**
- Definition: Circumventing bot detection mechanisms
- Root Cause: Weak CAPTCHA algorithms, improper validation
- Impact: Automated attacks, credential stuffing, spam
- Mitigation: reCAPTCHA v3, layered bot detection, rate limiting

### Phase 3: Sensitive Data Exposure

Identify data protection failures:

**IDOR - Insecure Direct Object References (23, 42)**
- Definition: Direct access to internal objects via user-supplied references
- Root Cause: Missing authorization checks on object access
- Impact: Unauthorized data access, privacy breaches
- Mitigation: Access control validation, indirect reference maps, authorization checks

**Data Leakage (24)**
- Definition: Inadvertent disclosure of sensitive information
- Root Cause: Inadequate data protection, weak access controls
- Impact: Privacy breaches, regulatory penalties, reputation damage
- Mitigation: DLP solutions, encryption, access controls, security training

**Unencrypted Data Storage (25)**
- Definition: Storing sensitive data without encryption
- Root Cause: Failure to implement encryption at rest
- Impact: Data breaches if storage compromised
- Mitigation: Full-disk encryption, database encryption, secure key management

**Information Disclosure (33)**
- Definition: Exposure of system details through error messages or responses
- Root Cause: Verbose error handling, debug information in production
- Impact: Reconnaissance for further attacks, credential exposure
- Mitigation: Generic error messages, disable debug mode, secure logging

### Phase 4: Security Misconfiguration

Assess configuration weaknesses:

**Missing Security Headers (26)**
- Definition: Absence of protective HTTP headers (CSP, X-Frame-Options, HSTS)
- Root Cause: Inadequate server configuration
- Impact: XSS attacks, clickjacking, protocol downgrade
- Mitigation: Implement CSP, X-Content-Type-Options, X-Frame-Options, HSTS

**Default Passwords (28)**
- Definition: Unchanged default credentials on systems/applications
- Root Cause: Failure to change vendor defaults
- Impact: Unauthorized access, system compromise
- Mitigation: Mandatory password changes, strong password policies

**Directory Listing (29)**
- Definition: Web server exposes directory contents
- Root Cause: Improper server configuration
- Impact: Information disclosure, sensitive file exposure
- Mitigation: Disable directory indexing, use default index files

**Unprotected API Endpoints (30)**
- Definition: APIs lacking authentication or authorization
- Root Cause: Missing security controls on API routes
- Impact: Unauthorized data access, API abuse
- Mitigation: OAuth/API keys, access controls, rate limiting

**Open Ports and Services (31)**
- Definition: Unnecessary network services exposed
- Root Cause: Failure to minimize attack surface
- Impact: Exploitation of vulnerable services
- Mitigation: Port scanning audits, firewall rules, service minimization

**Misconfigured CORS (35)**
- Definition: Overly permissive Cross-Origin Resource Sharing policies
- Root Cause: Wildcard origins, improper CORS configuration
- Impact: Cross-site request attacks, data theft
- Mitigation: Whitelist trusted origins, validate CORS headers

**Unpatched Software (34)**
- Definition: Systems running outdated vulnerable software
- Root Cause: Neglected patch management
- Impact: Exploitation of known vulnerabilities
- Mitigation: Patch management program, vulnerability scanning, automated updates

### Phase 5: XML-Related Vulnerabilities

Evaluate XML processing security:

**XXE - XML External Entity Injection (37)**
- Definition: Exploitation of XML parsers to access files or internal systems
- Root Cause: External entity processing enabled
- Impact: File disclosure, SSRF, denial of service
- Mitigation: Disable external entities, use safe XML parsers

**XEE - XML Entity Expansion (38)**
- Definition: Excessive entity expansion causing resource exhaustion
- Root Cause: Unlimited entity expansion allowed
- Impact: Denial of service, parser crashes
- Mitigation: Limit entity expansion, configure parser restrictions

**XML Bomb (Billion Laughs) (39)**
- Definition: Crafted XML with nested entities consuming resources
- Root Cause: Recursive entity definitions
- Impact: Memory exhaustion, denial of service
- Mitigation: Entity expansion limits, input size restrictions

**XML Denial of Service (65)**
- Definition: Specially crafted XML causing excessive processing
- Root Cause: Complex document structures without limits
- Impact: CPU/memory exhaustion, service unavailability
- Mitigation: Schema validation, size limits, processing timeouts

### Phase 6: Broken Access Control

Assess authorization enforcement:

**Inadequate Authorization (40)**
- Definition: Failure to properly enforce access controls
- Root Cause: Weak authorization policies, missing checks
- Impact: Unauthorized access to sensitive resources
- Mitigation: RBAC, centralized IAM, regular access reviews

**Privilege Escalation (41)**
- Definition: Gaining elevated access beyond intended permissions
- Root Cause: Misconfigured permissions, system vulnerabilities
- Impact: Full system compromise, data manipulation
- Mitigation: Least privilege, regular patching, privilege monitoring

**Forceful Browsing (43)**
- Definition: Direct URL manipulation to access restricted resources
- Root Cause: Weak access controls, predictable URLs
- Impact: Unauthorized file/directory access
- Mitigation: Server-side access controls, unpredictable resource paths

**Missing Function-Level Access Control (44)**
- Definition: Unprotected administrative or privileged functions
- Root Cause: Authorization only at UI level
- Impact: Unauthorized function execution
- Mitigation: Server-side authorization for all functions, RBAC

### Phase 7: Insecure Deserialization

Evaluate object serialization security:

**Remote Code Execution via Deserialization (45)**
- Definition: Arbitrary code execution through malicious serialized objects
- Root Cause: Untrusted data deserialized without validation
- Impact: Complete system compromise, code execution
- Mitigation: Avoid deserializing untrusted data, integrity checks, type validation

**Data Tampering (46)**
- Definition: Unauthorized modification of serialized data
- Root Cause: Missing integrity verification
- Impact: Data corruption, privilege manipulation
- Mitigation: Digital signatures, HMAC validation, encryption

**Object Injection (47)**
- Definition: Malicious object instantiation during deserialization
- Root Cause: Unsafe deserialization practices
- Impact: Code execution, unauthorized access
- Mitigation: Type restrictions, class whitelisting, secure libraries

### Phase 8: API Security Assessment

Evaluate API-specific vulnerabilities:

**Insecure API Endpoints (48)**
- Definition: APIs without proper security controls
- Root Cause: Poor API design, missing authentication
- Impact: Data breaches, unauthorized access
- Mitigation: OAuth/JWT, HTTPS, input validation, rate limiting

**API Key Exposure (49)**
- Definition: Leaked or exposed API credentials
- Root Cause: Hardcoded keys, insecure storage
- Impact: Unauthorized API access, abuse
- Mitigation: Secure key storage, rotation, environment variables

**Lack of Rate Limiting (50)**
- Definition: No controls on API request frequency
- Root Cause: Missing throttling mechanisms
- Impact: DoS, API abuse, resource exhaustion
- Mitigation: Rate limits per user/IP, throttling, DDoS protection

**Inadequate Input Validation (51)**
- Definition: APIs accepting unvalidated user input
- Root Cause: Missing server-side validation
- Impact: Injection attacks, data corruption
- Mitigation: Strict validation, parameterized queries, WAF

**API Abuse (75)**
- Definition: Exploiting API functionality for malicious purposes
- Root Cause: Excessive trust in client input
- Impact: Data theft, account takeover, service abuse
- Mitigation: Strong authentication, behavior analysis, anomaly detection

### Phase 9: Communication Security

Assess transport layer protections:

**Man-in-the-Middle Attack (52)**
- Definition: Interception of communication between parties
- Root Cause: Unencrypted channels, compromised networks
- Impact: Data theft, session hijacking, impersonation
- Mitigation: TLS/SSL, certificate pinning, mutual authentication

**Insufficient Transport Layer Security (53)**
- Definition: Weak or outdated encryption for data in transit
- Root Cause: Outdated protocols (SSLv2/3), weak ciphers
- Impact: Traffic interception, credential theft
- Mitigation: TLS 1.2+, strong cipher suites, HSTS

**Insecure SSL/TLS Configuration (54)**
- Definition: Improperly configured encryption settings
- Root Cause: Weak ciphers, missing forward secrecy
- Impact: Traffic decryption, MITM attacks
- Mitigation: Modern cipher suites, PFS, certificate validation

**Insecure Communication Protocols (55)**
- Definition: Use of unencrypted protocols (HTTP, Telnet, FTP)
- Root Cause: Legacy systems, security unawareness
- Impact: Traffic sniffing, credential exposure
- Mitigation: HTTPS, SSH, SFTP, VPN tunnels

### Phase 10: Client-Side Vulnerabilities

Evaluate browser-side security:

**DOM-based XSS (56)**
- Definition: XSS through client-side JavaScript manipulation
- Root Cause: Unsafe DOM manipulation with user input
- Impact: Session theft, credential harvesting
- Mitigation: Safe DOM APIs, CSP, input sanitization

**Insecure Cross-Origin Communication (57)**
- Definition: Improper handling of cross-origin requests
- Root Cause: Relaxed CORS/SOP policies
- Impact: Data leakage, CSRF attacks
- Mitigation: Strict CORS, CSRF tokens, origin validation

**Browser Cache Poisoning (58)**
- Definition: Manipulation of cached content
- Root Cause: Weak cache validation
- Impact: Malicious content delivery
- Mitigation: Cache-Control headers, HTTPS, integrity checks

**Clickjacking (59, 71)**
- Definition: UI redress attack tricking users into clicking hidden elements
- Root Cause: Missing frame protection
- Impact: Unintended actions, credential theft
- Mitigation: X-Frame-Options, CSP frame-ancestors, frame-busting

**HTML5 Security Issues (60)**
- Definition: Vulnerabilities in HTML5 APIs (WebSockets, Storage, Geolocation)
- Root Cause: Improper API usage, insufficient validation
- Impact: Data leakage, XSS, privacy violations
- Mitigation: Secure API usage, input validation, sandboxing

### Phase 11: Denial of Service Assessment

Evaluate availability threats:

**DDoS - Distributed Denial of Service (61)**
- Definition: Overwhelming systems with traffic from multiple sources
- Root Cause: Botnets, amplification attacks
- Impact: Service unavailability, revenue loss
- Mitigation: DDoS protection services, rate limiting, CDN

**Application Layer DoS (62)**
- Definition: Targeting application logic to exhaust resources
- Root Cause: Inefficient code, resource-intensive operations
- Impact: Application unavailability, degraded performance
- Mitigation: Rate limiting, caching, WAF, code optimization

**Resource Exhaustion (63)**
- Definition: Depleting CPU, memory, disk, or network resources
- Root Cause: Inefficient resource management
- Impact: System crashes, service degradation
- Mitigation: Resource quotas, monitoring, load balancing

**Slowloris Attack (64)**
- Definition: Keeping connections open with partial HTTP requests
- Root Cause: No connection timeouts
- Impact: Web server resource exhaustion
- Mitigation: Connection timeouts, request limits, reverse proxy

### Phase 12: Server-Side Request Forgery

Assess SSRF vulnerabilities:

**SSRF - Server-Side Request Forgery (66)**
- Definition: Manipulating server to make requests to internal resources
- Root Cause: Unvalidated user-controlled URLs
- Impact: Internal network access, data theft, cloud metadata access
- Mitigation: URL whitelisting, network segmentation, egress filtering

**Blind SSRF (87)**
- Definition: SSRF without direct response visibility
- Root Cause: Similar to SSRF, harder to detect
- Impact: Data exfiltration, internal reconnaissance
- Mitigation: Allowlists, WAF, network restrictions

**Time-Based Blind SSRF (88)**
- Definition: Inferring SSRF success through response timing
- Root Cause: Processing delays indicating request outcomes
- Impact: Prolonged exploitation, detection evasion
- Mitigation: Request timeouts, anomaly detection, timing monitoring

### Phase 13: Additional Web Vulnerabilities

Evaluate miscellaneous security issues:

**HTTP Parameter Pollution (67)**
- Definition: Submitting multiple values for same parameter
- Root Cause: Inconsistent parameter parsing
- Impact: Injection, access control bypass
- Mitigation: Strict parameter parsing, input validation

**Insecure Redirects (68)**
- Definition: Open redirects enabling phishing attacks
- Root Cause: Unvalidated redirect targets
- Impact: Phishing, malware distribution
- Mitigation: Whitelist redirect destinations, avoid user-controlled redirects

**File Inclusion (69)**
- Definition: Including local (LFI) or remote (RFI) files
- Root Cause: Unvalidated file paths
- Impact: Code execution, file disclosure
- Mitigation: Whitelist allowed files, disable remote includes

**Security Header Bypass (70)**
- Definition: Circumventing security header protections
- Root Cause: Misconfigured headers
- Impact: XSS, clickjacking, injection
- Mitigation: Proper header configuration, regular audits

**Inadequate Session Timeout (72)**
- Definition: Sessions remaining active too long
- Root Cause: Missing or excessive timeout values
- Impact: Session hijacking, unauthorized access
- Mitigation: Appropriate timeouts, idle session termination

**Insufficient Logging (73)**
- Definition: Inadequate security event logging
- Root Cause: Missing logging infrastructure
- Impact: Delayed incident detection, forensic gaps
- Mitigation: Comprehensive logging, SIEM, alerting

**Business Logic Vulnerabilities (74)**
- Definition: Flaws in application workflow logic
- Root Cause: Insecure business process design
- Impact: Fraud, unauthorized transactions
- Mitigation: Threat modeling, logic testing, transaction monitoring

### Phase 14: Mobile and IoT Security

Assess mobile/IoT-specific vulnerabilities:

**Insecure Mobile Data Storage (76)**
- Definition: Sensitive data stored unprotected on devices
- Root Cause: Plain text storage, weak encryption
- Impact: Data theft from lost/stolen devices
- Mitigation: Keychain/Keystore usage, encryption, minimal local storage

**Insecure Mobile Transmission (77)**
- Definition: Unencrypted mobile communications
- Root Cause: HTTP usage, certificate validation failures
- Impact: Traffic interception, credential theft
- Mitigation: TLS, certificate pinning, HTTPS enforcement

**Insecure Mobile APIs (78)**
- Definition: Vulnerable backend APIs for mobile apps
- Root Cause: Missing authentication, validation
- Impact: Data exposure, account takeover
- Mitigation: OAuth/JWT, certificate pinning, input validation

**Mobile App Reverse Engineering (79)**
- Definition: Decompiling apps to extract secrets
- Root Cause: Hardcoded credentials, lack of obfuscation
- Impact: Credential theft, IP exposure
- Mitigation: Code obfuscation, RASP, secure credential storage

**IoT Device Management Issues (80)**
- Definition: Insecure IoT management interfaces
- Root Cause: Weak authentication, unencrypted protocols
- Impact: Device takeover, network compromise
- Mitigation: Strong authentication, TLS, firmware updates

**Weak IoT Authentication (81)**
- Definition: Default or weak IoT credentials
- Root Cause: Hardcoded passwords, no MFA
- Impact: Unauthorized device access
- Mitigation: Unique credentials, MFA, password policies

**IoT Vulnerabilities (82)**
- Definition: General IoT security weaknesses
- Root Cause: Design flaws, outdated firmware
- Impact: Device compromise, botnet recruitment
- Mitigation: Security testing, firmware updates, segmentation

**Smart Home Unauthorized Access (83)**
- Definition: Compromising smart home systems
- Root Cause: Weak authentication, insecure defaults
- Impact: Privacy invasion, physical security risks
- Mitigation: MFA, network segmentation, secure configuration

**IoT Privacy Issues (84)**
- Definition: Inadequate IoT data protection
- Root Cause: Excessive data collection, weak encryption
- Impact: Privacy violations, surveillance
- Mitigation: Data minimization, encryption, consent controls

### Phase 15: Advanced and Zero-Day Threats

Address emerging vulnerability categories:

**MIME Sniffing (89)**
- Definition: Browser guessing content types
- Root Cause: Missing X-Content-Type-Options
- Impact: Content spoofing, XSS
- Mitigation: X-Content-Type-Options: nosniff

**CSP Bypass (91)**
- Definition: Circumventing Content Security Policy
- Root Cause: Weak CSP configuration, unsafe-inline
- Impact: XSS execution despite CSP
- Mitigation: Strict CSP, avoid unsafe-inline, nonce/hash usage

**Inconsistent Validation (92)**
- Definition: Varying validation across application components
- Root Cause: Decentralized validation logic
- Impact: Bypass of security controls
- Mitigation: Centralized validation libraries, consistent standards

**Race Conditions (93)**
- Definition: Timing vulnerabilities in concurrent operations
- Root Cause: Missing synchronization
- Impact: Data corruption, privilege escalation
- Mitigation: Proper locking, transactional operations

**Business Logic Flaws (94-95)**
- Definition: Order processing/price manipulation vulnerabilities
- Root Cause: Insufficient business rule validation
- Impact: Financial fraud, revenue loss
- Mitigation: Server-side validation, transaction monitoring

**Account Enumeration (96)**
- Definition: Determining valid accounts through differential responses
- Root Cause: Different error messages for valid/invalid users
- Impact: Targeted attacks, credential stuffing
- Mitigation: Uniform responses, consistent timing

**Unknown/Unpatched Vulnerabilities (98-99)**
- Definition: Undiscovered or unpatched security flaws
- Root Cause: Limited visibility, patch delays
- Impact: Zero-day exploitation
- Mitigation: Security testing, patch management, threat intelligence

**Zero-Day Exploits (100)**
- Definition: Attacks on previously unknown vulnerabilities
- Root Cause: Discovery before patches available
- Impact: Unmitigated exploitation
- Mitigation: Defense in depth, anomaly detection, rapid response

---

## Quick Reference

### Vulnerability Categories Summary

| Category | Vulnerability Numbers | Key Controls |
|----------|----------------------|--------------|
| Injection | 1-13 | Parameterized queries, input validation, output encoding |
| Authentication | 14-23, 85-86 | MFA, session management, account lockout |
| Data Exposure | 24-27 | Encryption at rest/transit, access controls, DLP |
| Misconfiguration | 28-36 | Secure defaults, hardening, patching |
| XML | 37-39, 65 | Disable external entities, limit expansion |
| Access Control | 40-44 | RBAC, least privilege, authorization checks |
| Deserialization | 45-47 | Avoid untrusted data, integrity validation |
| API Security | 48-51, 75 | OAuth, rate limiting, input validation |
| Communication | 52-55 | TLS 1.2+, certificate validation, HTTPS |
| Client-Side | 56-60 | CSP, X-Frame-Options, safe DOM |
| DoS | 61-65 | Rate limiting, DDoS protection, resource limits |
| SSRF | 66, 87-88 | URL whitelisting, egress filtering |
| Mobile/IoT | 76-84 | Encryption, authentication, secure storage |
| Business Logic | 74, 92-97 | Threat modeling, logic testing |
| Zero-Day | 98-100 | Defense in depth, threat intelligence |

### Critical Security Headers

```
Content-Security-Policy: default-src 'self'; script-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=()
```

### OWASP Top 10 Mapping

| OWASP 2021 | Related Vulnerabilities |
|------------|------------------------|
| A01: Broken Access Control | 40-44, 23, 74 |
| A02: Cryptographic Failures | 24-25, 53-55 |
| A03: Injection | 1-13, 37-39 |
| A04: Insecure Design | 74, 92-97 |
| A05: Security Misconfiguration | 26-36 |
| A06: Vulnerable Components | 34, 98-100 |
| A07: Auth Failures | 14-23, 85-86 |
| A08: Data Integrity | 45-47 |
| A09: Logging Failures | 73 |
| A10: SSRF | 66, 87-88 |

---

## Constraints and Limitations

- Vulnerability definitions represent common patterns; specific implementations vary
- Mitigations must be adapted to technology stack and architecture
- New vulnerabilities emerge continuously; reference should be updated
- Some vulnerabilities overlap across categories (e.g., IDOR appears in multiple contexts)
- Effectiveness of mitigations depends on proper implementation
- Automated scanners cannot detect all vulnerability types (especially business logic)

---

## Troubleshooting

### Common Assessment Challenges

| Challenge | Solution |
|-----------|----------|
| False positives in scanning | Manual verification, contextual analysis |
| Business logic flaws missed | Manual testing, threat modeling, abuse case analysis |
| Encrypted traffic analysis | Proxy configuration, certificate installation |
| WAF blocking tests | Rate adjustment, IP rotation, payload encoding |
| Session handling issues | Cookie management, authentication state tracking |
| API discovery | Swagger/OpenAPI enumeration, traffic analysis |

### Vulnerability Verification Techniques

| Vulnerability Type | Verification Approach |
|-------------------|----------------------|
| Injection | Payload testing with encoded variants |
| XSS | Alert boxes, cookie access, DOM inspection |
| CSRF | Cross-origin form submission testing |
| SSRF | Out-of-band DNS/HTTP callbacks |
| XXE | External entity with controlled server |
| Access Control | Horizontal/vertical privilege testing |
| Authentication | Credential rotation, session analysis |

---

## References

- OWASP Top 10 Web Application Security Risks
- CWE/SANS Top 25 Most Dangerous Software Errors
- OWASP Testing Guide
- OWASP Application Security Verification Standard (ASVS)
- NIST Cybersecurity Framework
- Source: Kumar MS - Top 100 Web Vulnerabilities
