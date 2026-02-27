---
name: eng-spec
description: "Generate an Engineering Specification. Use when the user says /eng-spec, asks to create a technical spec, engineering spec, system design document, or translate a PRD into a technical plan. Triggers: eng-spec, engineering spec, technical spec, system design, technical design, architecture spec."
---

# Engineering Specification

## Purpose

Generate a detailed engineering specification that translates product requirements into a concrete technical plan. The spec gives engineers a clear blueprint for implementation, covering architecture, data models, APIs, and operational concerns.

## When to Use

- After a PRD has been approved and engineering work is about to begin
- When a technical approach needs to be documented for review
- When onboarding engineers to an existing system's design

## Inputs

- **Feature or system name**: What is being specified
- **Product requirements or PRD**: The "what" that this spec addresses
- **Technical context**: Existing stack, services, constraints (optional but helpful)
- **Scale expectations**: Expected load, data volume, user count (optional)

## Output Format

Produce a markdown document with the following sections:

### 1. System Overview
A brief description of what the system or feature does and how it fits into the broader architecture. Include a high-level diagram description if helpful.

### 2. Architecture Decisions
List key technical decisions using the format:
- **Decision**: What was decided
- **Rationale**: Why this approach was chosen
- **Alternatives considered**: What else was evaluated and why it was rejected

### 3. Data Models
Define the core entities, their fields, types, and relationships. Use table format:

| Field | Type | Required | Description |
|-------|------|----------|-------------|

Include notes on indexing, constraints, and migration from existing schemas.

### 4. API Contracts
For each endpoint or interface, specify:
- Method and path (or function signature)
- Request parameters and body schema
- Response schema with status codes
- Authentication and authorization requirements
- Rate limiting or usage constraints

### 5. Error Handling
Define the error strategy:
- Error categories and codes
- Retry behavior and idempotency
- User-facing vs. internal error messages
- Logging and alerting requirements

### 6. Testing Strategy
Outline the testing approach:
- **Unit tests**: Key logic to cover
- **Integration tests**: Service boundaries to validate
- **End-to-end tests**: Critical user flows
- **Performance tests**: Load and latency benchmarks

### 7. Migration Plan
If this changes existing systems:
- Step-by-step migration sequence
- Data backfill requirements
- Feature flag strategy
- Rollback procedure

### 8. Security Considerations
Address authentication, authorization, data encryption, input validation, and any compliance requirements.

### 9. Open Questions and Risks
List unresolved technical questions and identified risks with proposed mitigations.

## Example

**Input**: "We need an eng spec for the PDF export feature described in the PRD. Our stack is Node.js, PostgreSQL, and AWS."

**Output**: A full engineering spec covering the architecture (async job queue with S3 storage for generated PDFs), data models (ExportJob, ExportTemplate tables), API contracts (POST /exports to trigger, GET /exports/:id to poll status, GET /exports/:id/download for retrieval), error handling (retry failed renders up to 3 times, notify user on permanent failure), testing strategy (unit tests for template rendering, integration tests for the job pipeline, load tests for concurrent generation), and migration plan (new tables, no schema changes to existing).

## Guidelines

- Be precise about data types, constraints, and expected values
- Separate the "happy path" from edge cases and error flows
- Include enough detail for an engineer to implement without ambiguity
- Call out dependencies on other teams or services explicitly
- Keep security and observability as first-class concerns, not afterthoughts
