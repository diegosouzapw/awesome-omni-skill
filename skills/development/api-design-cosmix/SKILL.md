---
name: api-design
description: Designs RESTful APIs, GraphQL schemas, and RPC interfaces following best practices for consistency, usability, and scalability. Trigger keywords: api design, REST, RESTful, REST API, GraphQL, GraphQL schema, gRPC, OpenAPI, Swagger, endpoint, route, resource, CRUD, HTTP method, GET, POST, PUT, PATCH, DELETE, status code, pagination, filtering, sorting, versioning, HATEOAS, API versioning, schema design, RPC, service design.
allowed-tools: Read, Grep, Glob, Edit, Write
---

# API Design

## Overview

This skill guides the design of well-structured APIs that are intuitive, consistent, and scalable. It covers REST, GraphQL, and gRPC patterns with focus on developer experience and long-term maintainability.

## Instructions

### 1. Understand Requirements

- Identify API consumers and use cases
- Define resource models and relationships
- Determine authentication/authorization needs
- Plan for versioning strategy

### 2. Design Resource Structure

- Define clear resource hierarchies
- Establish naming conventions
- Plan URL patterns
- Design request/response schemas

### 3. Define Operations

- Map CRUD operations to HTTP methods
- Design query parameters for filtering/sorting
- Plan pagination approach
- Define error response formats

### 4. Document the API

- Create OpenAPI/Swagger specification
- Include examples for all endpoints
- Document authentication flows
- Provide SDK usage examples

## Best Practices

### RESTful API Design

1. **Use Nouns for Resources**: `/users`, not `/getUsers`
2. **Consistent Naming**: Use snake_case or camelCase consistently across all endpoints
3. **Proper HTTP Methods**:
   - GET (read/fetch), POST (create), PUT (full update), PATCH (partial update), DELETE (remove)
4. **Meaningful Status Codes**:
   - 200 (OK), 201 (Created), 204 (No Content)
   - 400 (Bad Request), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found)
   - 409 (Conflict), 422 (Unprocessable Entity)
   - 500 (Internal Server Error), 503 (Service Unavailable)
5. **Idempotency**: PUT and DELETE should be idempotent; consider idempotency keys for POST
6. **HATEOAS**: Include links to related resources in responses
7. **Filtering & Sorting**: Use query parameters (`?status=active&sort=-created_at`)
8. **Pagination**: Cursor-based for large datasets, offset for simpler cases

### GraphQL Schema Design

1. **Use Descriptive Types**: Clear, self-documenting type and field names
2. **Connection Pattern**: Use relay-style connections for paginated lists
3. **Input Objects**: Separate input types for mutations (CreateUserInput, UpdateUserInput)
4. **Error Handling**: Return errors as part of payload, not just top-level exceptions
5. **Nullability**: Be explicit about nullable vs non-nullable fields
6. **Mutations Return Payloads**: Include both data and errors in mutation responses
7. **Avoid Over-fetching**: Design fragments and fields for common query patterns

### gRPC Service Design

1. **Service Naming**: Use verbs for RPC methods (CreateUser, GetUser, ListUsers)
2. **Message Reuse**: Define shared message types for common patterns
3. **Pagination**: Use page_token pattern for cursor-based pagination
4. **Errors**: Use google.rpc.Status for rich error details
5. **Versioning**: Use package versioning (myapi.v1, myapi.v2)
6. **Streaming**: Leverage server/client/bidirectional streaming where appropriate

### API Versioning Strategies

1. **URL Path Versioning**: `/v1/users`, `/v2/users` (explicit, simple, cacheable)
2. **Header Versioning**: `API-Version: 2` or `Accept: application/vnd.api.v2+json` (cleaner URLs)
3. **Query Parameter**: `?version=2` (fallback option, less recommended)
4. **Semantic Versioning**: Major versions for breaking changes, minor for features, patch for fixes
5. **Deprecation Policy**: Announce sunset dates, support N-1 versions minimum

## Examples

### Example 1: RESTful Resource Design

```yaml
# OpenAPI 3.0 Specification
openapi: 3.0.0
info:
  title: E-Commerce API
  version: 1.0.0

paths:
  /products:
    get:
      summary: List all products
      parameters:
        - name: category
          in: query
          schema:
            type: string
        - name: min_price
          in: query
          schema:
            type: number
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: per_page
          in: query
          schema:
            type: integer
            default: 20
            maximum: 100
      responses:
        "200":
          description: Paginated list of products
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/Product"
                  pagination:
                    $ref: "#/components/schemas/Pagination"

    post:
      summary: Create a new product
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/ProductInput"
      responses:
        "201":
          description: Product created
        "400":
          description: Validation error

  /products/{id}:
    get:
      summary: Get a specific product
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        "200":
          description: Product details
        "404":
          description: Product not found
```

### Example 2: GraphQL Schema Design

```graphql
type Query {
  """
  Fetch a user by ID
  """
  user(id: ID!): User

  """
  List users with optional filtering
  """
  users(filter: UserFilter, first: Int = 20, after: String): UserConnection!
}

type Mutation {
  """
  Create a new user account
  """
  createUser(input: CreateUserInput!): CreateUserPayload!

  """
  Update an existing user
  """
  updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!
}

type User {
  id: ID!
  email: String!
  name: String!
  createdAt: DateTime!
  orders(first: Int, after: String): OrderConnection!
}

input CreateUserInput {
  email: String!
  name: String!
  password: String!
}

type CreateUserPayload {
  user: User
  errors: [UserError!]!
}

type UserError {
  field: String
  message: String!
  code: UserErrorCode!
}
```

### Example 3: gRPC Service Definition

```protobuf
syntax = "proto3";

package ecommerce.v1;

import "google/protobuf/timestamp.proto";

service ProductService {
  // Fetch a single product by ID
  rpc GetProduct(GetProductRequest) returns (GetProductResponse);

  // List products with filtering and pagination
  rpc ListProducts(ListProductsRequest) returns (ListProductsResponse);

  // Create a new product
  rpc CreateProduct(CreateProductRequest) returns (CreateProductResponse);

  // Update an existing product
  rpc UpdateProduct(UpdateProductRequest) returns (UpdateProductResponse);

  // Delete a product
  rpc DeleteProduct(DeleteProductRequest) returns (DeleteProductResponse);
}

message Product {
  string id = 1;
  string name = 2;
  string description = 3;
  int64 price_cents = 4;
  string category = 5;
  google.protobuf.Timestamp created_at = 6;
  google.protobuf.Timestamp updated_at = 7;
}

message ListProductsRequest {
  // Optional category filter
  string category = 1;

  // Minimum price filter (in cents)
  optional int64 min_price_cents = 2;

  // Maximum number of items to return
  int32 page_size = 3;

  // Page token from previous response
  string page_token = 4;
}

message ListProductsResponse {
  repeated Product products = 1;
  string next_page_token = 2;
  int32 total_count = 3;
}

message CreateProductRequest {
  string name = 1;
  string description = 2;
  int64 price_cents = 3;
  string category = 4;
}

message CreateProductResponse {
  Product product = 1;
}
```

### Example 4: Error Response Design (REST)

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "Email must be a valid email address"
      },
      {
        "field": "age",
        "code": "OUT_OF_RANGE",
        "message": "Age must be between 18 and 120"
      }
    ],
    "request_id": "req_abc123xyz",
    "documentation_url": "https://api.example.com/docs/errors#VALIDATION_ERROR"
  }
}
```

### Example 5: API Versioning with Header

```http
# Request with API version header
GET /users/123 HTTP/1.1
Host: api.example.com
Accept: application/vnd.api.v2+json
Authorization: Bearer eyJ...

# Response includes deprecation warning
HTTP/1.1 200 OK
Content-Type: application/vnd.api.v2+json
Sunset: Sat, 31 Dec 2026 23:59:59 GMT
Deprecation: true
Link: <https://api.example.com/v3/users/123>; rel="successor-version"

{
  "id": "123",
  "email": "user@example.com",
  "name": "John Doe"
}
```
