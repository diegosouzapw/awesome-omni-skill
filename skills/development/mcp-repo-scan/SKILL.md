---
name: mcp-repo-scan
description: Comprehensive RE-Engine repository health audit, issue resolution, and architectural enhancement through systematic codebase analysis
---

# Purpose
Perform systematic repository integrity audit, resolve architectural/operational deficiencies, and enhance system performance through detailed analysis of RE-Engine's codebase structure, workflow implementations, and MCP integration patterns.

# Pre-Scan Validation Requirements
- [ ] Repository synchronized with latest remote commit
- [ ] Zero uncommitted changes in working directory
- [ ] Full build validation across all components:
  - `engine/`: `npm run build`
  - `web-dashboard/`: `npm run build`
  - `playwright/`: `npm run build`
  - `mcp/*/`: `npm run build` (all MCP servers)

# Detailed Scan Execution Phases

## Phase 1: Repository Structure & Code Quality Audit
1. **Technology Stack Assessment**:
   - TypeScript composition: 60.5% (type safety analysis)
   - JavaScript composition: 25.2% (migration opportunities)
   - Runtime requirements: Node.js v22+, React 18+, Playwright
   - Data layer: PostgreSQL (primary), Redis (caching/queues)

2. **Component Architecture Analysis**:
   - `engine/`: Core backend API, business logic orchestration, message routing engine
   - `web-dashboard/`: React-based approval interface, real-time monitoring dashboard
   - `playwright/`: Browser automation framework for human-in-the-loop workflow execution
   - `mcp/`: Model Context Protocol servers (reengine-core, reengine-browser, reengine-tinyfish)
   - `scripts/`: Deployment automation, utility scripts, maintenance routines

3. **Dependency & Entrypoint Validation**:
   - Package.json dependency audit (security vulnerabilities, outdated packages)
   - Entrypoint file analysis (index.ts, main.js, server.js)
   - Circular dependency detection
   - Unused import elimination

## Phase 2: Flow Analysis & Issue Resolution
4. **Outreach Workflow Mapping**:
   - Lead ingestion → CSV normalization → data enrichment → approval queue population
   - Approval workflow → multi-channel dispatch (WhatsApp API, Telegram Bot, SMTP, LinkedIn API, Facebook Graph)
   - Automated scheduling infrastructure (daily draft generation @ 08:00 UTC, IMAP polling @ 15-min intervals)
   - Response capture → sentiment analysis → CRM integration

5. **End-to-End Flow Debugging**:
   - Request parsing → controller routing → service layer → job queue → persistence layer → external API calls
   - Data transformation validation (DTOs, mappers, serializers)
   - Error propagation analysis and recovery mechanisms
   - Transaction boundary identification and optimization

## Phase 3: Performance Optimization
6. **Development Workflow Enhancement**:
   - Build optimization: `npm run build` (parallel compilation, tree-shaking)
   - Testing strategies: `npm test` (unit), `npm run test:integration` (API), `npm run test:smoke` (E2E)
   - Code quality: `npm run lint` (ESLint), `npm run format` (Prettier), `npm run typecheck` (TypeScript)
   - Local development: `npm run dashboard` (UI), `npm run mcp:start` (servers)

7. **Deployment Pipeline Optimization**:
   - Staging deployment: `npm run deploy:staging` (manual) or GitHub Actions (develop branch auto)
   - Production deployment: `npm run deploy:production` (manual) or GitHub Actions (release branch auto)
   - CI/CD enhancements: security scanning → dependency analysis → build validation → automated deployment

8. **Environment Configuration Management**:
   - Required environment variables (`.env.example` validation)
   - Database connection pooling (PostgreSQL max_connections, Redis cluster config)
   - External service integration health checks (WhatsApp API status, email provider deliverability)

## Phase 4: MCP Server Deep Dive
9. **MCP Server Architecture Review**:
   - `mcp-reengine-core/`: Business logic abstraction layer, tool orchestration
   - `mcp-reengine-browser/`: Playwright integration, browser automation primitives
   - `mcp-reengine-tinyfish/`: Specialized domain tools, custom workflow implementations

10. **MCP Server Implementation Fixes**:
    - Tool implementation validation (input sanitization, output formatting)
    - JSON schema compliance for request/response structures
    - Authentication token validation and refresh mechanisms
    - Error handling standardization (retry policies, circuit breakers)

11. **MCP Performance Validation**:
    - Load testing: `npm run mcp:start` with concurrent request simulation
    - Memory leak detection in long-running processes
    - Tool execution timeout optimization
    - Inter-server communication efficiency

## Phase 5: Documentation Enhancement & Reporting
12. **Documentation Updates**:
    - `docs/ARCHITECTURE.md`: Updated component interaction diagrams
    - `docs/FLOWS.md`: Detailed workflow sequence diagrams
    - `docs/OPERATIONS.md`: Enhanced development/deployment procedures
    - `docs/MCP_SERVERS.md`: Comprehensive MCP server implementation guide

13. **Comprehensive Audit Report Generation**:
    - Issue inventory with severity classification
    - Performance improvement metrics (before/after benchmarks)
    - MCP server optimization results
    - Documentation change log
    - Strategic recommendations for continued optimization

# Deliverables
Complete audit and enhancement package including:
- Automated fix application scripts
- Interactive issue investigation interface
- Continuous optimization monitoring framework