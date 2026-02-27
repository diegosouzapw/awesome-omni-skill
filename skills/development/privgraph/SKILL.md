---
name: privgraph
description: "Privacy and security annotation for Mermaid diagrams. Use this skill when: (1) a user shares or creates a Mermaid flowchart or sequence diagram and asks for privacy, security, or compliance review, (2) a user asks to annotate a diagram with data classifications, controls, trust boundaries, or compliance scope, (3) a user mentions 'privgraph', '@pg:', or 'risk-informed diagram', (4) a user asks about data flow risks, missing controls, or GDPR/HIPAA/PCI compliance gaps in an architecture diagram, (5) a user is writing a design doc with Mermaid diagrams and wants privacy/security feedback embedded in the diagram itself. Do NOT use for general Mermaid syntax help, diagram styling, or non-security topics."
---

# PrivGraph: Mermaid Security & Privacy Annotation

PrivGraph annotates Mermaid diagrams with privacy and security metadata using comment-based directives, then evaluates configurable rules to surface control gaps.

## Core Workflow

When a user presents a Mermaid diagram for privacy/security review:

1. **Parse** the diagram — identify nodes, edges, subgraphs, and any existing `@pg:` annotations
2. **Classify** — determine what data flows where (direct identifiers, health data, credentials, etc.)
3. **Annotate** — add `@pg:` directives as Mermaid comments for data classes, controls, boundaries, compliance
4. **Evaluate** — check the annotated diagram against the rule set (see `references/rules.yaml`)
5. **Report** — surface findings with severity, description, and remediation guidance

When a user asks to create a new annotated diagram from scratch, follow steps 2–5 after helping them build the diagram.

## Annotation Syntax

All annotations are Mermaid comments and do not affect rendering:

```
%% @pg:<directive> <target> <parameters>
```

### Directives

**`@pg:data-class`** — classify data on a flow
```
%% @pg:data-class A-->B DIRECT_ID, LOCATION
```

**`@pg:control`** — declare present or missing (`!`) controls on a node or flow
```
%% @pg:control UserDB encrypted-at-rest, access-controlled
%% @pg:control UserDB !retention-policy
%% @pg:control A-->B encrypted-in-transit
```

**`@pg:boundary`** — declare trust boundary crossing on a flow
```
%% @pg:boundary A-->B third-party
```

**`@pg:compliance`** — declare applicable compliance frameworks
```
%% @pg:compliance UserDB GDPR, CCPA
```

**`@pg:risk-accept`** — suppress a rule finding with justification
```
%% @pg:risk-accept Cache PG-003 "24hr TTL enforced at Redis level. Approved 2024-01-10"
```

**`@pg:meta`** — diagram-level metadata
```
%% @pg:meta owner "platform-team"
%% @pg:meta system "checkout-service"
%% @pg:meta reviewed "2024-01-15"
```

### Data Classifications

| Classification | Description | Examples |
|---------------|-------------|----------|
| `DIRECT_ID` | Direct identifiers | Names, emails, SSNs, phone numbers |
| `INDIRECT_ID` | Indirect/pseudonymous identifiers | Opaque IDs, UUIDs, session tokens, pseudonyms |
| `PHI` | Protected health information | EHR, Diagnoses, prescriptions, lab results |
| `PCI` | Payment card data | Card numbers, CVVs |
| `CREDENTIALS` | Authentication secrets | Passwords, API keys, tokens |
| `BIOMETRIC` | Biometric data | Fingerprints, face data, voice prints |
| `LOCATION` | Location data | GPS coordinates, IP-derived location |
| `FINANCIAL` | Financial data | Bank accounts, transactions |
| `SENSITIVE` | Other sensitive data | Catch-all for sensitive categories |
| `PUBLIC` | Non-sensitive data | Public content, marketing copy |

### Controls

`encrypted-at-rest`, `encrypted-in-transit`, `access-controlled`, `audit-logged`, `retention-policy`, `anonymized`, `minimized`, `consent-managed`, `dpa-in-place`

Prefix with `!` to mark a control as explicitly missing.

### Boundaries

`internal`, `third-party`, `cross-region`, `cross-cloud`, `public-internet`, `user-device`

### Compliance Frameworks

`GDPR`, `CCPA`, `HIPAA`, `PCI-DSS`, `SOC2`, `FERPA`

## Rule Evaluation

After annotating, evaluate the diagram against the rule set. Load `references/rules.yaml` for the full rule definitions. Here is the summary:

| Rule | Sev | Trigger |
|------|-----|---------|
| PG-001 | HIGH | DIRECT_ID/INDIRECT_ID/PHI flow missing `encrypted-in-transit` |
| PG-002 | HIGH | DIRECT_ID/INDIRECT_ID/PHI node missing `encrypted-at-rest` |
| PG-003 | MED | Sensitive data node missing `retention-policy` |
| PG-004 | HIGH | CREDENTIALS flow missing encryption |
| PG-005 | CRIT | DIRECT_ID to `third-party` missing `dpa-in-place` |
| PG-006 | HIGH | PHI to `third-party` boundary |
| PG-007 | MED | `cross-region` flow missing compliance annotation |
| PG-008 | HIGH | HIPAA-scoped flow missing `audit-logged` |
| PG-009 | MED | GDPR-scoped node missing `retention-policy` |
| PG-010 | LOW | Flow crosses boundary with no controls documented |
| PG-011 | INFO | Node has zero controls documented |

For each violation, check if a `@pg:risk-accept` suppresses it (matching target + rule ID). Suppressed findings should be noted but not reported as active violations.

## Reporting Format

Present findings grouped by severity. Use this structure:

```
## PrivGraph Review: [system name or file]

### Findings

🔴 **CRITICAL — PG-005: Direct Identifiers to Third Party Without DPA**
Flow: UserSvc → Analytics
DIRECT_ID is transferred to a third party without a documented DPA.
→ Add: `%% @pg:control Analytics dpa-in-place`

🟡 **HIGH — PG-001: Unencrypted Identifier Transfer**
Flow: Mobile → API
DIRECT_ID transferred without encryption in transit.
→ Add: `%% @pg:control Mobile-->API encrypted-in-transit`

### Summary
X critical, Y high, Z medium | N findings suppressed by @pg:risk-accept
```

Always output the **complete annotated diagram** (original diagram with all `@pg:` annotations added) so the user can copy-paste it directly.

## Handling Ambiguity

When data classifications are unclear from the diagram alone:
- Ask the user what data flows through ambiguous edges
- Default to the more sensitive classification when context suggests it
- Flag uncertain classifications with a comment: `%% TODO: Confirm data classification for A-->B`

When a diagram has no annotations yet, annotate it fully rather than just listing what's missing. Show the user what a complete annotation looks like.

## Supported Diagram Types

- **Flowcharts**: `flowchart LR/TB/etc.` — nodes, edges, subgraphs all supported
- **Sequence diagrams**: `sequenceDiagram` — participants map to nodes, messages map to edges, boxes map to subgraphs

For annotation target syntax in sequence diagrams, use participant names: `%% @pg:data-class Client->>Server DIRECT_ID`

## Examples

See `examples/flowchart-annotated.md` and `examples/sequence-annotated.md` for complete worked examples with annotations and findings.
