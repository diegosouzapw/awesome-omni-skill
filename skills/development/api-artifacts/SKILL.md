---
name: api-artifacts
description: Templates and rules for generating OpenAPI specs, Postman collections, AsyncAPI specs, and GraphQL schemas from architecture manifests
---

# API Artifacts

Generate ready-to-use API documentation and testing artifacts from the system manifest. These artifacts let developers import specs directly into tools like Swagger UI, Postman, and AsyncAPI Studio.

---

## OpenAPI 3.1 Specification

Generate one OpenAPI spec per REST API service in the manifest.

### Structure

```yaml
openapi: 3.1.0
info:
  title: "{{service-name}} API"
  description: "{{service-description}}"
  version: "0.1.0"
servers:
  - url: http://localhost:{{port}}
    description: Local development
  - url: https://{{service-name}}.{{domain}}
    description: Production

security:
  - bearerAuth: []

paths:
  # Generate from service responsibilities + shared types
  /health:
    get:
      summary: Health check
      operationId: getHealth
      tags: [system]
      security: []
      responses:
        "200":
          description: Service is healthy
          content:
            application/json:
              schema:
                type: object
                properties:
                  status:
                    type: string
                    example: "ok"
                  service:
                    type: string
                    example: "{{service-name}}"

  /{{resource}}:
    get:
      summary: "List {{resource}}"
      operationId: "list{{Resource}}"
      tags: [{{resource}}]
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
            maximum: 100
      responses:
        "200":
          description: "List of {{resource}}"
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/{{Resource}}"
                  pagination:
                    $ref: "#/components/schemas/Pagination"
        "401":
          $ref: "#/components/responses/Unauthorized"

    post:
      summary: "Create {{resource}}"
      operationId: "create{{Resource}}"
      tags: [{{resource}}]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/Create{{Resource}}"
            example:
              # Derive from shared type fields
      responses:
        "201":
          description: "{{Resource}} created"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/{{Resource}}"
        "400":
          $ref: "#/components/responses/BadRequest"
        "401":
          $ref: "#/components/responses/Unauthorized"

  /{{resource}}/{id}:
    get:
      summary: "Get {{resource}} by ID"
      operationId: "get{{Resource}}"
      tags: [{{resource}}]
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        "200":
          description: "{{Resource}} details"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/{{Resource}}"
        "404":
          $ref: "#/components/responses/NotFound"

    put:
      summary: "Update {{resource}}"
      operationId: "update{{Resource}}"
      tags: [{{resource}}]
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/Update{{Resource}}"
      responses:
        "200":
          description: "{{Resource}} updated"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/{{Resource}}"

    delete:
      summary: "Delete {{resource}}"
      operationId: "delete{{Resource}}"
      tags: [{{resource}}]
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        "204":
          description: "{{Resource}} deleted"

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  schemas:
    # Derive from shared types in manifest
    {{Resource}}:
      type: object
      properties:
        id:
          type: string
          format: uuid
        # ... fields from shared type
        createdAt:
          type: string
          format: date-time
        updatedAt:
          type: string
          format: date-time
      required: [id]

    Create{{Resource}}:
      type: object
      properties:
        # ... writable fields only (exclude id, timestamps)
      required: [# ... required fields]

    Update{{Resource}}:
      type: object
      properties:
        # ... writable fields, all optional

    Pagination:
      type: object
      properties:
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        totalPages:
          type: integer

    Error:
      type: object
      properties:
        code:
          type: string
        message:
          type: string
        details:
          type: object

  responses:
    BadRequest:
      description: Invalid request
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Error"
          example:
            code: "VALIDATION_ERROR"
            message: "Invalid request body"
            details: {}
    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Error"
          example:
            code: "UNAUTHORIZED"
            message: "Authentication required"
    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Error"
          example:
            code: "NOT_FOUND"
            message: "Resource not found"
```

### Generation Rules

1. **Derive paths from service responsibilities** — Each responsibility maps to a resource. `ticket-management` → `/tickets`, `user-management` → `/users`.
2. **Derive schemas from shared types** — Each shared type becomes a component schema. Use its fields for properties.
3. **Always include standard CRUD** — GET list, GET by ID, POST, PUT, DELETE for each resource unless the service is read-only or write-only.
4. **Always include health endpoint** — Every service gets `/health` with no auth.
5. **Always include standard error responses** — 400, 401, 403, 404, 500 with consistent Error schema.
6. **Use realistic examples** — Generate example values that make sense for the domain (not "string" or "test").
7. **Include pagination** — All list endpoints use page/limit pagination.
8. **Match auth to manifest** — Use the auth method from the communication section (JWT bearer, API key, etc.).

---

## Postman Collection

Generate one Postman collection per REST API service. Use Postman Collection v2.1 format.

### Structure

```json
{
  "info": {
    "name": "{{service-name}} API",
    "description": "{{service-description}}",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "auth": {
    "type": "bearer",
    "bearer": [
      {
        "key": "token",
        "value": "{{auth_token}}",
        "type": "string"
      }
    ]
  },
  "variable": [
    {
      "key": "base_url",
      "value": "http://localhost:{{port}}"
    },
    {
      "key": "auth_token",
      "value": ""
    }
  ],
  "item": [
    {
      "name": "Health",
      "item": [
        {
          "name": "Health Check",
          "request": {
            "method": "GET",
            "url": "{{base_url}}/health",
            "auth": { "type": "noauth" }
          }
        }
      ]
    },
    {
      "name": "{{Resource}}",
      "item": [
        {
          "name": "List {{resource}}",
          "request": {
            "method": "GET",
            "url": {
              "raw": "{{base_url}}/{{resource}}?page=1&limit=20",
              "host": ["{{base_url}}"],
              "path": ["{{resource}}"],
              "query": [
                { "key": "page", "value": "1" },
                { "key": "limit", "value": "20" }
              ]
            }
          }
        },
        {
          "name": "Get {{resource}} by ID",
          "request": {
            "method": "GET",
            "url": "{{base_url}}/{{resource}}/{{resource_id}}"
          }
        },
        {
          "name": "Create {{resource}}",
          "request": {
            "method": "POST",
            "url": "{{base_url}}/{{resource}}",
            "header": [
              { "key": "Content-Type", "value": "application/json" }
            ],
            "body": {
              "mode": "raw",
              "raw": "// JSON body with example values from shared types"
            }
          }
        },
        {
          "name": "Update {{resource}}",
          "request": {
            "method": "PUT",
            "url": "{{base_url}}/{{resource}}/{{resource_id}}",
            "header": [
              { "key": "Content-Type", "value": "application/json" }
            ],
            "body": {
              "mode": "raw",
              "raw": "// JSON body with example values"
            }
          }
        },
        {
          "name": "Delete {{resource}}",
          "request": {
            "method": "DELETE",
            "url": "{{base_url}}/{{resource}}/{{resource_id}}"
          }
        }
      ]
    }
  ]
}
```

### Generation Rules

1. **One folder per resource** — Group requests by domain resource (Users, Tickets, etc.).
2. **Include example bodies** — POST and PUT requests include realistic JSON bodies derived from shared types.
3. **Use collection variables** — `base_url` and `auth_token` as variables so the user can switch environments.
4. **Include auth setup** — Collection-level bearer auth with `auth_token` variable.
5. **Order requests logically** — List → Get → Create → Update → Delete within each folder.
6. **Add descriptions** — Each request gets a one-line description of what it does.

---

## AsyncAPI 2.6 Specification

Generate one AsyncAPI spec for event-driven / message queue connections in the manifest.

### Structure

```yaml
asyncapi: 2.6.0
info:
  title: "{{project-name}} Events"
  version: "0.1.0"
  description: "Event channels for {{project-name}} architecture"

servers:
  development:
    url: localhost:6379
    protocol: redis
    description: Local Redis for BullMQ
  production:
    url: "{{redis-url}}"
    protocol: redis
    description: Production Redis

channels:
  {{queue-name}}:
    description: "{{queue-description}}"
    publish:
      operationId: "publish{{EventName}}"
      summary: "{{event-summary}}"
      message:
        $ref: "#/components/messages/{{EventName}}"
    subscribe:
      operationId: "consume{{EventName}}"
      summary: "{{consumer-summary}}"
      message:
        $ref: "#/components/messages/{{EventName}}"

components:
  messages:
    {{EventName}}:
      name: {{EventName}}
      title: "{{event-title}}"
      contentType: application/json
      payload:
        $ref: "#/components/schemas/{{EventPayload}}"

  schemas:
    {{EventPayload}}:
      type: object
      properties:
        eventId:
          type: string
          format: uuid
        eventType:
          type: string
          enum: [{{event-types}}]
        timestamp:
          type: string
          format: date-time
        data:
          type: object
          properties:
            # Derive from shared types and event contracts
```

### Generation Rules

1. **One channel per message queue connection** — Derive from manifest `communication` entries with `pattern: message-queue` or `pattern: event-bus`.
2. **Derive message schemas from event contracts** — Use the `shared.contracts` with `type: event-schema` to define payloads.
3. **Include both publish and subscribe** — Show which service publishes and which consumes.
4. **Match protocol to infrastructure** — Redis for BullMQ, AMQP for RabbitMQ, Kafka for event bus.
5. **Include standard envelope** — Every event has `eventId`, `eventType`, `timestamp`, and `data`.

---

## GraphQL Schema

Generate for services with `type: graphql` in the manifest.

### Structure

```graphql
type Query {
  """List {{resource}} with pagination"""
  {{resource}}s(page: Int = 1, limit: Int = 20): {{Resource}}Connection!

  """Get a single {{resource}} by ID"""
  {{resource}}(id: ID!): {{Resource}}
}

type Mutation {
  """Create a new {{resource}}"""
  create{{Resource}}(input: Create{{Resource}}Input!): {{Resource}}!

  """Update an existing {{resource}}"""
  update{{Resource}}(id: ID!, input: Update{{Resource}}Input!): {{Resource}}!

  """Delete a {{resource}}"""
  delete{{Resource}}(id: ID!): Boolean!
}

type {{Resource}} {
  id: ID!
  # ... fields from shared types
  createdAt: DateTime!
  updatedAt: DateTime!
}

input Create{{Resource}}Input {
  # ... writable fields
}

input Update{{Resource}}Input {
  # ... writable fields, all optional
}

type {{Resource}}Connection {
  edges: [{{Resource}}!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type PageInfo {
  page: Int!
  limit: Int!
  totalPages: Int!
  hasNextPage: Boolean!
}

scalar DateTime
```

### Generation Rules

1. **Derive types from shared types** — Each shared type consumed by the GraphQL service becomes a GraphQL type.
2. **Separate Input types** — Create and Update inputs are separate from the main type.
3. **Connection pattern for lists** — Use Relay-style connections with edges and pageInfo.
4. **Include subscriptions** — If the service has real-time features, add subscription types.

---

## Artifact Output Format

When generating artifacts in the blueprint, present each one in a fenced code block with the appropriate language tag:

- OpenAPI → ` ```yaml ` with filename comment `# {{service-name}}-openapi.yaml`
- Postman → ` ```json ` with filename comment at top
- AsyncAPI → ` ```yaml ` with filename comment `# {{project-name}}-asyncapi.yaml`
- GraphQL → ` ```graphql ` with filename comment `# {{service-name}}-schema.graphql`

Always include a usage note after each artifact explaining how to import it into the relevant tool.

## When to Generate Which Artifact

| Manifest Condition | Artifacts to Generate |
|---|---|
| Service with `type: rest-api` | OpenAPI spec + Postman collection |
| Service with `type: graphql` | GraphQL schema |
| Communication with `pattern: message-queue` or `pattern: event-bus` | AsyncAPI spec |
| Service with `type: websocket` | AsyncAPI spec (WebSocket protocol) |
| Any combination | Generate all applicable artifacts |

If no services have APIs (e.g., agent-only project), skip this deliverable and note why.
