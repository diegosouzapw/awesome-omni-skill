---
name: feature-dev
description: Develop new features from requirements, specifications, or user stories following the full DevinClaw SDLC pipeline. Use this skill when implementing new microservices, API endpoints, UI components, or application features for the enterprise modernized applications, when building from an OpenAPI spec or Figma design, or when executing a complete SDD-to-delivery workflow for any new functionality.
---

# New Feature Development

## Overview

This skill implements new features end-to-end through the complete DevinClaw SDLC pipeline: specification, test-first development, implementation, review, and delivery. It takes requirements in any form — natural language descriptions, Jira tickets, OpenAPI specifications, Figma designs, or formal requirements documents — and produces production-ready code with full test coverage, security compliance, and audit trail. The skill leverages DeepWiki MCP for codebase understanding, generates SDD specifications and TDD test plans before writing implementation code, and can coordinate between Devin Cloud for autonomous build sessions and Devin CLI for interactive development requiring human-in-the-loop decisions.

The enterprise modernization program is replacing legacy applications with modern microservices architectures built on Java/Spring Boot, Python/FastAPI, React/TypeScript, and cloud-native infrastructure. Every new feature must comply with SOC 2 / ISO 27001, industry security CIS Benchmark, and cloud security compliance requirements from day one. This skill ensures that new development follows the same rigorous SDD→TDD→Build→Review pipeline regardless of whether the feature is a new REST endpoint, a React dashboard component, or an entire microservice.

## What's Needed From User

- **Feature requirements**: A description of the feature to build. Accepted formats:
 - Natural language description.
 - Jira ticket URL or ID.
 - OpenAPI/Swagger specification for API features.
 - Figma design URL for UI features.
 - Formal requirements document (Confluence page, PDF, or markdown).
- **Target repository**: The repository where the feature will be implemented.
- **Target branch**: The branch to base the feature branch on. Default: `main` or `develop`.
- **Technology stack** (optional): If not auto-detected from the repository (e.g., Java 17/Spring Boot 3, Python 3.11/FastAPI, React 18/TypeScript 5).
- **Acceptance criteria** (optional): Explicit acceptance criteria beyond what's in the requirements. If not provided, DevinClaw will derive criteria from the requirements and confirm with the user.
- **Architecture constraints** (optional): Specific patterns to follow (e.g., hexagonal architecture, CQRS, event sourcing) or constraints to observe.
- **Dependencies** (optional): New libraries or services this feature depends on that may need to be provisioned.
- **Interactive mode preference** (optional): Whether to use Devin Cloud (fully autonomous) or Devin CLI (interactive pair programming). Default: Devin Cloud for backend, Devin CLI for UI.

## Procedure

1. **Index codebase and understand architecture with DeepWiki**
 - Connect to the target repository via DeepWiki MCP and build or refresh the semantic index.
 - Identify the application architecture: package structure, layer boundaries, dependency injection patterns, configuration mechanisms.
 - Catalog existing patterns: how existing endpoints are structured, how services are organized, how database access is implemented, how tests are written.
 - Identify shared utilities, base classes, and cross-cutting concerns (logging, security, error handling) that the new feature must integrate with.
 - Map the existing API surface: current endpoints, data models, and authentication mechanisms.
 - Record the project's build system, linting rules, formatting configuration, and CI/CD pipeline structure.

2. **Parse requirements and derive acceptance criteria**
 - Ingest the feature requirements from the provided source:
 - For Jira tickets: extract via Jira MCP (summary, description, acceptance criteria, linked issues).
 - For OpenAPI specs: parse endpoint definitions, request/response schemas, and authentication requirements.
 - For Figma designs: extract component hierarchy, interaction patterns, responsive breakpoints, and accessibility requirements.
 - For natural language: extract functional requirements, non-functional requirements, and implicit constraints.
 - Derive testable acceptance criteria using the Given-When-Then format:
 - `Given [precondition], When [action], Then [expected outcome]`.
 - Identify security requirements: authentication, authorization, input validation, data classification.
 - Identify performance requirements: response time targets, throughput expectations, concurrent user support.
 - Present acceptance criteria to the user for confirmation before proceeding.

3. **Generate SDD specification**
 - Produce a Software Design Document covering:
 - **Feature overview**: Business purpose, user stories addressed, and scope boundaries.
 - **Architecture decisions**: How the feature fits into the existing application architecture. New packages, classes, database tables, API endpoints to be created.
 - **Data model**: New entities, relationships, database migrations (Flyway/Liquibase), and DTO mappings.
 - **API contract**: Request/response schemas, HTTP methods, status codes, error responses, pagination strategy. For GraphQL: queries, mutations, subscriptions, and type definitions.
 - **Security design**: Authentication requirements, authorization rules (RBAC/ABAC), input validation rules, data encryption requirements per SOC 2 / ISO 27001 and CIS Benchmark.
 - **Integration points**: External services, message queues, caches, and file systems the feature interacts with.
 - **Error handling**: Expected error conditions, error response format, retry strategies, circuit breaker configuration.
 - **Observability**: Logging requirements, metrics to emit, health check additions, tracing spans.
 - Submit the SDD to Advanced Devin for architectural review.
 - Advanced Devin validates: consistency with existing patterns, security compliance, scalability considerations, and completeness.

4. **Generate TDD test suite (failing tests)**
 - From the acceptance criteria and SDD, generate a comprehensive test suite:
 - **Unit tests**:
 - One test class per new class/module.
 - Tests for every public method covering happy path, edge cases, and error conditions.
 - Mock all dependencies at architectural boundaries.
 - Follow project's existing test naming and organization patterns.
 - **Integration tests**:
 - API endpoint tests covering all HTTP methods and response codes.
 - Database integration tests using Testcontainers.
 - Message queue integration tests if applicable.
 - Authentication and authorization enforcement tests.
 - **Contract tests** (for microservices):
 - Consumer-driven contract tests (Pact or Spring Cloud Contract).
 - API schema validation against the OpenAPI specification.
 - **UI tests** (for frontend features):
 - Component rendering tests with React Testing Library.
 - User interaction tests (click, type, submit, navigate).
 - Accessibility tests with axe-core integration.
 - End-to-end tests with Playwright for critical user flows.
 - Execute all tests to confirm they fail for the right reasons (compilation succeeds, tests fail on missing implementation).

5. **Implement the feature**
 - Select the execution spoke:
 - **Devin Cloud**: For backend features, API endpoints, database migrations, and service logic. Fully autonomous execution.
 - **Devin CLI**: For UI components, complex business logic requiring human judgment, and features touching sensitive data handling.
 - Implement in this order:
 - Database migrations first (new tables, columns, indexes, constraints).
 - Domain models and entities.
 - Repository/data access layer.
 - Service/business logic layer.
 - API controllers/handlers.
 - UI components (if applicable).
 - Configuration and wiring (Spring beans, dependency injection, environment variables).
 - Follow existing project patterns exactly. New code must be stylistically indistinguishable from existing code.
 - Apply security controls during implementation, not after:
 - Input validation on all user-facing parameters (CIS Benchmark INPUT-001).
 - Parameterized queries for all database access (CIS Benchmark SEC-XXX).
 - Structured audit logging for security-relevant actions (CIS Benchmark AUDIT-001).
 - Error handling with generic user messages and detailed internal logs (CIS Benchmark HEADERS-001).
 - Run all tests (new and pre-existing) after implementation. Fix any failures.

6. **Run guardrail checks**
 - Execute the full DevinClaw guardrail suite:
 - **HG-001**: All tests pass (unit, integration, e2e).
 - **HG-002**: CIS Benchmark scan clean (zero High findings).
 - **HG-003**: Lint and format pass (project's configured linter and formatter).
 - **HG-004**: Coverage >= 80% for all new and modified code.
 - **HG-005**: No secrets detected (TruffleHog/GitLeaks scan).
 - **HG-006**: No Critical/High CVEs in any new dependencies.
 - **HG-007**: All commits GPG-signed.
 - **HG-008**: Audit trail complete.
 - **HG-009**: License compliance for new dependencies.
 - **HG-010**: validated.
 - Fix any guardrail failures. Retry up to 3 times before escalating to human operator.

7. **Generate API documentation and OpenAPI spec**
 - For API features:
 - Generate or update the OpenAPI 3.1 specification.
 - Include request/response examples for every endpoint.
 - Document authentication requirements and error responses.
 - Validate the generated spec against the implementation.
 - For UI features:
 - Generate Storybook stories for new components.
 - Document component props, state management, and accessibility features.
 - Update project README or Confluence documentation as appropriate.

8. **Create pull request and invoke Devin Review**
 - Create a feature branch following the project's branching convention (e.g., `feature/JIRA-1234-add-work-order-endpoint`).
 - Create a PR with a descriptive title and body including:
 - Feature summary and Jira ticket link.
 - Architecture decisions made and rationale.
 - New API endpoints or UI components added.
 - Database migration summary.
 - Test coverage report (before/after delta).
 - Security controls implemented.
 - Screenshots or screen recordings for UI features.
 - Invoke Devin Review for automated review:
 - Code quality and pattern consistency.
 - Security vulnerability detection.
 - Performance anti-pattern detection.
 - Test quality and coverage validation.
 - Address all review findings before marking the PR as ready for human review.
 - Attach full audit trail as a PR comment.
 - Update DeepWiki with the new feature's architecture and design decisions.

## Specifications

- **Branch naming**: `feature/{ticket-id}-{short-description}` (e.g., `feature/enterprise-1234-work-order-api`).
- **Commit messages**: Conventional Commits format: `feat(scope): description`, `test(scope): description`, `docs(scope): description`.
- **PR size limit**: Maximum 50 files and 5,000 lines changed per PR (DevinClaw session limits). Split larger features into incremental PRs.
- **Test coverage**: Minimum 80% branch coverage for all new code. 100% coverage for security-critical paths (authentication, authorization, input validation).
- **API versioning**: All new API endpoints must be versioned (e.g., `/api/v1/`). No breaking changes to existing versioned endpoints.
- **Database migrations**: All schema changes via Flyway or Liquibase. Migrations must be idempotent and include rollback scripts. No manual DDL.
- **Dependency policy**: New dependencies require justification. Maximum 10 new dependencies per session. All must pass license compliance (HG-009) and vulnerability scanning (HG-006).
- **Security**: All user input validated (whitelist), all queries parameterized, all secrets from Vault (never hardcoded), all HTTP responses include security headers per SECURITY.md.
- **Accessibility**: All UI features must meet WCAG 2.1 AA and Section 508 compliance. axe-core must report zero violations.
- **Performance**: API endpoints must respond within 200ms at p95 for read operations, 500ms for write operations, under expected load.
- **Observability**: All new services must emit structured JSON logs, Prometheus metrics, and OpenTelemetry traces.

## Advice and Pointers

- Always implement the database migration layer first. Everything else depends on the data model being stable. Changing the data model after writing service and test code causes cascading rework.
- For Spring Boot features, follow the existing project's layer pattern exactly. If the project uses `controller → service → repository`, do not introduce a `handler` or `facade` layer even if you think it would be cleaner. Consistency trumps local optimization.
- When building from an OpenAPI spec, generate the server stubs first and then implement the business logic. This ensures the API contract is honored exactly. Use OpenAPI Generator for initial scaffolding but hand-code the business logic.
- For React/TypeScript UI features, build components bottom-up: atomic components first, then composite components, then page-level components. This enables incremental testing and review.
- enterprise applications often have complex authorization models (role-based + attribute-based). Test authorization exhaustively: every role against every endpoint, including edge cases like expired tokens and revoked roles.
- When adding new microservices, include health check endpoints (`/actuator/health` for Spring Boot, `/health` for others) from the start. The enterprise infrastructure monitoring depends on consistent health check contracts.
- If the feature requires a new database table, always include indexes for columns that will be used in WHERE clauses, JOIN conditions, or ORDER BY expressions. The organization's PostgreSQL instances handle millions of records and index-free queries cause cascading performance issues.
- For features that process enterprise-specific data formats (data standard, FIXM, notification), validate against the official schemas. These formats are strictly specified by international standards bodies and any deviation breaks interoperability.
- Interactive mode (Devin CLI) is preferred when the feature requires UI/UX decisions, complex business rule interpretation, or touches safety-critical code paths. Use Devin Cloud for well-specified backend work.
- When splitting a large feature across multiple PRs, ensure each PR is independently deployable and does not break existing functionality. Use feature flags if the feature needs to be hidden until complete.


## Self-Verification Loop (Devin 2.2)

After completing the primary procedure:

1. **Self-verify**: Run all applicable verification gates:
 - Build/test gates: compilation, unit tests, integration tests, lint, typecheck
 - Security gates: dependency scan, security benchmark compliance check, input validation audit
 - UI gates: computer-use E2E smoke test (if applicable), Section 508 accessibility validation
2. **Auto-fix**: If any verification gate fails, attempt automated repair — adjust code, configuration, or test fixtures to resolve the failure.
3. **Re-verify**: Run all verification gates again after fixes. Confirm each gate transitions from FAIL to PASS.
4. **Escalate**: If auto-fix fails after 2 attempts, escalate to human reviewer with a complete evidence pack. Include the failing gate identifier, error output, attempted fixes, and root cause hypothesis.

## Artifact Contract

Every stage of this skill produces paired outputs for machine-consumable handoff:

| Stage | Markdown Output | JSON Output |
|-------|----------------|-------------|
| Requirements Analysis | `requirements.md` | `requirements.json` |
| SDD Specification | `sdd_spec.md` | `sdd_spec.json` |
| TDD Test Stubs | `tdd_stubs.md` | `tdd_stubs.json` |
| Implementation | `implementation.md` | `implementation.json` |
| Integration Testing | `integration_test.md` | `integration_test.json` |

JSON outputs must conform to the schema defined in `audit/artifact-schemas/`. Markdown outputs are the human-readable narrative; JSON outputs are the machine-consumable contract consumed by the next stage or by OpenClaw for artifact validation.

## Evidence Pack

On completion, produce `evidence-pack.json` containing:

```json
{
 "session_id": "<Devin session identifier>",
 "timestamp": "<ISO 8601 completion time>",
 "skill_id": "devinclaw.feature_dev.v1",
 "artifacts": [
 {
 "filename": "<output file>",
 "sha256": "<SHA-256 hash of file contents>",
 "stage": "<which stage produced this artifact>"
 }
 ],
 "verification": {
 "gates_run": ["<gate_1>", "<gate_2>"],
 "gates_passed": ["<gate_1>", "<gate_2>"],
 "gates_failed": [],
 "auto_fix_attempts": 0,
 "test_summary": {"passed": 0, "failed": 0, "skipped": 0},
 "scan_summary": {"critical": 0, "high": 0, "medium": 0, "low": 0}
 },
 "knowledge_updates": [
 {
 "action": "created|updated",
 "knowledge_id": "<Devin knowledge entry ID>",
 "summary": "<what was learned>"
 }
 ],
 "escalations": [
 {
 "gate": "<failing gate>",
 "reason": "<why auto-fix failed>",
 "evidence": "<link to error output>"
 }
 ]
}
```

## Escalation Policy

- **Divergence threshold**: 0.35 — if parallel verification sessions disagree beyond this threshold on key findings, escalate to human reviewer with both evidence packs for adjudication.
- **Human approval required for**: new API contract definitions, database schema changes, authentication/authorization modifications, safety-critical feature logic.
- **Auto-escalate on**: Any security finding rated HIGH or CRITICAL, any risk of data loss or corruption, any changes to authentication or authorization logic, any modification to safety-critical code paths (DO-178C applicable systems).

## Forbidden Actions

- Do not begin implementation before the SDD specification is generated and reviewed by Advanced Devin. Every feature must have a documented design.
- Do not begin implementation before TDD test stubs are generated and confirmed to fail correctly. Tests come before code.
- Do not modify existing API contracts in backward-incompatible ways. If a new version of an endpoint is needed, create a new versioned endpoint and deprecate the old one.
- Do not introduce new dependencies without checking license compliance (HG-009) and vulnerability status (HG-006). No GPL/AGPL dependencies in Apache-licensed modules.
- Do not hardcode configuration values, secrets, API keys, or environment-specific URLs. All configuration must come from environment variables, Vault, or externalized configuration.
- Do not skip guardrail checks. All 10 hard gates must pass before the PR is created.
- Do not deploy to production environments. This skill creates PRs for human review and approval. Deployment is a separate authorized action.
- Do not ignore or suppress linter warnings. Fix all warnings or document the justification for a suppression with an inline comment referencing the specific rule ID.
- Do not write raw SQL queries with string concatenation. All database queries must use parameterized queries or an ORM's query builder to prevent SQL injection (CIS Benchmark SEC-XXX).
- Do not generate or store test data containing real PII, PHI, or sensitive enterprise operational data. All test data must be synthetic.
