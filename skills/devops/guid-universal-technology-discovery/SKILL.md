---
name: guid-universal-technology-discovery
description: "Discover and identify any technology, framework, or system without prior knowledge. Analyzes files, configurations, dependencies, and usage patterns to understand project technology stack from C source files to Kubernetes manifests."
model: inherit
---

# Universal Technology Discovery

## Pattern Description

**What**: Systematically identifies and documents all technologies, frameworks, libraries, and their versions used within a project, from low-level source files to high-level infrastructure manifests.

**When**: Use this skill when you need to understand the technology stack of a project, especially when:
- Analyzing unfamiliar projects or codebases.
- Working with mixed or hybrid technology environments.
- Discovering technologies beyond the obvious or explicitly stated ones.
- Understanding how technologies are actually used versus just installed.
- Identifying custom, proprietary, or legacy technologies.

**Context**: Crucial for understanding the technical landscape of any software project, regardless of its size or complexity. It applies to diverse environments, including on-premise, cloud-based, containerized, serverless deployments, and multi-language projects.

## Project-Specific Conventions

### Universal Technology Indicators (Discovery Patterns)

### Standard Technology Detection Indicators

**Package Manager Usage**
- **Primary Package Manager**: [e.g., `npm`, `yarn`, `pnpm` for Node.js; `pip`, `poetry` for Python; `Maven`, `Gradle` for Java]
- **Dependency Types**: [e.g., `dependencies`, `devDependencies`, `peerDependencies`]
- **Lock File Strategy**: [e.g., always commit `package-lock.json`/`yarn.lock`]

**Framework Detection Indicators**
- **Frontend**: [e.g., presence of `package.json` scripts like `start`, `build`, specific component file extensions like `.jsx`, `.tsx`, `.vue`, `.svelte`]
- **Backend**: [e.g., specific entry points like `app.js`/`server.ts`, database connection strings, framework-specific configuration files]
- **Infrastructure**: [e.g., `Dockerfile`, `docker-compose.yml`, `kubernetes/*.yaml`, `terraform/*.tf`]

**Versioning Strategy**
- **Dependency Pinning**: [e.g., exact versions, caret `^`, tilde `~`]
- **Tooling Versions**: [e.g., Node.js version in `.nvmrc` or `package.json`, Python version in `pyproject.toml`]

### Technology Discovery Protocol (Discovery Approach)

**Step 1: Comprehensive File Analysis**
- **File Discovery**: Use glob patterns to find all relevant files; analyze file types and extensions; examine file sizes and modification dates; map directory structure and organization; identify technology-specific file patterns.
- **Content Analysis**: Parse configuration files for technology indicators; analyze package files for dependencies; study build files for build tools and processes; examine code files for language and framework usage.
- **Pattern Recognition**: Identify technology-specific file patterns; recognize common technology configurations; map technology integration patterns; discover custom or proprietary technologies.

**Step 2: Technology Inference**
- **Syntax Analysis**: Analyze code syntax and structure; identify programming languages and versions; recognize framework-specific code patterns; discover library and API usage patterns.
- **Configuration Analysis**: Parse configuration files and settings; identify technology-specific configurations; map integration and connection settings; discover deployment and operational configurations.
- **Dependency Analysis**: Map direct and transitive dependencies; identify technology versions and compatibility; analyze dependency injection patterns; study service discovery and connection patterns.

**Step 3: Validation and Verification**
- **Usage Verification**: Verify technologies are actually used, not just installed; analyze how technologies are integrated and used; identify technology alternatives and migrations; validate technology versions and compatibility.
- **Integration Validation**: Verify technology integrations work correctly; test communication patterns and protocols; validate data flow and dependencies; confirm technology interactions and relationships.
- **Operational Validation**: Verify deployment and operational configurations; test build and packaging processes; validate infrastructure and hosting setups; confirm monitoring and logging configurations.

### Custom and Mixed Environment Discovery

**Proprietary and Custom Technologies**:
- Analyze custom file formats and configurations; study proprietary APIs and protocols; identify custom build tools and processes; map domain-specific technologies and frameworks; recognize custom integration patterns.

**Legacy Technology Analysis**:
- Identify legacy programming languages and frameworks; analyze outdated patterns and approaches; map modernization and migration opportunities; study maintenance and support patterns; plan technology evolution strategies.

**Mixed Environment Discovery**:
- **Hybrid Technology Stacks**: Map technology boundaries and interactions; identify integration challenges and solutions; study technology transition patterns; analyze communication between different technologies; plan technology consolidation strategies.
- **Multi-Language Projects**: Identify language boundaries and responsibilities; map inter-language communication patterns; analyze shared resources and data structures; study build and packaging across languages; understand development workflow coordination.

## Common Pitfalls

### ❌ Surface-Level Analysis
**Problem**: Relying only on package.json or visible dependencies; assuming technologies based on file extensions alone; ignoring configuration files and build scripts; missing indirectly used technologies.
**Why It Fails**: Leads to an inaccurate understanding of the actual runtime environment and potential conflicts.
**Better Approach**: Always perform multi-layer analysis (files, configuration, code) and consider deployment and operational environments.

### ❌ Static vs. Dynamic Analysis Confusion
**Problem**: Confusing installed technologies with actually used ones; missing runtime-discovered dependencies; ignoring dynamically loaded technologies or those discovered through configuration.
**Why It Fails**: Provides an incomplete or incorrect view of the active technology stack.
**Better Approach**: Use both static analysis and runtime behavior understanding; always verify technologies are actually used.

### ❌ Complex Integrations Oversight
**Problem**: Missing technologies behind reverse proxies or gateways; ignoring external service integrations; overlooking technologies in different repositories; missing technologies discovered through environment variables.
**Why It Fails**: Leads to blind spots in the system's architecture and potential integration failures.
**Better Approach**: Analyze all project areas, not just obvious ones; test technology integrations and dependencies; validate configuration accuracy and completeness.

### ❌ Outdated Information
**Problem**: The documented technology stack does not reflect the current state of the project.
**Why It Fails**: Causes confusion, incorrect assumptions, and potential security risks from unpatched versions.
**Better Approach**: Automate technology detection during CI/CD or periodically review and update documentation.

## Notes

The universal technology discovery provides the foundation for comprehensive project analysis by identifying all technologies present, regardless of how they're implemented or integrated. Effective discovery requires comprehensive analysis, rigorous validation, and clear documentation.

## Related Resources

### Related Skills
- **convention-extraction**: For understanding how discovered technologies influence coding styles.
- **integration-mapping-discovery**: For mapping how technologies connect.
- **operational-intelligence-discovery**: For understanding deployment and maintenance.
- **repository-pattern-recognition**: For understanding how technologies influence repository structure and patterns.
- **tool-selection-guidance**: For selecting appropriate tools based on discovered technology stack.
- **skill-creation**: For creating skills based on discovered technology patterns and conventions.

### Related Agents
- **security-specialist**: For auditing detected technologies for vulnerabilities.
- **devops-specialist**: For understanding how technologies are deployed and managed.
- **architecture-specialist**: For in-depth analysis and recommendation of architectural patterns.

### External Resources
- [Snyk Advisor](https://snyk.io/advisor/)
- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)
- [The Twelve-Factor App](https://12factor.net/)
- [Site Reliability Engineering (SRE) Principles](https://sre.google/sre-book/table-of-contents/)