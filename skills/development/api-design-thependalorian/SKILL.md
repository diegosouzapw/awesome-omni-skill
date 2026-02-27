---
name: api-design
description: This skill should be used when designing APIs, choosing between REST, GraphQL, or gRPC, implementing API protocols, or ensuring API security. It provides guidance on RESTful APIs, GraphQL, gRPC, HTTP/WebSockets, and API best practices.
---

# API Design

This skill provides comprehensive guidance on designing APIs, from RESTful conventions to GraphQL and gRPC, including protocol selection and security best practices.

## When to Use This Skill

Use this skill when:
- Designing new APIs
- Choosing between REST, GraphQL, or gRPC
- Implementing API endpoints
- Ensuring API security
- Optimizing API performance
- Documenting APIs

## API Fundamentals

### What is an API?
**Application Programming Interface** - defines how software components interact

```
┌──────────┐                      ┌──────────┐
│  Client  │ ◄── API Contract ──► │  Server  │
│(Browser/ │    - Requests        │          │
│ Mobile)  │    - Responses       │          │
└──────────┘                      └──────────┘
```

### Four Essential Design Principles

| Principle | Description |
|-----------|-------------|
| **Consistency** | Same naming, casing, patterns throughout |
| **Simplicity** | Focus on core use cases, intuitive design |
| **Security** | Authentication, authorization, rate limiting, validation |
| **Performance** | Caching, pagination, minimal payloads, reduce round trips |

## RESTful APIs

### Core Concepts
- **Resource-based** approach using HTTP methods
- **Stateless** - each request contains all needed information
- Uses standard HTTP methods

### HTTP Methods (CRUD Operations)

| Method | Operation | Example | Idempotent |
|--------|-----------|---------|------------|
| GET | Read | `GET /products/123` | ✅ Yes |
| POST | Create | `POST /products` | ❌ No |
| PUT | Full Update | `PUT /products/123` | ✅ Yes |
| PATCH | Partial Update | `PATCH /products/123` | ✅ Yes |
| DELETE | Remove | `DELETE /products/123` | ✅ Yes |

### Status Codes

| Range | Category | Examples |
|-------|----------|----------|
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirection | 301 Moved Permanently, 304 Not Modified |
| 4xx | Client Error | 400 Bad Request, 401 Unauthorized, 404 Not Found |
| 5xx | Server Error | 500 Internal Server Error, 503 Service Unavailable |

### REST Best Practices

**1. Use Plural Nouns (not verbs):**
```
✅ GET /products
✅ GET /products/123
❌ GET /getProducts
❌ POST /createProduct
```

**2. Filtering, Sorting, Pagination:**
```
GET /products?category=electronics&in_stock=true    # Filtering
GET /products?sort=price_asc                          # Sorting
GET /products?page=3&limit=10                         # Pagination
GET /products?offset=20&limit=10                     # Alternative pagination
```

**3. API Versioning:**
```
GET /api/v1/products
GET /api/v2/products
```

**4. Nested Resources:**
```
GET /products/123/reviews        # Reviews for product 123
GET /users/456/orders            # Orders for user 456
```

## GraphQL APIs

### Why GraphQL Exists
Created by Facebook to solve:
- Multiple API calls for single view
- Over-fetching data
- Under-fetching data

### REST vs GraphQL

**REST (Multiple calls):**
```
GET /users/123
GET /users/123/posts
GET /users/123/followers
```

**GraphQL (Single call):**
```graphql
query {
  user(id: "123") {
    name
    posts {
      title
      content
    }
    followers {
      name
    }
  }
}
```

### Schema Definition

```graphql
type User {
  id: ID!
  name: String!
  email: String
  posts: [Post]
}

type Post {
  id: ID!
  title: String!
  content: String
  author: User
}

type Query {
  user(id: ID!): User
  posts: [Post]
}

type Mutation {
  createUser(name: String!): User
  createPost(title: String!, content: String!): Post
}
```

### Operations

| Operation | Purpose | REST Equivalent |
|-----------|---------|-----------------|
| Query | Read data | GET |
| Mutation | Modify data | POST, PUT, PATCH, DELETE |
| Subscription | Real-time updates | WebSockets |

### Error Handling
GraphQL always returns **200 OK** - errors in response body:
```json
{
  "data": { "user": null },
  "errors": [{
    "message": "User not found",
    "path": ["user"],
    "extensions": { "code": "NOT_FOUND" }
  }]
}
```

## gRPC

### Overview
- High-performance RPC framework by Google
- Uses **Protocol Buffers** for serialization
- Runs on **HTTP/2**

### Best For
- Microservices communication
- Internal system-to-system calls
- When performance is critical

### Comparison

| Feature | REST | GraphQL | gRPC |
|---------|------|---------|------|
| Protocol | HTTP/1.1 | HTTP | HTTP/2 |
| Format | JSON | JSON | Protocol Buffers |
| Streaming | ❌ | Subscription | ✅ Bidirectional |
| Browser Support | ✅ Full | ✅ Full | ⚠️ Limited |
| Best For | Public APIs | Complex UIs | Microservices |

## API Protocols

### HTTP/HTTPS

**Request Structure:**
```
GET /api/products/123 HTTP/1.1
Host: api.example.com
Authorization: Bearer <token>
Content-Type: application/json
```

**Response Structure:**
```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=3600

{"id": 123, "name": "Product"}
```

> ⚠️ **Always use HTTPS** - encrypts data in transit with TLS/SSL

### WebSockets

**Problem with HTTP for real-time:**
```
Client: "Any messages?" → Server: "No"
Client: "Any messages?" → Server: "No"
Client: "Any messages?" → Server: "Yes, here's one"
(Wasteful polling!)
```

**WebSocket Solution:**
```
Client ←──── Bidirectional ────→ Server
       │                         │
       │  Server can push data   │
       │  without client asking  │
```

**Use Cases:**
- Chat applications
- Live notifications
- Real-time gaming
- Stock tickers

### AMQP (Advanced Message Queuing Protocol)

```
┌──────────┐     ┌─────────────┐     ┌──────────┐
│ Producer │ ──► │   Message   │ ──► │ Consumer │
│ (Payment │     │    Queue    │     │(Process  │
│  System) │     │   (Broker)  │     │ Orders)  │
└──────────┘     └─────────────┘     └──────────┘
```

**Benefits:**
- Decouples producers and consumers
- Handles traffic spikes (queue buffers)
- Guaranteed delivery

## TCP vs UDP

### Transport Layer Protocols

```
Application Layer (HTTP, WebSocket, gRPC)
              │
              ▼
Transport Layer (TCP or UDP)  ◄── We're here
              │
              ▼
Network Layer (IP)
```

### TCP (Transmission Control Protocol)

**Like sending a package with tracking & signature:**
- ✅ Guaranteed delivery
- ✅ Ordered packets
- ✅ Error checking
- ❌ Slower (overhead)

**Three-Way Handshake:**
```
Client ──── SYN ────► Server
Client ◄── SYN-ACK ── Server
Client ──── ACK ────► Server
(Connection established!)
```

### UDP (User Datagram Protocol)

**Like sending postcards:**
- ✅ Fast
- ✅ Low overhead
- ❌ No delivery guarantee
- ❌ No ordering

### When to Use

| TCP | UDP |
|-----|-----|
| Banking, payments | Video streaming |
| Email | Online gaming |
| File transfers | Voice calls |
| APIs | Live broadcasts |

## API Design Best Practices

### Performance
- Implement caching strategies
- Use pagination for large datasets
- Minimize payload sizes
- Reduce round trips (batch requests when possible)

### Security
- Use HTTPS always
- Implement authentication (JWT, OAuth 2.0)
- Use authorization (RBAC, ABAC)
- Rate limiting
- Input validation
- CORS configuration

### Documentation
- Document request/response schemas
- Include error response formats
- Provide example requests
- Use OpenAPI/Swagger for REST APIs
- Use GraphQL schema introspection

### Versioning
- Version your APIs (`/api/v1/`, `/api/v2/`)
- Maintain backward compatibility
- Deprecate old versions gracefully
- Communicate breaking changes

## Reference Material

For detailed examples and explanations, refer to:
- `references/SYSTEM_DESIGN_MASTER_GUIDE.md` - Part 3: API Design section
