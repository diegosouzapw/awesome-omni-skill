---
name: project-planner
description: Transforms project ideas into structured documentation (overview + specifications). Use when starting new projects or when brief needs project-level planning with vision, features, and technical requirements.
---

# Project Planner Skill

**Purpose**: Generate comprehensive project documentation from high-level descriptions.

**Trigger Words**: new project, project overview, project spec, technical requirements, project planning, architecture, system design

---

## Quick Decision: Use Project Planning?

```python
def needs_project_planning(context: dict) -> bool:
    """Fast evaluation for project-level planning."""

    # Indicators of project-level work
    project_indicators = [
        "new project", "project overview", "system design",
        "architecture", "technical requirements", "project spec",
        "build a", "create a", "develop a platform",
        "microservices", "full stack", "api + frontend"
    ]

    description = context.get("description", "").lower()
    return any(indicator in description for indicator in project_indicators)
```

---

## Output Structure

Generates TWO documents in `project-management/`:

### 1. PROJECT-OVERVIEW.md
High-level vision and goals

### 2. SPECIFICATIONS.md
Detailed technical requirements

---

## Document 1: PROJECT-OVERVIEW.md

### Template Structure

```markdown
# {Project Name}

> {Tagline - one compelling sentence}

## Vision

{2-3 sentences describing the ultimate goal and impact}

## Goals

1. {Primary goal}
2. {Secondary goal}
3. {Tertiary goal}

## Key Features

- **{Feature 1}**: {Brief description}
- **{Feature 2}**: {Brief description}
- **{Feature 3}**: {Brief description}
- **{Feature 4}**: {Brief description}
- **{Feature 5}**: {Brief description}

## Success Criteria

1. **{Metric 1}**: {Target}
2. **{Metric 2}**: {Target}
3. **{Metric 3}**: {Target}

## Constraints

- **Budget**: {If specified}
- **Timeline**: {If specified}
- **Technology**: {Required tech stack or limitations}
- **Team**: {Team size/composition if known}

## Out of Scope

- {What this project will NOT do}
- {Features explicitly excluded}
- {Future phases}
```

### Example Output

```markdown
# TaskFlow Pro

> Modern task management with AI-powered prioritization

## Vision

Build a task management platform that helps remote teams stay organized through intelligent prioritization, real-time collaboration, and seamless integrations with existing tools.

## Goals

1. Reduce task management overhead by 50%
2. Enable real-time team collaboration
3. Integrate with popular dev tools (GitHub, Jira, Slack)

## Key Features

- **AI Prioritization**: ML-based task ranking by urgency and impact
- **Real-time Collaboration**: Live updates, comments, mentions
- **Smart Integrations**: Auto-sync with GitHub issues, Jira tickets
- **Custom Workflows**: Configurable pipelines per team
- **Analytics Dashboard**: Team productivity insights

## Success Criteria

1. **User Adoption**: 1000 active users in 6 months
2. **Performance**: <200ms API response time
3. **Reliability**: 99.9% uptime

## Constraints

- Timeline: 6 months MVP
- Technology: Python backend, React frontend, PostgreSQL
- Team: 2 backend, 2 frontend, 1 ML engineer

## Out of Scope

- Mobile apps (Phase 2)
- Video conferencing
- Time tracking (separate product)
```

---

## Document 2: SPECIFICATIONS.md

### Template Structure

```markdown
# {Project Name} - Technical Specifications

## Functional Requirements

### Core Features

#### {Feature 1}
- **Description**: {What it does}
- **User Story**: As a {role}, I want {action} so that {benefit}
- **Acceptance Criteria**:
  - [ ] {Criterion 1}
  - [ ] {Criterion 2}
  - [ ] {Criterion 3}

#### {Feature 2}
{Repeat structure}

### User Flows

#### {Flow 1}: {Name}
1. User {action}
2. System {response}
3. User {next action}
4. Result: {outcome}

---

## Non-Functional Requirements

### Performance
- API response time: <200ms (p95)
- Page load time: <1s
- Concurrent users: 10,000+
- Database queries: <50ms

### Security
- Authentication: OAuth2 + JWT
- Authorization: Role-based access control (RBAC)
- Data encryption: AES-256 at rest, TLS 1.3 in transit
- Rate limiting: 100 req/min per user

### Reliability
- Uptime: 99.9% SLA
- Backup frequency: Daily
- Recovery time: <1 hour (RTO)
- Data loss: <5 minutes (RPO)

### Scalability
- Horizontal scaling: Auto-scale based on load
- Database: Read replicas for queries
- Cache: Redis for hot data
- CDN: Static assets

---

## API Contracts

### Authentication API

#### POST /api/auth/login
```json
// Request
{
  "email": "user@example.com",
  "password": "hashed_password"
}

// Response (200 OK)
{
  "token": "jwt_token_here",
  "user": {
    "id": "user_123",
    "email": "user@example.com",
    "name": "John Doe"
  }
}

// Error (401 Unauthorized)
{
  "error": "Invalid credentials"
}
```

#### POST /api/auth/logout
{Repeat structure for each endpoint}

### Tasks API

#### GET /api/tasks
```json
// Query params: ?page=1&per_page=50&status=active
// Response (200 OK)
{
  "tasks": [
    {
      "id": "task_123",
      "title": "Fix bug in auth",
      "status": "active",
      "priority": "high",
      "assignee": "user_456",
      "created_at": "2025-10-30T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 50,
    "total": 150
  }
}
```

{Continue for all major endpoints}

---

## Data Models

### User
```python
class User:
    id: str (UUID)
    email: str (unique, indexed)
    password_hash: str
    name: str
    role: Enum['admin', 'member', 'viewer']
    created_at: datetime
    updated_at: datetime
    last_login: datetime | None
```

### Task
```python
class Task:
    id: str (UUID)
    title: str (max 200 chars)
    description: str | None
    status: Enum['backlog', 'active', 'completed']
    priority: Enum['low', 'medium', 'high', 'urgent']
    assignee_id: str | None (FK -> User.id)
    project_id: str (FK -> Project.id)
    due_date: datetime | None
    created_at: datetime
    updated_at: datetime
```

{Continue for all major models}

---

## System Architecture

### Components
- **API Gateway**: Kong/NGINX for routing and rate limiting
- **Backend Services**: FastAPI/Django microservices
- **Database**: PostgreSQL (primary), Redis (cache)
- **Message Queue**: RabbitMQ for async tasks
- **Storage**: S3 for file uploads
- **Monitoring**: Prometheus + Grafana

### Deployment
- **Infrastructure**: AWS/GCP Kubernetes
- **CI/CD**: GitHub Actions
- **Environments**: dev, staging, production
- **Rollback**: Blue-green deployment

---

## Dependencies

### Backend
- Python 3.11+
- FastAPI or Django REST Framework
- SQLAlchemy or Django ORM
- Celery for background tasks
- pytest for testing

### Frontend
- React 18+ or Vue 3+
- TypeScript
- Tailwind CSS or Material-UI
- Axios for API calls
- Vitest or Jest for testing

### Infrastructure
- Docker + Docker Compose
- Kubernetes (production)
- PostgreSQL 15+
- Redis 7+
- NGINX or Caddy

---

## Development Phases

### Phase 1: MVP (Months 1-3)
- [ ] User authentication
- [ ] Basic task CRUD
- [ ] Simple prioritization
- [ ] API foundation

### Phase 2: Collaboration (Months 4-5)
- [ ] Real-time updates (WebSocket)
- [ ] Comments and mentions
- [ ] Team management

### Phase 3: Integrations (Month 6)
- [ ] GitHub integration
- [ ] Jira sync
- [ ] Slack notifications

---

## Testing Strategy

### Unit Tests
- Coverage: >80%
- All business logic functions
- Mock external dependencies

### Integration Tests
- API endpoint testing
- Database transactions
- Authentication flows

### E2E Tests
- Critical user flows
- Payment processing (if applicable)
- Admin workflows

---

## Security Considerations

### OWASP Top 10 Coverage
1. **Injection**: Parameterized queries, input validation
2. **Broken Auth**: JWT with refresh tokens, secure session management
3. **Sensitive Data**: Encryption at rest and in transit
4. **XXE**: Disable XML external entities
5. **Broken Access Control**: RBAC enforcement
6. **Security Misconfiguration**: Secure defaults, regular audits
7. **XSS**: Output escaping, CSP headers
8. **Insecure Deserialization**: Validate all input
9. **Known Vulnerabilities**: Dependency scanning (Snyk, Dependabot)
10. **Insufficient Logging**: Audit logs for sensitive actions

---

## Monitoring & Observability

### Metrics
- Request rate, error rate, latency (RED method)
- Database connection pool usage
- Cache hit/miss ratio
- Background job queue length

### Logging
- Structured JSON logs
- Centralized logging (ELK stack or CloudWatch)
- Log levels: DEBUG (dev), INFO (staging), WARN/ERROR (prod)

### Alerting
- Error rate >5% (P1)
- API latency >500ms (P2)
- Database connections >80% (P2)
- Disk usage >90% (P1)

---

## Documentation Requirements

- [ ] API documentation (OpenAPI/Swagger)
- [ ] Setup guide (README.md)
- [ ] Architecture diagrams
- [ ] Deployment runbook
- [ ] Troubleshooting guide

```

---

## Generation Process

### Step 1: Extract Project Context
```python
def extract_project_info(prompt: str) -> dict:
    """Parse project description for key details."""

    info = {
        "name": None,
        "description": prompt,
        "features": [],
        "tech_stack": [],
        "constraints": {},
        "goals": []
    }

    # Extract from prompt:
    # - Project name (if mentioned)
    # - Desired features
    # - Technology preferences
    # - Timeline/budget constraints
    # - Success metrics

    return info
```

### Step 2: Apply Output Style
Use `output-style-selector` to determine:
- **PROJECT-OVERVIEW.md**: Bullet-points, concise
- **SPECIFICATIONS.md**: Table-based for API contracts, YAML-structured for models

### Step 3: Generate Documents
1. Create `project-management/` directory if needed
2. Write PROJECT-OVERVIEW.md (vision-focused)
3. Write SPECIFICATIONS.md (technical details)
4. Validate completeness

### Step 4: Validation Checklist
```markdown
## Generated Documents Validation

PROJECT-OVERVIEW.md:
- [ ] Project name and tagline present
- [ ] Vision statement (2-3 sentences)
- [ ] 3+ goals defined
- [ ] 5-10 key features listed
- [ ] Success criteria measurable
- [ ] Constraints documented
- [ ] Out-of-scope items listed

SPECIFICATIONS.md:
- [ ] Functional requirements detailed
- [ ] Non-functional requirements (perf, security, reliability)
- [ ] API contracts with examples (if applicable)
- [ ] Data models defined
- [ ] Architecture overview
- [ ] Dependencies listed
- [ ] Development phases outlined
- [ ] Testing strategy included
```

---

## Integration with Commands

### With `/lazy plan`
```bash
# Generate project docs first
/lazy plan --project "Build AI-powered task manager"

→ project-planner skill triggers
→ Generates PROJECT-OVERVIEW.md + SPECIFICATIONS.md
→ Then creates first user story from specifications

# Or start from enhanced prompt
/lazy plan --file enhanced_prompt.md

→ Detects project-level scope
→ Runs project-planner
→ Creates foundational docs
→ Proceeds with story creation
```

### With `/lazy code`
```bash
# Reference specifications during implementation
/lazy code @US-3.4.md

→ context-packer loads SPECIFICATIONS.md
→ API contracts and data models available
→ Implementation follows spec
```

---

## What This Skill Does NOT Do

❌ Generate actual code (that's for `coder` agent)
❌ Create user stories (that's for `project-manager` agent)
❌ Make architectural decisions (provides template, you decide)
❌ Replace technical design documents (TDDs)

✅ **DOES**: Create structured foundation documents for new projects.

---

## Configuration

```bash
# Minimal specs (faster, less detail)
export LAZYDEV_PROJECT_SPEC_MINIMAL=1

# Skip API contracts (non-API projects)
export LAZYDEV_PROJECT_NO_API=1

# Focus on specific aspects
export LAZYDEV_PROJECT_FOCUS="security,performance"
```

---

## Tips for Effective Project Planning

### For PROJECT-OVERVIEW.md
1. **Vision**: Think big picture - why does this exist?
2. **Goals**: Limit to 3-5 measurable outcomes
3. **Features**: High-level only (not task-level details)
4. **Success Criteria**: Must be measurable (numbers, percentages)

### For SPECIFICATIONS.md
1. **API Contracts**: Start with authentication and core resources
2. **Data Models**: Include relationships and constraints
3. **Non-Functional**: Don't skip - these prevent tech debt
4. **Security**: Reference OWASP Top 10 coverage
5. **Phases**: Break into 2-3 month chunks maximum

### Best Practices
- **Keep PROJECT-OVERVIEW under 2 pages**: Executive summary only
- **SPECIFICATIONS can be longer**: This is the source of truth
- **Update specs as you learn**: Living documents
- **Version control both**: Track changes over time

---

## Example Trigger Scenarios

### Scenario 1: New Greenfield Project
```
User: "I want to build a real-time chat platform with video calls"

→ project-planner triggers
→ Generates:
  - PROJECT-OVERVIEW.md (vision: modern communication platform)
  - SPECIFICATIONS.md (WebSocket APIs, video streaming, etc.)
→ Ready for user story creation
```

### Scenario 2: From Enhanced Prompt
```
User: /lazy plan --file enhanced_prompt.md
# enhanced_prompt contains: detailed project requirements, tech stack, timeline

→ project-planner parses prompt
→ Extracts structured information
→ Generates both documents
→ Proceeds to first user story
```

### Scenario 3: Partial Information
```
User: "Build a task manager, not sure about details yet"

→ project-planner generates template
→ Marks sections as [TODO: Specify...]
→ User fills in gaps incrementally
→ Re-generate or update manually
```

---

## Output Format (Completion)

```markdown
## Project Planning Complete

**Documents Generated**:

1. **PROJECT-OVERVIEW.md** (2.4KB)
   - Project: TaskFlow Pro
   - Vision: Modern task management with AI
   - Features: 5 key features defined
   - Success criteria: 3 measurable metrics

2. **SPECIFICATIONS.md** (8.1KB)
   - Functional requirements: 5 core features detailed
   - API contracts: 12 endpoints documented
   - Data models: 6 models defined
   - Architecture: Microservices with Kubernetes
   - Development phases: 3 phases over 6 months

**Location**: `./project-management/`

**Next Steps**:
1. Review and refine generated documents
2. Run: `/lazy plan "First user story description"`
3. Begin implementation with `/lazy code`

**Estimated Setup Time**: 15-20 minutes to review/customize
```

---

**Version**: 1.0.0
**Output Size**: 10-15KB total (both documents)
**Generation Time**: ~30 seconds
