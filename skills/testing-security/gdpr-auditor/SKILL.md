---
name: gdpr-auditor
description: This skill should be used when analyzing codebases, applications, databases, or systems for GDPR (General Data Protection Regulation) compliance. Use this skill when users need to audit data protection practices, identify potential compliance issues, assess data handling procedures, review privacy policies, or ensure adherence to EU data protection requirements.
license: MIT
---

# GDPR Auditor Skill

This skill provides comprehensive guidance for auditing systems and codebases for GDPR compliance.

## Purpose

The GDPR auditor skill equips Claude with specialized knowledge to:

1. Identify personal data collection, storage, and processing practices
2. Assess compliance with GDPR principles (lawfulness, fairness, transparency, purpose limitation, data minimization, accuracy, storage limitation, integrity and confidentiality)
3. Review data subject rights implementation (access, rectification, erasure, portability, objection)
4. Evaluate security measures and data protection by design/default
5. Identify data breach risks and incident response procedures
6. Assess third-party data processor agreements and international data transfers
7. Review documentation and record-keeping practices

## When to Use This Skill

Use this skill when:

- Auditing a codebase for GDPR compliance issues
- Reviewing database schemas for personal data handling
- Analyzing privacy policies and consent mechanisms
- Evaluating data retention and deletion practices
- Assessing API endpoints that handle personal data
- Reviewing authentication and authorization systems
- Preparing for GDPR compliance certifications
- Investigating potential data protection violations
- Designing data protection impact assessments (DPIAs)

## When NOT to Use This Skill

Do **NOT** use this skill for:

- **Live system penetration testing** - This audits static code, not running systems
- **Runtime behavior analysis** - Cannot observe application execution or user flows
- **Live database auditing** - Does not connect to running databases (only analyzes schema files)
- **Network traffic monitoring** - No real-time traffic analysis capability
- **Third-party API testing** - Cannot call or test external services directly
- **Legal compliance certification** - This is a technical tool, not official legal certification
- **Non-EU jurisdictions** - Use CCPA Auditor for California, HIPAA Auditor for US healthcare
- **Production system scanning** - Works with source code repositories, not deployed systems
- **Active vulnerability exploitation** - Defensive analysis only

**This skill analyzes static files only** (source code, configuration files, database schema files, documentation). For live system testing or runtime analysis, consult qualified security professionals.

## Output Format

This skill generates a **GDPR Compliance Audit Report** in Markdown format.

**Report Structure:**

```markdown
# GDPR Compliance Audit Report
Generated: [Date]
Application: [Name/Description]
Audited by: GDPR Auditor Skill

## Executive Summary
- Overall Compliance Status: [Compliant / Partially Compliant / Non-Compliant]
- Critical Issues: [Number]
- High Priority Issues: [Number]
- Overall Risk Level: [Low / Medium / High / Critical]
- Primary Concerns: [Brief list]

## Critical Issues
[Issues requiring immediate attention]

1. [Issue Title]
   - **GDPR Article(s):** [Relevant articles]
   - **File/Location:** [file.py:line or description]
   - **Risk:** Critical
   - **Finding:** [Detailed description]
   - **Recommendation:** [Specific remediation steps]

## High-Priority Recommendations
[Important improvements needed for compliance]

## Medium-Priority Recommendations
[Suggested enhancements and best practices]

## Compliant Areas
[Aspects that meet GDPR requirements - positive findings]

## Data Subject Rights Assessment
- Right to Access: [Implemented / Not Implemented / Partial]
- Right to Rectification: [Status]
- Right to Erasure: [Status]
- Right to Data Portability: [Status]
- Right to Object: [Status]

## Compliance Roadmap
**Phase 1 (Immediate - 0-30 days):**
[Critical fixes]

**Phase 2 (Short-term - 1-3 months):**
[High-priority items]

**Phase 3 (Medium-term - 3-6 months):**
[Enhancements and optimizations]

## GDPR Articles Referenced
[List of GDPR articles cited in findings]

## Next Steps
[Prioritized action items]
```

**Deliverable:** A comprehensive, actionable audit report with specific code references, GDPR article citations, and prioritized remediation guidance.

## Workflow

### Initial Assessment

Start by understanding the scope of the audit:

1. Identify the type of system being audited (web application, mobile app, database, API, etc.)
2. Determine the categories of personal data processed
3. Understand the legal basis for processing (consent, contract, legitimate interest, etc.)
4. Identify data subjects (EU residents, employees, customers, etc.)

### Static Code and File Analysis

When analyzing codebase files and configurations, examine:

1. **Data Collection Points** (Source Code Analysis)
   - Forms, input fields, API endpoint definitions in code
   - Third-party integration code (analytics, advertising, social media SDKs)
   - Cookie and tracking technology implementations
   - Use `scripts/scan_data_collection.py` to automatically identify data collection patterns in source files

2. **Data Storage** (Schema File Analysis)
   - Database schema files (SQL DDL, migrations, ORM models)
   - Field types and constraints in schema definitions
   - Encryption configuration in code
   - Data retention policies defined in code or configuration
   - Use `scripts/analyze_database_schema.py` to review database schema files and migration scripts

3. **Data Processing**
   - How personal data flows through the system
   - Third-party processors and data sharing
   - Automated decision-making and profiling
   - Cross-border data transfers

4. **Data Subject Rights**
   - Implementation of access requests (right to access)
   - Data export functionality (right to portability)
   - Deletion mechanisms (right to erasure)
   - Opt-out and consent withdrawal
   - Use `scripts/check_dsr_implementation.py` to verify data subject rights implementation

5. **Security Measures**
   - Authentication and authorization
   - Encryption in transit (TLS/SSL)
   - Access controls and audit logs
   - Vulnerability assessments
   - Use `scripts/security_audit.py` to check security implementations

### Reference Materials

When detailed information is needed, load relevant reference documents:

- `references/gdpr_articles.md` - Complete GDPR articles and requirements
- `references/personal_data_categories.md` - Categories of personal data and special categories
- `references/legal_bases.md` - Legal bases for processing and when to use each
- `references/dsr_requirements.md` - Data subject rights implementation requirements
- `references/security_measures.md` - Technical and organizational security measures
- `references/breach_procedures.md` - Data breach notification requirements
- `references/dpia_guidelines.md` - When and how to conduct DPIAs
- `references/international_transfers.md` - Rules for international data transfers

### Reporting Findings

Structure audit findings using the following format:

1. **Executive Summary** - High-level overview of compliance status
2. **Critical Issues** - Violations that require immediate attention
3. **High-Priority Recommendations** - Important improvements needed
4. **Medium-Priority Recommendations** - Suggested enhancements
5. **Compliant Areas** - Aspects that meet GDPR requirements
6. **Next Steps** - Actionable items with priorities

For each finding, include:
- **Issue Description** - What the problem is
- **GDPR Article(s)** - Which requirements are affected
- **Risk Level** - Critical/High/Medium/Low
- **Current Implementation** - What exists now
- **Recommendation** - Specific steps to resolve
- **Code References** - File paths and line numbers where applicable

### Best Practices

Follow these principles when conducting audits:

1. **Be Thorough** - Check all areas where personal data might be processed
2. **Be Specific** - Provide exact file paths, line numbers, and code examples
3. **Be Practical** - Offer actionable recommendations with implementation guidance
4. **Be Risk-Aware** - Prioritize findings based on potential harm to data subjects
5. **Be Educational** - Explain why something is a GDPR concern
6. **Document Everything** - Create a clear audit trail

### Common GDPR Violations to Check

Always verify the following common compliance issues:

- Missing or inadequate consent mechanisms
- No privacy policy or outdated privacy notices
- Lack of data retention/deletion schedules
- No encryption for sensitive personal data
- Missing data breach notification procedures
- Inadequate vendor/processor agreements
- No data protection impact assessments for high-risk processing
- Failure to implement data subject rights
- Excessive data collection (data minimization violations)
- No legal basis documented for processing activities
- Insufficient access controls
- No audit logging for personal data access
- International transfers without adequate safeguards

## Using the Scripts

The skill includes Python scripts for **static file analysis** in the `scripts/` directory:

- **scan_data_collection.py** - Scans source code files for data collection patterns (forms, inputs, API endpoint definitions)
- **analyze_database_schema.py** - Analyzes database schema files (SQL DDL, migration files, ORM models) for personal data fields
- **check_dsr_implementation.py** - Scans code for data subject rights endpoint implementations
- **security_audit.py** - Reviews code and configuration files for security patterns
- **generate_audit_report.py** - Formats findings into a structured audit report (Markdown)

**Important:** All scripts work with **static files only** (source code, schemas, configuration files). They do NOT:
- Connect to live databases or running systems
- Execute code or make network requests
- Require system access or credentials
- Modify any files

Execute scripts when relevant to automate parts of the audit process. Scripts are defensive security tools designed to identify compliance issues for remediation, not to exploit systems.

## Note on Defensive Security

This skill is designed exclusively for defensive security purposes: identifying compliance gaps, recommending improvements, and helping organizations protect personal data. Do not use this skill to exploit vulnerabilities, harvest data, or circumvent security measures.
