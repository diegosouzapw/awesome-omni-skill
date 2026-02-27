---
name: saas-compliance-frameworks
description: Security and compliance requirements for SaaS applications. Covers SOC 2, GDPR, HIPAA, and common compliance patterns with implementation guidance.
allowed-tools: Read, Glob, Grep, Task, mcp__perplexity__search, mcp__perplexity__reason, mcp__microsoft-learn__microsoft_docs_search, mcp__microsoft-learn__microsoft_docs_fetch
---

# SaaS Compliance Frameworks Skill

Guidance for implementing security and compliance requirements in SaaS applications.

## When to Use This Skill

Use this skill when:

- **Saas Compliance Frameworks tasks** - Working on security and compliance requirements for saas applications. covers soc 2, gdpr, hipaa, and common compliance patterns with implementation guidance
- **Planning or design** - Need guidance on Saas Compliance Frameworks approaches
- **Best practices** - Want to follow established patterns and standards

## Overview

SaaS applications serving enterprise customers typically need compliance certifications. This skill covers the major frameworks and their implementation requirements.

## Framework Comparison

```text
+------------------------------------------------------------------+
|                  Compliance Framework Overview                    |
+------------------------------------------------------------------+
| Framework | Focus Area        | Required By              | Audit  |
+-----------+-------------------+--------------------------+--------+
| SOC 2     | Security controls | Enterprise B2B           | Annual |
| GDPR      | Data privacy      | EU customers/data        | N/A    |
| HIPAA     | Health data       | Healthcare/PHI           | Annual |
| PCI DSS   | Payment data      | Card payments            | Annual |
| ISO 27001 | InfoSec mgmt      | Enterprise/Government    | Annual |
| CCPA      | Privacy (CA)      | California consumers     | N/A    |
+-----------+-------------------+--------------------------+--------+
```

## SOC 2

### Trust Service Criteria

```text
SOC 2 Trust Service Criteria:
+------------------------------------------------------------------+
| Criteria       | Focus                    | Key Controls         |
+----------------+--------------------------+----------------------+
| Security       | Protection from          | Access control,      |
| (Required)     | unauthorized access      | encryption, firewall |
+----------------+--------------------------+----------------------+
| Availability   | System uptime and        | Monitoring, DR/BC,   |
|                | accessibility            | SLAs, redundancy     |
+----------------+--------------------------+----------------------+
| Processing     | Complete, accurate       | QA, validation,      |
| Integrity      | processing               | error handling       |
+----------------+--------------------------+----------------------+
| Confidentiality| Protection of            | Encryption, access   |
|                | confidential info        | restrictions, DLP    |
+----------------+--------------------------+----------------------+
| Privacy        | Collection, use, and     | Consent, data rights,|
|                | retention of PII         | retention policies   |
+----------------+--------------------------+----------------------+
```

### SOC 2 Implementation Checklist

```text
Security Controls:
[ ] Multi-factor authentication (MFA) for all users
[ ] Role-based access control (RBAC)
[ ] Encryption at rest (AES-256)
[ ] Encryption in transit (TLS 1.2+)
[ ] Network segmentation/firewalls
[ ] Vulnerability scanning (regular)
[ ] Penetration testing (annual)
[ ] Security awareness training
[ ] Incident response plan
[ ] Vendor security assessments

Availability Controls:
[ ] Uptime monitoring and alerting
[ ] Disaster recovery plan
[ ] Regular backups (tested)
[ ] Redundant infrastructure
[ ] Documented SLAs
[ ] Capacity planning

Processing Integrity:
[ ] Input validation
[ ] Error handling and logging
[ ] Data integrity checks
[ ] Change management process
[ ] QA/testing procedures

Confidentiality:
[ ] Data classification policy
[ ] Access logging and monitoring
[ ] Secure data disposal
[ ] NDA with vendors
[ ] DLP controls (if applicable)
```

### SOC 2 Type I vs Type II

```text
Comparison:
+------------------------------------------------------------------+
| Aspect          | Type I              | Type II                  |
+-----------------+---------------------+--------------------------+
| Scope           | Point-in-time       | Period of time (6-12 mo) |
| Proves          | Controls designed   | Controls operating       |
| Duration        | 1-2 months          | 6-12 months observation  |
| Cost            | Lower               | Higher                   |
| Trust Level     | Lower               | Higher (preferred)       |
| Start With      | Type I first        | Then Type II             |
+------------------------------------------------------------------+
```

## GDPR

### Key Requirements

```text
GDPR Principles:
+------------------------------------------------------------------+
| Principle           | Requirement                                |
+---------------------+--------------------------------------------+
| Lawfulness          | Valid legal basis for processing           |
| Purpose Limitation  | Collect for specified, explicit purposes   |
| Data Minimization   | Only collect what's necessary              |
| Accuracy            | Keep data accurate and up to date          |
| Storage Limitation  | Don't keep longer than needed              |
| Integrity           | Protect against unauthorized processing    |
| Accountability      | Demonstrate compliance                     |
+------------------------------------------------------------------+
```

### Data Subject Rights

```csharp
// Interface for handling data subject requests
public interface IDataSubjectRequestHandler
{
    // Right to Access (Art. 15)
    Task<PersonalDataExport> ExportPersonalDataAsync(
        Guid userId,
        CancellationToken ct = default);

    // Right to Erasure / Right to be Forgotten (Art. 17)
    Task<DeletionResult> DeletePersonalDataAsync(
        Guid userId,
        CancellationToken ct = default);

    // Right to Rectification (Art. 16)
    Task UpdatePersonalDataAsync(
        Guid userId,
        PersonalDataUpdate update,
        CancellationToken ct = default);

    // Right to Data Portability (Art. 20)
    Task<byte[]> ExportPortableDataAsync(
        Guid userId,
        string format = "json",  // or "csv"
        CancellationToken ct = default);

    // Right to Restriction (Art. 18)
    Task RestrictProcessingAsync(
        Guid userId,
        CancellationToken ct = default);
}
```

### GDPR Implementation Patterns

```csharp
// Personal data inventory tracking
public sealed record PersonalDataField
{
    public required string FieldName { get; init; }
    public required string DataCategory { get; init; }  // "identity", "contact", "financial"
    public required string LegalBasis { get; init; }    // "consent", "contract", "legitimate_interest"
    public required string RetentionPeriod { get; init; }
    public required bool IsRequired { get; init; }
    public string? Purpose { get; init; }
}

// Consent management
public sealed record ConsentRecord
{
    public required Guid UserId { get; init; }
    public required string Purpose { get; init; }
    public required bool Granted { get; init; }
    public required DateTimeOffset Timestamp { get; init; }
    public required string Source { get; init; }  // "signup_form", "settings", "api"
    public required string Version { get; init; }  // Policy version
    public string? IpAddress { get; init; }
}

// Data retention enforcement
public sealed class DataRetentionService(IDbContext db, ILogger<DataRetentionService> logger)
{
    public async Task EnforceRetentionPoliciesAsync(CancellationToken ct)
    {
        // Delete expired personal data based on retention policies
        var policies = await GetRetentionPoliciesAsync(ct);

        foreach (var policy in policies)
        {
            var cutoffDate = DateTimeOffset.UtcNow - policy.RetentionPeriod;

            var deleted = await db.Set<PersonalData>()
                .Where(d => d.Category == policy.Category)
                .Where(d => d.LastActivityDate < cutoffDate)
                .Where(d => !d.HasLegalHold)
                .ExecuteDeleteAsync(ct);

            logger.LogInformation(
                "Deleted {Count} records for category {Category} older than {Cutoff}",
                deleted, policy.Category, cutoffDate);
        }
    }
}
```

## HIPAA

### Protected Health Information (PHI)

```text
PHI Identifiers (18 types):
+------------------------------------------------------------------+
| Category        | Examples                                       |
+-----------------+------------------------------------------------+
| Direct          | Name, SSN, medical record #, health plan #     |
| Geographic      | Address, city, state, ZIP (if <20K population) |
| Temporal        | Dates (birth, admission, discharge, death)     |
| Contact         | Phone, fax, email, URLs, IP addresses          |
| Biometric       | Fingerprints, voice prints, photos             |
| Unique          | Account #, vehicle ID, device serial #         |
+------------------------------------------------------------------+
```

### HIPAA Safeguards

```text
Administrative Safeguards:
[ ] Security Officer designation
[ ] Risk analysis (annual)
[ ] Workforce training
[ ] Access management procedures
[ ] Incident response procedures
[ ] Business Associate Agreements (BAAs)
[ ] Contingency planning

Physical Safeguards:
[ ] Facility access controls
[ ] Workstation security
[ ] Device/media controls
[ ] Disposal procedures

Technical Safeguards:
[ ] Access controls (unique user ID, auto-logoff)
[ ] Audit controls (logging)
[ ] Integrity controls (checksums)
[ ] Transmission security (encryption)
[ ] Authentication
```

### HIPAA Implementation

```csharp
// PHI access logging (required for audit controls)
public sealed class PhiAccessLogger(IAuditRepository repository)
{
    public async Task LogAccessAsync(PhiAccessEvent accessEvent)
    {
        await repository.LogAsync(new AuditEntry
        {
            Timestamp = DateTimeOffset.UtcNow,
            UserId = accessEvent.UserId,
            Action = accessEvent.Action,  // "view", "modify", "export", "delete"
            ResourceType = "PHI",
            ResourceId = accessEvent.PatientId,
            Details = accessEvent.Details,
            IpAddress = accessEvent.IpAddress,
            Success = accessEvent.Success
        });
    }
}

// Minimum necessary access enforcement
public sealed class PhiAccessPolicy
{
    public static bool CanAccess(
        UserRole role,
        PhiCategory category,
        AccessType accessType)
    {
        // Implement minimum necessary principle
        return (role, category, accessType) switch
        {
            (UserRole.Clinician, _, AccessType.Read) => true,
            (UserRole.Clinician, _, AccessType.Write) => true,
            (UserRole.BillingStaff, PhiCategory.Billing, _) => true,
            (UserRole.BillingStaff, PhiCategory.Clinical, _) => false,
            (UserRole.Admin, _, AccessType.Read) => true,
            (UserRole.Admin, _, AccessType.Write) => false,  // Admin shouldn't modify PHI
            _ => false
        };
    }
}
```

## Cross-Framework Controls

### Common Requirements

```text
Controls Required by Multiple Frameworks:
+------------------------------------------------------------------+
| Control                | SOC 2 | GDPR | HIPAA | PCI  | ISO 27001 |
+------------------------+-------+------+-------+------+-----------+
| Encryption at rest     |   X   |  X   |   X   |  X   |     X     |
| Encryption in transit  |   X   |  X   |   X   |  X   |     X     |
| Access control (RBAC)  |   X   |  X   |   X   |  X   |     X     |
| Audit logging          |   X   |  X   |   X   |  X   |     X     |
| Incident response      |   X   |  X   |   X   |  X   |     X     |
| Vendor management      |   X   |  X   |   X   |  X   |     X     |
| Security training      |   X   |      |   X   |  X   |     X     |
| MFA                    |   X   |      |       |  X   |     X     |
| Vulnerability scanning |   X   |      |   X   |  X   |     X     |
| Penetration testing    |   X   |      |       |  X   |     X     |
| Backup and recovery    |   X   |      |   X   |  X   |     X     |
| Data classification    |   X   |  X   |   X   |  X   |     X     |
+------------------------+-------+------+-------+------+-----------+
```

### Multi-Tenant Compliance

```text
Tenant Isolation for Compliance:
+------------------------------------------------------------------+
| Compliance Need          | Isolation Level  | Pattern            |
+--------------------------+------------------+--------------------+
| SOC 2 (standard)         | Logical (Pool)   | RLS + audit        |
| HIPAA BAA                | Logical minimum  | RLS + encryption   |
| HIPAA (strict)           | Database/Silo    | Separate DB        |
| PCI DSS                  | Segment network  | Network isolation  |
| Data sovereignty (GDPR)  | Regional deploy  | Geo-specific infra |
+------------------------------------------------------------------+
```

## Compliance Monitoring

### Continuous Compliance

```csharp
public interface IComplianceMonitor
{
    // Run compliance checks
    Task<ComplianceReport> RunChecksAsync(
        string framework,
        CancellationToken ct = default);

    // Check specific control
    Task<ControlResult> CheckControlAsync(
        string controlId,
        CancellationToken ct = default);

    // Get compliance score
    Task<ComplianceScore> GetScoreAsync(
        string framework,
        CancellationToken ct = default);
}

public sealed record ComplianceScore
{
    public required string Framework { get; init; }
    public required int TotalControls { get; init; }
    public required int PassingControls { get; init; }
    public required int FailingControls { get; init; }
    public required int NotApplicable { get; init; }
    public decimal PercentCompliant => TotalControls > 0
        ? (decimal)PassingControls / TotalControls * 100
        : 0;
}
```

## References

Load for detailed implementation:

- `references/soc2-requirements.md` - SOC 2 control details
- `references/gdpr-implementation.md` - GDPR technical implementation
- `references/hipaa-checklist.md` - HIPAA safeguards checklist

## Related Skills

- `audit-logging` - Immutable audit trails
- `tenant-data-isolation` - Data isolation patterns
- `data-residency` - Geographic data requirements

## MCP Research

For current compliance guidance:

```text
perplexity: "SOC 2 Type II 2024 requirements" "SaaS compliance best practices"
microsoft-learn: "Azure compliance" "SOC 2 Azure" "HIPAA Azure"
```
