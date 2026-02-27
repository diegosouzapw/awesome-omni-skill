---
name: tm-init
description: Initialize a threat modeling project by analyzing architecture documentation. Creates threat model structure with asset inventory, data flows, trust boundaries, and attack surface mapping. Use when starting new threat modeling work, setting up threat model for a project, or creating initial security assessment.
allowed-tools: Read, Write, Glob, Grep, Bash(mkdir:*), Bash(ls:*)
---

# Threat Model Initialization

## Purpose

Initialize a comprehensive threat model by analyzing your system's architecture documentation. This skill discovers and catalogs:

- **Assets**: Systems, data stores, services, and integrations
- **Data Flows**: How data moves between components
- **Trust Boundaries**: Where privilege levels change
- **Attack Surface**: Entry points exposed to potential attackers

## Usage

```
/tm-init [--docs <path>] [--scope <pattern>] [--framework stride|pasta]
```

**Arguments** (parsed from $ARGUMENTS):
- `--docs <path>`: Path to architecture documentation (default: `./docs`)
- `--scope <pattern>`: Limit analysis to matching components
- `--framework`: Threat framework to use (default: `stride`)

## Process

### Step 1: Discover Documentation

Scan the documentation directory for architecture artifacts:

```
Glob patterns to search:
- **/*.md (Markdown documentation)
- **/README* (Project readmes)
- **/openapi.yaml, **/openapi.json (API specs)
- **/swagger.* (Swagger specs)
- **/*.mmd, **/*.puml (Diagrams)
- **/docker-compose.* (Infrastructure)
- **/Dockerfile* (Containerization)
- **/*.tf (Terraform)
- **/k8s/**, **/kubernetes/** (Kubernetes)
```

### Step 2: Extract Assets

For each component found, identify and classify:

**Asset Types**:
| Type | Description | Look For |
|------|-------------|----------|
| `data-store` | Persists data | PostgreSQL, MySQL, MongoDB, Redis, S3, etc. |
| `service` | Backend logic | API servers, microservices, workers |
| `client` | User interfaces | Web apps, mobile apps, CLIs |
| `integration` | External systems | Payment gateways, email services, third-party APIs |
| `infrastructure` | Platform components | Load balancers, CDN, DNS, queues |
| `identity` | Auth systems | IdP, OAuth providers, SSO |
| `secret` | Sensitive material | API keys, certificates, credentials |

**Data Classifications**:
- `public`: Publicly available information
- `internal`: Internal business data
- `confidential`: Sensitive business data
- `restricted`: PII, PHI, financial data, credentials

### Step 3: Map Data Flows

Identify how data moves between components:

- Source and destination assets
- Data types being transmitted
- Protocol (HTTP, HTTPS, gRPC, WebSocket, etc.)
- Authentication method
- Encryption status
- Whether it crosses a trust boundary

### Step 4: Define Trust Boundaries

Identify where security context changes:

**Trust Boundary Types**:
- `network`: Public/DMZ/Internal network segmentation
- `process`: Process/container isolation
- `privilege`: User/admin/system privilege changes
- `environment`: Dev/staging/prod boundaries
- `organizational`: Third-party/vendor boundaries
- `data-classification`: Sensitivity level changes

### Step 5: Catalog Attack Surface

Document all entry points:

**Attack Surface Types**:
- `api`: REST, GraphQL, gRPC endpoints
- `web-ui`: Web application interfaces
- `mobile`: Mobile application entry points
- `cli`: Command-line interfaces
- `admin`: Administrative interfaces
- `integration`: Webhooks, callbacks
- `file-upload`: File upload functionality
- `message-queue`: Message queue consumers

### Step 6: Generate Diagrams

Create Mermaid diagrams for visualization.

## Output Structure

Create the following directory structure:

```
.threatmodel/
├── config.yaml
├── state/
│   ├── assets.json
│   ├── dataflows.json
│   ├── trust-boundaries.json
│   ├── attack-surface.json
│   └── sequences.json
├── diagrams/
│   ├── architecture.mmd
│   ├── dataflow.mmd
│   └── trust-boundaries.mmd
├── reports/
├── baseline/
└── policies/
```

## Config File Template

Create `.threatmodel/config.yaml`:

```yaml
project:
  name: "[Project Name]"
  version: "1.0.0"
  description: "[Description]"

analysis:
  framework: "stride"
  depth: "standard"

documentation:
  paths:
    - "./docs"
  patterns:
    - "**/*.md"
    - "**/openapi.yaml"

verification:
  code_paths:
    - "./src"
  exclude_paths:
    - "./node_modules"
    - "./**/*.test.*"

compliance:
  frameworks:
    - owasp
```

## JSON Output Format

### assets.json
```json
{
  "version": "1.0",
  "generated": "ISO-8601 timestamp",
  "assets": [
    {
      "id": "asset-001",
      "name": "User Database",
      "type": "data-store",
      "classification": "restricted",
      "description": "PostgreSQL database storing user data",
      "owner": "platform-team",
      "data_types": ["pii", "credentials"],
      "code_references": ["src/db/connection.ts"]
    }
  ]
}
```

### dataflows.json
```json
{
  "version": "1.0",
  "generated": "ISO-8601 timestamp",
  "dataflows": [
    {
      "id": "flow-001",
      "name": "User Login",
      "source": {"asset_id": "asset-001", "component": "LoginPage"},
      "destination": {"asset_id": "asset-002", "component": "AuthService"},
      "data_types": ["credentials"],
      "protocol": "HTTPS",
      "encryption": {"in_transit": true},
      "crosses_trust_boundary": true,
      "trust_boundary_id": "tb-001"
    }
  ]
}
```

## Instructions for Claude

When executing this skill:

1. **Ask for documentation path** if not provided in arguments

2. **Explore the documentation**:
   - Use Glob to find all relevant files
   - Read architecture docs, README files, API specs
   - Look for existing diagrams or system descriptions

3. **Build understanding of the system**:
   - List all named components
   - Understand how they connect
   - Note external dependencies
   - Identify where data enters/exits

4. **Create the threat model structure**:
   - Create `.threatmodel/` directory
   - Write config.yaml with project info
   - Write each state file with discovered data
   - Generate Mermaid diagrams

5. **Validate completeness**:
   - Every asset should have at least one data flow
   - Every external-facing component should be in attack surface
   - Trust boundaries should be identified

6. **Write visual discovery report** (`.threatmodel/reports/discovery-report.md`):
   ```markdown
   # Discovery Report

   **Project**: [Name]
   **Generated**: [Date]

   ## System Overview

   ```
   DISCOVERY SUMMARY
   ═══════════════════════════════════════════════════════════

   ASSETS DISCOVERED: 14
   ─────────────────────────────────────────────────────────
   Services      │████████████████░░░░░░░░░░░░░░░░░░░░░░░░│  4
   Data Stores   │████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░│  3
   Clients       │████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│  2
   Integrations  │████████████████████░░░░░░░░░░░░░░░░░░░░│  5

   DATA FLOWS: 22 (8 cross trust boundaries)
   TRUST BOUNDARIES: 5
   ATTACK SURFACE ENTRIES: 12
   ```

   ## Assets by Classification

   | Asset | Type | Classification |
   |-------|------|----------------|
   | User Database | data-store | Restricted |
   | API Gateway | service | Internal |
   ...
   ```

7. **Console summary** (also display to user):
   ```
   Threat Model Initialized
   ========================

   Project: [Name]
   Framework: STRIDE

   Discovered:
     - X assets (breakdown by type)
     - Y data flows (Z cross trust boundaries)
     - N trust boundaries
     - M attack surface entries

   Created:
     .threatmodel/config.yaml
     .threatmodel/state/assets.json
     .threatmodel/state/dataflows.json
     .threatmodel/state/trust-boundaries.json
     .threatmodel/state/attack-surface.json
     .threatmodel/reports/discovery-report.md
     .threatmodel/diagrams/architecture.mmd
     .threatmodel/diagrams/dataflow.mmd

   Next Steps:
     Run /tm-threats to analyze threats
   ```

## Reference Files

- [STRIDE Framework](../../shared/frameworks/stride.md)
- [Data Schema](../../shared/schemas/threat-model-schema.json)
