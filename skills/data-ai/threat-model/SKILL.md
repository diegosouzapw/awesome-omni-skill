---
name: threat-model
description: "Threat modeling methodology and risk assessment process. Use when designing new features, reviewing architecture for security, performing STRIDE analysis, creating attack trees, or assessing risk with CVSS/DREAD. Also use when authentication/authorization is added, data flows cross trust boundaries, third-party integrations are introduced, sensitive data handling changes, or analyzing security incidents. Essential for data flow diagrams and security design reviews."
---

# Threat Model Skill

## Overview

Threat modeling finds security issues before code is written. This skill guides systematic identification of threats through structured analysis—because finding vulnerabilities in design is 100x cheaper than finding them in production.

**Core principle:** Think like an attacker, document like an engineer. Threat modeling isn't about paranoia—it's about systematic analysis of what could go wrong.

## The Threat Modeling Process

### Phase 1: Understand the System

**Before identifying threats:**

1. **Define Scope**
   - What are we modeling?
   - What's in scope / out of scope?
   - What are the security requirements?

2. **Create Data Flow Diagram**
   - What are the components?
   - How does data flow between them?
   - Where are the trust boundaries?

3. **Identify Assets**
   - What data matters?
   - What functionality matters?
   - What would an attacker want?

### Phase 2: Identify Threats

**Systematically find threats:**

1. **Apply STRIDE to Each Component**
   - For each element in the DFD
   - Consider each STRIDE category
   - Document potential threats

2. **Consider Attack Paths**
   - How could an attacker reach this?
   - What would they need to exploit it?
   - What's the blast radius?

3. **Document Assumptions**
   - What are we assuming is secure?
   - What are we trusting?
   - What could invalidate assumptions?

### Phase 3: Prioritize and Mitigate

**Then address what matters:**

1. **Score Risks**
   - Use DREAD or CVSS consistently
   - Consider both impact and likelihood
   - Factor in existing controls

2. **Identify Mitigations**
   - How do we prevent this threat?
   - How do we detect if it happens?
   - How do we respond?

3. **Track to Resolution**
   - Create actionable tickets
   - Assign ownership
   - Verify mitigations implemented

## STRIDE Framework

**Apply to each component in your data flow diagram:**

| Threat | Property Violated | Questions to Ask |
|--------|-------------------|------------------|
| **S**poofing | Authentication | Can someone pretend to be someone else? |
| **T**ampering | Integrity | Can data be modified without detection? |
| **R**epudiation | Non-repudiation | Can someone deny they did something? |
| **I**nformation Disclosure | Confidentiality | Can sensitive data leak? |
| **D**enial of Service | Availability | Can the system be made unavailable? |
| **E**levation of Privilege | Authorization | Can someone gain unauthorized access? |

## Red Flags - STOP and Analyze

### Architecture Red Flags

```
❌ CRITICAL: No authentication between services
   "Internal services trust each other"
   → Lateral movement after any compromise

❌ CRITICAL: Secrets in code or config
   "API keys are in the codebase"
   → Anyone with repo access has keys

❌ HIGH: No input validation at trust boundary
   "We trust data from the mobile app"
   → Mobile app can be reverse-engineered

❌ HIGH: Logging sensitive data
   "We log requests for debugging"
   → Logs become attack target

❌ MEDIUM: No rate limiting
   "We don't expect much traffic"
   → Brute force attacks enabled
```

### Data Flow Red Flags

```
❌ CRITICAL: Sensitive data over unencrypted channel
   "It's internal network only"
   → Insiders can sniff traffic

❌ HIGH: No encryption at rest for PII
   "Database is behind firewall"
   → Backup tapes, logs, replicas exposed

❌ HIGH: Cross-origin data access
   "Frontend needs to call multiple APIs"
   → CORS misconfiguration risks

❌ MEDIUM: Storing more data than needed
   "We might need it later"
   → Larger attack surface, compliance risk
```

## Common Rationalizations - Don't Accept These

| Excuse | Reality |
|--------|---------|
| "It's behind the firewall" | Firewalls get bypassed. Defense in depth. |
| "Only admins have access" | Admin credentials get stolen. Limit blast radius. |
| "We trust our employees" | Insider threat is real. Verify, don't trust. |
| "It's low priority data" | Low-priority breaches make news too. Protect it. |
| "We'll add security later" | Security debt compounds. Build it in. |
| "No one would attack us" | Automated attacks don't discriminate. |

## Threat Model Quality Checklist

Before finalizing threat model:

**Scope:**
- [ ] Clear boundaries defined
- [ ] Security requirements documented
- [ ] Assumptions listed

**Analysis:**
- [ ] Data flow diagram complete
- [ ] Trust boundaries identified
- [ ] STRIDE applied to each component
- [ ] Attack paths considered

**Prioritization:**
- [ ] Risks scored consistently
- [ ] Mitigations identified
- [ ] Residual risk documented

**Tracking:**
- [ ] Threats tracked as tickets
- [ ] Owners assigned
- [ ] Review scheduled

## Quick Patterns

### STRIDE Analysis Template

```markdown
## STRIDE Analysis: [Component Name]

**Component Description:** [What it does]
**Data Handled:** [What data flows through it]
**Trust Level:** [Who can access it]

### Spoofing
- **Threat:** [Description]
- **Likelihood:** [Low/Medium/High]
- **Impact:** [Low/Medium/High]
- **Mitigation:** [How to prevent]

### Tampering
- **Threat:** [Description]
- **Mitigation:** [How to prevent]

[Continue for R, I, D, E...]
```

### DREAD Risk Scoring

| Factor | 1 (Low) | 2 (Medium) | 3 (High) |
|--------|---------|------------|----------|
| **D**amage | Minor inconvenience | Service disruption | Data breach |
| **R**eproducibility | Complex conditions | Sometimes works | Reliable exploit |
| **E**xploitability | Expert knowledge | Some skill needed | Script kiddie |
| **A**ffected Users | Single user | Subset of users | All users |
| **D**iscoverability | Requires insider | Can find by probing | Publicly known |

**Score:** (D + R + E + A + D) / 5
- **2.5 - 3.0:** Critical - Fix immediately
- **1.5 - 2.4:** High - Fix soon
- **1.0 - 1.4:** Low - Fix eventually

### Abuser Story Template

```markdown
**As a** [malicious actor type],
**I want to** [malicious action],
**So that** [attacker's goal].

**Attack Vector:** [How they would do it]
**Prerequisites:** [What they need]
**Impact:** [What happens if successful]
```

**Example:**
> **As a** external attacker,
> **I want to** inject SQL into the login form,
> **So that** I can dump the user database.
>
> **Attack Vector:** Submit `' OR '1'='1` in username field
> **Prerequisites:** Knowledge of SQL injection, access to login page
> **Impact:** Full database compromise, credential theft

### Data Flow Diagram Elements

```
┌─────────────┐
│   Process   │  ○ Application logic (trust boundary)
└─────────────┘

╔═════════════╗
║  Data Store ║  ═ Database, file system
╚═════════════╝

┌ ─ ─ ─ ─ ─ ─ ┐
  External     │  □ User, third-party system
│  Entity
└ ─ ─ ─ ─ ─ ─ ┘

     ───────►     → Data flow (label with data type)

┊ ┊ ┊ ┊ ┊ ┊ ┊    ┊ Trust boundary (networks, privilege levels)
```

## Quick Commands

```bash
# Generate diagram from code
# (Use draw.io, Mermaid, or Threat Dragon)

# Automated threat analysis
pip install pytm  # Python threat modeling
threatspec         # Threat modeling as code

# Track findings
# Create issues in your tracker with:
# - Threat description
# - DREAD score
# - Mitigation plan
# - Owner
# - Target date
```

## References

Detailed patterns and examples in `references/`:
- `stride-examples.md` - STRIDE analysis walkthroughs
- `attack-patterns.md` - Common attack patterns by component
- `mitigation-catalog.md` - Standard mitigations for common threats
