---
name: graphql-api-design
description: GraphQL schema design, type systems, resolver patterns, DataLoader optimization, pagination, subscriptions, and query complexity management. Use when building GraphQL APIs, designing schemas, migrating from REST, or optimizing query performance.
version: 1.0.0
---

# GraphQL API Design

Master GraphQL schema design and implementation to build flexible, efficient APIs that clients love.

## When to Use This Skill

- Designing GraphQL schemas and type systems
- Building GraphQL resolvers and queries
- Implementing mutations with proper error handling
- Optimizing query performance with DataLoaders
- Implementing real-time features with subscriptions
- Preventing N+1 queries and query complexity attacks
- Migrating from REST APIs to GraphQL
- Setting up pagination and filtering strategies

## GraphQL Design Fundamentals

### Schema-First Development

Design your GraphQL schema BEFORE writing resolvers:

1. **Define Types**: Represent your domain model
2. **Define Queries**: Read operations for fetching data
3. **Define Mutations**: Write operations for modifying data
4. **Define Subscriptions**: Real-time updates
5. **Implement Resolvers**: Connect schema to data sources

**Benefits:**
- Clear contract between client and server
- Introspection documentation for free
- Type safety across the entire stack
- Schema can be evolved gradually

### Core Types

**Basic scalar types:**
```graphql
String      # Text
Int         # 32-bit integer
Float       # Floating point
Boolean     # True/False
ID          # Unique identifier

# Custom scalars
scalar DateTime
scalar Email
scalar URL
scalar JSON
scalar Money
```

**Type definitions:**
```graphql
# Object type
type User {
  id: ID!                    # Non-null ID
  email: String!             # Required string
  name: String!
  phone: String              # Optional string
  posts: [Post!]!            # Non-null array of non-null posts
  tags: [String!]            # Nullable array of non-null strings
  createdAt: DateTime!
}

# Enum for fixed set of values
enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

# Interface for shared fields
interface Node {
  id: ID!
  createdAt: DateTime!
}

# Implementation of interface
type Post implements Node {
  id: ID!
  createdAt: DateTime!
  title: String!
  content: String!
  status: PostStatus!
}

# Union for multiple return types
union SearchResult = User | Post | Comment

# Input type for mutations
input CreateUserInput {
  email: String!
  name: String!
  password: String!
  profileInput: ProfileInput
}

input ProfileInput {
  bio: String
  avatar: URL
}
```

## Schema Organization

### Modular Schema Structure

Organize schema across multiple files:

```graphql
# user.graphql
type User {
  id: ID!
  email: String!
  name: String!
  posts(first: Int, after: String): PostConnection!
}

extend type Query {
  user(id: ID!): User
  users(first: Int, after: String): UserConnection!
}

extend type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
}

# post.graphql
type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  status: PostStatus!
}

extend type Query {
  post(id: ID!): Post
  posts(first: Int, after: String): PostConnection!
}

extend type Mutation {
  createPost(input: CreatePostInput!): CreatePostPayload!
}
```

## Queries and Filtering

### Root Query Structure

```graphql
type Query {
  # Single resource
  user(id: ID!): User
  post(id: ID!): Post

  # Collections
  users(
    first: Int = 20
    after: String
    filter: UserFilter
    sort: UserSort
  ): UserConnection!

  # Search
  search(query: String!): [SearchResult!]!
}

input UserFilter {
  status: UserStatus
  email: String
  createdAfter: DateTime
}

input UserSort {
  field: UserSortField = CREATED_AT
  direction: SortDirection = DESC
}

enum UserSortField {
  CREATED_AT
  UPDATED_AT
  NAME
}

enum SortDirection {
  ASC
  DESC
}
```

## Pagination Patterns

### 1. Relay Cursor Pagination (Recommended)

Best for: Infinite scroll, real-time data, consistent results

```graphql
type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type UserEdge {
  node: User!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

type Query {
  users(first: Int, after: String, last: Int, before: String): UserConnection!
}

# Usage
{
  users(first: 10, after: "cursor123") {
    edges {
      cursor
      node {
        id
        name
      }
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

### 2. Offset Pagination (Simpler)

Best for: Traditional pagination UI

```graphql
type UserList {
  items: [User!]!
  total: Int!
  page: Int!
  pageSize: Int!
  pages: Int!
}

type Query {
  users(page: Int = 1, pageSize: Int = 20): UserList!
}
```

## Mutations and Error Handling

### Input/Payload Pattern

Always use input types and return structured payloads:

```graphql
input CreatePostInput {
  title: String!
  content: String!
  tags: [String!]
}

type CreatePostPayload {
  post: Post
  errors: [Error!]
  success: Boolean!
}

type Error {
  field: String
  message: String!
  code: ErrorCode!
}

enum ErrorCode {
  VALIDATION_ERROR
  UNAUTHORIZED
  NOT_FOUND
  INTERNAL_ERROR
}

type Mutation {
  createPost(input: CreatePostInput!): CreatePostPayload!
}
```

**Implementation (Python/Ariadne):**

```python
@mutation.field("createPost")
async def resolve_create_post(obj, info, input: dict) -> dict:
    try:
        # Validate input
        if not input.get("title"):
            return {
                "post": None,
                "errors": [{"field": "title", "message": "Title required"}],
                "success": False
            }

        # Create post
        post = await create_post(
            title=input["title"],
            content=input["content"],
            tags=input.get("tags", [])
        )

        return {
            "post": post,
            "errors": [],
            "success": True
        }
    except Exception as e:
        return {
            "post": None,
            "errors": [{"message": str(e), "code": "INTERNAL_ERROR"}],
            "success": False
        }
```

### Batch Mutations

```graphql
input BatchCreateUserInput {
  users: [CreateUserInput!]!
}

type BatchCreateUserPayload {
  results: [CreateUserResult!]!
  successCount: Int!
  errorCount: Int!
}

type CreateUserResult {
  user: User
  errors: [Error!]
  index: Int!
}

type Mutation {
  batchCreateUsers(input: BatchCreateUserInput!): BatchCreateUserPayload!
}
```

## Resolver Implementation

### Basic Resolvers

```python
from ariadne import QueryType, ObjectType, MutationType

query = QueryType()
user_type = ObjectType("User")
mutation = MutationType()

@query.field("user")
async def resolve_user(obj, info, id: str) -> dict:
    """Resolve single user by ID."""
    return await fetch_user_by_id(id)

@query.field("users")
async def resolve_users(obj, info, first: int = 20, after: str = None) -> dict:
    """Resolve paginated user list."""
    offset = decode_cursor(after) if after else 0
    users = await fetch_users(limit=first + 1, offset=offset)
    has_next = len(users) > first
    if has_next:
        users = users[:first]

    edges = [
        {"node": user, "cursor": encode_cursor(offset + i)}
        for i, user in enumerate(users)
    ]

    return {
        "edges": edges,
        "pageInfo": {
            "hasNextPage": has_next,
            "hasPreviousPage": offset > 0,
            "startCursor": edges[0]["cursor"] if edges else None,
            "endCursor": edges[-1]["cursor"] if edges else None
        }
    }

@user_type.field("posts")
async def resolve_user_posts(user: dict, info, first: int = 20) -> dict:
    """Resolve user's posts (with DataLoader to prevent N+1)."""
    loader = info.context["loaders"]["posts_by_user"]
    return await loader.load(user["id"])
```

## N+1 Query Prevention with DataLoaders

The **N+1 problem**: Fetching related data one-by-one instead of batching

**Problem example:**
```python
# This creates N+1 queries!
for user in users:
    user.posts = await fetch_posts_for_user(user.id)  # N queries!
```

**Solution with DataLoader:**

```python
from aiodataloader import DataLoader

class PostsByUserLoader(DataLoader):
    """Batch load posts for multiple users."""

    async def batch_load_fn(self, user_ids: list) -> list:
        """Load posts for multiple users in ONE query."""
        posts = await fetch_posts_by_user_ids(user_ids)

        # Group posts by user_id
        posts_by_user = {}
        for post in posts:
            user_id = post["user_id"]
            if user_id not in posts_by_user:
                posts_by_user[user_id] = []
            posts_by_user[user_id].append(post)

        # Return in input order
        return [posts_by_user.get(uid, []) for uid in user_ids]

# Setup context with loaders
def create_context():
    return {
        "loaders": {
            "posts_by_user": PostsByUserLoader()
        }
    }

# Use in resolver
@user_type.field("posts")
async def resolve_user_posts(user: dict, info) -> list:
    loader = info.context["loaders"]["posts_by_user"]
    return await loader.load(user["id"])
```

## Query Complexity and Security

### Depth Limiting

Prevent excessively nested queries:

```python
def depth_limit_validator(max_depth: int):
    def validate_depth(context, node, ancestors):
        depth = len(ancestors)
        if depth > max_depth:
            raise GraphQLError(
                f"Query depth {depth} exceeds max {max_depth}"
            )
    return validate_depth

# Usage in schema validation
from graphql import validate

depth_validator = depth_limit_validator(10)
errors = validate(schema, parsed_query, [depth_validator])
```

### Query Complexity Analysis

Limit query complexity to prevent expensive operations:

```python
def calculate_complexity(field_nodes, type_info, complexity_args):
    """Calculate complexity score for a query."""
    complexity = 1

    if type_info.type and isinstance(type_info.type, GraphQLList):
        # List fields multiply complexity
        list_size = complexity_args.get("first", 10)
        complexity *= list_size

    return complexity

# Usage
from graphql import validate

complexity_validator = QueryComplexityValidator(max_complexity=1000)
errors = validate(schema, parsed_query, [complexity_validator])
```

## Subscriptions for Real-Time Updates

```graphql
type Subscription {
  postAdded: Post!
  postUpdated(postId: ID!): Post!
  userStatusChanged(userId: ID!): UserStatus!
}

type UserStatus {
  userId: ID!
  online: Boolean!
  lastSeen: DateTime!
}

# Client usage
subscription {
  postAdded {
    id
    title
    author { name }
  }
}
```

**Implementation:**

```python
subscription = SubscriptionType()

@subscription.source("postAdded")
async def post_added_generator(obj, info):
    """Subscribe to new posts."""
    async for post in info.context["pubsub"].subscribe("posts"):
        yield post

@subscription.field("postAdded")
def post_added_resolver(post, info):
    return post
```

## Custom Scalars

```graphql
scalar DateTime
scalar Email
scalar URL
scalar JSON
scalar Money

type User {
  email: Email!
  website: URL
  createdAt: DateTime!
  metadata: JSON
}

type Product {
  price: Money!
}
```

## Directives

### Built-in Directives

```graphql
type User {
  name: String!
  email: String! @deprecated(reason: "Use emails field")
  emails: [String!]!

  privateData: String @include(if: $isOwner)
}

query GetUser($isOwner: Boolean!) {
  user(id: "123") {
    name
    privateData @include(if: $isOwner)
  }
}
```

### Custom Directives

```graphql
directive @auth(requires: Role = USER) on FIELD_DEFINITION

enum Role {
  USER
  ADMIN
  MODERATOR
}

type Mutation {
  deleteUser(id: ID!): Boolean! @auth(requires: ADMIN)
}
```

## Schema Versioning

### Field Deprecation

```graphql
type User {
  name: String! @deprecated(reason: "Use firstName and lastName")
  firstName: String!
  lastName: String!
}
```

### Schema Evolution (Backward Compatible)

```graphql
# v1
type User {
  name: String!
}

# v2 - Add optional field
type User {
  name: String!
  email: String
}

# v3 - Deprecate old field
type User {
  name: String! @deprecated(reason: "Use firstName/lastName")
  firstName: String!
  lastName: String!
  email: String
}
```

## Best Practices Summary

1. **Nullable by Default**: Make fields nullable initially, mark as non-null when guaranteed
2. **Input Types**: Always use input types for mutations (never raw arguments)
3. **Payload Pattern**: Return errors within mutation payloads
4. **Cursor Pagination**: Use for infinite scroll, offset for simple cases
5. **DataLoaders**: Prevent N+1 queries with batch loading
6. **Naming**: camelCase for fields, PascalCase for types
7. **Deprecation**: Use `@deprecated` for backward compatibility
8. **Query Limits**: Enforce depth and complexity limits
9. **Custom Scalars**: Model domain types (Email, DateTime)
10. **Documentation**: Document schema fields with descriptions

## Common Pitfalls to Avoid

- Using nullable for fields that should always exist
- Forgetting to batch load related data (N+1 queries)
- Over-nesting schemas (design flat hierarchies)
- Not limiting query complexity (vulnerable to attacks)
- Removing fields instead of deprecating
- Tight coupling between schema and database schema
- Missing error handling in mutations
- Not implementing pagination

## Cross-Skill References

- **rest-api-design skill** - For REST comparison
- **api-architecture skill** - For security, versioning, monitoring
- **api-testing skill** - For testing GraphQL queries and mutations

## Additional Resources

For detailed guidance, see:
- `references/schema-patterns.md` - Complete schema design patterns
- `references/resolver-patterns.md` - Resolver and DataLoader patterns
- `references/subscriptions-guide.md` - Real-time subscription patterns
