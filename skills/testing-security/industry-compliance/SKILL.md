---
name: industry-compliance
description: "Use this skill when you need industry-specific regulatory compliance for Banking & Finance (FFIEC, FINRA, Basel III, PSD2, DORA), Healthcare & Life Sciences (FDA 21 CFR Part 11, HITRUST CSF, HL7 FHIR security, GxP), Hi-Tech & Semiconductors (ITAR, EAR, CMMC), or Retail/Consumer (PCI-DSS, CPRA). Trigger for sector-specific compliance programs, regulated industry deployments, or when standard frameworks alone are insufficient."
license: Apache-2.0
metadata:
  author: aviskaar
  version: "1.0"
  tags: [banking-compliance, healthcare-compliance, finra, ffiec, hitrust, fda-21cfr, itar, cmmc, pci-dss, dora, life-sciences, hi-tech, industry-standards]
---

# Industry Compliance — Industry-Specific Security Standards Specialist

## Role

The Industry Compliance specialist owns the security and regulatory requirements unique to specific regulated industries. This skill supplements the baseline frameworks (SOC 2, NIST, ISO 27001) with sector-specific mandates that carry unique technical controls, audit regimes, and regulatory penalty structures.

---

## Sector 1 — Banking & Financial Services

### FFIEC Cybersecurity Assessment Tool (CAT):

**Inherent Risk Profile (assess first):**
```
Risk Categories:
1. Technologies and Connection Types
   - Internet-facing systems, mobile banking, external connectivity
2. Delivery Channels
   - ATM, remote deposit, online banking, wire transfers
3. Online/Mobile Products and Technology Services
   - Transaction volume, customer-facing systems
4. Organizational Characteristics
   - Asset size, geography, complexity
5. External Threats
   - Industry threat intelligence, recent attack patterns

Inherent Risk Levels: Least → Minimal → Moderate → Significant → Most
```

**Cybersecurity Maturity (5 domains):**
```
Domain 1: Cyber Risk Management and Oversight
- Board-level cybersecurity oversight; CISO reports to Board
- Cybersecurity strategy integrated into business strategy
- Risk appetite defined; security risk included in ERM

Domain 2: Threat Intelligence and Collaboration
- Financial sector threat intel sharing (FS-ISAC membership)
- Threat intelligence program; automated IOC ingestion
- Sharing of cyber threat information with sector peers

Domain 3: Cybersecurity Controls
- Controls aligned to NIST CSF; FFIEC-specific controls implemented
- Network security; endpoint security; application security
- Encryption: FIPS 140-2/3 validated modules required for regulated data

Domain 4: External Dependency Management
- Third-party risk management; critical vendors audited annually
- Cloud provider due diligence; contract security requirements
- Concentration risk: no single vendor for critical functions

Domain 5: Cyber Incident Management and Resilience
- Incident response plan; table-top exercises (annual minimum)
- Regulatory reporting: notify OCC/Fed/FDIC within 36h of significant incident
- Business continuity: tested recovery; RTO/RPO defined for critical systems
```

### FINRA / SEC Cybersecurity Rules:

```
SEC Rule 10b-5 / Regulation S-P:
- Safeguards Rule: protect customer financial records and information
- Annual cybersecurity review required
- Material breach: disclose within 4 business days (Form 8-K / Item 1.05)
- Cybersecurity risk management: board oversight; documented program

FINRA Requirements:
- Written supervisory procedures (WSP) include cybersecurity
- Annual cybersecurity risk assessment
- Multi-factor authentication for all firm systems containing customer data
- Vendor management: written agreements; security requirements
- Penetration testing: recommended annually
- Incident reporting to FINRA if customer data compromised

Basel III / IV (Operational Risk):
- Cyber risk as operational risk; capital charge implications
- Scenario analysis: cyber risk scenarios in ICAAP/ILAAP
- Operational Resilience: critical business functions identified; impact tolerances set
```

### DORA (EU Digital Operational Resilience Act):

```
Applicability: EU financial entities (banks, insurers, payment institutions, investment firms)
Key Requirements:
□ ICT Risk Management Framework — comprehensive, integrated into overall ERM
□ ICT Incident Reporting:
  - Major incidents: initial report to authority within 4h; intermediate within 72h; final within 1 month
□ Digital Operational Resilience Testing:
  - Basic testing (annually): vulnerability assessments, open-source analyses
  - Advanced TLPT (Threat-Led Penetration Testing): every 3 years; TIBER-EU framework
□ ICT Third-Party Risk Management:
  - Register of all ICT third-party providers
  - Critical third parties: direct oversight by EU supervisors
  - Exit strategies for critical providers
□ Information and Intelligence Sharing:
  - Voluntary participation in cyber threat intelligence sharing
```

### PSD2 / Open Banking Security:

```
Strong Customer Authentication (SCA):
- Authentication uses ≥2 of: knowledge (PIN/password), possession (device/card), inherence (biometric)
- Dynamic linking: authentication code linked to specific transaction amount and payee
- SCA exemptions: low-value (<€30), trusted beneficiary, corporate, low-risk (TRA)

API Security for Open Banking:
- OAuth 2.0 + OIDC for TPP (Third-Party Provider) access
- mTLS certificates for TPP authentication
- Certificate management: QWAC (Qualified Website Authentication Certificate)
- API rate limiting; fraud monitoring on open banking flows
- Consent management: granular; time-limited; revocable
```

---

## Sector 2 — Healthcare & Life Sciences

### HITRUST CSF (Common Security Framework):

```
HITRUST Assurance Levels:
- e1 (Essentials): 44 requirements; basic cyber hygiene; fastest to achieve
- i1 (Implemented): 182 requirements; validated implementation
- r2 (Risk-Based): 375 requirements; 3rd party validated; gold standard for healthcare

Key HITRUST Control Categories:
1. Information Protection Program (01.x)
2. Endpoint Protection (07.x)
3. Portable Media Security (08.x)
4. Mobile Device Security (09.x)
5. Wireless Protection (10.x)
6. Configuration Management (11.x)
7. Vulnerability Management (12.x)
8. Network Protection (13.x)
9. Password Management (14.x)
10. Access Control (15.x)
11. Audit Logging (16.x)
12. Education, Training & Awareness (17.x)
13. Third Party Assurance (18.x)
14. Incident Management (19.x)
15. Business Continuity & DR (20.x)
16. Risk Management (21.x)

HITRUST assessment process:
1. Readiness Assessment (self-assessment) → identify gaps
2. Validated Assessment (HITRUST CSF Assessor) → formal certification
3. QA Review by HITRUST → 90 business day review cycle
4. Certification issued (valid 2 years; interim assessment at 1 year)
```

### FDA 21 CFR Part 11 (Electronic Records & Signatures):

```
Applicability: FDA-regulated industries (pharma, medical device, biotech, clinical trials)

Technical Controls Required:
□ System validation: software validated for intended use; IQ/OQ/PQ documentation
□ Audit trails: computer-generated, date/time stamped; cannot be disabled or modified
□ Record security: access limited by roles; modification access separate from view access
□ Electronic signatures: legally binding; linked to records; non-repudiable
□ System access: unique user IDs; no shared accounts; periodic review
□ Authority checks: only authorized users can use system; appropriate actions
□ Operational checks: sequencing of steps enforced by system

Audit Trail Requirements:
- Record: who did what, when, and why (reason for change)
- Immutable: cannot be altered by any user including admin
- Retained: same period as the record it supports (often 15+ years for clinical data)
- Available: accessible to FDA inspectors on demand
```

### HL7 FHIR Security (Healthcare Interoperability):

```
FHIR API Security Controls:
- Authentication: SMART on FHIR (OAuth 2.0 profile)
- Authorization: scopes aligned to FHIR resource types and operations
- Consent enforcement: FHIR Consent resource governing data access
- TLS 1.2+ for all FHIR API communications
- Audit events: FHIR AuditEvent resource for all access

PHI in FHIR:
- De-identification: Safe Harbor or Expert Determination method
- Minimum necessary: only expose data required for use case
- Patient rights: enable patient access to their own FHIR data
```

### GxP (Good x Practice — Pharma/MedDev):

```
GxP principles for IT systems:
- CSV (Computer System Validation): all GxP systems validated
- Data integrity: ALCOA+ (Attributable, Legible, Contemporaneous, Original, Accurate + Complete, Consistent, Enduring, Available)
- Change control: all changes to validated systems follow change management
- Audit readiness: systems ready for FDA inspection at all times
- Supplier qualification: software vendors assessed for quality systems
```

---

## Sector 3 — Hi-Tech & Defense

### CMMC 2.0 (Cybersecurity Maturity Model Certification):

```
Level 1 (Foundational): 17 practices — basic cyber hygiene; annual self-assessment
Level 2 (Advanced): 110 practices — aligned to NIST SP 800-171; triennial C3PAO assessment
Level 3 (Expert): 110+ practices — NIST SP 800-172; government-led assessment

Level 2 Key Practice Areas:
- Access Control (AC): 22 practices — least privilege; remote access controls
- Audit and Accountability (AU): 9 practices — audit logs; review and reporting
- Configuration Management (CM): 9 practices — baselines; change control
- Identification and Authentication (IA): 11 practices — MFA; password management
- Incident Response (IR): 3 practices — IR plan; testing; reporting
- Maintenance (MA): 6 practices — controlled maintenance; media sanitization
- Media Protection (MP): 9 practices — CUI on media; sanitization
- Personnel Security (PS): 2 practices — screening; termination
- Physical Protection (PE): 6 practices — physical access controls
- Risk Assessment (RA): 3 practices — risk assessments; vulnerability scanning
- Security Assessment (CA): 4 practices — periodic assessments; POA&M
- System/Communications Protection (SC): 16 practices — boundary protection; encryption
- System and Information Integrity (SI): 7 practices — malware protection; security alerts
```

### ITAR / EAR (Export Controls):

```
ITAR (International Traffic in Arms Regulations):
- Applicability: defense articles, services, technical data (USML items)
- Registration: all manufacturers/exporters register with DDTC
- Export authorization: license or exemption for any foreign person access
- Technical data controls: access limited to US persons unless licensed
- Encryption: use of foreign encryption in ITAR items requires review
- Violation: criminal penalties up to $1M/violation; civil up to $1.35M

IT Controls for ITAR:
- Access control: only US persons access ITAR-controlled systems and data
- Physical: ITAR data on physically controlled systems; no foreign access
- Cloud: US-only cloud regions; US-person cloud provider employees only
- Training: ITAR awareness training for all personnel with access
```

---

## Sector 4 — Manufacturing & Energy

### ICS/OT Security (NERC CIP / ISA/IEC 62443):

```
NERC CIP (Critical Infrastructure Protection — Energy):
- CIP-002: BES Cyber System categorization (High/Medium/Low impact)
- CIP-004: Security awareness training; personnel risk assessment
- CIP-005: Electronic security perimeters; ESP access controls
- CIP-006: Physical security of BES Cyber Systems
- CIP-007: Systems security management; ports/services; malware prevention
- CIP-008: Incident reporting and response planning
- CIP-010: Configuration management; vulnerability management
- CIP-011: Information protection; BES Cyber System Information

OT/ICS Security Principles:
- Air gap or strict DMZ between IT and OT networks
- No direct internet connectivity to OT systems
- Purdue Model / ISA-95 segmentation enforced
- Change management: any OT change requires engineering + security sign-off
- Vendor access: time-limited, monitored, requires business justification
- Incident response: separate OT IR playbook (safety first, then security)
```

---

## Non-Negotiable Industry Compliance Rules

1. **Sector regulator requirements supersede internal policy** — if FFIEC/FDA/NERC requires it, it is mandatory
2. **Regulatory reporting timelines are law** — breach notification deadlines are not negotiable
3. **Audit readiness is permanent** — FDA, OCC, FINRA can inspect without advance notice
4. **Industry certification lapses are business-critical** — HITRUST, PCI-DSS, CMMC lapse = customer loss
5. **Export controls are strict liability** — ITAR/EAR violations do not require intent; compliance mandatory
6. **Patient safety supersedes all else** — for healthcare, security controls that protect patient safety are non-negotiable
