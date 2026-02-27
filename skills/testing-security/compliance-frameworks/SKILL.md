---
name: compliance-frameworks
description: ISO 27001, NIST CSF 2.0, CIS Controls v8.1, EU CRA compliance mapping, multi-standard alignment per Hack23 ISMS policies
license: MIT
---

# Compliance Frameworks Skill

## Context

This skill applies when:
- Mapping security controls to compliance frameworks
- Creating compliance documentation
- Conducting compliance audits
- Implementing EU Cyber Resilience Act (CRA) requirements
- Generating compliance evidence portfolios
- Responding to compliance questionnaires
- Preparing for security certifications

This skill enforces **[Hack23 ISMS policies](https://github.com/Hack23/ISMS-PUBLIC)** for multi-standard compliance alignment.

## Rules

### 1. ISO 27001:2022 (Secure Development Policy 📜)

1. **A.5**: Organizational controls (policies, roles)
2. **A.8**: Asset management (handling, classification, deletion)
3. **A.12**: Operations security (change management, vulnerabilities)
4. **A.13**: Communications security (network controls, encryption)
5. **A.14**: System acquisition and development (secure SDLC)

### 2. NIST Cybersecurity Framework 2.0 (Secure Development Policy 📜)

6. **GOVERN**: Risk management, policy, oversight
7. **IDENTIFY**: Asset management, risk assessment
8. **PROTECT**: Access control, data security, awareness
9. **DETECT**: Continuous monitoring, anomaly detection
10. **RESPOND**: Incident analysis, mitigation, communication
11. **RECOVER**: Recovery planning, improvements, communications

### 3. CIS Controls v8.1 (Secure Development Policy 📜)

12. **IG1 (Implementation Group 1)**: Basic cyber hygiene (8 controls)
13. **IG2 (Implementation Group 2)**: Medium-sized organizations (56 additional controls)
14. **IG3 (Implementation Group 3)**: Large organizations/high security (additional 64 controls)

### 4. EU Cyber Resilience Act (Open Source Policy 🛡️)

15. **Conformity Assessment**: CE marking for digital products
16. **Security Updates**: Timely vulnerability patching
17. **Risk Management**: Classification-based security requirements
18. **Documentation**: Technical documentation for 10 years
19. **Incident Reporting**: Report actively exploited vulnerabilities

## Examples

### ✅ Good Pattern: ISO 27001:2022 Control Mapping

```typescript
/**
 * ISO 27001:2022 Control Implementation Matrix
 * 
 * ISMS Policy: Compliance Framework Integration
 * Evidence: https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md#compliance-framework-integration
 */

const iso27001Controls = {
  // A.5 Organizational Controls
  'A.5.1': {
    control: 'Policies for information security',
    implementation: 'SECURITY.md, Open_Source_Policy.md, Secure_Development_Policy.md',
    evidence: [
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/SECURITY.md',
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/Open_Source_Policy.md'
    ],
    status: 'Implemented',
    lastReview: '2026-02-16'
  },
  
  // A.8 Asset Management
  'A.8.3': {
    control: 'Handling of assets',
    implementation: 'Input validation with Zod, data classification per Data_Classification_Policy.md',
    evidence: [
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/src/validation/',
      'https://github.com/Hack23/ISMS-PUBLIC/blob/main/Data_Classification_Policy.md'
    ],
    status: 'Implemented',
    lastReview: '2026-02-16'
  },
  
  'A.8.10': {
    control: 'Information deletion',
    implementation: 'LRU cache with TTL, GDPR right to erasure support',
    evidence: [
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/src/cache.ts',
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/.github/skills/gdpr-compliance/SKILL.md'
    ],
    status: 'Implemented',
    lastReview: '2026-02-16'
  },
  
  // A.12 Operations Security
  'A.12.1.2': {
    control: 'Change management',
    implementation: 'GitHub PRs, code review, CI/CD pipeline',
    evidence: [
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/.github/workflows/',
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/PULL_REQUEST_TEMPLATE.md'
    ],
    status: 'Implemented',
    lastReview: '2026-02-16'
  },
  
  'A.12.6.1': {
    control: 'Management of technical vulnerabilities',
    implementation: 'Dependabot, npm audit, CodeQL, vulnerability remediation SLAs',
    evidence: [
      'https://github.com/Hack23/European-Parliament-MCP-Server/security/dependabot',
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/.github/workflows/codeql.yml',
      'https://github.com/Hack23/ISMS-PUBLIC/blob/main/Open_Source_Policy.md#vulnerability-management'
    ],
    status: 'Implemented',
    lastReview: '2026-02-16'
  },
  
  // A.13 Communications Security
  'A.13.1.1': {
    control: 'Network controls',
    implementation: 'HTTPS only for EP API, TLS 1.3, no open network ports',
    evidence: [
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/src/api/client.ts',
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/SECURITY_HEADERS.md'
    ],
    status: 'Implemented',
    lastReview: '2026-02-16'
  },
  
  // A.14 System Acquisition, Development and Maintenance
  'A.14.2.1': {
    control: 'Secure development policy',
    implementation: 'Secure_Development_Policy.md, security by design, threat modeling',
    evidence: [
      'https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md',
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/THREAT_MODEL.md'
    ],
    status: 'Implemented',
    lastReview: '2026-02-16'
  },
  
  'A.14.2.5': {
    control: 'Secure system engineering principles',
    implementation: 'Defense in depth, fail secure, input validation, least privilege',
    evidence: [
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/ARCHITECTURE.md',
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/.github/skills/security-by-design/SKILL.md'
    ],
    status: 'Implemented',
    lastReview: '2026-02-16'
  }
};

// Export for compliance reporting
export function generateISO27001Report(): string {
  const implemented = Object.values(iso27001Controls).filter(c => c.status === 'Implemented').length;
  const total = Object.keys(iso27001Controls).length;
  
  return `
# ISO 27001:2022 Compliance Report

**Project**: European Parliament MCP Server  
**Date**: ${new Date().toISOString()}  
**Status**: ${implemented}/${total} controls implemented (${Math.round(implemented/total*100)}%)

## Control Implementation Summary

${Object.entries(iso27001Controls).map(([id, control]) => `
### ${id}: ${control.control}

**Implementation**: ${control.implementation}  
**Status**: ${control.status}  
**Last Review**: ${control.lastReview}

**Evidence**:
${control.evidence.map(e => `- ${e}`).join('\n')}
`).join('\n')}

## Compliance Statement

This system implements security controls aligned with ISO 27001:2022 standard, demonstrating commitment to information security management per [Hack23 ISMS](https://github.com/Hack23/ISMS-PUBLIC).
  `;
}
```

**Policy Reference**: [Secure Development Policy Section 📜](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md#compliance-framework-integration)

**Evidence**: [CIA ISO 27001 Mapping](https://github.com/Hack23/cia/blob/master/ISO27001_MAPPING.md)

### ✅ Good Pattern: NIST CSF 2.0 Function Mapping

```typescript
/**
 * NIST Cybersecurity Framework 2.0 Implementation
 * 
 * Functions: GOVERN, IDENTIFY, PROTECT, DETECT, RESPOND, RECOVER
 */

const nistCSF2Mapping = {
  // GOVERN (GV)
  'GV.OC-01': {
    function: 'GOVERN',
    category: 'Organizational Context',
    subcategory: 'Organizational mission, objectives, and activities are understood',
    implementation: 'ISMS policies define organizational security requirements',
    evidence: 'https://github.com/Hack23/ISMS-PUBLIC/blob/main/Information_Security_Policy.md'
  },
  
  'GV.RM-01': {
    function: 'GOVERN',
    category: 'Risk Management',
    subcategory: 'Risk management objectives are established',
    implementation: 'Threat modeling, risk register, classification framework',
    evidence: [
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/THREAT_MODEL.md',
      'https://github.com/Hack23/ISMS-PUBLIC/blob/main/Risk_Register.md'
    ]
  },
  
  // IDENTIFY (ID)
  'ID.AM-01': {
    function: 'IDENTIFY',
    category: 'Asset Management',
    subcategory: 'Physical devices and systems are inventoried',
    implementation: 'SBOM generation, dependency tracking, asset inventory',
    evidence: [
      'https://github.com/Hack23/European-Parliament-MCP-Server/releases',
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/package-lock.json'
    ]
  },
  
  'ID.RA-01': {
    function: 'IDENTIFY',
    category: 'Risk Assessment',
    subcategory: 'Vulnerabilities are identified and documented',
    implementation: 'CodeQL scanning, Dependabot, npm audit, OSSF Scorecard',
    evidence: [
      'https://github.com/Hack23/European-Parliament-MCP-Server/security/code-scanning',
      'https://securityscorecards.dev/viewer/?uri=github.com/Hack23/European-Parliament-MCP-Server'
    ]
  },
  
  // PROTECT (PR)
  'PR.AC-01': {
    function: 'PROTECT',
    category: 'Access Control',
    subcategory: 'Identities and credentials are issued, managed, and verified',
    implementation: 'MCP stdio transport (process-level isolation), no network auth needed',
    evidence: 'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/src/index.ts'
  },
  
  'PR.DS-01': {
    function: 'PROTECT',
    category: 'Data Security',
    subcategory: 'Data-at-rest is protected',
    implementation: 'HTTPS for transit, no persistent storage of personal data',
    evidence: 'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/src/api/client.ts'
  },
  
  // DETECT (DE)
  'DE.CM-01': {
    function: 'DETECT',
    category: 'Continuous Monitoring',
    subcategory: 'Networks and network services are monitored',
    implementation: 'Audit logging, error monitoring, GDPR access logs',
    evidence: 'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/src/audit.ts'
  },
  
  // RESPOND (RS)
  'RS.AN-01': {
    function: 'RESPOND',
    category: 'Analysis',
    subcategory: 'Notifications are investigated',
    implementation: 'Vulnerability disclosure process, security incident response',
    evidence: 'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/SECURITY.md'
  },
  
  // RECOVER (RC)
  'RC.RP-01': {
    function: 'RECOVER',
    category: 'Recovery Planning',
    subcategory: 'Recovery plan is executed',
    implementation: 'Incident response procedures, backup and restore capabilities',
    evidence: 'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/SECURITY.md#incident-response'
  }
};
```

**Policy Reference**: [Secure Development Policy Section 📜](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md#compliance-framework-integration)

**Evidence**: [Black Trigram NIST CSF Mapping](https://github.com/Hack23/blacktrigram/blob/main/NIST_CSF_MAPPING.md)

### ✅ Good Pattern: CIS Controls v8.1 Implementation

```typescript
/**
 * CIS Controls v8.1 Safeguards
 * Implementation Groups: IG1 (Basic), IG2 (Foundational), IG3 (Organizational)
 */

const cisControlsMapping = {
  // CIS Control 1: Inventory and Control of Enterprise Assets
  '1.1': {
    control: 'Establish and Maintain Detailed Enterprise Asset Inventory',
    safeguard: 'Basic',
    ig: 'IG1',
    implementation: 'SBOM generation with CycloneDX, dependency tracking',
    evidence: 'https://github.com/Hack23/European-Parliament-MCP-Server/releases/latest/download/sbom.json'
  },
  
  // CIS Control 2: Inventory and Control of Software Assets
  '2.1': {
    control: 'Establish and Maintain Software Inventory',
    safeguard: 'Basic',
    ig: 'IG1',
    implementation: 'package.json, package-lock.json, SBOM',
    evidence: 'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/package.json'
  },
  
  '2.3': {
    control: 'Address Unauthorized Software',
    safeguard: 'Basic',
    ig: 'IG1',
    implementation: 'License scanning with FOSSA, approved license list enforcement',
    evidence: [
      'https://app.fossa.com/projects/git%2Bgithub.com%2FHack23%2FEuropean-Parliament-MCP-Server',
      'https://github.com/Hack23/ISMS-PUBLIC/blob/main/Open_Source_Policy.md#approved-licenses'
    ]
  },
  
  // CIS Control 3: Data Protection
  '3.1': {
    control: 'Establish and Maintain Data Management Process',
    safeguard: 'Basic',
    ig: 'IG1',
    implementation: 'Data classification, GDPR compliance, data minimization',
    evidence: 'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/.github/skills/gdpr-compliance/SKILL.md'
  },
  
  '3.3': {
    control: 'Configure Data Access Control Lists',
    safeguard: 'Basic',
    ig: 'IG1',
    implementation: 'Least privilege, process-level isolation via stdio',
    evidence: 'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/src/index.ts'
  },
  
  // CIS Control 8: Audit Log Management
  '8.2': {
    control: 'Collect Audit Logs',
    safeguard: 'Basic',
    ig: 'IG1',
    implementation: 'Structured audit logging for all security events',
    evidence: 'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/src/audit.ts'
  },
  
  '8.5': {
    control: 'Collect Detailed Audit Logs',
    safeguard: 'Foundational',
    ig: 'IG2',
    implementation: 'Detailed logs with timestamps, event types, actors, outcomes',
    evidence: 'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/src/audit.ts'
  },
  
  // CIS Control 16: Application Software Security
  '16.1': {
    control: 'Establish and Maintain Secure Application Development Process',
    safeguard: 'Basic',
    ig: 'IG1',
    implementation: 'Secure Development Policy, security by design, threat modeling',
    evidence: 'https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md'
  },
  
  '16.10': {
    control: 'Apply Secure Design Principles in Application Architectures',
    safeguard: 'Foundational',
    ig: 'IG2',
    implementation: 'Defense in depth, fail secure, input validation, least privilege',
    evidence: [
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/ARCHITECTURE.md',
      'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/.github/skills/security-by-design/SKILL.md'
    ]
  }
};
```

**Policy Reference**: [Secure Development Policy Section 📜](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md#compliance-framework-integration)

### ✅ Good Pattern: EU Cyber Resilience Act (CRA) Compliance

```typescript
/**
 * EU Cyber Resilience Act Conformity Assessment
 * 
 * ISMS Policy: Open Source Policy Section 🛡️
 * Evidence: https://github.com/Hack23/ISMS-PUBLIC/blob/main/Open_Source_Policy.md#cra-conformity-assessment-evidence
 */

const craCompliance = {
  product: 'European Parliament MCP Server',
  classification: 'Important (Class I)',  // Based on risk assessment
  
  // Essential Requirements
  essentialRequirements: {
    security: {
      'Art. 10': {
        requirement: 'Products with digital elements shall be delivered without known exploitable vulnerabilities',
        implementation: 'CodeQL scanning, Dependabot, npm audit, OSSF Scorecard ≥7.0',
        evidence: [
          'https://github.com/Hack23/European-Parliament-MCP-Server/security/code-scanning',
          'https://securityscorecards.dev/viewer/?uri=github.com/Hack23/European-Parliament-MCP-Server'
        ]
      },
      'Art. 11': {
        requirement: 'Products shall be delivered with a secure by default configuration',
        implementation: 'Security by design, threat modeling, input validation mandatory',
        evidence: 'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/THREAT_MODEL.md'
      }
    },
    
    updates: {
      'Art. 12': {
        requirement: 'Manufacturers shall provide security updates for the expected lifetime',
        implementation: 'Dependabot automated updates, 5-year support lifecycle',
        evidence: 'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/SECURITY.md#supported-versions'
      }
    },
    
    documentation: {
      'Art. 13': {
        requirement: 'Technical documentation shall be maintained for 10 years',
        implementation: 'SECURITY_ARCHITECTURE.md, THREAT_MODEL.md, compliance mappings maintained in git',
        evidence: 'https://github.com/Hack23/European-Parliament-MCP-Server'
      }
    },
    
    incidentReporting: {
      'Art. 14': {
        requirement: 'Actively exploited vulnerabilities shall be reported within 24 hours',
        implementation: 'Vulnerability disclosure process, CSIRT coordination',
        evidence: 'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/SECURITY.md#reporting-a-vulnerability'
      }
    }
  },
  
  // Conformity Assessment
  conformityAssessment: {
    type: 'Module A (Internal Control)',  // For Class I products
    ceMarking: false,  // Not required for open source tools
    declarationOfConformity: true,
    technicalDocumentation: {
      description: 'European Parliament MCP Server - Model Context Protocol server for parliamentary data',
      architecture: 'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/SECURITY_ARCHITECTURE.md',
      threatModel: 'https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/THREAT_MODEL.md',
      sbom: 'https://github.com/Hack23/European-Parliament-MCP-Server/releases/latest/download/sbom.json'
    }
  }
};
```

**Policy Reference**: [Open Source Policy Section 🛡️](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Open_Source_Policy.md#cra-conformity-assessment-evidence)

## Anti-Patterns

### ❌ Bad: No Compliance Mapping

```
// Just code, no compliance documentation
```

### ❌ Bad: Outdated Compliance Evidence

```
Last compliance review: 2022  // More than 1 year old!
```

## Evidence Portfolio

### Reference Implementations

1. **Citizen Intelligence Agency (CIA)**
   - ISO 27001 Mapping: https://github.com/Hack23/cia/blob/master/ISO27001_MAPPING.md
   - NIST CSF: https://github.com/Hack23/cia/blob/master/NIST_CSF_MAPPING.md
   - CIS Controls: https://github.com/Hack23/cia/blob/master/CIS_CONTROLS_MAPPING.md

2. **Black Trigram Game**
   - Compliance Dashboard: https://github.com/Hack23/blacktrigram/blob/main/COMPLIANCE.md

3. **CIA Compliance Manager**
   - Multi-Framework Mapping: https://github.com/Hack23/cia-compliance-manager/blob/main/COMPLIANCE_MAPPING.md

### Policy Documents

- **Secure Development Policy**: https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md
- **Open Source Policy**: https://github.com/Hack23/ISMS-PUBLIC/blob/main/Open_Source_Policy.md
- **ISMS-PUBLIC Repository**: https://github.com/Hack23/ISMS-PUBLIC

## ISMS Compliance

This skill enforces:
- **CF-001**: ISO 27001:2022 control mapping
- **CF-002**: NIST CSF 2.0 function alignment
- **CF-003**: CIS Controls v8.1 safeguards
- **CF-004**: EU CRA conformity assessment

**Policy Reference**: [Hack23 ISMS-PUBLIC](https://github.com/Hack23/ISMS-PUBLIC)
