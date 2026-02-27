---
name: azure-enterprise-governance
description: Enterprise-grade Azure governance, security, and compliance framework. Combines Microsoft Cloud Adoption Framework (CAF) naming standards with comprehensive security architecture (Zero Trust), compliance frameworks (NIST, SOC2, PCI-DSS, HIPAA), and best practices. Provides naming validation, security audits, RBAC design, and compliance checklists for production-ready Azure deployments.
---

# Azure Enterprise Governance Framework

## Overview

Master enterprise-level Azure governance, security, and compliance. This skill combines Microsoft Cloud Adoption Framework (CAF) naming standards with comprehensive security architecture (Zero Trust principles), compliance frameworks (NIST, SOC 2, PCI-DSS, HIPAA), and operational best practices. Design secure, compliant, and scalable Azure infrastructure aligned with industry standards.

## Core Capabilities

### 1. Naming Convention Design & Validation
Design and validate Azure resource naming strategies that are:
- **Compliant**: Follow Microsoft Cloud Adoption Framework (CAF) standards
- **Scalable**: Support hundreds of resources across multiple environments
- **Auditable**: Enable automated compliance checking
- **Human-friendly**: Clear, consistent, and easy to parse

**🎯 Interactive Decision Guide:**

Use `references/naming-decision-guide.md` for step-by-step guidance:
- **Decision flowchart** - Visual guide for choosing naming patterns
- **Questionnaire** - Answer questions to determine your needs
- **Template library** - Ready-to-use naming templates for common scenarios:
  - Lab/Experimental environments (for learning and POC)
  - Single application deployment (simple architectures)
  - Microservices architecture (distributed systems)
  - Multi-tenant SaaS (platform services)
- **Interactive naming generator** - Auto-generate naming schemes

**📚 Detailed Reference:**

See `references/naming-conventions.md` for:
- Microsoft-recommended naming format
- Resource type abbreviations from official CAF documentation
- Naming constraints and restrictions per resource type
- Multi-environment naming strategies
- Hierarchical resource organization patterns

**Usage Pattern:**
1. **Start with decision guide**: Run `python scripts/generate_naming.py` for interactive help
2. Review your organization structure (org, department, project)
3. Select naming template based on scenario (lab, app, microservices, multi-tenant)
4. Define abbreviations for resource types and environments
5. Validate naming scheme: `python scripts/validate_naming.py --resource-group <name>`
6. Apply naming scheme consistently across all resources

### 2. Security & Compliance Framework
Implement security controls across Azure infrastructure using:
- **Zero Trust Architecture**: Assume breach, verify everything
- **NIST Cybersecurity Framework**: Security standards and controls
- **Azure Well-Architected Framework**: Security pillar best practices
- **Managed Identity**: Eliminate shared credentials and key management

Reference `references/security-best-practices.md` for:
- Identity and access management (IAM) patterns
- Network security and isolation strategies
- Data protection and encryption requirements
- Compliance frameworks (NIST, SOC 2, PCI-DSS, HIPAA)
- Security assessment checklist
- Common security misconfigurations and fixes

**Key Security Principles:**
- Never use secrets in code (use Key Vault + Managed Identity)
- Implement defense in depth (network, application, data layers)
- Enable monitoring and alerting on all resources
- Enforce role-based access control (RBAC)
- Require multi-factor authentication (MFA)
- Use private endpoints for sensitive services
- Encrypt data in transit and at rest
- Regular security assessments and penetration testing

### 3. Automated Validation & Compliance Checking
Validate resource naming and security configurations using Python scripts in `scripts/`:

**validate_naming.py**
- Check resource names against CAF standards
- Verify naming constraints (length, characters, uniqueness)
- Detect naming pattern violations
- Generate compliance reports
- Usage: `python scripts/validate_naming.py --resource-group mygroup --check-all`

**security_audit.py**
- Audit Azure resources for security misconfigurations
- Check for managed identity usage
- Verify encryption settings (data, transport)
- Validate network isolation (NSGs, private endpoints)
- Identify overly permissive RBAC assignments
- Generate security assessment report
- Usage: `python scripts/security_audit.py --resource-group mygroup --severity high`

**compliance_checker.py**
- Verify compliance with organizational policies
- Check naming convention compliance
- Validate security controls alignment
- Generate audit trail for compliance documentation
- Support multiple compliance frameworks (NIST, SOC2, etc.)
- Usage: `python scripts/compliance_checker.py --framework nist --resource-group mygroup`

### 4. Organization Hierarchy & Governance

Structure Azure resources using hierarchies that support:
- **Multi-tenant organizations**: Separate by customer/tenant
- **Environment management**: dev, test, stg, prod isolation
- **Cost allocation**: Easy chargeback and cost center mapping
- **Access control**: Align resource hierarchy with RBAC
- **Disaster recovery**: Regional isolation and failover strategy

**Hierarchy Template:**
```
Subscription (billing boundary)
├── Resource Group: rg-{org}-{workload}-{env}
│   ├── Compute: asp-{org}-{workload}-{env}
│   ├── Storage: st{org}{env}001
│   ├── Database: sqldb-{org}-{workload}-{env}
│   └── Security: kv-{org}-{env}
├── Resource Group: rg-{org}-{workload}-{env}
└── Resource Group: rg-{org}-platform-{env}
```

## Best Practices Checklist

### Before Deployment

- [ ] **Naming Validated**: Run `validate_naming.py` against all resource names
- [ ] **Security Review**: Complete `security-best-practices.md` checklist
- [ ] **RBAC Configured**: Use managed identities, no shared credentials
- [ ] **Encryption Enabled**: Data at rest and in transit encrypted
- [ ] **Monitoring Setup**: Application Insights, Log Analytics configured
- [ ] **Network Isolation**: Private endpoints for sensitive services
- [ ] **Compliance Check**: Run `compliance_checker.py` for your framework
- [ ] **Documentation**: Resource hierarchy and naming documented
- [ ] **Access Control**: Principle of least privilege applied
- [ ] **Backup Strategy**: Automated backups configured and tested

### Post-Deployment

- [ ] **Audit Baseline**: Run `security_audit.py` to establish baseline
- [ ] **Monitoring Active**: Alerts configured for security events
- [ ] **Regular Reviews**: Monthly compliance and security reviews
- [ ] **Access Reviews**: Quarterly RBAC access reviews
- [ ] **Threat Analysis**: Regular threat modeling and updates
- [ ] **Incident Response**: Runbooks documented and tested
- [ ] **Disaster Recovery**: DR procedures documented and practiced

## Common Use Cases

### Scenario 1: Migrate 100+ Resources to Compliant Naming
```bash
# Validate current resources
python scripts/validate_naming.py --resource-group oldgroup --check-all

# Identify violations
python scripts/validate_naming.py --resource-group oldgroup --report violations.json

# Create migration plan with new compliant names
# Use references/naming-conventions.md to determine new names
```

### Scenario 2: Implement Zero Trust Security
1. Review `references/security-best-practices.md` section on Zero Trust
2. Audit current state: `python scripts/security_audit.py --resource-group mygroup`
3. Identify gaps compared to Zero Trust checklist
4. Implement controls: Managed Identity, Private Endpoints, NSGs
5. Re-audit and validate: `python scripts/security_audit.py --resource-group mygroup`

### Scenario 3: Prepare for SOC 2 / HIPAA Compliance
1. Select compliance framework: `python scripts/compliance_checker.py --framework soc2`
2. Review required controls in `references/security-best-practices.md`
3. Generate gap analysis report
4. Implement required security controls
5. Document compliance evidence and controls
6. Schedule regular audits: `python scripts/compliance_checker.py --framework soc2 --schedule monthly`

### Scenario 4: Design Multi-Tenant Naming Strategy
1. Review `references/naming-conventions.md` section on multi-tenant patterns
2. Define tenant/customer identifier (e.g., tenant ID, subdomain)
3. Create resource group naming pattern: `rg-{tenant}-{workload}-{env}`
4. Map resources to resource groups by tenant
5. Enforce access isolation using RBAC and subscriptions per tenant

## Related Skills

- **azure-expert**: Comprehensive Azure service architecture and deployment
- **skill-creator**: Create and manage AI skills in VS Code

## Additional Resources

- [Microsoft Cloud Adoption Framework - Naming](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming)
- [Microsoft Cloud Adoption Framework - Naming Abbreviations](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations)
- [Azure Well-Architected Framework - Security Pillar](https://learn.microsoft.com/azure/well-architected/security/)
- [Azure Security Best Practices](https://learn.microsoft.com/security/benchmark/azure/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework/)
- [Zero Trust Principles - Microsoft](https://learn.microsoft.com/security/zero-trust/)
