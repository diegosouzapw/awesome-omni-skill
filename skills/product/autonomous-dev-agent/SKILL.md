---
name: autonomous-dev-agent
description: Autonomous development agent that orchestrates the complete SDLC from requirements to deployment. Use this skill when users want to "build a complete app", "create a full application from scratch", "automate the entire development process", or "I want you to develop this autonomously". This agent uses requirements-analysis, repo-management, repo-hygiene, frontend-development, visual-testing, and production-deployment-skill to deliver production-ready applications. Trigger when users want end-to-end autonomous development without manual intervention at each step.
---

# Autonomous Development Agent

Complete end-to-end autonomous development agent that takes an idea and delivers a production-ready application by orchestrating all SDLC skills.

## When to Use This Agent

- User wants a **complete application built autonomously**
- "Build me an app that does X" without wanting to guide each step
- "Create this from scratch and deploy it" 
- End-to-end development from idea → production
- Rapid prototyping with full production setup

**Do NOT use when**:
- User wants to guide each phase manually
- User only needs one specific phase (requirements, frontend, etc.)
- User wants to learn the development process step-by-step

## How This Agent Works

This agent **orchestrates all SDLC skills** in sequence:

```
1. Requirements Analysis → Gather and document requirements
2. Repository Setup → Initialize Git, configure hygiene
3. Frontend Development → Build UI with React + TypeScript
4. Visual Testing → Automated E2E and visual regression
5. Deployment → Production infrastructure and CI/CD
6. Verification → Comprehensive validation
```

Each phase uses specialized skills and produces deliverables that feed into the next phase.

## Agent Workflow

### Phase 1: Requirements Gathering (Autonomous)

**Uses**: `requirements-analysis` skill

**Actions**:
1. Ask clarifying questions about the project
2. Identify target users and core workflows
3. Create user stories with acceptance criteria
4. Prioritize features with MoSCoW
5. Document technical constraints
6. Generate formal requirements specification

**Output**: 
- `requirements.md` - Complete requirements document
- `user-stories.md` - Prioritized backlog
- `technical-specs.md` - Technical constraints and stack decisions

**Decision Point**: Present requirements to user for approval before proceeding.

### Phase 2: Repository Setup (Autonomous)

**Uses**: `repo-management` + `repo-hygiene` skills

**Actions**:
1. Initialize Git repository with main branch
2. Create comprehensive .gitignore (includes Claude file patterns)
3. Set up project structure (src/, tests/, docs/)
4. Create README.md with project overview
5. Add CONTRIBUTING.md with guidelines
6. Set up pre-commit hooks (Husky + lint-staged)
7. Configure GitHub Actions or Azure Pipelines
8. Add PR template and issue templates
9. Create .env.example (never .env)
10. Initial commit with proper message

**Output**:
- Fully configured Git repository
- All essential project files
- CI/CD workflows ready
- Pre-commit hooks preventing junk commits

### Phase 3: Frontend Development (Autonomous)

**Uses**: `frontend-development` skill

**Actions**:
1. Initialize React + TypeScript + Vite project
2. Install dependencies (React Router, Zustand, TanStack Query, Tailwind)
3. Configure TypeScript, ESLint, Prettier, Tailwind
4. Create project structure (components/, pages/, hooks/, services/)
5. Implement all pages from requirements
6. Build reusable UI components
7. Set up state management (Zustand stores)
8. Implement API services with TanStack Query
9. Add responsive design with Tailwind
10. Ensure accessibility (ARIA labels, semantic HTML)
11. Write unit tests for critical components
12. Create Docker configuration for production

**Output**:
- Complete React application
- All features from requirements implemented
- Production-ready build configuration
- Component tests

### Phase 4: Backend Development (If Needed)

**Uses**: Patterns from `tire-retail-system` reference

**Actions**:
1. Set up Express + TypeScript server
2. Configure Prisma ORM with PostgreSQL
3. Implement authentication (JWT with refresh tokens)
4. Create all API endpoints from requirements
5. Add input validation (Zod schemas)
6. Implement middleware (auth, error handling, logging)
7. Set up Redis caching
8. Configure CORS and security headers
9. Write integration tests
10. Create Docker Compose for local development

**Output**:
- Complete REST API
- Database migrations
- API documentation
- Integration tests

### Phase 5: Visual Testing (Autonomous)

**Uses**: `visual-testing` skill

**Actions**:
1. Configure Playwright with multi-browser support
2. Create comprehensive spider test (crawls all pages)
3. Implement E2E tests for critical user flows
4. Set up visual regression testing (BackstopJS or Percy)
5. Add accessibility audits (Axe) for all pages
6. Configure performance testing (Lighthouse)
7. Create Page Object Models for maintainability
8. Generate comprehensive test reports
9. Integrate tests into CI/CD pipeline

**Output**:
- Complete E2E test suite
- Visual regression baselines
- Accessibility audit reports
- Performance benchmarks
- CI/CD integration

### Phase 6: Deployment (Autonomous)

**Uses**: `production-deployment-skill`

**Actions**:
1. Run security scan (code vulnerabilities)
2. Execute comprehensive test suite
3. Generate production Docker images
4. Create Kubernetes manifests (if applicable)
5. Set up infrastructure as code (Terraform)
6. Configure CI/CD pipeline with deployment stages
7. Set up monitoring and logging (health checks)
8. Configure environment variables
9. Deploy to production (or staging first)
10. Verify deployment with smoke tests

**Output**:
- Production deployment
- CI/CD pipeline operational
- Monitoring and logging configured
- Health checks passing
- Deployment documentation

### Phase 7: Final Verification (Autonomous)

**Actions**:
1. Run spider through deployed application
2. Verify all user stories are implemented
3. Check accessibility compliance
4. Validate performance metrics
5. Confirm security scans pass
6. Test all critical user flows
7. Generate final project report

**Output**:
- Verification report
- Known issues (if any)
- Production URLs
- Access credentials (securely stored)

## Usage Examples

### Example 1: Simple Request

**User**: "Build me a todo list app with user authentication"

**Agent Response**:
```
I'll build a complete todo list application autonomously. Here's my plan:

Phase 1: Requirements Analysis
- Target users: Individual users managing personal tasks
- Core features: User registration, task CRUD, categories, due dates
- Stack: React + TypeScript, Node.js + Express, PostgreSQL, JWT auth

Phase 2-6: Autonomous Development
[Agent proceeds through all phases automatically]

Phase 7: Delivery
✅ Application deployed to: https://your-app.com
✅ Repository: github.com/user/todo-app
✅ All tests passing (45 E2E tests, 120 unit tests)
✅ Accessibility: WCAG 2.1 AA compliant
✅ Performance: 98/100 Lighthouse score

Next steps:
- Review the deployed application
- Check GitHub repository for code
- Review documentation in docs/ folder
```

### Example 2: Complex Application

**User**: "Create a multi-vendor e-commerce platform with vendor dashboards, inventory management, and Stripe payments"

**Agent Response**:
```
This is a complex application. Let me gather requirements first...

[Agent asks 3-4 clarifying questions about vendors, products, payment flow]

Based on your answers, here's what I'll build:

**Requirements Summary:**
- Vendor registration and approval system
- Individual vendor dashboards
- Product catalog with multi-vendor support
- Inventory tracking per vendor
- Order management with commission splitting
- Stripe Connect for vendor payouts
- Admin dashboard for platform management

**Technical Stack:**
- Frontend: React 18 + TypeScript + Tailwind
- Backend: Node.js + Express + Prisma
- Database: PostgreSQL
- Payments: Stripe Connect
- File Storage: AWS S3
- Deployment: Docker + Kubernetes

**Estimated Complexity:** High (2-3 hours autonomous development)

Proceed with autonomous development? [Yes/No]

[If Yes, agent proceeds through all 7 phases automatically]
```

## Agent Decision-Making Process

### When to Ask User

The agent asks for user input only at critical decision points:

1. **After Requirements** - Confirm scope and stack before proceeding
2. **Major Architecture Decisions** - Choice between significantly different approaches
3. **Budget/Scale Constraints** - When cost or scale affects architecture
4. **External Integrations** - API keys, third-party services setup
5. **Deployment Target** - AWS, Azure, Vercel, etc.

### What Agent Decides Autonomously

The agent makes these decisions without asking:

✅ **Technology choices** within approved stack (React, Express, PostgreSQL)
✅ **Component structure** and code organization
✅ **State management** approach (Zustand vs Context)
✅ **Testing strategies** (unit, integration, E2E coverage)
✅ **CI/CD configuration** based on repo platform
✅ **Docker configuration** for development and production
✅ **Git branching** strategy (GitHub Flow for simple, Git Flow for complex)
✅ **Error handling** patterns
✅ **Security measures** (HTTPS, CORS, rate limiting, input validation)
✅ **Performance optimizations** (code splitting, caching, lazy loading)

## Configuration Options

### Autonomy Level

**Level 1: Guided** (Default for first-time users)
- Agent asks for approval after each major phase
- User reviews requirements, code structure, before proceeding
- Best for: Learning, high-stakes projects

**Level 2: Semi-Autonomous** (Recommended)
- Agent asks for approval after requirements
- Proceeds autonomously through development and testing
- Asks before deployment to production
- Best for: Most projects, balanced control

**Level 3: Fully Autonomous** (For experienced users)
- Agent proceeds through all phases without stopping
- Only stops on errors or completion
- User reviews final deliverable
- Best for: Rapid prototyping, trusted AI collaboration

### Customization

**Stack Preferences**:
```yaml
frontend:
  framework: react  # or vue, svelte
  language: typescript
  styling: tailwind  # or styled-components, scss
  state: zustand  # or redux, jotai

backend:
  runtime: nodejs  # or python, go
  framework: express  # or fastapi, gin
  database: postgresql  # or mysql, mongodb
  orm: prisma  # or typeorm, sqlalchemy

deployment:
  platform: docker  # or kubernetes, serverless
  hosting: aws  # or azure, vercel, render
```

## Error Handling

### When Agent Encounters Errors

**Build Errors**:
1. Analyze error message
2. Attempt fix (up to 3 retries)
3. If unresolved, report to user with context
4. Suggest manual intervention steps

**Test Failures**:
1. Review failing tests
2. Identify root cause
3. Fix implementation or update test
4. Re-run test suite
5. If still failing, report to user

**Deployment Errors**:
1. Check logs and health checks
2. Attempt rollback if critical
3. Report error with diagnostics
4. Provide manual resolution steps

**Integration Errors** (external APIs):
1. Verify credentials and configuration
2. Check API status
3. Implement retry logic
4. Report to user if persistent

## Quality Standards

The agent ensures all deliverables meet:

### Code Quality
- ✅ TypeScript strict mode enabled
- ✅ ESLint configured with no warnings
- ✅ Prettier formatted
- ✅ No console.logs in production
- ✅ Proper error handling throughout
- ✅ Commented complex logic

### Testing
- ✅ >80% code coverage (unit tests)
- ✅ All critical user flows have E2E tests
- ✅ No accessibility violations (WCAG 2.1 AA)
- ✅ Performance score >90 (Lighthouse)
- ✅ All tests passing before deployment

### Security
- ✅ No secrets in code
- ✅ Input validation on all endpoints
- ✅ Authentication on protected routes
- ✅ HTTPS only in production
- ✅ Security headers configured
- ✅ Dependencies scanned for vulnerabilities

### Documentation
- ✅ README with setup instructions
- ✅ API documentation (if backend)
- ✅ Component documentation (if complex)
- ✅ Architecture decision records
- ✅ Deployment guide

## Deliverables

### Complete Package Includes

**1. Source Code**
- Git repository with clean history
- All code following best practices
- Complete test coverage
- Production-ready configuration

**2. Documentation**
- README.md with setup and usage
- ARCHITECTURE.md with design decisions
- API.md with endpoint documentation
- DEPLOYMENT.md with deployment guide
- CONTRIBUTING.md with development guidelines

**3. Infrastructure**
- Docker Compose for local development
- Production Docker images
- Kubernetes manifests (if applicable)
- CI/CD pipelines configured
- Environment variable templates

**4. Tests**
- Unit tests (Vitest)
- Integration tests
- E2E tests (Playwright)
- Visual regression baselines
- Performance benchmarks

**5. Deployment**
- Production environment configured
- Monitoring and logging set up
- Health checks operational
- Backup strategy implemented
- Rollback procedure documented

**6. Reports**
- Requirements specification
- Test coverage report
- Accessibility audit results
- Performance metrics
- Security scan results

## Comparison with Manual Development

| Aspect | Manual Development | Autonomous Agent |
|--------|-------------------|------------------|
| Requirements | Hours of meetings | 10-15 minutes Q&A |
| Setup | 1-2 hours | 5 minutes |
| Development | Days to weeks | 1-3 hours |
| Testing | Hours per feature | Automated |
| Deployment | Half day | 15 minutes |
| Documentation | Often incomplete | Comprehensive |
| **Total Time** | **Weeks** | **Hours** |

## Limitations & Constraints

### What Agent CAN Build

✅ CRUD applications (admin dashboards, CMS)
✅ E-commerce platforms (single or multi-vendor)
✅ SaaS applications (project management, CRM)
✅ Content platforms (blogs, forums, social networks)
✅ Internal tools (analytics dashboards, reporting)
✅ APIs and microservices
✅ Static websites with dynamic content

### What Agent CANNOT Build (Yet)

❌ Mobile native apps (iOS/Android) - web apps only
❌ Real-time multiplayer games
❌ Blockchain/cryptocurrency applications
❌ Machine learning model training
❌ Hardware integrations (IoT devices)
❌ Applications requiring deep domain expertise (medical, legal)

### Technical Constraints

- **Frontend**: Limited to React ecosystem (Vue/Svelte not supported yet)
- **Backend**: Node.js or Python only (no Go, Rust, Java)
- **Database**: Relational databases only (PostgreSQL, MySQL)
- **Scale**: Optimized for <100K users (larger scale needs manual architecture)
- **Complexity**: Best for applications with <50 user stories

## Best Practices

### Getting Best Results

1. **Clear Initial Description**: Provide detailed description upfront
2. **Answer Questions Thoughtfully**: Agent's questions guide architecture
3. **Trust the Process**: Let agent work autonomously through phases
4. **Review Deliverables**: Always review final code and documentation
5. **Provide Feedback**: Help agent learn your preferences

### When to Intervene

- **Critical Security Concerns**: If you spot a security issue
- **Business Logic Errors**: If agent misunderstands requirements
- **Stack Preferences**: If you have strong technology preferences
- **Budget Constraints**: If costs are exceeding expectations

## Example Usage Script

```typescript
// Example: Invoking the autonomous agent

// Simple invocation
"Build me a task management app with teams and projects"

// Detailed invocation
"Create a SaaS application for freelancers to:
- Manage clients and projects
- Track time
- Generate invoices
- Accept payments via Stripe
- Export financial reports

Target users: Freelancers and small agencies (1-10 people)
Scale: Expect 1,000 users in first 6 months
Budget: Keep infrastructure costs under $200/month"

// With specific preferences
"Build an e-commerce platform using:
- React + TypeScript for frontend
- Express + Prisma for backend
- PostgreSQL database
- Deploy to AWS with Docker
- Stripe for payments
- S3 for product images"
```

## Monitoring Agent Progress

The agent provides real-time updates:

```
[Phase 1: Requirements] Analyzing requirements... ✓
[Phase 1: Requirements] Generated 15 user stories... ✓
[Phase 1: Requirements] Created technical specifications... ✓

[Phase 2: Repository] Initializing Git repository... ✓
[Phase 2: Repository] Configured .gitignore with Claude file patterns... ✓
[Phase 2: Repository] Set up pre-commit hooks... ✓

[Phase 3: Frontend] Installing dependencies... ✓
[Phase 3: Frontend] Created 12 components... ✓
[Phase 3: Frontend] Implemented 8 pages... ✓
[Phase 3: Frontend] Added 24 unit tests... ✓

[Phase 4: Testing] Configured Playwright... ✓
[Phase 4: Testing] Created spider test (found 8 pages)... ✓
[Phase 4: Testing] Implemented 15 E2E tests... ✓
[Phase 4: Testing] All tests passing... ✓

[Phase 5: Deployment] Building Docker images... ✓
[Phase 5: Deployment] Deploying to production... ✓
[Phase 5: Deployment] Health checks passing... ✓

✅ Application successfully deployed!
```

## Post-Deployment Support

After deployment, the agent provides:

1. **Deployment Summary**: URLs, credentials, repositories
2. **Monitoring Dashboard**: Links to logs and metrics
3. **Quick Start Guide**: For team onboarding
4. **Maintenance Runbook**: Common tasks and troubleshooting
5. **Feature Addition Guide**: How to add new features

## Integration with SDLC Skills

This agent is a **meta-skill** that orchestrates:

- `requirements-analysis` - Phase 1
- `repo-management` - Phase 2
- `repo-hygiene` - Phase 2
- `frontend-development` - Phase 3
- `visual-testing` - Phase 5
- `production-deployment-skill` - Phase 6

Each skill is invoked programmatically with appropriate context and parameters.

## Conclusion

The Autonomous Development Agent transforms high-level ideas into production-ready applications by orchestrating all SDLC phases. It makes intelligent decisions, follows best practices, and delivers comprehensive, well-documented, tested applications in hours instead of weeks.

**Use this agent when**: You want to rapidly prototype, validate ideas, or build production applications without managing every development step manually.

**Key Benefit**: Focus on business logic and requirements while the agent handles the technical implementation, testing, and deployment infrastructure.
