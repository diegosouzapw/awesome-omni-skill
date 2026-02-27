---
name: extract-constraints
description: Extract technical constraints (C-*) from ecosystem E(t) - timeouts, API limits, compliance requirements, platform dependencies. Acknowledges given constraints, not design choices. Use to document ecosystem realities that code must work within.
allowed-tools: [Read, Write, Edit]
---

# extract-constraints

**Skill Type**: Actuator (Requirements Disambiguation)
**Purpose**: Extract C-* constraints acknowledging ecosystem E(t)
**Prerequisites**: REQ-* requirement exists

---

## Agent Instructions

You are extracting **constraints** (C-*) from the **ecosystem E(t)**.

**Constraints are GIVEN** (external reality), not **CHOSEN** (design decisions).

**Examples**:
- ✅ C-001: Stripe API timeout is 10 seconds (given by Stripe)
- ❌ C-002: We will use PostgreSQL (this is a design choice → ADR)

**Goal**: Acknowledge ecosystem constraints that code must work within.

---

## C-* Categories

### 1. API/Service Constraints

**Timeouts**:
```yaml
C-001: Stripe API timeout
  - Value: 10 seconds (given by Stripe documentation)
  - Source: https://stripe.com/docs/api#timeouts
  - Fallback: Return error to user
  - Monitoring: Alert if approaching timeout (>8s)
```

**Rate limits**:
```yaml
C-010: Stripe API rate limit
  - Limit: 100 requests per second
  - Source: Stripe API documentation
  - Behavior: Implement exponential backoff
  - Error: "Payment service temporarily unavailable"
```

---

### 2. Compliance Constraints

**PCI-DSS**:
```yaml
C-020: PCI-DSS Level 1 compliance
  - Requirement: Never store full credit card numbers
  - Implementation: Tokenize via Stripe
  - Validation: No card numbers in logs or database
  - Audit: Regular PCI scans required
```

**GDPR**:
```yaml
C-030: GDPR data portability
  - Requirement: Users can export their data
  - Format: JSON or CSV
  - Timeline: Within 30 days of request
  - Scope: All personal data
```

---

### 3. Platform Constraints

**Language/runtime**:
```yaml
C-040: Python version
  - Minimum: Python 3.8
  - Reason: Using dataclasses, type hints
  - Source: Project decision (E(t) = team knows Python)
  - Validation: Check at startup
```

**Dependencies**:
```yaml
C-050: bcrypt library
  - Library: bcrypt
  - Version: >=4.0.0
  - Reason: Password hashing (security requirement)
  - Source: Security team standard
```

---

### 4. Performance Constraints

**Response time SLAs**:
```yaml
C-060: Login response time
  - Max time: 500ms (p95)
  - Measurement: End-to-end from request to response
  - Monitoring: Datadog APM
  - Alerting: >400ms warning, >500ms critical
```

**Resource limits**:
```yaml
C-070: Database connection pool
  - Max connections: 20
  - Source: RDS instance limit
  - Behavior: Queue requests if pool exhausted
  - Timeout: 5 seconds wait for connection
```

---

### 5. Security Constraints

**Authentication**:
```yaml
C-080: HTTPS requirement
  - Protocol: All auth requests must be HTTPS
  - Source: Security policy
  - Enforcement: Reject HTTP requests
  - Exception: None (no HTTP fallback)
```

**Session management**:
```yaml
C-090: Session timeout
  - Duration: 30 minutes of inactivity
  - Source: Security team policy
  - Token: JWT with exp claim
  - Storage: Redis with TTL
```

---

## Constraint vs Design Decision

**Constraint (C-*)** - **GIVEN** by ecosystem:
```yaml
✅ C-001: Stripe API timeout is 10 seconds
   Source: Stripe documentation (external reality)
   We MUST work within this constraint

✅ C-002: PCI-DSS prohibits storing card numbers
   Source: Payment Card Industry regulation
   We MUST comply

✅ C-003: Team knows Python, not Java
   Source: Team capabilities (E(t))
   We work within this reality
```

**Design Decision** - **CHOSEN** by us:
```yaml
❌ "We will use PostgreSQL" → This is ADR, not C-*
❌ "We will implement REST API" → This is ADR, not C-*
❌ "We will use MVC pattern" → This is ADR, not C-*
```

**Rule**: If it's a choice between alternatives → **ADR** (Design stage)
If it's a given reality we must work within → **C-*** (Constraint)

---

## Extraction Process

### Step 1: Identify Ecosystem Sources

**Ask**:
1. What external APIs/services are we using? (Stripe, Auth0, AWS)
2. What compliance requirements apply? (PCI-DSS, GDPR, HIPAA)
3. What platform constraints exist? (language version, libraries)
4. What performance SLAs? (response time, uptime)
5. What security policies? (HTTPS, encryption, session timeout)
6. What team constraints? (skills, preferences, existing systems)

---

### Step 2: Document Each Constraint

**Template**:
```yaml
C-{ID}: {Constraint Name}
  - Value/Requirement: {What is constrained}
  - Source: {Where this constraint comes from - API docs, regulation, policy}
  - Ecosystem E(t): {What ecosystem component imposes this}
  - Implementation: {How code must handle this}
  - Validation: {How to verify compliance}
  - Autogenerate: {Yes/No}
```

---

## Output Format

```
[EXTRACT CONSTRAINTS - <REQ-ID>]

Requirement: User login

Constraints Extracted:

API/Service Constraints (2):
  ✓ C-001: Database query timeout (100ms, RDS limit)
  ✓ C-002: Session storage (Redis, existing infrastructure)

Compliance Constraints (2):
  ✓ C-003: HTTPS required (security policy)
  ✓ C-004: Password hashing (bcrypt, security standard)

Performance Constraints (1):
  ✓ C-005: Login response time (<500ms, SLA requirement)

Total: 5 constraints

Ecosystem E(t) Acknowledged:
  - External: RDS connection limits, Redis infrastructure
  - Compliance: Security policies (HTTPS, bcrypt)
  - Performance: SLA requirements

Updated: docs/requirements/authentication.md
  Added: Constraints section with 5 C-*

✅ Constraint Extraction Complete!
```

---

## Notes

**Why extract constraints?**
- **Acknowledge ecosystem E(t)**: Document external realities
- **Design context**: Constraints inform ADRs (architecture decisions)
- **Code generation**: Some constraints can be autogenerated (timeout checks)
- **Compliance**: Document regulatory requirements

**Homeostasis Goal**:
```yaml
desired_state:
  all_ecosystem_constraints_documented: true
  constraints_vs_choices_clear: true
```

**"Excellence or nothing"** 🔥
