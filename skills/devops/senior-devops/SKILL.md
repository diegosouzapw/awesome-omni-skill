---

# === CORE IDENTITY ===
name: senior-devops
title: Senior Devops Skill Package
description: Comprehensive DevOps skill for CI/CD, infrastructure automation, containerization, and cloud platforms (AWS, GCP, Azure). Includes pipeline setup, infrastructure as code, deployment automation, and monitoring. Use when setting up pipelines, deploying applications, managing infrastructure, implementing monitoring, or optimizing deployment processes.
domain: engineering
subdomain: devops-operations

# === WEBSITE DISPLAY ===
difficulty: advanced
time-saved: "TODO: Quantify time savings"
frequency: "TODO: Estimate usage frequency"
use-cases:
  - Setting up infrastructure as code with Terraform or CloudFormation
  - Implementing CI/CD pipelines with automated testing and deployment
  - Configuring container orchestration with Kubernetes
  - Monitoring and alerting setup for production systems

# === RELATIONSHIPS ===
related-agents: []
related-skills: []
related-commands: []
orchestrated-by: []

# === TECHNICAL ===
dependencies:
  scripts: [pipeline_generator.py, terraform_scaffolder.py, deployment_manager.py, servicenow_change_manager.py]
  references: [cicd_pipeline_guide.md, infrastructure_as_code.md, deployment_strategies.md, servicenow_change_mgmt.md]
  assets: [servicenow-change-template.json]
compatibility:
  python-version: 3.8+
  platforms: [macos, linux, windows]
tech-stack: [Python 3.8+, Markdown]

# === EXAMPLES ===
examples:
  -
    title: Example Usage
    input: "TODO: Add example input for senior-devops"
    output: "TODO: Add expected output"

# === ANALYTICS ===
stats:
  downloads: 0
  stars: 0
  rating: 0.0
  reviews: 0

# === VERSIONING ===
version: v1.0.0
author: Claude Skills Team
contributors: []
created: 2025-10-19
updated: 2025-11-23
license: MIT

# === DISCOVERABILITY ===
tags: [automation, ci/cd, cloud, devops, engineering, senior, servicenow, change-management, itsm]
featured: false
verified: true
---


# Senior Devops

Complete toolkit for senior devops with modern tools and best practices.

## Overview

This skill provides comprehensive DevOps capabilities through three core Python automation tools and extensive reference documentation. Whether setting up CI/CD pipelines, automating infrastructure, deploying containerized applications, or implementing monitoring, this skill delivers production-ready DevOps solutions.

Senior DevOps engineers use this skill for continuous integration/deployment, infrastructure as code (Terraform), containerization (Docker, Kubernetes), cloud platforms (AWS, GCP, Azure), pipeline automation (GitHub Actions, CircleCI), and observability (Prometheus, Grafana). The skill covers deployment strategies, infrastructure automation, and reliability engineering.

**Core Value:** Accelerate deployment pipelines by 75%+ while improving reliability, consistency, and infrastructure automation through proven DevOps patterns and tools.

## Quick Start

### Main Capabilities

This skill provides four core capabilities through automated scripts:

```bash
# Script 1: Pipeline Generator
python scripts/pipeline_generator.py [options]

# Script 2: Terraform Scaffolder
python scripts/terraform_scaffolder.py [options]

# Script 3: Deployment Manager
python scripts/deployment_manager.py [options]

# Script 4: ServiceNow Change Manager - ITIL change request automation
python scripts/servicenow_change_manager.py --deployment-file deploy.json --change-type normal
```

## Core Capabilities

- **CI/CD Pipeline Setup** - GitHub Actions, CircleCI, GitLab CI configuration with automated testing, building, and deployment
- **Infrastructure as Code** - Terraform scaffolding for AWS, GCP, Azure with modular architecture and state management
- **Container Orchestration** - Docker and Kubernetes configuration, Helm charts, service mesh setup
- **Deployment Automation** - Blue-green, canary, rolling deployments with automated rollback strategies
- **Monitoring & Observability** - Prometheus, Grafana dashboards, logging (ELK/EFK), alerting, and incident response
- **Security & Compliance** - Secret management, RBAC, network policies, security scanning in pipelines

## Python Tools

### 1. Pipeline Generator

Generate production-ready CI/CD pipelines for multiple platforms.

**Key Features:**
- GitHub Actions workflow generation
- CircleCI config creation
- GitLab CI pipeline templates
- Multi-stage builds (test, build, deploy)
- Automated testing integration
- Docker image building and pushing

**Common Usage:**
```bash
# Generate GitHub Actions pipeline
python scripts/pipeline_generator.py --platform github --language nodejs --output .github/workflows/

# CircleCI pipeline
python scripts/pipeline_generator.py --platform circleci --language python --test pytest

# Full stack pipeline
python scripts/pipeline_generator.py --platform github --stack fullstack --deploy kubernetes

# Help
python scripts/pipeline_generator.py --help
```

**Use Cases:**
- Setting up CI/CD for new projects
- Standardizing pipelines across teams
- Adding deployment stages to existing pipelines

### 2. Terraform Scaffolder

Scaffold infrastructure as code with Terraform best practices.

**Key Features:**
- Modular Terraform structure generation
- AWS, GCP, Azure provider templates
- Remote state configuration (S3, GCS, Azure Blob)
- Module organization (networking, compute, database)
- Variable and output management
- Security best practices built-in

**Common Usage:**
```bash
# Scaffold AWS infrastructure
python scripts/terraform_scaffolder.py --provider aws --modules vpc,eks,rds --output infrastructure/

# GCP infrastructure
python scripts/terraform_scaffolder.py --provider gcp --modules gke,cloudsql --remote-state gcs

# Multi-environment setup
python scripts/terraform_scaffolder.py --provider aws --environments dev,staging,prod

# Help
python scripts/terraform_scaffolder.py --help
```

**Use Cases:**
- Starting new infrastructure projects
- Organizing existing Terraform code
- Creating reusable infrastructure modules

### 3. Deployment Manager

Automate application deployments with multiple strategies.

**Key Features:**
- Blue-green deployment automation
- Canary release management
- Rolling update strategies
- Automated rollback on failure
- Health check integration
- Multi-environment deployment

**Common Usage:**
```bash
# Blue-green deployment
python scripts/deployment_manager.py --strategy blue-green --app myapp --version v2.0

# Canary deployment
python scripts/deployment_manager.py --strategy canary --app myapp --version v2.0 --canary-percentage 10

# Rollback
python scripts/deployment_manager.py --rollback --app myapp --to-version v1.9

# Help
python scripts/deployment_manager.py --help
```

**Use Cases:**
- Zero-downtime deployments
- Testing new releases with minimal risk
- Automating deployment workflows

### 4. ServiceNow Change Manager

Generate ServiceNow change request payloads from deployment configurations for ITIL-compliant change management.

**Key Features:**
- Change request generation (Standard, Normal, Emergency)
- Automatic risk assessment based on deployment scope
- Backout plan documentation from deployment config
- Test plan generation with validation steps
- CMDB Configuration Item linking
- CAB approval workflow support
- curl command generation for API testing

**Common Usage:**
```bash
# Generate normal change request from deployment
python scripts/servicenow_change_manager.py \
  --deployment-file deploy-config.json \
  --change-type normal \
  --ci-names "pandora-api-prod,pandora-db-prod" \
  --start-time "2025-01-15T10:00:00Z" \
  --end-time "2025-01-15T12:00:00Z" \
  --output json

# Generate standard change (pre-approved)
python scripts/servicenow_change_manager.py \
  --deployment-file deploy-config.json \
  --change-type standard \
  --output curl

# Generate emergency change for hotfix
python scripts/servicenow_change_manager.py \
  --deployment-file hotfix.json \
  --change-type emergency \
  --output curl

# Help
python scripts/servicenow_change_manager.py --help
```

**Use Cases:**
- ITIL-compliant deployment change management
- Audit trail for production deployments
- CAB approval automation
- Compliance tracking (SOX, PCI-DSS)
- Linking deployments to CMDB Configuration Items

See [servicenow_change_mgmt.md](references/servicenow_change_mgmt.md) for change management best practices.

See [cicd_pipeline_guide.md](references/cicd_pipeline_guide.md) for comprehensive documentation.

## Reference Documentation

### Cicd Pipeline Guide

Comprehensive guide available in `references/cicd_pipeline_guide.md`:

- Detailed patterns and practices
- Code examples
- Best practices
- Anti-patterns to avoid
- Real-world scenarios

### Infrastructure As Code

Complete workflow documentation in `references/infrastructure_as_code.md`:

- Step-by-step processes
- Optimization strategies
- Tool integrations
- Performance tuning
- Troubleshooting guide

### Deployment Strategies

Technical reference guide in `references/deployment_strategies.md`:

- Technology stack details
- Configuration examples
- Integration patterns
- Security considerations
- Scalability guidelines

### ServiceNow Change Management

ITIL change management integration guide in `references/servicenow_change_mgmt.md`:

- Change request types (Standard, Normal, Emergency)
- Deployment-to-change workflow automation
- Change Request API patterns
- CAB approval workflows
- CI/CD pipeline integration (GitHub Actions, Jenkins)
- Risk assessment and impact analysis
- Backout plan documentation
- Post-implementation review

## Tech Stack

**Languages:** TypeScript, JavaScript, Python, Go, Swift, Kotlin
**Frontend:** React, Next.js, React Native, Flutter
**Backend:** Node.js, Express, GraphQL, REST APIs
**Database:** PostgreSQL, Prisma, NeonDB, Supabase
**DevOps:** Docker, Kubernetes, Terraform, GitHub Actions, CircleCI
**Cloud:** AWS, GCP, Azure

## Key Workflows

### 1. CI/CD Pipeline Setup

**Time:** 2-3 hours for complete pipeline

1. **Define Pipeline Stages** - Test, build, scan, deploy stages with appropriate triggers
2. **Generate Pipeline Configuration** - Use pipeline generator for platform-specific config
   ```bash
   # Generate GitHub Actions pipeline
   python scripts/pipeline_generator.py --platform github --language nodejs --deploy kubernetes
   ```
3. **Configure Secrets** - Setup repository secrets for cloud credentials, API keys
4. **Test Pipeline** - Trigger test run, validate all stages execute correctly
5. **Enable Branch Protection** - Require pipeline success before merging

See [cicd_pipeline_guide.md](references/cicd_pipeline_guide.md) for pipeline patterns.

### 2. Infrastructure Provisioning with Terraform

**Time:** 4-6 hours for initial infrastructure

1. **Design Infrastructure** - Define required resources (networking, compute, database, storage)
2. **Scaffold Terraform** - Generate modular Terraform structure
   ```bash
   # Scaffold AWS infrastructure
   python scripts/terraform_scaffolder.py --provider aws --modules vpc,eks,rds --output infrastructure/
   ```
3. **Configure Remote State** - Setup S3/GCS backend for state management
4. **Apply Infrastructure** - Plan and apply Terraform configuration
   ```bash
   terraform init
   terraform plan -out=tfplan
   terraform apply tfplan
   ```
5. **Document Architecture** - Create diagrams and runbooks

See [infrastructure_as_code.md](references/infrastructure_as_code.md) for IaC best practices.

### 3. Kubernetes Application Deployment

**Time:** 3-4 hours for initial deployment

1. **Containerize Application** - Create optimized Dockerfile with multi-stage build
2. **Create Kubernetes Manifests** - Deployment, Service, Ingress, ConfigMap, Secret
3. **Deploy Application** - Use deployment manager for controlled rollout
   ```bash
   # Blue-green deployment
   python scripts/deployment_manager.py --strategy blue-green --app myapp --version v1.0
   ```
4. **Configure Monitoring** - Setup Prometheus metrics, Grafana dashboards
5. **Test & Validate** - Health checks, load testing, rollback testing

### 4. Monitoring & Alerting Setup

**Time:** 2-3 hours for complete observability stack

1. **Deploy Monitoring Stack** - Prometheus, Grafana, Alertmanager
2. **Configure Metrics Collection** - Application metrics, infrastructure metrics, logs
3. **Create Dashboards** - Service health, resource usage, error rates
4. **Setup Alerts** - Error rate thresholds, latency SLOs, capacity warnings
5. **Test Incident Response** - Validate alerting, runbooks, escalation

See [deployment_strategies.md](references/deployment_strategies.md) for deployment and monitoring patterns.

## Development Workflow

### 1. Setup and Configuration

```bash
# Install dependencies
npm install
# or
pip install -r requirements.txt

# Configure environment
cp .env.example .env
```

### 2. Run Quality Checks

```bash
# Use the analyzer script
python scripts/terraform_scaffolder.py .

# Review recommendations
# Apply fixes
```

### 3. Implement Best Practices

Follow the patterns and practices documented in:
- `references/cicd_pipeline_guide.md`
- `references/infrastructure_as_code.md`
- `references/deployment_strategies.md`

## Best Practices Summary

### Code Quality
- Follow established patterns
- Write comprehensive tests
- Document decisions
- Review regularly

### Performance
- Measure before optimizing
- Use appropriate caching
- Optimize critical paths
- Monitor in production

### Security
- Validate all inputs
- Use parameterized queries
- Implement proper authentication
- Keep dependencies updated

### Maintainability
- Write clear code
- Use consistent naming
- Add helpful comments
- Keep it simple

## Common Commands

```bash
# Development
npm run dev
npm run build
npm run test
npm run lint

# Analysis
python scripts/terraform_scaffolder.py .
python scripts/deployment_manager.py --analyze

# Deployment
docker build -t app:latest .
docker-compose up -d
kubectl apply -f k8s/
```

## Troubleshooting

### Common Issues

Check the comprehensive troubleshooting section in `references/deployment_strategies.md`.

### Getting Help

- Review reference documentation
- Check script output messages
- Consult tech stack documentation
- Review error logs

## Resources

- Pattern Reference: `references/cicd_pipeline_guide.md`
- Workflow Guide: `references/infrastructure_as_code.md`
- Technical Guide: `references/deployment_strategies.md`
- ServiceNow Guide: `references/servicenow_change_mgmt.md`
- ServiceNow Template: `assets/servicenow-change-template.json`
- Tool Scripts: `scripts/` directory
