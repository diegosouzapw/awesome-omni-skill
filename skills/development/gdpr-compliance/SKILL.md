---
name: gdpr-compliance
description: Use when handling personal data, user registration, consent forms, cookie consent banners, data export, account deletion, privacy policies, or any code processing EU/EEA user data — GDPR coding patterns, Article 5, Article 6, lawful basis, legitimate interest, right to erasure, right to access, data portability, DPIA, DPO, 72-hour breach notification, Privacy Shield, Standard Contractual Clauses, data minimization, consent management, data subject rights, privacy by design
---

# GDPR Compliance Coding Guidelines

## 1. Overview

The General Data Protection Regulation (GDPR), effective 25 May 2018, governs how organizations collect, store, process, and delete personal data of individuals in the EU/EEA. It applies to ANY service that processes data of EU/EEA residents, regardless of where the service is hosted. Non-compliance carries penalties of up to 4% of annual global turnover or EUR 20 million, whichever is greater. Every developer writing code that touches user data — names, emails, IP addresses, cookies, device IDs, location data, or any information that can identify a natural person — must follow these patterns.

## 2. The 7 Principles (Art. 5)

| # | Principle | What Developers Must Do |
|---|-----------|------------------------|
| 1 | **Lawfulness, fairness, transparency** | Collect data only with a valid lawful basis (Art. 6). Show clear consent UI. Link to a readable privacy policy before any data collection. |
| 2 | **Purpose limitation** | Collect data for specified, explicit purposes. Use separate consent checkboxes per purpose. Never repurpose collected data without new consent. |
| 3 | **Data minimization** | Collect only what is strictly needed. Remove "nice to have" fields from forms and database schemas. If you do not need a birthdate, do not ask for it. |
| 4 | **Accuracy** | Provide edit-profile endpoints. Validate inputs at collection. Allow users to correct their data at any time. |
| 5 | **Storage limitation** | Define retention periods per data type. Implement automated cleanup jobs. Delete or anonymize data when the retention period expires. |
| 6 | **Integrity & confidentiality** | Encrypt PII at rest and in transit. Enforce role-based access control. Maintain audit logs of data access. |
| 7 | **Accountability** | Document all processing activities. Maintain a Record of Processing Activities (ROPA). Log consent events with timestamps and versions. |

## 3. Lawful Bases for Processing (Art. 6)

| Lawful Basis | When to Use | Code Implication |
|---|---|---|
| **Consent** (Art. 6(1)(a)) | Marketing emails, analytics cookies, non-essential data collection | Explicit opt-in UI, granular per purpose, must be withdrawable |
| **Contract** (Art. 6(1)(b)) | Processing necessary to deliver a service the user signed up for | User registration data, order processing, account management |
| **Legal obligation** (Art. 6(1)(c)) | Tax records, fraud prevention mandated by law | Retain invoices for statutory period even after account deletion |
| **Vital interests** (Art. 6(1)(d)) | Emergency medical situations | Rare in software — almost never the correct basis |
| **Public task** (Art. 6(1)(e)) | Government or public authority functions | Only applies to public sector applications |
| **Legitimate interests** (Art. 6(1)(f)) | Fraud prevention, network security, internal analytics | Requires documented balance test — user rights vs. business need |

### Consent Collection Key Rules

- Consent checkboxes must NEVER be pre-checked. Each purpose requires a separate checkbox.
- Store consent with: timestamp, policy version, IP address, user agent, and exact wording shown.
- Withdrawal must be as easy as granting consent.
- Double opt-in is required for email marketing in many EU jurisdictions.

> **Code patterns:** See [resources/consent-patterns.md](resources/consent-patterns.md) for HTML consent forms, SQL storage schema, consent recording (PHP), withdrawal (Python), double opt-in, cookie consent (JS), API endpoints, and validation-before-processing patterns.

## 4. Data Subject Rights (Developer Summary)

| # | Right | Article | What to Implement |
|---|-------|---------|-------------------|
| 1 | **Access** | Art. 15 | Export all personal data held about the user (JSON/CSV). Respond within 30 days. |
| 2 | **Rectification** | Art. 16 | Allow users to correct inaccurate personal data via edit-profile endpoints. |
| 3 | **Erasure (Right to be Forgotten)** | Art. 17 | Delete personal data. Anonymize what must be retained for legal obligations. |
| 4 | **Data Portability** | Art. 20 | Provide data in structured, machine-readable format with documented schema. |
| 5 | **Restriction of Processing** | Art. 18 | Allow users to pause processing without deleting data. Guard all processing functions. |
| 6 | **Object** | Art. 21 | Accept objections to processing based on legitimate interests, profiling, and direct marketing. |
| 7 | **Automated Decision-Making** | Art. 22 | Offer human review for decisions with legal or significant effects based on automated processing. |

> **Code patterns:** See [resources/data-subject-rights.md](resources/data-subject-rights.md) for full implementation patterns for all 7 rights (Python, PHP, SQL).

## 5. Privacy by Design (Art. 25)

Key principles every developer must follow:

- **Data Minimization in Schemas** — Collect only fields that are strictly needed. Remove "nice to have" columns. If you do not need a birthdate, do not add a birthdate column.
- **Pseudonymization** — Replace direct identifiers with one-way hashes in analytics and logs. Store salts separately with access controls.
- **Encryption** — Encrypt PII at rest using field-level encryption (e.g., Fernet). Encrypt in transit with TLS.
- **Privacy-Friendly Defaults** — New user settings must default to the most restrictive options (profile private, marketing off, analytics off, third-party sharing off).
- **Separate Storage for Sensitive Data** — Store PII in a separate table (or database) with stricter access controls. Services that do not need PII never get PII credentials.
- **Purpose-Based Access Control** — Define which fields each processing purpose can access. Marketing can only read email (and only with consent). Analytics gets only pseudonymized IDs.

> **Code patterns:** See [resources/privacy-design-retention.md](resources/privacy-design-retention.md) for schema examples, pseudonymization, encryption, privacy defaults, separated storage, purpose-based access control, retention policies, cleanup jobs, soft/hard delete, and anonymization patterns.

## 6. Consent Management

Consent management covers cookie consent banners, server-side consent APIs, and validation-before-processing guards. All consent UI must:

- Never use cookie walls (blocking site access without consent violates Art. 7(4)).
- Categorize cookies (necessary, analytics, marketing) with separate toggles.
- Store consent server-side with audit trail (timestamp, version, IP).
- Allow withdrawal at any time, as easily as granting.

> **Code patterns:** See [resources/consent-patterns.md](resources/consent-patterns.md) for cookie consent JS implementation, consent API endpoints (Python), and validation-before-processing guards.

## 7. Data Retention & Deletion

Every data type must have a defined retention period. Implement automated cleanup that runs on a schedule (daily cron).

| Data Type | Retention | Action |
|-----------|-----------|--------|
| Sessions | 30 days | Delete |
| Activity logs | 90 days | Delete |
| Analytics events | 1 year | Anonymize |
| Support tickets | 2 years | Anonymize |
| Invoices | 7 years | Retain (tax law) |
| Deleted user backups | 30 days | Delete |
| Consent records | 7 years | Retain (accountability) |
| Password reset tokens | 1 day | Delete |

Three deletion strategies: **Soft delete** (mark deleted, 30-day grace period), **Hard delete** (permanent removal), **Anonymize** (remove PII, keep record for analytics/business).

> **Code patterns:** See [resources/privacy-design-retention.md](resources/privacy-design-retention.md) for retention policy definitions, automated cleanup jobs, soft/hard/anonymize implementations, and SQL anonymization patterns.

## 8. Data Breach Notification (Art. 33, Art. 34)

### Requirements

- Report to supervisory authority within **72 hours** of becoming aware of a breach (Art. 33).
- Notify affected users **without undue delay** if the breach poses a **high risk** to their rights (Art. 34).

### Breach Report Contents

The notification must include:
1. Nature of the breach (what data, how many records, how many individuals).
2. Contact details of the Data Protection Officer.
3. Likely consequences of the breach.
4. Measures taken or proposed to address the breach.

> **Code patterns:** See [resources/breach-notification.md](resources/breach-notification.md) for incident response code (Python), containment actions, notification logic, and breach log SQL schema.

## 9. Cross-Border Data Transfers (Art. 44-49)

### 9.1 Adequacy Decisions

Countries with an EU adequacy decision (data can flow freely): Andorra, Argentina, Canada (PIPEDA), Faroe Islands, Guernsey, Israel, Isle of Man, Japan, Jersey, New Zealand, Republic of Korea, Switzerland, United Kingdom, Uruguay, and the US (EU-US Data Privacy Framework, for certified companies).

### 9.2 When Standard Contractual Clauses (SCCs) Are Needed

If transferring data to a country WITHOUT an adequacy decision, you must implement SCCs or another transfer mechanism (Art. 46).

### 9.3 Data Residency Patterns

```python
# Configuration — restrict data storage to EU regions
DATA_RESIDENCY = {
    "default_region": "eu-west-1",
    "allowed_regions": ["eu-west-1", "eu-central-1", "eu-north-1"],
    "blocked_regions": ["us-east-1", "ap-southeast-1"],
}

def get_storage_region(user_country: str) -> str:
    """Determine storage region based on user location."""
    if user_country in EU_EEA_COUNTRIES:
        return DATA_RESIDENCY["default_region"]
    # For non-EU users, still store in EU if that is your policy
    return DATA_RESIDENCY["default_region"]

def validate_storage_region(region: str) -> bool:
    if region in DATA_RESIDENCY["blocked_regions"]:
        raise ValueError(f"Data storage in {region} is not permitted under GDPR policy")
    return region in DATA_RESIDENCY["allowed_regions"]
```

```sql
-- Add region tracking to user data
ALTER TABLE users ADD COLUMN data_region VARCHAR(20) NOT NULL DEFAULT 'eu-west-1';

-- Ensure queries only access data in permitted regions
-- Application layer must enforce this
```

### 9.4 Third-Party Data Processor Checks

```python
def register_third_party_processor(processor_name: str, country: str, has_scc: bool) -> None:
    """Record third-party processors and their transfer basis."""
    is_adequate = country in ADEQUATE_COUNTRIES

    if not is_adequate and not has_scc:
        raise ValueError(
            f"Cannot register processor in {country} without SCCs or adequacy decision"
        )

    db.execute(
        "INSERT INTO data_processors (name, country, adequate, has_scc, registered_at) "
        "VALUES (%s, %s, %s, %s, NOW())",
        (processor_name, country, is_adequate, has_scc),
    )
```

## 10. Data Protection Impact Assessment (DPIA) (Art. 35)

### 10.1 When a DPIA Is Required

A DPIA is mandatory before processing that is likely to result in a high risk to individuals:
- Large-scale processing of special categories of data (health, biometrics, religion, political opinions).
- Systematic monitoring of a publicly accessible area (CCTV).
- Large-scale profiling with legal or significant effects.
- Automated decision-making with legal effects.
- Processing of children's data at scale.
- Combining datasets from different sources.

### 10.2 What to Document

```python
DPIA_TEMPLATE = {
    "project_name": "",
    "description": "",
    "data_categories": [],             # personal, special category, children
    "processing_purposes": [],
    "lawful_basis": "",                # consent, contract, legitimate interest, etc.
    "data_subjects": "",               # customers, employees, children, etc.
    "data_volume": "",                 # approximate number of records
    "retention_period": "",
    "third_parties": [],               # processors and their countries
    "risks": [
        {
            "risk": "",
            "likelihood": "",          # low, medium, high
            "impact": "",              # low, medium, high
            "mitigation": "",
        }
    ],
    "security_measures": [],           # encryption, access control, pseudonymization
    "dpo_consulted": False,
    "supervisory_authority_consulted": False,
    "assessment_date": "",
    "review_date": "",                 # DPIAs must be reviewed periodically
}
```

### 10.3 Risk Assessment in Code Reviews

When reviewing code that processes personal data, check:

```text
[ ] Is there a lawful basis for this processing?
[ ] Is the data minimized — only collecting what is needed?
[ ] Is there a defined retention period?
[ ] Is the data encrypted at rest and in transit?
[ ] Are access controls in place (who can read this data)?
[ ] Is there a deletion path for this data?
[ ] If consent-based, can the user withdraw consent?
[ ] Is the data exported to any third party? If so, is there an SCC or adequacy basis?
[ ] Are there audit logs for access to this data?
[ ] If this involves automated decisions, is there a human review path?
```

## 11. Common Mistakes

| # | Mistake | Why It Violates GDPR | Fix |
|---|---------|---------------------|-----|
| 1 | Pre-checked consent checkboxes | Consent must be a clear affirmative act (Art. 4(11), Recital 32) | All consent checkboxes must default to unchecked |
| 2 | No account deletion endpoint | Violates Right to Erasure (Art. 17) | Implement delete account API with cascading cleanup |
| 3 | Collecting unnecessary fields | Violates Data Minimization (Art. 5(1)(c)) | Audit forms — remove every field that is not strictly necessary |
| 4 | No data retention policy | Violates Storage Limitation (Art. 5(1)(e)) | Define retention periods and implement automated cleanup |
| 5 | Storing consent without timestamp | Cannot prove when consent was given (Accountability, Art. 5(2)) | Always store timestamp, policy version, IP, and exact consent text |
| 6 | Cookie wall (blocking site access without consent) | Consent is not freely given if access depends on it (Art. 7(4)) | Allow access with only necessary cookies; make consent optional |
| 7 | Bundled consent (one checkbox for everything) | Consent must be granular and purpose-specific (Art. 6(1)(a)) | Separate checkbox per purpose |
| 8 | No data export feature | Violates Right to Portability (Art. 20) | Provide JSON or CSV export of user data |
| 9 | Logging PII in application logs | Uncontrolled processing, no retention, hard to delete | Pseudonymize or exclude PII from logs; use user ID hashes |
| 10 | Storing EU data outside EU without SCCs | Violates Transfer rules (Art. 44-49) | Use EU regions or ensure valid SCCs are in place |
| 11 | No way to withdraw consent | Withdrawal must be as easy as giving consent (Art. 7(3)) | Provide withdraw button next to each consent preference |
| 12 | Sharing data with third parties without disclosure | Violates transparency (Art. 13(1)(e)) | List all recipients in privacy policy; get consent if needed |
| 13 | No breach notification process | Must notify authority within 72 hours (Art. 33) | Implement breach detection, logging, and notification workflow |
| 14 | Using personal data for a new purpose without consent | Violates Purpose Limitation (Art. 5(1)(b)) | Get new consent before repurposing data |
| 15 | Hardcoded deletion without anonymization option | Some records must be retained (invoices for tax law) | Implement selective deletion: delete PII, anonymize business records, retain legal records |

## 12. Quick Reference

### DO

```text
[x] Collect explicit, granular, unchecked opt-in consent for each purpose
[x] Store consent records with timestamp, version, IP, and exact text shown
[x] Provide data export in JSON and CSV formats (Art. 15, Art. 20)
[x] Implement account deletion with cascading cleanup (Art. 17)
[x] Define and enforce retention periods for every data type
[x] Encrypt PII at rest and in transit
[x] Pseudonymize user IDs in analytics and logs
[x] Set privacy-friendly defaults for new accounts
[x] Implement consent withdrawal that is as easy as consent granting
[x] Separate sensitive PII into isolated storage with stricter access
[x] Log all access to personal data for audit purposes
[x] Validate that third-party processors have SCCs or adequacy decisions
[x] Run a DPIA before large-scale processing of sensitive data
[x] Implement 72-hour breach notification workflow
[x] Provide human review for automated decisions with legal effects
[x] Allow users to restrict processing without deleting their data
[x] Document all processing activities (Record of Processing Activities)
[x] Run automated retention cleanup jobs on a schedule
```

### DON'T

```text
[ ] Pre-check consent checkboxes
[ ] Bundle multiple consent purposes into one checkbox
[ ] Collect data "just in case" — remove unnecessary form fields
[ ] Store PII in application logs or error tracking systems
[ ] Use cookie walls that block site access without consent
[ ] Transfer data to non-adequate countries without SCCs
[ ] Repurpose collected data without obtaining new consent
[ ] Make consent withdrawal harder than consent granting
[ ] Skip the deletion endpoint — every app needs one
[ ] Forget to include timestamps and versions in consent records
[ ] Log full IP addresses in analytics without consent or legitimate interest
[ ] Store passwords in plaintext (not GDPR-specific but relevant to Art. 32 security)
[ ] Ignore data subject access requests — you have 30 days to respond
[ ] Deploy profiling or automated decisions without a human review fallback
[ ] Assume GDPR does not apply because your servers are outside the EU
```

### Key GDPR Articles Reference

| Article | Topic |
|---------|-------|
| Art. 4 | Definitions (personal data, processing, consent, controller, processor) |
| Art. 5 | Principles (lawfulness, minimization, accuracy, storage limitation, integrity) |
| Art. 6 | Lawful bases for processing |
| Art. 7 | Conditions for consent |
| Art. 12-14 | Transparency and information obligations |
| Art. 15 | Right of access |
| Art. 16 | Right to rectification |
| Art. 17 | Right to erasure |
| Art. 18 | Right to restriction of processing |
| Art. 20 | Right to data portability |
| Art. 21 | Right to object |
| Art. 22 | Automated individual decision-making |
| Art. 25 | Data protection by design and by default |
| Art. 32 | Security of processing |
| Art. 33 | Notification of breach to supervisory authority (72 hours) |
| Art. 34 | Communication of breach to data subject |
| Art. 35 | Data protection impact assessment |
| Art. 44-49 | Transfers to third countries |
| Art. 83 | Penalties (up to 4% turnover or EUR 20M) |

## Resources

| File | Contents |
|------|----------|
| [resources/consent-patterns.md](resources/consent-patterns.md) | Consent forms (HTML), storage schema (SQL), recording/withdrawal (PHP/Python), double opt-in, cookie consent (JS), consent API endpoints, validation-before-processing |
| [resources/data-subject-rights.md](resources/data-subject-rights.md) | Implementation patterns for all 7 data subject rights: access/export, rectification, erasure, portability, restriction, objection, automated decisions |
| [resources/privacy-design-retention.md](resources/privacy-design-retention.md) | Privacy by Design code (schema minimization, pseudonymization, encryption, defaults, separated storage, purpose-based access) + Data Retention & Deletion (policies, cleanup jobs, soft/hard delete, anonymization) |
| [resources/breach-notification.md](resources/breach-notification.md) | Data breach incident response code, containment actions, notification triggers, breach log schema |
