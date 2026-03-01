---
name: api-documentor
description: Generates OpenAPI/Swagger/GraphQL documentation and SDKs from code
version: 1.0.0
tags: [api, documentation, openapi, swagger, graphql, sdk]
---

# API Documentor Skill

## Purpose

The API Documentor Skill automatically generates comprehensive API documentation from code, including OpenAPI/Swagger specs, GraphQL schemas, and client SDKs. It eliminates manual documentation efforts and ensures API docs stay synchronized with implementation.

**Key Capabilities:**
- Generate OpenAPI 3.0 specifications from REST APIs
- Create GraphQL schema documentation
- Auto-generate client SDKs (Python, JavaScript, Go, Java)
- Generate API reference documentation
- Create interactive API explorers
- Validate API implementations against specs

**Target Token Savings:** 80% (from ~3000 tokens to ~600 tokens)

## When to Use

Use the API Documentor Skill when:

- Building or updating REST APIs
- Creating GraphQL services
- Generating client SDKs
- Publishing API documentation
- Validating API implementations
- Creating API mock servers
- Onboarding API consumers
- Versioning APIs

**Trigger Phrases:**
- "Generate OpenAPI spec"
- "Create API documentation"
- "Generate client SDK"
- "Document GraphQL schema"
- "Create Swagger docs"
- "Generate API reference"

## Operations

### 1. generate-openapi
Generates OpenAPI 3.0 specification from REST API code.

**Features:**
- Automatic endpoint detection
- Request/response schema extraction
- Authentication documentation
- Error code documentation
- Example request/responses

### 2. generate-graphql
Creates GraphQL schema documentation from resolvers.

**Features:**
- Type definitions
- Query documentation
- Mutation documentation
- Subscription support
- Directive documentation

### 3. generate-sdk
Auto-generates client SDKs in multiple languages.

**Languages:**
- Python (requests-based)
- JavaScript/TypeScript
- Go
- Java
- Ruby

### 4. generate-docs
Creates human-readable API reference documentation.

**Formats:**
- HTML
- Markdown
- PDF

### 5. validate-api
Validates API implementation against OpenAPI spec.

**Checks:**
- Endpoint availability
- Request/response schemas
- Authentication requirements
- Error responses

### 6. create-mock
Generates API mock server from specification.

**Features:**
- Example-based responses
- Schema-based response generation
- Delayed responses
- Error simulation

## Scripts

### Generate OpenAPI Specification

```bash
# Generate from Flask/FastAPI application
python ~/.claude/skills/api-documentor/scripts/main.py \
  --operation generate-openapi \
  --app-file app.py \
  --output openapi.yaml

# Generate with custom info
python ~/.claude/skills/api-documentor/scripts/main.py \
  --operation generate-openapi \
  --app-file app.py \
  --title "My API" \
  --version "1.0.0" \
  --output openapi.yaml
```

### Generate GraphQL Documentation

```bash
# Generate from GraphQL schema
python ~/.claude/skills/api-documentor/scripts/main.py \
  --operation generate-graphql \
  --schema-file schema.graphql \
  --output graphql-docs.html
```

### Generate Client SDK

```bash
# Generate Python SDK
python ~/.claude/skills/api-documentor/scripts/main.py \
  --operation generate-sdk \
  --spec openapi.yaml \
  --language python \
  --output-dir ./sdks/python

# Generate TypeScript SDK
python ~/.claude/skills/api-documentor/scripts/main.py \
  --operation generate-sdk \
  --spec openapi.yaml \
  --language typescript \
  --output-dir ./sdks/typescript
```

### Generate Documentation

```bash
# Generate HTML docs
python ~/.claude/skills/api-documentor/scripts/main.py \
  --operation generate-docs \
  --spec openapi.yaml \
  --format html \
  --output api-docs.html
```

## Configuration

```json
{
  "api-documentor": {
    "openapi": {
      "version": "3.0.0",
      "title": "API Documentation",
      "description": "Auto-generated API documentation",
      "contact": {
        "name": "API Team",
        "email": "api@example.com"
      },
      "servers": [
        {"url": "https://api.example.com", "description": "Production"},
        {"url": "https://api-staging.example.com", "description": "Staging"}
      ]
    },
    "sdk": {
      "languages": ["python", "javascript", "typescript", "go"],
      "package_name": "api-client",
      "include_examples": true
    },
    "output": {
      "format": "yaml",
      "pretty_print": true
    }
  }
}
```

## Examples

### Example 1: Generate OpenAPI from Flask App

```bash
python ~/.claude/skills/api-documentor/scripts/main.py \
  --operation generate-openapi \
  --app-file app.py
```

**Output:**
```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
paths:
  /users:
    get:
      summary: List users
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
```

### Example 2: Generate Python SDK

```bash
python ~/.claude/skills/api-documentor/scripts/main.py \
  --operation generate-sdk \
  --spec openapi.yaml \
  --language python
```

**Output:** Complete Python SDK with methods for all API endpoints

## Token Economics

**Without Skill:** ~3000 tokens (manual documentation)
**With Skill:** ~600 tokens (80% savings)

## Success Metrics

- Execution time: <500ms for OpenAPI generation
- SDK generation: <2s for full client
- Accuracy: 100% spec compliance
- Token savings: 80%

---

**API Documentor Skill v1.0.0** - Automated API documentation generation
