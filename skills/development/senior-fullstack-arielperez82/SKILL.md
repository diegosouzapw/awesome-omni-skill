---

# === CORE IDENTITY ===
name: senior-fullstack
title: Senior Fullstack Skill Package
description: Comprehensive fullstack development skill for building complete web applications with React, Next.js, Node.js, GraphQL, and PostgreSQL. Includes project scaffolding, code quality analysis, architecture patterns, and complete tech stack guidance. Use when building new projects, analyzing code quality, implementing design patterns, or setting up development workflows.
domain: engineering
subdomain: fullstack-development

# === WEBSITE DISPLAY ===
difficulty: advanced
time-saved: "TODO: Quantify time savings"
frequency: "TODO: Estimate usage frequency"
use-cases:
  - Primary workflow for Senior Fullstack
  - Analysis and recommendations for senior fullstack tasks
  - Best practices implementation for senior fullstack
  - Integration with related skills and workflows

# === RELATIONSHIPS ===
related-agents: []
related-skills: []
related-commands: []
orchestrated-by: []

# === TECHNICAL ===
dependencies:
  scripts: []
  references: []
  assets: []
compatibility:
  python-version: 3.8+
  platforms: [macos, linux, windows]
tech-stack: [Python 3.8+, Markdown]

# === EXAMPLES ===
examples:
  -
    title: Example Usage
    input: "TODO: Add example input for senior-fullstack"
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
tags:
  - analysis
  - architecture
  - design
  - development
  - engineering
  - fullstack
  - senior
featured: false
verified: true
---

# Senior Fullstack Developer

Expert-level fullstack development skill with production-ready tools for modern web application development. Covers architecture patterns, tech stack mastery, and automated quality analysis.

## Overview

This skill provides comprehensive fullstack development capabilities through three core automation tools and extensive reference documentation. Whether you're scaffolding a new project, analyzing code quality, or implementing complex architecture patterns, this skill delivers production-ready solutions.

**Use this skill when:**
- Starting new fullstack projects with modern tech stacks
- Analyzing and improving code quality
- Implementing microservices or clean architecture
- Setting up development workflows and DevOps pipelines
- Making technology stack decisions

## Core Capabilities

### 1. Project Scaffolder

Generate production-ready fullstack projects with complete infrastructure.

**Features:**
- Multiple stack templates (Next.js, React, Vue + GraphQL/REST)
- Docker Compose configuration
- CI/CD pipelines (GitHub Actions)
- Testing infrastructure (Jest, Cypress)
- Database setup and migrations
- TypeScript, ESLint, Prettier pre-configured

**Usage:**
```bash
python scripts/project_scaffolder.py my-project --type nextjs-graphql
cd my-project && docker-compose up -d
```

**Supported Stacks:**
- Next.js + GraphQL + PostgreSQL
- React + REST + MongoDB
- Vue + GraphQL + MySQL
- Express + TypeScript + PostgreSQL

### 2. Code Quality Analyzer

Comprehensive code analysis with actionable recommendations.

**Features:**
- Security vulnerability scanning
- Performance issue detection
- Test coverage assessment
- Documentation quality analysis
- Dependency audit
- Prioritized recommendations

**Usage:**
```bash
python scripts/code_quality_analyzer.py /path/to/project
python scripts/code_quality_analyzer.py /path/to/project --json
```

### 3. Fullstack Scaffolder

Rapid fullstack application generation with best practices built-in.

**Usage:**
```bash
python scripts/fullstack_scaffolder.py my-app --stack nextjs-graphql
```

## Python Tools

### 1. Project Scaffolder

Generate production-ready fullstack projects with complete infrastructure.

**Key Features:**
- Multiple stack templates (Next.js, React, Vue + GraphQL/REST)
- Docker Compose configuration
- CI/CD pipelines (GitHub Actions)
- Testing infrastructure (Jest, Cypress)
- Database setup and migrations
- TypeScript, ESLint, Prettier pre-configured

**Common Usage:**
```bash
# Create Next.js + GraphQL project
python scripts/project_scaffolder.py my-project --type nextjs-graphql

# React + REST API
python scripts/project_scaffolder.py my-app --type react-rest

# Start services
cd my-project && docker-compose up -d

# Help
python scripts/project_scaffolder.py --help
```

**Use Cases:**
- Starting new fullstack projects with best practices
- Creating proof-of-concept applications
- Standardizing project structure across teams

### 2. Code Quality Analyzer

Comprehensive code analysis with actionable recommendations.

**Key Features:**
- Security vulnerability scanning
- Performance issue detection
- Test coverage assessment
- Documentation quality analysis
- Dependency audit
- Prioritized recommendations

**Common Usage:**
```bash
# Analyze project
python scripts/code_quality_analyzer.py /path/to/project

# JSON output for CI/CD
python scripts/code_quality_analyzer.py /path/to/project --json

# Focus on security
python scripts/code_quality_analyzer.py /path/to/project --security-only

# Help
python scripts/code_quality_analyzer.py --help
```

**Use Cases:**
- Pre-deployment quality checks
- Technical debt identification
- Security audit automation

### 3. Fullstack Scaffolder

Rapid fullstack application generation with best practices built-in.

**Key Features:**
- Quick project setup
- Modern tech stack selection
- Best practices integration
- Ready-to-deploy configuration

**Common Usage:**
```bash
# Scaffold fullstack app
python scripts/fullstack_scaffolder.py my-app --stack nextjs-graphql

# Custom configuration
python scripts/fullstack_scaffolder.py my-app --frontend react --backend express --db postgresql

# Help
python scripts/fullstack_scaffolder.py --help
```

**Use Cases:**
- Rapid prototyping
- Hackathon projects
- Client POC development

See [tech-stacks.md](references/tech-stacks.md) for comprehensive tool documentation.

## Key Workflows

### 1. New Project Setup

**Time:** 30 minutes for complete setup

1. **Generate Project Structure** - Scaffold with optimal tech stack
   ```bash
   # Create project
   python scripts/project_scaffolder.py my-app --type nextjs-graphql
   ```
2. **Configure Environment** - Setup environment variables, database connection
   ```bash
   cd my-app
   cp .env.example .env
   # Edit .env with configuration
   ```
3. **Start Development Services** - Launch Docker containers
   ```bash
   docker-compose up -d
   ```
4. **Run Database Migrations** - Initialize database schema
   ```bash
   npm run migrate
   ```
5. **Start Development Server** - Begin development
   ```bash
   npm run dev
   ```

See [architecture-patterns.md](references/architecture-patterns.md) for architecture guidance.

### 2. Code Quality Assessment

**Time:** 15-20 minutes for full analysis

1. **Run Comprehensive Analysis** - Analyze entire codebase
   ```bash
   python scripts/code_quality_analyzer.py ./
   ```
2. **Review Recommendations** - Prioritize issues by severity
3. **Fix Security Vulnerabilities** - Update dependencies
   ```bash
   npm audit fix
   ```
4. **Fix Linting Errors** - Automated fixes where possible
   ```bash
   npm run lint -- --fix
   ```
5. **Run Test Suite** - Ensure all tests pass
   ```bash
   npm test
   ```
6. **Build for Production** - Verify production build succeeds
   ```bash
   npm run build
   ```
7. **Re-analyze** - Verify improvements
   ```bash
   python scripts/code_quality_analyzer.py ./ --json > quality-report.json
   ```

### 3. Production Deployment

**Time:** 1-2 hours for initial deployment

1. **Quality Checks** - Ensure tests and build pass
   ```bash
   npm test && npm run build
   ```
2. **Build Docker Image** - Create production container
   ```bash
   docker build -t my-app:latest .
   ```
3. **Deploy with Docker Compose** - Production deployment
   ```bash
   docker-compose -f docker-compose.prod.yml up -d
   ```
   Or deploy to Kubernetes:
   ```bash
   kubectl apply -f k8s/
   ```
4. **Verify Deployment** - Health check and smoke tests
   ```bash
   curl https://your-app.com/health
   ```
5. **Monitor** - Setup logging and monitoring dashboards

See [best-practices.md](references/best-practices.md) for deployment best practices.

## Reference Documentation

Detailed guides available in the `references/` directory:

### Architecture Patterns
**[architecture-patterns.md](references/architecture-patterns.md)** - Comprehensive architecture guide covering:
- Microservices architecture and service design
- Clean architecture and layer patterns
- Domain-driven design (DDD)
- Frontend architecture (atomic design, state management)
- Backend patterns (service layer, repository pattern)
- Performance optimization strategies
- Security patterns and deployment approaches

### Technology Stacks
**[tech-stacks.md](references/tech-stacks.md)** - Complete technology reference including:
- Languages (TypeScript, Python, Go, Kotlin, Swift)
- Frontend frameworks (React, Next.js, React Native, Flutter)
- Backend frameworks (Node.js, Express, GraphQL, REST)
- Databases (PostgreSQL, Prisma, NeonDB, Supabase, MongoDB)
- DevOps tools (Docker, Kubernetes, Terraform, CI/CD)
- Cloud platforms (AWS, GCP, Azure, Vercel, Railway)
- Testing frameworks and development tools

### Best Practices
**[best-practices.md](references/best-practices.md)** - Industry standards and guidelines for:
- Code quality and SOLID principles
- Testing strategies (unit, integration, e2e)
- Performance optimization (frontend and backend)
- Security best practices (authentication, authorization, validation)
- Error handling and logging
- Documentation standards
- Git workflow and deployment practices

## Quick Start Workflows

### Workflow 1: New Project Setup

```bash
# 1. Generate project structure
python scripts/project_scaffolder.py my-app --type nextjs-graphql

# 2. Configure environment
cd my-app
cp .env.example .env
# Edit .env with your configuration

# 3. Start development services
docker-compose up -d

# 4. Run database migrations
npm run migrate

# 5. Start development server
npm run dev
```

### Workflow 2: Code Quality Assessment

```bash
# 1. Run comprehensive analysis
python scripts/code_quality_analyzer.py ./

# 2. Review recommendations and fix issues
npm audit fix                    # Security vulnerabilities
npm run lint -- --fix           # Linting errors
npm test                        # Run test suite

# 3. Build for production
npm run build

# 4. Re-analyze to verify improvements
python scripts/code_quality_analyzer.py ./ --json
```

### Workflow 3: Production Deployment

```bash
# 1. Ensure quality checks pass
npm test && npm run build

# 2. Build Docker image
docker build -t my-app:latest .

# 3. Deploy with Docker Compose
docker-compose -f docker-compose.prod.yml up -d

# Or deploy to Kubernetes
kubectl apply -f k8s/

# 4. Verify deployment
curl https://your-app.com/health
```

## Tech Stack Summary

**Frontend:** React, Next.js, TypeScript, Tailwind CSS
**Backend:** Node.js, Express, GraphQL, REST APIs
**Database:** PostgreSQL, Prisma ORM, NeonDB, Supabase
**DevOps:** Docker, Kubernetes, Terraform, GitHub Actions
**Cloud:** AWS, GCP, Azure, Vercel
**Testing:** Jest, React Testing Library, Cypress

For detailed technology guides, see [tech-stacks.md](references/tech-stacks.md).

## Common Use Cases

**1. E-commerce Platform**
- Next.js for SEO-optimized storefront
- GraphQL API for product catalog
- PostgreSQL for transactions
- Stripe integration for payments

**2. SaaS Dashboard**
- React SPA with complex state management
- REST API with Node.js/Express
- Real-time updates with WebSockets
- Role-based access control

**3. Mobile + Web App**
- React Native for iOS/Android
- Next.js for web presence
- Shared GraphQL API
- Supabase for backend services

**4. Content Management System**
- Next.js with ISR (Incremental Static Regeneration)
- Headless CMS integration
- PostgreSQL for structured data
- CDN distribution via Vercel

## Additional Resources

- **Architecture Guide:** [references/architecture-patterns.md](references/architecture-patterns.md)
- **Technology Reference:** [references/tech-stacks.md](references/tech-stacks.md)
- **Best Practices:** [references/best-practices.md](references/best-practices.md)
- **Python Tools:** `scripts/` directory

## Getting Help

1. **Architecture questions:** Review [architecture-patterns.md](references/architecture-patterns.md)
2. **Technology selection:** Consult [tech-stacks.md](references/tech-stacks.md)
3. **Code quality issues:** Run code quality analyzer and review output
4. **Best practices:** See [best-practices.md](references/best-practices.md)
5. **Tool usage:** Run any script with `--help` flag

---

**Version:** 1.0.0
**Last Updated:** 2025-11-08
**Documentation Structure:** Progressive disclosure with references/
