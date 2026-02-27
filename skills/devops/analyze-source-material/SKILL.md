---
name: analyze-source-material
description: Analyze mixed repositories (application code + infrastructure + policies + docs) to extract ALL components for ONE unified threat model. Use when analyzing codebases with multiple source types for threat modeling. Architecture modeling only - do NOT identify vulnerabilities.
---

# Source Material Analysis Instructions

## Executive Summary
Analyze mixed repositories (application code + infrastructure + policies + docs) to extract ALL components for ONE unified threat model. Your role: architecture modeling only—extract components, trust zones, and data flows. Do NOT identify vulnerabilities or security flaws; IriusRisk handles that automatically. Create a single comprehensive threat model covering all layers.

## Critical Principle: One Comprehensive Threat Model
Create ONE unified threat model including ALL components from ALL source types. Do NOT create separate models for code vs. infrastructure—IriusRisk works best with a complete, holistic view of the entire system.

## Scope-Aware Analysis for Multi-Repository Projects

**FIRST: Check for repository scope definition** in `.iriusrisk/project.json`:

```json
{
  "name": "E-commerce Platform",
  "reference_id": "ecommerce-platform",
  "scope": "AWS cloud infrastructure via Terraform. Provisions ECS for backend API 
           (api-backend repo), RDS PostgreSQL, ALB, CloudFront for frontend static 
           assets (web-frontend repo). All application components from other repos 
           should run within these AWS services."
}
```

### When Scope is Defined

**Your analysis must be FOCUSED on the scope:**

1. **Understand the scope boundaries:**
   - What is THIS repository responsible for?
   - What components from OTHER repositories are mentioned?
   - What is the architectural relationship to other parts?

2. **Focus your component extraction:**
   - Extract components relevant to THIS repository's scope
   - Note references to components from other repositories
   - Understand how your components relate to the broader system

3. **Scope-specific extraction strategies:**

   **Infrastructure scope** (e.g., "AWS infrastructure", "Kubernetes platform"):
   - **Primary focus**: Cloud resources, networking, compute platforms
   - **Extract**: VPCs, load balancers, container orchestration, managed databases, CDN, storage services
   - **Note**: Application components mentioned in scope (e.g., "backend API from api-backend repo")
   - **Expectation**: Your components will CONTAIN or HOST application components from other repos

   **Application scope** (e.g., "Backend API", "Payment service"):
   - **Primary focus**: Business logic, services, APIs
   - **Extract**: Service endpoints, business logic components, data access layers, auth modules
   - **Note**: Infrastructure mentioned in scope (e.g., "deployed to ECS from terraform repo")
   - **Expectation**: Your components will RUN INSIDE infrastructure components from other repos

   **Frontend scope** (e.g., "React SPA", "Mobile app"):
   - **Primary focus**: Client-side components, UI layers
   - **Extract**: Frontend application, API clients, authentication flows, client-side state management
   - **Note**: Backend services and CDN mentioned in scope
   - **Expectation**: Your components connect to backend APIs and run on CDN/infrastructure

### When Scope is NOT Defined

**Use comprehensive, all-encompassing analysis:**
- Extract ALL components from ALL source types
- Create complete, holistic threat model
- Include application, infrastructure, data, and integration layers
- Standard single-repository analysis approach

## Your Role: Architecture Modeling Only
**Do:** Extract components, trust zones, and data flows  
**Do NOT:** Identify vulnerabilities, threats, security flaws, or speculate about weaknesses  
**Why:** IriusRisk performs all security analysis automatically

## 🚨 CRITICAL: Tags Are For Architecture, NOT Vulnerabilities

When analyzing source code, you will inevitably notice security issues (SQL injection, weak crypto, etc.). **DO NOT add these as tags to components or dataflows.**

### ✅ CORRECT Tag Usage - Architectural Categorization

**Tags describe the NATURE and PURPOSE of components:**

```yaml
components:
  - id: "payment-api"
    name: "Payment Service"
    tags:
      - "payment-processing"      # ✅ Business function
      - "pci-dss-scope"          # ✅ Compliance relevance
      - "customer-data"          # ✅ Data sensitivity
      - "public-facing"          # ✅ Network exposure
```

### ❌ WRONG Tag Usage - Vulnerability Documentation

**NEVER add tags for vulnerabilities you find in code:**

```yaml
# ❌ DO NOT DO THIS - Even if you find these issues in source code
components:
  - id: "flask-app"
    tags:
      - "sql-injection-vulnerable"      # ❌ NO! This is a threat
      - "insecure-deserialization"      # ❌ NO! This is a vulnerability
```

### Why This Rule Exists

1. **IriusRisk finds vulnerabilities** - Its threat library identifies these based on component types
2. **Tags clutter diagrams** - Vulnerability labels make the diagram unreadable
3. **Defeats automation** - If you manually tag all issues, why use IriusRisk?
4. **Mixes concerns** - OTM = architecture (what IS), Threats = security analysis (what's WRONG)

## Component Types to Extract

### 1. Application/Functional Components
**From source code, APIs, microservices:**
- **Business Logic Components**: Payment processing, user authentication, order management
- **Application Services**: User management service, notification service, audit service
- **API Endpoints**: REST APIs, GraphQL endpoints, webhook receivers
- **Background Processes**: Batch jobs, scheduled tasks, data processing pipelines

### 2. Data Components
**From databases, data flows, storage systems:**
- **Data Stores**: SQL databases, NoSQL databases, data warehouses, caches
- **Data Processing**: ETL pipelines, stream processing, data analytics engines
- **Data Storage**: File systems, object storage (S3), content delivery networks

### 3. Infrastructure/Network Components
**From Terraform, cloud configurations, network diagrams:**
- **Compute Resources**: Virtual machines, containers, serverless functions
- **Network Infrastructure**: VPCs, subnets, NAT gateways, internet gateways
- **Load Balancing**: Application load balancers, network load balancers, API gateways

### 4. Cloud Services Components
**From cloud provider configurations:**
- **Serverless**: Lambda functions, Step Functions, EventBridge, SQS, SNS
- **Managed Services**: RDS, DynamoDB, ElastiCache, Elasticsearch
- **Storage Services**: S3 buckets, EFS, EBS volumes, Glacier

### 5. Integration Components
**From API configurations, message queues, external services:**
- **Message Queues**: SQS, RabbitMQ, Kafka, EventBridge
- **External APIs**: Third-party payment processors, social media APIs
- **Integration Platforms**: API management platforms, ESBs

## Source Analysis Strategy

### Phase 1: Repository Scanning and Categorization
1. **Identify all source types** in the repository:
   - Application source code (multiple languages/frameworks)
   - Infrastructure as Code (Terraform, CloudFormation, Kubernetes)
   - Configuration files (Docker, CI/CD, environment configs)
   - Security policies and compliance documentation
   - Architecture documentation and diagrams

2. **Catalog components by source type**:
   - Create inventory of what each source type reveals
   - Note overlaps and relationships between sources
   - Identify gaps where components are referenced but not defined

### Phase 2: Component Extraction and Classification

**From Application Code:** Extract business logic (auth services, business domain services, API endpoints, background jobs). Extract components separately from infrastructure—they'll be nested within infrastructure components (containers, VMs).

**From Infrastructure Code (Terraform/CloudFormation):** Extract cloud resources, security groups/ACLs, load balancers, database instances, monitoring configs, IAM roles.

**From Security Policies/Documentation:** Identify required controls, compliance frameworks, data classification, network segmentation policies.

**From Configuration/Deployment Files:** Discover container definitions, orchestration, environment configs, CI/CD pipelines, monitoring/observability.

### Phase 3: Component Consolidation and Relationship Mapping

**1. Merge overlapping components:** Consolidate same logical component appearing in multiple sources into one with comprehensive properties.

**2. Plan nesting hierarchy:**
- Infrastructure layer → Cloud resources, VMs, containers, managed services
- Business logic layer → Application services nested within infrastructure
- Data layer → Databases/storage (nested or standalone)
- Integration layer → External APIs, message queues, third-party services

**3. Establish relationships:** Nesting (business logic within infrastructure), data flows (between components and data stores), network connections (between infrastructure).

**4. Define trust zones:**
- Internet Zone (trust rating: 1) - Public-facing components, external APIs
- DMZ Zone (3) - Load balancers, web servers, API gateways
- Application Zone (5) - Business logic services, application servers
- Data Zone (7) - Databases, caches, data processing
- Management Zone (8) - Admin interfaces, monitoring, logging

## Workflow Integration

1. Call analyze_source_material() for guidance
2. Call create_threat_model() for OTM creation workflow
3. Execute: **sync() FIRST (mandatory, even if local files exist)** → create OTM → import_otm() → project_status() → sync()

**⚠️ IMPORTANT:** Always call sync() before creating or modifying the OTM, even if `.iriusrisk/current-threat-model.otm` already exists locally. The user may have modified the threat model via the IriusRisk web interface since the last sync. Using a stale local file will overwrite their changes.

Result: Single, comprehensive threat model for holistic IriusRisk analysis across all system layers.
