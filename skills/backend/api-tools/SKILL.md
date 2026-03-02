---
name: api-tools
description: API testing, documentation, and development tools
domain: tools-integrations
version: 1.0.0
tags: [postman, insomnia, swagger, openapi, curl, httpie]
triggers:
  keywords:
    primary: [postman, insomnia, curl, httpie, swagger, openapi]
    secondary: [api test, request, response, collection, mock server, bruno]
  context_boost: [api, endpoint, http, rest]
  context_penalty: [frontend, ui, design]
  priority: medium
---

# API Tools

## Overview

Tools and techniques for API development, testing, documentation, and debugging.

---

## HTTP Clients

### cURL

```bash
# Basic GET
curl https://api.example.com/users

# GET with headers
curl -H "Authorization: Bearer TOKEN" \
     -H "Accept: application/json" \
     https://api.example.com/users

# POST with JSON body
curl -X POST \
     -H "Content-Type: application/json" \
     -d '{"name": "John", "email": "john@example.com"}' \
     https://api.example.com/users

# POST with form data
curl -X POST \
     -F "file=@document.pdf" \
     -F "name=My Document" \
     https://api.example.com/upload

# PUT request
curl -X PUT \
     -H "Content-Type: application/json" \
     -d '{"name": "Updated Name"}' \
     https://api.example.com/users/123

# DELETE request
curl -X DELETE https://api.example.com/users/123

# Show response headers
curl -i https://api.example.com/users

# Verbose output (debugging)
curl -v https://api.example.com/users

# Follow redirects
curl -L https://example.com/redirect

# Save response to file
curl -o response.json https://api.example.com/users

# With authentication
curl -u username:password https://api.example.com/protected
curl --oauth2-bearer TOKEN https://api.example.com/protected

# Measure timing
curl -w "@curl-format.txt" -o /dev/null -s https://api.example.com
```

```txt
# curl-format.txt
     time_namelookup:  %{time_namelookup}s\n
        time_connect:  %{time_connect}s\n
     time_appconnect:  %{time_appconnect}s\n
    time_pretransfer:  %{time_pretransfer}s\n
       time_redirect:  %{time_redirect}s\n
  time_starttransfer:  %{time_starttransfer}s\n
                     ----------\n
          time_total:  %{time_total}s\n
```

### HTTPie

```bash
# GET request (more readable output)
http https://api.example.com/users

# With headers
http https://api.example.com/users \
  Authorization:"Bearer TOKEN" \
  Accept:application/json

# POST with JSON (default)
http POST https://api.example.com/users \
  name=John \
  email=john@example.com

# POST with raw JSON
http POST https://api.example.com/users \
  < user.json

# Form data
http -f POST https://api.example.com/login \
  username=john \
  password=secret

# File upload
http -f POST https://api.example.com/upload \
  file@document.pdf

# Download file
http --download https://api.example.com/files/document.pdf

# Session persistence
http --session=user-session POST https://api.example.com/login
http --session=user-session GET https://api.example.com/profile

# Only headers
http --headers https://api.example.com/users

# Only body
http --body https://api.example.com/users
```

---

## API Testing

### Postman Collections

```json
{
  "info": {
    "name": "User API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "variable": [
    {
      "key": "baseUrl",
      "value": "https://api.example.com"
    },
    {
      "key": "token",
      "value": ""
    }
  ],
  "item": [
    {
      "name": "Auth",
      "item": [
        {
          "name": "Login",
          "request": {
            "method": "POST",
            "url": "{{baseUrl}}/auth/login",
            "header": [
              { "key": "Content-Type", "value": "application/json" }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\"email\": \"{{email}}\", \"password\": \"{{password}}\"}"
            }
          },
          "event": [
            {
              "listen": "test",
              "script": {
                "exec": [
                  "pm.test('Status code is 200', function () {",
                  "    pm.response.to.have.status(200);",
                  "});",
                  "",
                  "pm.test('Response has token', function () {",
                  "    var jsonData = pm.response.json();",
                  "    pm.expect(jsonData.token).to.be.a('string');",
                  "    pm.collectionVariables.set('token', jsonData.token);",
                  "});"
                ]
              }
            }
          ]
        }
      ]
    },
    {
      "name": "Users",
      "item": [
        {
          "name": "List Users",
          "request": {
            "method": "GET",
            "url": {
              "raw": "{{baseUrl}}/users?page=1&limit=10",
              "query": [
                { "key": "page", "value": "1" },
                { "key": "limit", "value": "10" }
              ]
            },
            "header": [
              { "key": "Authorization", "value": "Bearer {{token}}" }
            ]
          },
          "event": [
            {
              "listen": "test",
              "script": {
                "exec": [
                  "pm.test('Status code is 200', function () {",
                  "    pm.response.to.have.status(200);",
                  "});",
                  "",
                  "pm.test('Response is an array', function () {",
                  "    var jsonData = pm.response.json();",
                  "    pm.expect(jsonData.data).to.be.an('array');",
                  "});",
                  "",
                  "pm.test('Response time is less than 500ms', function () {",
                  "    pm.expect(pm.response.responseTime).to.be.below(500);",
                  "});"
                ]
              }
            }
          ]
        }
      ]
    }
  ]
}
```

### Newman CLI

```bash
# Run Postman collection
newman run collection.json

# With environment
newman run collection.json -e environment.json

# Generate HTML report
newman run collection.json \
  --reporters cli,html \
  --reporter-html-export report.html

# Run specific folder
newman run collection.json --folder "Users"

# With iteration data
newman run collection.json -d data.json -n 10

# Set environment variables
newman run collection.json \
  --env-var "baseUrl=https://staging.api.example.com" \
  --env-var "token=xxx"
```

---

## OpenAPI / Swagger

### OpenAPI Specification

```yaml
# openapi.yaml
openapi: 3.0.3
info:
  title: User API
  description: API for managing users
  version: 1.0.0
  contact:
    email: api@example.com
  license:
    name: MIT

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: https://staging-api.example.com/v1
    description: Staging

tags:
  - name: Users
    description: User management operations
  - name: Auth
    description: Authentication operations

paths:
  /users:
    get:
      summary: List users
      description: Returns a paginated list of users
      operationId: listUsers
      tags:
        - Users
      security:
        - bearerAuth: []
      parameters:
        - name: page
          in: query
          description: Page number
          schema:
            type: integer
            minimum: 1
            default: 1
        - name: limit
          in: query
          description: Items per page
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
        - name: search
          in: query
          description: Search term
          schema:
            type: string
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'
        '401':
          $ref: '#/components/responses/Unauthorized'

    post:
      summary: Create user
      operationId: createUser
      tags:
        - Users
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUser'
      responses:
        '201':
          description: User created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'

  /users/{id}:
    get:
      summary: Get user by ID
      operationId: getUserById
      tags:
        - Users
      security:
        - bearerAuth: []
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'

components:
  schemas:
    User:
      type: object
      required:
        - id
        - email
        - createdAt
      properties:
        id:
          type: string
          format: uuid
          example: "123e4567-e89b-12d3-a456-426614174000"
        email:
          type: string
          format: email
          example: "user@example.com"
        name:
          type: string
          example: "John Doe"
        role:
          type: string
          enum: [user, admin]
          default: user
        createdAt:
          type: string
          format: date-time

    CreateUser:
      type: object
      required:
        - email
        - password
      properties:
        email:
          type: string
          format: email
        password:
          type: string
          minLength: 8
        name:
          type: string

    UserList:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/User'
        meta:
          $ref: '#/components/schemas/PaginationMeta'

    PaginationMeta:
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

  responses:
    BadRequest:
      description: Bad request
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

    Unauthorized:
      description: Unauthorized
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

### Generate Client SDK

```bash
# Generate TypeScript client
npx openapi-generator-cli generate \
  -i openapi.yaml \
  -g typescript-fetch \
  -o ./generated/client

# Generate Python client
openapi-generator generate \
  -i openapi.yaml \
  -g python \
  -o ./generated/python-client
```

---

## Mock Servers

### Prism (OpenAPI Mock Server)

```bash
# Start mock server
npx @stoplight/prism-cli mock openapi.yaml

# With dynamic responses
npx @stoplight/prism-cli mock openapi.yaml --dynamic

# Custom port
npx @stoplight/prism-cli mock openapi.yaml -p 4010
```

### JSON Server (Quick REST API)

```json
// db.json
{
  "users": [
    { "id": 1, "name": "John", "email": "john@example.com" },
    { "id": 2, "name": "Jane", "email": "jane@example.com" }
  ],
  "posts": [
    { "id": 1, "title": "Hello World", "userId": 1 }
  ]
}
```

```bash
# Start server
npx json-server --watch db.json --port 3001

# Routes automatically created:
# GET    /users
# GET    /users/1
# POST   /users
# PUT    /users/1
# DELETE /users/1
# GET    /posts?userId=1
```

---

## API Debugging

### Request/Response Logging

```typescript
import axios from 'axios';

// Request interceptor
axios.interceptors.request.use((config) => {
  console.log('Request:', {
    method: config.method?.toUpperCase(),
    url: config.url,
    params: config.params,
    data: config.data,
    headers: config.headers,
  });
  return config;
});

// Response interceptor
axios.interceptors.response.use(
  (response) => {
    console.log('Response:', {
      status: response.status,
      headers: response.headers,
      data: response.data,
    });
    return response;
  },
  (error) => {
    console.log('Error:', {
      status: error.response?.status,
      data: error.response?.data,
    });
    return Promise.reject(error);
  }
);
```

---

## Related Skills

- [[api-design]] - API design patterns
- [[testing-strategies]] - API testing
- [[backend]] - Backend development

