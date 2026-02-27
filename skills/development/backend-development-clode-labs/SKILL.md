---
name: backend-development
description: Build backend services, APIs, and server-side applications. Use this skill for creating REST/GraphQL APIs, database integrations, authentication systems, and server-side business logic in Go, Python, or Node.js.
category: development
tags: [backend, api, rest, golang, python, nodejs, database]
license: MIT
---

# Backend Development

Build APIs following project patterns. Implement proper validation, error handling, and security. Create Dockerfile and docker-compose.yml for containerization.

## Session Continuity

Your session persists. You may be started fresh OR resumed with new context.

### Trigger Types

| Trigger | Meaning |
|---------|---------|
| `start` | Normal task execution (first time) |
| `resume` | User provided additional context |
| `task_chat` | Direct message from task chat UI |

### Resume Trigger Format

When resumed, you receive:
```
## Task Resumed

The user has provided additional context:

<user's message>

Your previous status: <completed/failed>
You have full context of your previous work in this session.
```

### How to Handle Resume

| Previous Status | User Intent | Action |
|-----------------|-------------|--------|
| `failed` | Providing fix info | Retry with new context |
| `completed` | Wants modification | Review and update |
| `completed` | Asking question | Answer from your context |
| `in_progress` | Adding context | Incorporate and continue |

### Q&A Mode

If resumed with `mode="qa"`:
- Only answer questions
- Do NOT perform new work
- Use `TaskChatResponse` to reply

## Inputs

- `requirements`: What to build
- `files_to_create`: Files to create (including Dockerfile, docker-compose.yml)
- `files_to_modify`: Existing files to modify
- `patterns_to_follow`: Reference patterns in codebase
- `include_docker`: Whether to create Dockerfile and docker-compose.yml (default: true)
- `validation_criteria`: Self-validation criteria
  - `critical`: MUST pass before completing
  - `expected`: SHOULD pass (log warning if not)
  - `nice_to_have`: Optional improvements

## Constraints

- Always validate input data
- Use parameterized queries (never string concatenation for SQL)
- Follow OWASP security guidelines
- **Do NOT create documentation files** unless explicitly requested
- **Always create Dockerfile and docker-compose.yml** unless explicitly excluded

## Docker Containerization

When building backend services, create containerization files:

### Dockerfile

Create a multi-stage Dockerfile optimized for the language:

**Go:**
```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

# Runtime stage
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
EXPOSE 8080
CMD ["./main"]
```

**Python (FastAPI/Flask):**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8080
CMD ["python", "main.py"]
```

**Node.js:**
```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Runtime stage
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 8080
CMD ["node", "server.js"]
```

### docker-compose.yml

Create a complete docker-compose.yml with all services:

```yaml
version: '3.8'

services:
  backend:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://postgres:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}
      - PORT=8080
      - NODE_ENV=production
    depends_on:
      - postgres
    networks:
      - app-network

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=${POSTGRES_DB:-myapp}
      - POSTGRES_USER=${POSTGRES_USER:-postgres}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:

networks:
  app-network:
    driver: bridge
```

### Best Practices

1. **Multi-stage builds**: Reduce final image size
2. **Security**:
   - Run as non-root user
   - Use minimal base images (alpine, distroless)
   - Don't include secrets in images
3. **Optimization**:
   - Layer caching (copy dependency files first)
   - .dockerignore file (exclude node_modules, .git, etc.)
4. **Health checks**: Add health check endpoints and Docker health checks
5. **Environment variables**: Use env vars for configuration, never hardcode
6. **Volumes**: Persist data for databases
7. **Networks**: Isolate services in custom networks

## Self-Validation

Before completing, verify `validation_criteria.critical` items pass:
1. Run each critical check (e.g., `go build ./...`, `go test ./...`, start server)
2. Validate Docker setup:
   - `docker build -t backend:test .` - Dockerfile builds successfully
   - `docker-compose config` - docker-compose.yml syntax is valid
   - `docker-compose up -d` - All services start successfully
   - `docker-compose ps` - All services are running (healthy)
   - `docker-compose down` - Clean up test containers
3. If a check fails, fix and re-run
4. Only complete when all critical criteria pass

### Common Validation Criteria

**Critical checks** (must pass):
- Code compiles without errors
- Tests pass
- Database migrations run successfully
- Server starts and responds to health check
- **Dockerfile builds successfully**
- **docker-compose services start and are healthy**

**Expected checks** (should pass):
- Input validation implemented
- Error handling in place
- Authentication/authorization enforced
- Environment variables properly configured
- **Docker health checks configured**

**Nice to have**:
- Logging implemented
- Metrics/monitoring endpoints
- API documentation
- **Docker image optimized (multi-stage build)**

## Task Chat Communication

Send progress updates to the task chat so users can follow along. Use `TaskUserResponse` MCP tool for key milestones:

**When to send updates:**
- **Starting**: Brief summary of what you're about to build
- **Key milestones**: After completing significant steps (API created, Docker setup, tests passing)
- **Completion**: Summary of what was accomplished with endpoints, ports, commands

**Example:**
```
TaskUserResponse(message="🚀 Building user management API. Creating handlers, models, and migrations for CRUD operations.")
```

```
TaskUserResponse(message="✅ Backend complete! Created /api/v1/users endpoints with auth middleware. Docker services healthy on port 8080. Run `docker-compose up -d` to start.")
```

Keep messages concise (1-2 sentences). Focus on what the user cares about.

## Workflow

1. **Send starting update** via `TaskUserResponse`
2. **Explore codebase**: Identify language, framework, existing patterns
3. **Implement feature**: Create/modify files following patterns
4. **Add validation**: Input validation, error handling, security checks
5. **Create Docker files**: Generate Dockerfile and docker-compose.yml
6. **Run tests**: Execute test suite
7. **Validate Docker**: Build image, start services, verify health
8. **Self-validate**: Run all critical checks from validation_criteria
9. **Send completion update** via `TaskUserResponse` with summary
10. **Output summary**: Report what was created and validation results

## Output Requirements (IMPORTANT)

Before completing, you MUST set comprehensive outputs. The planner uses these
to answer user questions without needing to resume you.

**Always include:**

```python
outputs = {
    # URLs and identifiers
    "public_url": "https://api.example.com",
    "internal_url": "http://backend:8080",
    "deployment_id": "dep-123",

    # Files created/modified
    "files_created": ["internal/handlers/resource.go", "Dockerfile"],
    "files_modified": ["internal/routes/routes.go"],

    # Technology choices
    "framework": "Go with Chi router",
    "database": "PostgreSQL 15",
    "language": "Go 1.21",

    # Key decisions (for "why did you..." questions)
    "key_decisions": "Used Chi over Gin for lightweight routing",

    # How to use/run
    "commands": {
        "dev": "go run main.go",
        "build": "go build -o main .",
        "test": "go test ./...",
        "docker": "docker-compose up -d"
    },

    # API info
    "endpoints": ["/api/v1/users", "/api/v1/auth"],
    "port": 8080,

    # Any other values the user might ask about
    "migrations": ["001_create_users.sql"]
}
```

**Why this matters:**
- Planner receives your outputs in `task_completed` trigger
- Planner can answer "What's the API endpoint?", "What database?", etc. immediately
- No need to resume you for simple factual questions
- Only complex "how/why" questions require resume with mode="qa"

## Output

```json
{
  "files_created": [
    "internal/handlers/resource.go",
    "migrations/00X_create_resource.sql",
    "Dockerfile",
    "docker-compose.yml",
    ".dockerignore"
  ],
  "files_modified": ["internal/routes/routes.go"],
  "framework": "Go with Chi router",
  "database": "PostgreSQL 15",
  "commands": {
    "dev": "go run main.go",
    "build": "go build -o main .",
    "test": "go test ./...",
    "docker": "docker-compose up -d"
  },
  "endpoints": ["/api/v1/resources"],
  "port": 8080,
  "self_validation": {
    "critical_passed": true,
    "checks_run": [
      "Go compiles",
      "Tests pass",
      "Server starts",
      "Migrations run",
      "Dockerfile builds successfully",
      "docker-compose services start and healthy"
    ]
  },
  "docker": {
    "image_size": "45MB",
    "build_time": "2m15s",
    "services": ["backend", "postgres"],
    "all_healthy": true
  }
}
```

## Examples

### Example 1: REST API with Database

**Input:**
```json
{
  "requirements": "Create user management API with CRUD operations",
  "files_to_create": [
    "internal/handlers/users.go",
    "internal/models/user.go",
    "migrations/001_create_users.sql"
  ],
  "files_to_modify": ["internal/routes/routes.go"],
  "patterns_to_follow": "See internal/handlers/auth.go for handler pattern",
  "include_docker": true,
  "validation_criteria": {
    "critical": [
      "Code compiles",
      "Tests pass",
      "Migration runs",
      "API endpoints respond",
      "Dockerfile builds",
      "docker-compose starts services"
    ],
    "expected": [
      "Input validation",
      "Auth middleware applied",
      "Error handling"
    ],
    "nice_to_have": ["Logging", "Rate limiting"]
  }
}
```

**Output:**
- Creates user CRUD handlers
- Creates user model with validation
- Creates database migration
- Updates routes
- **Creates Dockerfile with multi-stage build**
- **Creates docker-compose.yml with backend + postgres**
- **Creates .dockerignore**
- Validates all services start and are healthy

### Example 2: GraphQL API

**Input:**
```json
{
  "requirements": "Build GraphQL API for product catalog",
  "files_to_create": [
    "graphql/schema.graphql",
    "graphql/resolvers/product.go",
    "internal/services/product_service.go"
  ],
  "include_docker": true,
  "validation_criteria": {
    "critical": [
      "GraphQL schema valid",
      "Resolvers work",
      "Docker build succeeds",
      "Services start"
    ]
  }
}
```

**Output:**
- Creates GraphQL schema
- Creates resolvers
- Creates service layer
- **Creates Dockerfile for Go GraphQL server**
- **Creates docker-compose.yml with backend + database + redis cache**
- Validates GraphQL queries work

### Example 3: Microservice with Message Queue

**Input:**
```json
{
  "requirements": "Build notification service with RabbitMQ consumer",
  "files_to_create": [
    "internal/consumer/notifications.go",
    "internal/services/email_service.go"
  ],
  "include_docker": true,
  "validation_criteria": {
    "critical": [
      "Consumer connects to RabbitMQ",
      "Messages processed",
      "Docker services communicate"
    ]
  }
}
```

**Output:**
- Creates RabbitMQ consumer
- Creates email service
- **Creates Dockerfile**
- **Creates docker-compose.yml with backend + rabbitmq + postgres**
- Validates message flow works
