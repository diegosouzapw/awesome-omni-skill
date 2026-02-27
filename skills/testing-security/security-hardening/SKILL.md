---
name: security-hardening
description: Comprehensive security implementation covering authentication, authorization, input validation, vulnerability detection, compliance, and security standards (OWASP, ISO 27001, SOC2, CWE). Use when securing applications, APIs, and infrastructure.
license: Apache-2.0
compatibility: Python 3.10+, FastAPI, OAuth2, Docker, Kubernetes
metadata:
  author: paracle-core-team
  version: "2.0.0"
  category: security
  level: advanced
  display_name: "Security Hardening"
  tags:
    - security
    - authentication
    - authorization
    - validation
    - owasp
    - iso27001
    - soc2
    - compliance
    - vulnerability
    - sast
    - dast
    - sbom
  capabilities:
    - authentication_implementation
    - authorization_policies
    - input_validation
    - security_testing
    - vulnerability_scanning
    - compliance_checking
    - threat_modeling
    - secret_management
    - container_security
    - supply_chain_security
  standards:
    - OWASP_Top_10
    - OWASP_ASVS
    - CWE_Top_25
    - ISO_27001
    - ISO_42001
    - SOC2_Type_II
    - NIST_CSF
    - GDPR
    - SLSA
  tools:
    - bandit
    - semgrep
    - codeql
    - sonarqube
    - safety
    - trivy
    - snyk
    - gitleaks
    - checkov
  requirements:
    - skill_name: paracle-development
      min_level: advanced
allowed-tools: Read Write Bash(python:*) Bash(bandit:*) Bash(safety:*) Bash(semgrep:*) Bash(trivy:*) Bash(pip-audit:*)
---

# Security Hardening Skill

## When to use this skill

Use when:
- Implementing authentication
- Setting up authorization
- Validating user input
- Preventing security vulnerabilities
- Securing API endpoints

## Authentication

```python
# JWT-based authentication
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import jwt, JWTError

security = HTTPBearer()

SECRET_KEY = os.getenv("JWT_SECRET_KEY")
ALGORITHM = "HS256"

def create_access_token(user_id: str) -> str:
    payload = {
        "sub": user_id,
        "exp": datetime.utcnow() + timedelta(hours=24),
        "iat": datetime.utcnow(),
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
) -> User:
    token = credentials.credentials

    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id = payload.get("sub")
        if user_id is None:
            raise HTTPException(status_code=401, detail="Invalid token")
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")

    user = await get_user_by_id(user_id)
    if user is None:
        raise HTTPException(status_code=401, detail="User not found")

    return user

# Use in endpoints
@app.get("/agents")
async def list_agents(user: User = Depends(get_current_user)):
    # User is authenticated
    return await fetch_user_agents(user.id)
```

## Authorization (RBAC)

```python
# Role-Based Access Control
from enum import Enum

class Role(str, Enum):
    ADMIN = "admin"
    USER = "user"
    VIEWER = "viewer"

class Permission(str, Enum):
    CREATE_AGENT = "create:agent"
    READ_AGENT = "read:agent"
    UPDATE_AGENT = "update:agent"
    DELETE_AGENT = "delete:agent"

ROLE_PERMISSIONS = {
    Role.ADMIN: [
        Permission.CREATE_AGENT,
        Permission.READ_AGENT,
        Permission.UPDATE_AGENT,
        Permission.DELETE_AGENT,
    ],
    Role.USER: [
        Permission.CREATE_AGENT,
        Permission.READ_AGENT,
        Permission.UPDATE_AGENT,
    ],
    Role.VIEWER: [
        Permission.READ_AGENT,
    ],
}

def require_permission(permission: Permission):
    async def check_permission(user: User = Depends(get_current_user)):
        if permission not in ROLE_PERMISSIONS.get(user.role, []):
            raise HTTPException(
                status_code=403,
                detail=f"Permission denied: {permission}",
            )
        return user

    return check_permission

# Use in endpoints
@app.delete("/agents/{agent_id}")
async def delete_agent(
    agent_id: str,
    user: User = Depends(require_permission(Permission.DELETE_AGENT)),
):
    return await delete_agent_service(agent_id, user)
```

## Input Validation

```python
# Use Pydantic for validation
from pydantic import BaseModel, Field, validator

class CreateAgentRequest(BaseModel):
    name: str = Field(..., min_length=1, max_length=100, regex="^[a-z0-9-]+$")
    model: str = Field(..., regex="^(gpt-4|claude-3).*$")
    temperature: float = Field(default=0.7, ge=0.0, le=2.0)
    system_prompt: str = Field(..., max_length=10000)

    @validator("name")
    def validate_name(cls, v):
        # Additional validation
        if v in ["admin", "root", "system"]:
            raise ValueError("Reserved name")
        return v

# SQL Injection Prevention
from sqlalchemy import select, text

# Bad: SQL injection vulnerable
query = f"SELECT * FROM agents WHERE name = '{user_input}'"  # DON'T DO THIS

# Good: Use parameterized queries
query = select(Agent).where(Agent.name == user_input)  # Safe

# Or with raw SQL (use params)
query = text("SELECT * FROM agents WHERE name = :name")
result = await session.execute(query, {"name": user_input})  # Safe
```

## Rate Limiting

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

@app.post("/agents")
@limiter.limit("10/minute")  # Max 10 requests per minute
async def create_agent(
    request: Request,
    data: CreateAgentRequest,
    user: User = Depends(get_current_user),
):
    return await create_agent_service(data, user)
```

## Secret Management

```python
# Don't hardcode secrets
# Bad
API_KEY = "sk-1234567890abcdef"  # DON'T DO THIS

# Good: Use environment variables
import os
API_KEY = os.getenv("OPENAI_API_KEY")
if not API_KEY:
    raise ValueError("OPENAI_API_KEY not set")

# Better: Use secret management service
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()
client = SecretClient(vault_url="https://my-vault.vault.azure.net", credential=credential)
API_KEY = client.get_secret("openai-api-key").value
```

## CORS Configuration

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://app.example.com",  # Specific origins only
        # Don't use "*" in production
    ],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["*"],
    max_age=3600,
)
```

## Security Headers

```python
@app.middleware("http")
async def add_security_headers(request: Request, call_next):
    response = await call_next(request)
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["X-XSS-Protection"] = "1; mode=block"
    response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
    return response
```

## Error Handling

```python
# Don't leak sensitive info in errors
@app.exception_handler(Exception)
async def generic_exception_handler(request: Request, exc: Exception):
    # Log full error for debugging
    logger.error(f"Unexpected error: {exc}", exc_info=True)

    # Return generic message to user
    return JSONResponse(
        status_code=500,
        content={"detail": "Internal server error"},  # Don't expose stack trace
    )

# Validate error messages don't expose system info
class AgentNotFoundError(HTTPException):
    def __init__(self, agent_id: str):
        # Don't include internal IDs or paths
        super().__init__(
            status_code=404,
            detail="Agent not found",  # Generic message
        )
        # Log details server-side only
        logger.warning(f"Agent not found: {agent_id}")
```

## Security Testing

```python
# Test authentication
def test_requires_authentication():
    response = client.get("/agents")
    assert response.status_code == 401

# Test authorization
def test_viewer_cannot_delete():
    token = create_token_for_role(Role.VIEWER)
    response = client.delete(
        "/agents/123",
        headers={"Authorization": f"Bearer {token}"},
    )
    assert response.status_code == 403

# Test input validation
def test_rejects_invalid_agent_name():
    response = client.post(
        "/agents",
        json={"name": "invalid name!"},  # Spaces and ! not allowed
    )
    assert response.status_code == 422
```

## Best Practices

1. **Never trust user input** - Validate everything
2. **Use parameterized queries** - Prevent SQL injection
3. **Implement rate limiting** - Prevent abuse
4. **Store secrets securely** - Never in code
5. **Use HTTPS only** - Encrypt in transit
6. **Add security headers** - Prevent common attacks
7. **Log security events** - Monitor for attacks
8. **Keep dependencies updated** - Patch vulnerabilities

---

## Security Standards Reference

### OWASP Top 10 (2021)

| ID | Vulnerability | Prevention |
|----|--------------|------------|
| A01 | Broken Access Control | RBAC, least privilege, deny by default |
| A02 | Cryptographic Failures | TLS 1.3, strong hashing (bcrypt/Argon2), key rotation |
| A03 | Injection | Parameterized queries, input validation, ORM |
| A04 | Insecure Design | Threat modeling, secure design patterns |
| A05 | Security Misconfiguration | Hardened configs, disable defaults, CSP |
| A06 | Vulnerable Components | SCA scanning (Safety, Snyk), SBOM |
| A07 | Auth Failures | MFA, session management, secure password storage |
| A08 | Software/Data Integrity | Code signing, dependency verification, SLSA |
| A09 | Security Logging | Audit trails, SIEM integration, alerting |
| A10 | SSRF | URL validation, allowlists, network segmentation |

### CWE Top 25 (Most Dangerous)

```python
# CWE-79: Cross-site Scripting (XSS)
from markupsafe import escape
safe_output = escape(user_input)

# CWE-89: SQL Injection
query = select(User).where(User.id == user_id)  # Use ORM

# CWE-787: Out-of-bounds Write
# Use memory-safe languages/bounds checking

# CWE-20: Improper Input Validation
from pydantic import BaseModel, validator, constr

class SafeInput(BaseModel):
    name: constr(min_length=1, max_length=100, regex="^[a-zA-Z0-9_-]+$")

# CWE-125: Out-of-bounds Read
# Validate array indices before access

# CWE-22: Path Traversal
from pathlib import Path
safe_path = Path(base_dir) / Path(user_path).name  # Sanitize

# CWE-352: CSRF
from fastapi_csrf_protect import CsrfProtect
```

### ISO 27001 Controls

```yaml
# Key security controls for AI systems
A.5: Information Security Policies
A.6: Organization of Information Security
A.7: Human Resource Security
A.8: Asset Management
A.9: Access Control
A.10: Cryptography
A.12: Operations Security
A.13: Communications Security
A.14: System Acquisition, Development, Maintenance
A.16: Information Security Incident Management
A.18: Compliance
```

### ISO 42001 (AI-Specific)

```yaml
# AI Management System requirements
4.1: Understanding the organization context
5.1: Leadership and commitment to AI ethics
6.1: Risk assessment for AI systems
7.1: Support and resources for AI governance
8.1: AI development lifecycle controls
9.1: Performance evaluation of AI systems
10.1: Continual improvement of AI management
```

---

## Security Scanning Tools

### Static Application Security Testing (SAST)

```bash
# Bandit - Python security linter
bandit -r packages/ -f json -o bandit-report.json

# Semgrep - Pattern-based analysis
semgrep --config=p/owasp-top-ten --json -o semgrep-report.json packages/

# SonarQube - Continuous inspection
sonar-scanner \
  -Dsonar.projectKey=paracle \
  -Dsonar.sources=packages/ \
  -Dsonar.python.coverage.reportPaths=coverage.xml

# CodeQL - GitHub semantic analysis
codeql database create paracle-db --language=python
codeql database analyze paracle-db python-security-extended --format=sarif-latest
```

### Software Composition Analysis (SCA)

```bash
# Safety - PyPI vulnerability checker
safety check --json > safety-report.json

# pip-audit - Python package auditing
pip-audit --format json --output pip-audit-report.json

# Snyk - Comprehensive SCA
snyk test --json > snyk-report.json

# Trivy - Multi-scanner
trivy fs --security-checks vuln,secret,config .

# OSSF Scorecard - Supply chain security
scorecard --repo=github.com/user/paracle --format=json
```

### Secret Detection

```bash
# Gitleaks - Git history scanning
gitleaks detect --source . --report-format json --report-path gitleaks.json

# detect-secrets - Yelp's secret scanner
detect-secrets scan --all-files > .secrets.baseline

# TruffleHog - Credential scanner
trufflehog git file://. --json > trufflehog-report.json
```

### Container Security

```bash
# Trivy container scanning
trivy image paracle:latest --format json -o trivy-image.json

# Grype - SBOM-based scanning
grype paracle:latest -o json > grype-report.json

# Syft - SBOM generation (CycloneDX, SPDX)
syft paracle:latest -o cyclonedx-json > sbom.json

# Checkov - IaC security
checkov -d docker/ --framework dockerfile -o json
```

### Dynamic Application Security Testing (DAST)

```bash
# OWASP ZAP - Web app scanner
zap-cli quick-scan --self-contained http://localhost:8000 -r zap-report.html

# Nuclei - Template-based scanning
nuclei -u http://localhost:8000 -t cves/ -o nuclei-report.json
```

---

## Compliance Frameworks

### SOC2 Type II Controls

```python
# Trust Service Criteria implementation
class SOC2Controls:
    """SOC2 Type II control mapping."""

    # Security (CC6.1)
    LOGICAL_ACCESS = "CC6.1"  # RBAC implementation

    # Availability (A1.1)
    AVAILABILITY = "A1.1"  # Rate limiting, circuit breakers

    # Processing Integrity (PI1.1)
    INTEGRITY = "PI1.1"  # Input validation, checksums

    # Confidentiality (C1.1)
    CONFIDENTIALITY = "C1.1"  # Encryption, access controls

    # Privacy (P1.1)
    PRIVACY = "P1.1"  # Data minimization, consent
```

### GDPR Requirements

```python
# Privacy by Design implementation
from dataclasses import dataclass
from typing import Optional
from datetime import datetime

@dataclass
class GDPRCompliantData:
    """GDPR Article 25 - Data Protection by Design."""

    # Data minimization (Art. 5(1)(c))
    necessary_fields_only: bool = True

    # Purpose limitation (Art. 5(1)(b))
    processing_purpose: str = ""

    # Storage limitation (Art. 5(1)(e))
    retention_period_days: int = 365

    # Right to erasure (Art. 17)
    deletion_date: Optional[datetime] = None

    # Right to portability (Art. 20)
    export_format: str = "json"
```

### SLSA (Supply-chain Levels)

```yaml
# SLSA Level 3 requirements
slsa_level: 3
requirements:
  source:
    - version_controlled: true
    - verified_history: true
    - two_person_reviewed: true
  build:
    - build_service: GitHub Actions
    - hermetic: true
    - reproducible: true
  provenance:
    - signed: true
    - non_falsifiable: true
    - dependencies_complete: true
```

---

## Threat Modeling

### STRIDE Analysis

```python
# STRIDE threat categories
class ThreatCategory(Enum):
    SPOOFING = "Identity spoofing"        # Auth bypass
    TAMPERING = "Data tampering"          # Integrity violation
    REPUDIATION = "Repudiation"           # Non-attribution
    INFORMATION_DISCLOSURE = "Info leak"  # Confidentiality breach
    DENIAL_OF_SERVICE = "DoS"             # Availability impact
    ELEVATION_OF_PRIVILEGE = "Privilege"  # Authorization bypass

# Threat model template
threat_model = {
    "asset": "Agent execution",
    "threats": [
        {
            "category": ThreatCategory.ELEVATION_OF_PRIVILEGE,
            "description": "Agent escapes sandbox",
            "likelihood": "Medium",
            "impact": "High",
            "mitigations": [
                "Docker isolation",
                "seccomp profiles",
                "capability dropping"
            ]
        }
    ]
}
```

### DREAD Risk Scoring

```python
def calculate_dread_score(
    damage: int,        # 1-10: How bad is an exploit?
    reproducibility: int,  # 1-10: How easy to reproduce?
    exploitability: int,   # 1-10: How easy to exploit?
    affected_users: int,   # 1-10: How many affected?
    discoverability: int   # 1-10: How easy to discover?
) -> float:
    """Calculate DREAD risk score (0-10)."""
    return (damage + reproducibility + exploitability +
            affected_users + discoverability) / 5
```

---

## Security Checklist

### Pre-Development
- [ ] Threat model created (STRIDE/DREAD)
- [ ] Security requirements defined
- [ ] Secure architecture reviewed

### Development
- [ ] Authentication implemented (JWT/OAuth2)
- [ ] Authorization rules defined (RBAC/ABAC)
- [ ] Input validation with Pydantic
- [ ] SQL injection prevention (ORM/parameterized)
- [ ] XSS prevention (output encoding)
- [ ] CSRF protection enabled
- [ ] Rate limiting on endpoints
- [ ] Secrets in environment/vault
- [ ] HTTPS enforced
- [ ] CORS configured properly
- [ ] Security headers added
- [ ] Error messages sanitized

### Testing
- [ ] SAST scan passed (Bandit, Semgrep)
- [ ] SCA scan passed (Safety, Snyk)
- [ ] Secret scan passed (Gitleaks)
- [ ] Container scan passed (Trivy)
- [ ] DAST scan passed (ZAP)
- [ ] Penetration testing completed
- [ ] Security unit tests written

### Deployment
- [ ] SBOM generated
- [ ] Container hardened (non-root, read-only)
- [ ] Network policies applied
- [ ] Logging/monitoring enabled
- [ ] Incident response plan ready

### Compliance
- [ ] OWASP Top 10 addressed
- [ ] CWE Top 25 mitigated
- [ ] ISO 27001 controls mapped
- [ ] SOC2 evidence collected
- [ ] GDPR requirements met
- [ ] SLSA provenance generated

---

## Resources

### Standards
- OWASP Top 10: https://owasp.org/www-project-top-ten/
- OWASP ASVS: https://owasp.org/www-project-application-security-verification-standard/
- CWE Top 25: https://cwe.mitre.org/top25/
- NIST CSF: https://www.nist.gov/cyberframework
- ISO 27001: https://www.iso.org/isoiec-27001-information-security.html
- ISO 42001: https://www.iso.org/standard/81230.html
- SOC2: https://www.aicpa.org/soc2
- SLSA: https://slsa.dev/

### Tools
- Bandit: https://bandit.readthedocs.io/
- Semgrep: https://semgrep.dev/
- CodeQL: https://codeql.github.com/
- SonarQube: https://www.sonarqube.org/
- Trivy: https://trivy.dev/
- Snyk: https://snyk.io/
- OWASP ZAP: https://www.zaproxy.org/

### Paracle Docs
- Security Guide: `docs/security-agent.md`
- Compliance Guide: `docs/compliance-guide.md`
- Audit Guide: `docs/audit-guide.md`
