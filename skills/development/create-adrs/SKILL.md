---
name: create-adrs
description: Create Architecture Decision Records (ADRs) documenting strategic technical decisions while acknowledging ecosystem E(t) constraints. Use when choosing cloud providers, languages, frameworks, databases, or architectural patterns.
allowed-tools: [Read, Write, Edit]
---

# create-adrs

**Skill Type**: Actuator (Design Documentation)
**Purpose**: Document architecture decisions acknowledging ecosystem E(t)
**Prerequisites**: Strategic technical decision needed

---

## Agent Instructions

You are creating **Architecture Decision Records (ADRs)** that acknowledge **ecosystem E(t)** constraints.

**Critical**: ADRs document **GIVEN** constraints (E(t)), not just **CHOSEN** solutions.

**Format**: "Given E(t), we chose X" (not "we chose X")

---

## ADR Structure

### Template

```markdown
# ADR-{ID}: {Decision Title}

**Date**: {YYYY-MM-DD}
**Status**: Accepted | Rejected | Superseded | Deprecated
**Deciders**: {Who made this decision}

---

## Context

**Requirements**:
- <REQ-ID>: User authentication
- REQ-NFR-SEC-001: Secure password storage

**Problem**:
We need to choose a password hashing algorithm that is secure
against brute force attacks while maintaining acceptable performance.

**Ecosystem E(t) Constraints**:
- Team Experience: Team familiar with bcrypt
- Infrastructure: Running on AWS (bcrypt supported)
- Compliance: PCI-DSS recommends strong hashing
- Performance: Login response must be < 500ms (REQ-NFR-PERF-001)
- Libraries Available: bcrypt, argon2, scrypt available

---

## Decision

**Selected**: bcrypt with cost factor 12

**Rejected Alternatives**:
- SHA256: ❌ Too fast (vulnerable to brute force)
- MD5: ❌ Cryptographically broken
- Argon2: ⚠️ Better security but team unfamiliar, migration risk
- scrypt: ⚠️ Good security but less widely supported

**Rationale**:
1. bcrypt is industry standard (acknowledged E(t))
2. Team has experience with bcrypt (acknowledge E(t) - team capabilities)
3. Cost factor 12 balances security and performance
4. Meets PCI-DSS requirements (acknowledge E(t) - compliance)
5. Login tests show 200ms average (well within 500ms SLA)

---

## Ecosystem Constraints Acknowledged

**Team** (E(t)):
- Team knows: bcrypt, SHA256
- Team doesn't know: Argon2 implementation
- Risk: Learning curve with Argon2

**Infrastructure** (E(t)):
- Platform: AWS Lambda + RDS PostgreSQL
- bcrypt: Native support, no additional dependencies
- Argon2: Would need custom layer

**Compliance** (E(t)):
- PCI-DSS: Requires strong, salted hashing
- bcrypt: Explicitly mentioned in PCI guidelines
- Audit: External auditors familiar with bcrypt

**Performance** (E(t)):
- Requirement: Login < 500ms (REQ-NFR-PERF-001)
- bcrypt (cost 12): ~200ms (acceptable)
- Argon2: ~250ms (acceptable but no experience)

**Timeline** (E(t)):
- Sprint: 2 weeks
- bcrypt: No learning curve (immediate)
- Argon2: 2-3 days research + testing

---

## Constraints Imposed Downstream

**Code Stage** (constraints for implementation):
- MUST use bcrypt library (not custom hashing)
- MUST use cost factor 12 (not lower)
- MUST salt passwords (bcrypt does this automatically)

**Runtime Stage** (constraints for deployment):
- Infrastructure: bcrypt library must be available
- Performance monitoring: Track hash time (should be ~200ms)

**Testing Stage**:
- Test hash time (ensure < 500ms)
- Test bcrypt version (security updates)

---

## Consequences

**Positive**:
- ✅ Team productive immediately (no learning curve)
- ✅ Proven security (industry standard)
- ✅ PCI-DSS compliant (audit-friendly)
- ✅ Performance acceptable (< 500ms)

**Negative**:
- ⚠️ Not latest algorithm (Argon2 is newer)
- ⚠️ Vendor lock-in to bcrypt (migration costly)

**Neutral**:
- Cost factor 12 is balance (could be 10 for faster, 14 for stronger)

---

## Related Decisions

- ADR-001: Database selection (PostgreSQL)
- ADR-003: Session management (JWT)

**Supersedes**: None
**Superseded By**: None

---

## References

- PCI-DSS Password Requirements: https://www.pcisecuritystandards.org/
- bcrypt Documentation: https://en.wikipedia.org/wiki/Bcrypt
- OWASP Password Storage: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
```

---

## ADR Numbering

**Sequential**: ADR-001, ADR-002, ADR-003, ...

**Categories**:
- Infrastructure: Cloud provider, hosting, databases
- Security: Authentication, encryption, authorization
- Architecture: Patterns, frameworks, languages
- Integration: APIs, external services
- Data: Storage, schemas, migration

---

## When to Create ADRs

**Create ADR for**:
- ✅ Cloud provider choice (AWS vs GCP vs Azure)
- ✅ Language/framework choice (Python vs Node vs Java)
- ✅ Database choice (PostgreSQL vs MySQL vs MongoDB)
- ✅ Authentication approach (JWT vs sessions vs OAuth)
- ✅ API style (REST vs GraphQL vs gRPC)
- ✅ Architectural pattern (monolith vs microservices)

**Don't create ADR for**:
- ❌ Implementation details (already covered by code)
- ❌ Tactical decisions (variable names, file structure)
- ❌ Obvious choices (using standard library)

---

## Output Format

```
[CREATE ADR - bcrypt Password Hashing]

Decision: Use bcrypt for password hashing

Context:
  Requirements: <REQ-ID>, REQ-NFR-SEC-001
  Problem: Need secure password hashing
  Ecosystem E(t): Team knows bcrypt, PCI-DSS compliant, AWS supported

Decision:
  Selected: bcrypt (cost factor 12)
  Rejected: SHA256 (too fast), Argon2 (team unfamiliar)

Ecosystem Constraints Acknowledged:
  - Team capabilities: knows bcrypt ✓
  - Compliance: PCI-DSS requires strong hashing ✓
  - Performance: Must be < 500ms ✓
  - Infrastructure: AWS supports bcrypt ✓

Constraints Imposed:
  - Code must use bcrypt library
  - Code must use cost factor 12
  - Runtime must have bcrypt available

Created: docs/adrs/ADR-002-password-hashing.md

✅ ADR Created!
   Architecture decision documented
   Ecosystem E(t) acknowledged
   Downstream constraints clear
```

---

## Notes

**Why ADRs?**
- **Document "why"**: Explains reasoning behind decisions
- **Acknowledge E(t)**: Shows we understand ecosystem constraints
- **Prevent re-litigation**: Decision made, recorded, done
- **Onboarding**: New team members understand past decisions

**ADRs vs Constraints (C-*)**:
- **C-***: Given constraints we MUST work within
- **ADR**: Chosen solutions GIVEN those constraints

**Example**:
```
C-001: PCI-DSS requires strong password hashing (GIVEN)
ADR-002: We chose bcrypt (GIVEN C-001 and team skills)
```

**Homeostasis Goal**:
```yaml
desired_state:
  all_strategic_decisions_documented: true
  ecosystem_constraints_acknowledged: true
  rationale_clear: true
```

**"Excellence or nothing"** 🔥
