---
name: architecture-format-extended
description: Extended architecture templates with full examples. Imports architecture-format-core for base structure.
version: 1.1
tier: 2
requires: architecture-format-core
---

# Architecture Document Structure (Extended)

> [!NOTE]
> This skill provides **FULL** templates with detailed examples.
> It extends `architecture-format-core` with comprehensive sections 3-10.

## When to Load This Skill

| Condition | Action |
|-----------|--------|
| Creating NEW system from scratch | ✅ Load this skill |
| Major refactor (>3 components changed) | ✅ Load this skill |
| Sophisticated requirement / complex task | ✅ Load this skill |
| User explicitly requests full template | ✅ Load this skill |
| Minor architecture update | ❌ Use `architecture-format-core` only |

---

## 3. System Components (Extended Examples)

> [!TIP]
> **Example Template:** See `.agent/skills/architecture-format-extended/examples/component-template.md`

Use the detailed component template for defining new services or major components.

---

## 4. Data Model (Full)

> [!TIP]
> **Example Template:** See `.agent/skills/architecture-format-extended/examples/data-model.md`

### 4.1. Conceptual Data Model
Description of main entities and their relationships at a high level.

### 4.2. Logical Data Model
Detailed description considering storage technology (Relational vs NoSQL).

### 4.3. Data Model Diagram
ER-diagram in PlantUML format.

### 4.4. Migrations and Versioning
Strategy for DB schema changes.

---

## 5. Interfaces

> [!TIP]
> **Example Template:** See `.agent/skills/architecture-format-extended/examples/api-interface.md`

### 5.1. External APIs
Detailed REST/GraphQL/gRPC definitions including authentication and error handling.

### 5.2. Internal Interfaces
Interaction between system components (e.g., Message Queues, Events).

### 5.3. Integrations with External Systems
Third-party service purpose, protocol, and error handling strategies.

---

## 6. Technology Stack

### 6.1. Backend
- Programming Language & Framework with justification.

### 6.2. Frontend
- Framework with justification.

### 6.3. Database
- Type (SQL/NoSQL) with justification.

### 6.4. Infrastructure
- Containerization (Docker)
- Orchestration (K8s/Compose)
- Middleware (Redis, RabbitMQ)
- Observability (Prometheus, ELK)

---

## 7. Security

### 7.1. Authentication and Authorization
- Auth Mechanism (JWT/OAuth)
- Session Management

### 7.2. Data Protection
- Encryption (At Rest / In Transit)
- PII handling

### 7.3. Attack Protection
- OWASP Top 10 (SQLi, XSS, CSRF)
- Rate Limiting

---

## 8. Scalability and Performance

### 8.1. Scaling Strategy
- Horizontal vs Vertical scaling plans.

### 8.2. Caching
- Strategy, specific items, and invalidation rules.

### 8.3. DB Optimization
- Indexes, Partitioning, Replication.

---

## 9. Reliability and Fault Tolerance

### 9.1. Error Handling
- Degredation, Circuit Breakers, Retries.

### 9.2. Backup
- Strategy, Frequency, Storage.

### 9.3. Monitoring and Alerting
- Key Metrics (Latency, Errors, Saturation).

---

## 10. Deployment

### 10.1. Environments
- Dev, Staging, Prod definitions.

### 10.2. CI/CD Pipeline
- Build -> Test -> Deploy stages.

### 10.3. Configuration
- Env vars, Secrets management.

### 10.4. Deployment Instructions
- Step-by-step guide for deployment and migrations.
