---
name: threat-model-generation
description: Generate a STRIDE-based security threat model for a repository. Use when setting up security monitoring, after architecture changes, or for security audits.
version: 1.0.0
tags: [security, threat-model, stride]
---

# Threat Model Generation

Generate a comprehensive security threat model for a repository using the STRIDE methodology.

## When to Use This Skill

- **First-time setup** - New repository needs initial threat model
- **Architecture changes** - Significant changes to components, APIs, or data flows
- **Security audit** - Periodic review or compliance requirement
- **Manual request** - Security team requests updated threat model

## Inputs

| Input                   | Description                                             | Required                         |
| ----------------------- | ------------------------------------------------------- | -------------------------------- |
| Repository path         | Root directory to analyze                               | Yes (default: current directory) |
| Existing threat model   | Path to existing `.factory/threat-model.md` if updating | No                               |
| Compliance requirements | Frameworks to consider (SOC2, GDPR, HIPAA, etc.)        | No                               |

## Instructions

### Step 1: Analyze Repository Structure

Scan the codebase to understand the system:

1. **Identify languages and frameworks**
   - Check `package.json`, `requirements.txt`, `go.mod`, `Cargo.toml`, etc.
   - Note the primary tech stack

2. **Map components and services**
   - Look for `apps/`, `services/`, `packages/` directories
   - Identify entry points: API routes, CLI commands, web handlers
   - Note databases, caches, message queues

3. **Identify external interfaces**
   - HTTP endpoints (REST, GraphQL)
   - File upload handlers
   - Webhook receivers
   - OAuth/SSO integrations

4. **Trace data flows**
   - How does user input enter the system?
   - Where is sensitive data stored?
   - What external services are called?

### Step 2: Identify Trust Boundaries

Define security zones:

1. **Public Zone** (untrusted)
   - All external HTTP endpoints
   - Public APIs without authentication
   - User-uploaded files

2. **Authenticated Zone** (partially trusted)
   - Endpoints requiring valid session/token
   - User-specific data access
   - Rate-limited APIs

3. **Internal Zone** (trusted)
   - Service-to-service communication
   - Admin-only endpoints
   - Database connections
   - Secrets management

### Step 3: Inventory Critical Assets

Classify data by sensitivity:

1. **PII (Personally Identifiable Information)**
   - User emails, names, addresses, phone numbers
   - Document protection measures

2. **Credentials & Secrets**
   - Password hashes, API keys, OAuth tokens
   - JWT signing keys, encryption keys

3. **Business-Critical Data**
   - Transaction records, customer data
   - Proprietary algorithms, trade secrets

### Step 4: Apply STRIDE Analysis

For each major component, analyze threats:

#### S - Spoofing Identity
- Can attackers impersonate users or services?
- Are authentication mechanisms secure?

#### T - Tampering with Data
- Can attackers modify data in transit or at rest?
- Look for: SQL injection, XSS, mass assignment, missing input validation

#### R - Repudiation
- Can users deny actions they performed?
- Look for: missing audit logs, insufficient logging

#### I - Information Disclosure
- Can attackers access data they shouldn't?
- Look for: IDOR, verbose errors, hardcoded secrets

#### D - Denial of Service
- Can attackers disrupt service availability?
- Look for: missing rate limits, resource exhaustion

#### E - Elevation of Privilege
- Can attackers gain unauthorized access levels?
- Look for: missing authorization checks, role manipulation

### Step 5: Document Vulnerability Patterns

Create a library of code patterns specific to this codebase's tech stack.

### Step 6: Generate Output Files

Create two files:

#### 1. `.factory/threat-model.md`

Comprehensive threat model with:
- System overview with architecture description
- Trust boundaries and security zones
- Attack surface inventory
- Critical assets classification
- STRIDE threat analysis for each component
- Vulnerability pattern library
- Security testing strategy
- Assumptions and accepted risks
- Version changelog

#### 2. `.factory/security-config.json`

```json
{
  "threat_model_version": "1.0.0",
  "last_updated": "<ISO timestamp>",
  "security_team_contacts": [],
  "compliance_requirements": [],
  "scan_frequency": "on_commit",
  "severity_thresholds": {
    "block_merge": ["CRITICAL"],
    "require_review": ["HIGH", "CRITICAL"],
    "notify_security_team": ["CRITICAL"]
  },
  "vulnerability_patterns": {
    "enabled": [
      "sql_injection",
      "xss",
      "command_injection",
      "path_traversal",
      "auth_bypass",
      "idor"
    ]
  }
}
```

## Success Criteria

- [ ] `.factory/threat-model.md` exists with all sections populated
- [ ] `.factory/security-config.json` exists with valid JSON
- [ ] All major components have STRIDE analysis
- [ ] Vulnerability patterns match the tech stack
- [ ] Document is written in natural language (LLM-readable)
- [ ] No placeholder text remains

## Example Invocations

**Generate initial threat model:**
```
Generate a threat model for this repository.
```

**Update existing threat model:**
```
Update the threat model - we added a new payments service.
```

**Generate with compliance requirements:**
```
Generate a threat model for this repository. We need to comply with SOC2 and GDPR.
```
