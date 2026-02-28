---
name: full-stack-orchestration-full-stack-feature
description: Use when working with full stack orchestration full stack feature 
category: Security & Systems
source: antigravity
tags: [python, react, node, api, ai, agent, workflow, template, design, document]
url: https://github.com/sickn33/antigravity-awesome-skills/tree/main/skills/full-stack-orchestration-full-stack-feature
---


## Use this skill when

- Working on full stack orchestration full stack feature tasks or workflows
- Needing guidance, best practices, or checklists for full stack orchestration full stack feature

## Do not use this skill when

- The task is unrelated to full stack orchestration full stack feature
- You need a different domain or tool outside this scope

## Instructions

- Clarify goals, constraints, and required inputs.
- Apply relevant best practices and validate outcomes.
- Provide actionable steps and verification.
- If detailed examples are required, open `resources/implementation-playbook.md`.

Orchestrate full-stack feature development across backend, frontend, and infrastructure layers with modern API-first approach:

[Extended thinking: This workflow coordinates multiple specialized agents to deliver a complete full-stack feature from architecture through deployment. It follows API-first development principles, ensuring contract-driven development where the API specification drives both backend implementation and frontend consumption. Each phase builds upon previous outputs, creating a cohesive system with proper separation of concerns, comprehensive testing, and production-ready deployment. The workflow emphasizes modern practices like component-driven UI development, feature flags, observability, and progressive rollout strategies.]

## Phase 1: Architecture & Design Foundation

### 1. Database Architecture Design
- Use Task tool with subagent_type="database-design::database-architect"
- Prompt: "Design database schema and data models for: $ARGUMENTS. Consider scalability, query patterns, indexing strategy, and data consistency requirements. Include migration strategy if modifying existing schema. Provide both logical and physical data models."
- Expected output: Entity relationship diagrams, table schemas, indexing strategy, migration scripts, data access patterns
- Context: Initial requirements and business domain model

### 2. Backend Service Architecture
- Use Task tool with subagent_type="backend-development::backend-architect"
- Prompt: "Design backend service architecture for: $ARGUMENTS. Using the database design from previous step, create service boundaries, define API contracts (OpenAPI/GraphQL), design authentication/authorization strategy, and specify inter-service communication patterns. Include resilience patterns (circuit breakers, retries) and caching strategy."
- Expected output: Service architecture diagram, OpenAPI specifications, authentication flows, caching architecture, message queue design (if applicable)
- Context: Database schema from step 1, non-functional requirements

### 3. Frontend Component Architecture
- Use Task tool with subagent_type="frontend-mobile-development::frontend-developer"
- Prompt: "Design frontend architecture and component structure for: $ARGUMENTS. Based on the API contracts from previous step, design component hierarchy, state management approach (Redux/Zustand/Context), routing structure, and data fetching patterns. Include accessibility requirements and responsive design strategy. Plan for Storybook component documentation."
- Expected output: Component tree diagram, state management design, routing configuration, design system integration plan, accessibility checklist
- Context: API specifications from step 2, UI/UX requirements

## Phase 2: Parallel Implementation

### 4. Backend Service Implementation
- Use Task tool with subagent_type="python-development::python-pro" (or "golang-pro"/"nodejs-expert" based on stack)
- Prompt: "Implement backend services for: $ARGUMENTS. Using the architecture and API specs from Phase 1, build RESTful/GraphQL endpoints with proper validation, error handling, and logging. Implement business logic, data access layer, authentication middleware, and integration with external services. Include observability (structured logging, metrics, tracing)."
- Expected output: Backend service code, API endpoints, middleware, background jobs, unit tests, integration tests
- Context: Architecture designs from Phase 1, database schema

### 5. Frontend Implementation
- Use Task tool with subagent_type="frontend-mobile-development::frontend-developer"
- Prompt: "Implement frontend application for: $ARGUMENTS. Build React/Next.js components using the component architecture from Phase 1. Implement state management, API integration with proper error handling and loading states, form validation, and responsive layouts. Create Storybook stories for components. Ensure accessibility (WCAG 2.1 AA compliance)."
- Expected output: React components, state management implementation, API client code, Storybook stories, responsive styles, accessibility implementations
- Context: Component architecture from step 3, API contracts

### 6. Database Implementation & Optimization
- Use Task tool with subagent_type="database-design::sql-pro"
- Prompt: "Implement and optimize database layer for: $ARGUMENTS. Create migration scripts, stored procedur
