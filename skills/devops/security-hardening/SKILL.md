---
name: security-hardening
description: Security hardening checklists for applications, containers, CI/CD, and infrastructure. Use this skill when reviewing security posture, setting up new services, or responding to security concerns.
alwaysApply: false
---

# Security Hardening

You are a security engineer. Apply these hardening checklists when reviewing or setting up applications, containers, and infrastructure.

## Application Security Checklist

### Authentication
- [ ] Passwords hashed with bcrypt/scrypt/argon2 (never MD5/SHA1)
- [ ] Rate limiting on login endpoints (e.g., 5 attempts per minute)
- [ ] Account lockout or CAPTCHA after failed attempts
- [ ] Session tokens are random, unpredictable, and expire
- [ ] Multi-factor authentication available for sensitive operations
- [ ] OAuth tokens stored securely (httpOnly cookies, not localStorage)
- [ ] JWT tokens have reasonable expiry (15 min access, 7 day refresh)

### Input Validation
- [ ] All user input validated on the server side (never trust client-only validation)
- [ ] SQL parameterized queries / prepared statements (never string concatenation)
- [ ] HTML output escaped to prevent XSS (use framework auto-escaping)
- [ ] File uploads validated (type, size, content) and stored outside web root
- [ ] API rate limiting per endpoint and per user
- [ ] Request size limits configured

### Secrets Management
- [ ] No secrets in source code (`.env` files in `.gitignore`)
- [ ] No secrets in Docker images (use runtime env vars or secrets manager)
- [ ] No secrets in CI/CD logs (mask sensitive outputs)
- [ ] Secrets rotated periodically
- [ ] Different secrets per environment (dev/staging/prod)
- [ ] Use a secrets manager (Vault, AWS Secrets Manager, 1Password)

### HTTP Security Headers
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 0
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

### API Security
- [ ] Authentication required on all non-public endpoints
- [ ] Authorization checks (user A can't access user B's data)
- [ ] CORS configured restrictively (specific origins, not `*`)
- [ ] Rate limiting per API key / user
- [ ] Input validation with schema (Zod, JSON Schema)
- [ ] Error messages don't leak internal details
- [ ] API versioning for breaking changes

## Container Security

### Dockerfile
- [ ] Non-root user (`USER node`, `USER nobody`)
- [ ] Minimal base image (Alpine, Distroless)
- [ ] No secrets in build args or ENV
- [ ] Image scanned for vulnerabilities (Trivy, Scout)
- [ ] Specific version tags (not `latest`)
- [ ] Read-only filesystem where possible

### Runtime
```yaml
# Kubernetes SecurityContext
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
```

### Network
- [ ] Network policies restrict pod-to-pod communication
- [ ] Ingress controller with TLS termination
- [ ] Internal services not exposed externally
- [ ] Service mesh for mTLS between services (Istio, Linkerd)

## CI/CD Security

### Pipeline
- [ ] Secrets stored in CI platform secrets manager (not in config files)
- [ ] Action versions pinned by SHA (not tag)
- [ ] PR pipelines don't have access to production secrets
- [ ] Dependency scanning in pipeline (Dependabot, Snyk)
- [ ] SAST (Static Analysis) in pipeline
- [ ] Container image scanning before push
- [ ] Required reviews before merge to main

### Supply Chain
- [ ] Lock files committed (`package-lock.json`, `go.sum`)
- [ ] Dependency updates reviewed (not auto-merged)
- [ ] Private package registry for internal packages
- [ ] Signature verification for critical dependencies
- [ ] SBOM (Software Bill of Materials) generated

## Dependency Security

### Scanning
```bash
# Node.js
npm audit
npm audit fix

# Go
govulncheck ./...

# Python
pip-audit
safety check

# Docker
trivy image myapp:latest
docker scout cves myapp:latest
```

### Policy
- [ ] High/critical vulnerabilities fixed within 48 hours
- [ ] Medium vulnerabilities fixed within 2 weeks
- [ ] Automated PR for dependency updates (Dependabot, Renovate)
- [ ] No dependencies with known exploits in production

## OWASP Top 10 Quick Reference

| Risk | Prevention |
|------|-----------|
| **Injection** | Parameterized queries, input validation, ORMs |
| **Broken Auth** | Strong passwords, MFA, session management |
| **Sensitive Data Exposure** | TLS everywhere, encrypt at rest, minimize data collection |
| **XXE** | Disable XML external entities, use JSON |
| **Broken Access Control** | Server-side authz checks on every request |
| **Security Misconfiguration** | Hardened defaults, remove debug endpoints, update frameworks |
| **XSS** | Auto-escaping templates, Content-Security-Policy header |
| **Insecure Deserialization** | Validate input types, avoid native serialization |
| **Using Vulnerable Components** | Regular dependency audits, automated scanning |
| **Insufficient Logging** | Log auth events, failures, access anomalies |

## Quick Wins (Do These First)
1. **Enable HTTPS everywhere** — no exceptions, use HSTS
2. **Rotate default credentials** — database, admin panels, API keys
3. **Enable dependency scanning** — Dependabot or Renovate
4. **Add security headers** — CSP, HSTS, X-Frame-Options
5. **Restrict CORS** — specific origins, not wildcard
6. **Enable audit logging** — who did what, when
7. **Run as non-root** — containers, services, CI jobs
