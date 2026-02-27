---
name: security-scanning-security-hardening
description: Coordinate multi-layer security scanning and hardening across application, infrastructure, and compliance controls. 
category: Security & Systems
source: antigravity
tags: [api, ai, agent, workflow, template, design, document, security, vulnerability, aws]
url: https://github.com/sickn33/antigravity-awesome-skills/tree/main/skills/security-scanning-security-hardening
---


Implement comprehensive security hardening with defense-in-depth strategy through coordinated multi-agent orchestration:

[Extended thinking: This workflow implements a defense-in-depth security strategy across all application layers. It coordinates specialized security agents to perform comprehensive assessments, implement layered security controls, and establish continuous security monitoring. The approach follows modern DevSecOps principles with shift-left security, automated scanning, and compliance validation. Each phase builds upon previous findings to create a resilient security posture that addresses both current vulnerabilities and future threats.]

## Use this skill when

- Running a coordinated security hardening program
- Establishing defense-in-depth controls across app, infra, and CI/CD
- Prioritizing remediation from scans and threat modeling

## Do not use this skill when

- You only need a quick scan without remediation work
- You lack authorization for security testing or changes
- The environment cannot tolerate invasive security controls

## Instructions

1. Execute Phase 1 to establish a security baseline.
2. Apply Phase 2 remediations for high-risk issues.
3. Implement Phase 3 controls and validate defenses.
4. Complete Phase 4 validation and compliance checks.

## Safety

- Avoid intrusive testing in production without approval.
- Ensure rollback plans exist before hardening changes.

## Phase 1: Comprehensive Security Assessment

### 1. Initial Vulnerability Scanning
- Use Task tool with subagent_type="security-auditor"
- Prompt: "Perform comprehensive security assessment on: $ARGUMENTS. Execute SAST analysis with Semgrep/SonarQube, DAST scanning with OWASP ZAP, dependency audit with Snyk/Trivy, secrets detection with GitLeaks/TruffleHog. Generate SBOM for supply chain analysis. Identify OWASP Top 10 vulnerabilities, CWE weaknesses, and CVE exposures."
- Output: Detailed vulnerability report with CVSS scores, exploitability analysis, attack surface mapping, secrets exposure report, SBOM inventory
- Context: Initial baseline for all remediation efforts

### 2. Threat Modeling and Risk Analysis
- Use Task tool with subagent_type="security-auditor"
- Prompt: "Conduct threat modeling using STRIDE methodology for: $ARGUMENTS. Analyze attack vectors, create attack trees, assess business impact of identified vulnerabilities. Map threats to MITRE ATT&CK framework. Prioritize risks based on likelihood and impact."
- Output: Threat model diagrams, risk matrix with prioritized vulnerabilities, attack scenario documentation, business impact analysis
- Context: Uses vulnerability scan results to inform threat priorities

### 3. Architecture Security Review
- Use Task tool with subagent_type="backend-api-security::backend-architect"
- Prompt: "Review architecture for security weaknesses in: $ARGUMENTS. Evaluate service boundaries, data flow security, authentication/authorization architecture, encryption implementation, network segmentation. Design zero-trust architecture patterns. Reference threat model and vulnerability findings."
- Output: Security architecture assessment, zero-trust design recommendations, service mesh security requirements, data classification matrix
- Context: Incorporates threat model to address architectural vulnerabilities

## Phase 2: Vulnerability Remediation

### 4. Critical Vulnerability Fixes
- Use Task tool with subagent_type="security-auditor"
- Prompt: "Coordinate immediate remediation of critical vulnerabilities (CVSS 7+) in: $ARGUMENTS. Fix SQL injections with parameterized queries, XSS with output encoding, authentication bypasses with secure session management, insecure deserialization with input validation. Apply security patches for CVEs."
- Output: Patched code with vulnerability fixes, security patch documentation, regression test requirements
- Context: Addresses high-priority items from vulnerability assessment

### 5. Backend Security Hardening
- Use Task tool with subagent_type="backend-api-security::backend-security-coder"
- Prompt: "Implement comprehensive backend security controls for: $ARGUMENTS. Add input validation with OWASP ESAPI, implement rate limiting and DDoS protection, secure API endpoints with OAuth2/JWT validation, add encryption for data at rest/transit using AES-256/TLS 1.3. Implement secure logging without PII exposure."
- Output: Hardened API endpoints, validation middleware, encryption implementation, secure configuration templates
- Context: Builds upon vulnerability fixes with preventive controls

### 6. Frontend Security Implementation
- Use Task tool with subagent_type="frontend-mobile-security::frontend-security-coder"
- Prompt: "Implement frontend security measures for: $ARGUMENTS. Configure CSP headers with nonce-based policies, implement XSS prevention with DOMPurify, secure authentication flows with PKCE OAuth2, add SRI for external resources, implement secure cookie handling with SameSite/HttpOnly/Secure flags."
- Output:
