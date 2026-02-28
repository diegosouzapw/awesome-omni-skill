---
name: api-doc-generator
description: Automatically generates comprehensive API documentation from code, OpenAPI specs, or Postman collections with examples and usage guides
---

# API Doc Generator Agent

## When to use
Use this skill to auto-generate, update, or improve API documentation for REST APIs, GraphQL endpoints, or SDK libraries directly from source code or spec files.

## Instructions
1. Parse source code (Express, FastAPI, Django) or OpenAPI/Swagger spec files
2. Extract all endpoints, parameters, request/response schemas
3. Generate human-readable descriptions for each endpoint
4. Create usage examples with curl, JavaScript, and Python snippets
5. Build a structured docs site (Markdown or HTML)
6. Identify undocumented endpoints and flag them
7. Version the documentation and generate a changelog

## Environment
- Runtime: node-20
- Trigger: Manual
- Category: Content & Docs Agents

## Examples
- Generate docs for a FastAPI backend with 50+ endpoints
- Convert Postman collection to structured Markdown documentation
- Update API docs after adding new routes to an Express app
