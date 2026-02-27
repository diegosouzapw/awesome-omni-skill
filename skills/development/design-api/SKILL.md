---
name: design-api
description: Triggered when user asks to design APIs, create API specifications, or plan API endpoints. Automatically delegates to the api-designer agent.
allowed-tools: Read, Grep, Glob
context: fork
agent: api-designer
---

# Design API Skill

## Trigger Phrases

This skill is automatically triggered when the user:

- Asks to "design API" or "create API"
- Requests API endpoints or specifications
- Wants REST API design
- Mentions "API design", "endpoints", or "API documentation"
- Asks about API structure or patterns

## Delegation Instructions

When this skill is triggered:

1. **Delegate immediately** to the `api-designer` agent
2. Provide API requirements and use cases
3. Include resource definitions
4. Specify API style preferences (REST, GraphQL, etc.)
5. Include any constraints or standards

## Context to Pass

- **API Purpose**: What the API needs to do
- **Resources**: Resources the API will manage
- **Use Cases**: Main use cases and workflows
- **API Style**: REST, GraphQL, RPC, etc.
- **Standards**: Any API standards to follow
- **Constraints**: Rate limiting, authentication, etc.

## Agent Responsibilities

The api-designer agent will:

1. Design API structure and endpoints
2. Define request/response schemas
3. Plan error handling
4. Ensure RESTful design
5. Create API documentation
6. Follow API best practices
