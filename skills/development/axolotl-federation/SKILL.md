---
name: axolotl-federation
description: Axolotl micro-federation architecture - config, schema merging, mergeAxolotls, cross-module dependencies, best practices, and troubleshooting
---

## Micro-Federation (GraphQL Federation Patterns)

Micro-federation is one of Axolotl's most powerful features, allowing you to compose multiple GraphQL modules into a unified API while maintaining type safety and code organization.

### What is Micro-Federation?

Micro-federation in Axolotl is a modular architecture pattern where:

- Each domain (e.g., users, posts, products) has its own GraphQL schema and resolvers
- Schemas are automatically merged into a single supergraph at build time
- Each module maintains its own type safety with generated models
- Resolvers are intelligently merged at runtime to handle overlapping types

**Key Difference from Apollo Federation:** Axolotl's micro-federation is designed for **monorepo or single-project architectures** where all modules are built and deployed together, not for distributed microservices.

> **Note:** The `auth` and `users` modules are **core modules**. The `todos` module is an **example module** included for demonstration — it can be safely removed. Throughout this skill, we use `posts` as a hypothetical domain to illustrate federation patterns.

### When to Use Micro-Federation

**Good use cases:**

- Large monorepo applications with distinct domain modules
- Teams working on separate features within the same codebase
- Projects where you want to organize GraphQL code by business domain
- Applications that need to scale code organization without microservices complexity

**Not recommended for:**

- Distributed services that deploy independently (use Apollo Federation instead)
- Simple applications with only a few types and resolvers
- Projects where all types are tightly coupled

---

### Federation Configuration

> **Configuration:** See `AGENTS.md` → **Understanding axolotl.json** for the complete configuration. Key federation fields:
>
> - `federation[]` — array of `{ schema, models }` objects, one per module
> - `schema` — path to module's `.graphql` schema
> - `models` — path to module's auto-generated `models.ts`

---

### Federation Directory Structure

> **Project Structure:** See `AGENTS.md` → **File Structure** section for the complete backend directory tree. Key points for federation:
>
> - Each module lives in `backend/src/modules/{name}/` with its own `schema.graphql`, `models.ts`, `axolotl.ts`, and `resolvers/`
> - The root `backend/src/resolvers.ts` merges all modules via `mergeAxolotls()`
> - `backend/axolotl.json` lists all federated modules

---

### Creating Submodule Schemas

Each module defines its own schema:

**src/modules/auth/schema.graphql:**

```graphql
type Query {
  user: AuthorizedUserQuery @resolver
}

type Mutation {
  user: AuthorizedUserMutation @resolver
}

directive @resolver on FIELD_DEFINITION

schema {
  query: Query
  mutation: Mutation
}
```

> **Note:** The auth module owns the `Query.user` and `Mutation.user` gateway resolvers. These perform authentication and return the user object that becomes `source` for all child resolvers. Domain modules (e.g., users) should NOT duplicate these gateway fields.

**src/modules/users/schema.graphql:**

```graphql
type User {
  _id: String!
  email: String!
}

type Mutation {
  login(email: String!, password: String!): String! @resolver
  register(email: String!, password: String!): String! @resolver
}

type AuthorizedUserQuery {
  me: User! @resolver
}

directive @resolver on FIELD_DEFINITION

schema {
  query: Query
  mutation: Mutation
}
```

**src/modules/posts/schema.graphql (example domain):**

```graphql
type Post {
  _id: String!
  title: String!
  content: String!
  published: Boolean!
}

type AuthorizedUserMutation {
  createPost(title: String!, content: String!): String! @resolver
  postOps(_id: String!): PostOps! @resolver
}

type AuthorizedUserQuery {
  posts: [Post!] @resolver
  post(_id: String!): Post! @resolver
}

type PostOps {
  publish: Boolean @resolver
  delete: Boolean @resolver
}

directive @resolver on FIELD_DEFINITION

schema {
  query: Query
  mutation: Mutation
}
```

---

### Module Axolotl Instances

Each module needs its own `axolotl.ts` file to create type-safe resolver helpers:

**src/modules/users/axolotl.ts:**

```typescript
import { Models } from '@/src/modules/users/models.js';
import { Axolotl } from '@aexol/axolotl-core';
import { graphqlYogaAdapter } from '@aexol/axolotl-graphql-yoga';

export const { createResolvers, createDirectives, applyMiddleware } = Axolotl(graphqlYogaAdapter)<Models, unknown>();
```

**src/modules/posts/axolotl.ts:**

```typescript
import { Models } from '@/src/modules/posts/models.js';
import { Axolotl } from '@aexol/axolotl-core';
import { graphqlYogaAdapter } from '@aexol/axolotl-graphql-yoga';

export const { createResolvers, createDirectives, applyMiddleware } = Axolotl(graphqlYogaAdapter)<Models, unknown>();
```

### Module Adapter Usage (Critical)

Modules **MUST** use `graphqlYogaAdapter` (the basic adapter without context). The context-aware adapter `graphqlYogaWithContextAdapter<T>(buildContext)` is used **ONLY** in the root `backend/src/axolotl.ts`.

Context is initialized once by the root adapter and automatically flows to all module resolvers — modules never need to set up context themselves.

**Correct usage:**

```typescript
// ✅ CORRECT — Module axolotl.ts uses basic adapter
// src/modules/posts/axolotl.ts
import { Models } from '@/src/modules/posts/models.js';
import { Axolotl } from '@aexol/axolotl-core';
import { graphqlYogaAdapter } from '@aexol/axolotl-graphql-yoga';

export const { createResolvers } = Axolotl(graphqlYogaAdapter)<Models, unknown>();

// ✅ CORRECT — Only root axolotl.ts uses context adapter
// src/axolotl.ts
import { graphqlYogaWithContextAdapter } from '@aexol/axolotl-graphql-yoga';
const yogaAdapter = graphqlYogaWithContextAdapter<AppContext>(buildContext);
export const { createResolvers, adapter } = Axolotl(yogaAdapter)<Models, Scalars>();
```

**Incorrect usage:**

```typescript
// ❌ WRONG — Module should NOT use context adapter
// src/modules/posts/axolotl.ts
import { graphqlYogaWithContextAdapter } from '@aexol/axolotl-graphql-yoga';
const yogaAdapter = graphqlYogaWithContextAdapter<AppContext>(buildContext);
export const { createResolvers } = Axolotl(yogaAdapter)<Models, unknown>();
// This is wrong because:
// 1. Context is already initialized by the root adapter
// 2. Module resolvers automatically receive the context from the root
// 3. Duplicating context setup is redundant and error-prone
```

> **Why this works:** Context flows through Axolotl like this: the root `adapter()` initializes the GraphQL Yoga server with the context function → every incoming request runs `buildContext()` once → the resulting context object is available as `input[2]` in ALL resolvers, regardless of which module defined them. Modules only define resolver logic — they don't need to know how context is constructed.

**Accessing context in module resolvers:**

- Module resolvers access context via `input[2]` (or destructured as `[, , context]`)
- The context type comes from the root `axolotl.ts` — modules get it automatically
- If modules need typed access to custom context fields, import the context type: `const ctx = input[2] as AppContext`

---

### Schema Merging Rules

When you run `cd backend && npx @aexol/axolotl build`, schemas are merged using these rules:

**1. Types are merged by name:**

- If `User` type exists in multiple schemas, all fields are combined
- Fields with the same name **must have identical type signatures**
- If there's a conflict, the build fails with a detailed error

**Example - Types get merged:**

```graphql
# modules/users/schema.graphql
type User {
  _id: String!
  email: String!
}

# modules/posts/schema.graphql
type User {
  _id: String!
}

# Merged result in schema.graphql
type User {
  _id: String!
  email: String! # Field from users module
}
```

**2. Root types (Query, Mutation, Subscription) are automatically merged:**

```graphql
# modules/auth/schema.graphql
type Query {
  user: AuthorizedUserQuery @resolver
}

# modules/posts/schema.graphql
type Query {
  # No Query.user here — owned by auth module
}

# Merged result - fields combined
type Query {
  user: AuthorizedUserQuery @resolver
}
```

---

### Resolver Merging with mergeAxolotls

The `mergeAxolotls` function intelligently merges resolvers:

**1. Non-overlapping resolvers are combined:**

```typescript
// users: { Mutation: { login: fn1 } }
// posts: { Mutation: { createPost: fn2 } }
// Result: { Mutation: { login: fn1, createPost: fn2 } }
```

**2. Overlapping resolvers are executed in parallel and results are deep-merged:**

```typescript
// users: { Query: { user: () => ({ email: "john@example.com" }) } }
// posts: { Query: { user: () => ({ posts: [...] }) } }
// Result: { Query: { user: () => ({ email: "john@example.com", posts: [...] }) } }
```

This allows multiple modules to contribute different fields to the same resolver!

**3. Subscriptions cannot be merged - only the first one is used:**

```typescript
// If multiple modules define the same subscription, only the first is used
// This is because subscriptions have a single event stream
```

**Module resolvers example:**

```typescript
// src/modules/users/resolvers/resolvers.ts
import { createResolvers } from '../axolotl.js';
import Mutation from './Mutation/resolvers.js';
import AuthorizedUserQuery from './AuthorizedUserQuery/resolvers.js';

export default createResolvers({
  ...Mutation,
  ...AuthorizedUserQuery,
});
```

```typescript
// src/modules/users/resolvers/Mutation/login.ts
import { GraphQLError } from 'graphql';
import { createResolvers } from '../../axolotl.js';
import { prisma } from '@/src/db.js';
import { serializeSetCookie } from '@/src/lib/cookies.js';
import { verifyPassword, signToken, generateSessionToken, getSessionExpiryDate } from '@/src/lib/auth.js';

export default createResolvers({
  Mutation: {
    login: async (input, { password, email: rawEmail }) => {
      const email = rawEmail.toLowerCase().trim();
      const user = await prisma.user.findFirst({ where: { email } });
      if (!user) throw new GraphQLError('Invalid credentials', { extensions: { code: 'INVALID_CREDENTIALS' } });
      const isValid = await verifyPassword(password, user.password);
      if (!isValid) throw new GraphQLError('Invalid credentials', { extensions: { code: 'INVALID_CREDENTIALS' } });
      const sessionToken = generateSessionToken();
      const session = await prisma.session.create({
        data: { token: sessionToken, userId: user.id, expiresAt: getSessionExpiryDate() },
      });
      const jwtToken = signToken({ userId: user.id, email: user.email, jti: session.token });
      const context = input[2];
      const { res } = context;
      if (res) {
        res.setHeader('Set-Cookie', serializeSetCookie(jwtToken));
      }
      return jwtToken;
    },
  },
});
```

**Main resolvers (merge all modules):**

```typescript
// src/resolvers.ts
import { mergeAxolotls } from '@aexol/axolotl-core';
import authResolvers from '@/src/modules/auth/resolvers/resolvers.js';
import usersResolvers from '@/src/modules/users/resolvers/resolvers.js';

// Add more modules as your application grows:
// import postsResolvers from '@/src/modules/posts/resolvers/resolvers.js';

export default mergeAxolotls(authResolvers, usersResolvers);
```

---

### Type Generation Flow

```
1. Read backend/axolotl.json
   ↓
2. For each federation entry:
   - Parse schema file
   - Generate models.ts with TypeScript types
   ↓
3. Merge all schemas using graphql-js-tree
   ↓
4. Write merged schema to root schema file
   ↓
5. Generate root models.ts from supergraph
   ↓
6. Each module uses its own models for type safety
```

---

### Advanced Federation Topics

#### Sharing Types Across Modules

When modules need to reference the same types, define them in each schema:

```graphql
# src/modules/users/schema.graphql
type User {
  _id: String!
  email: String!
}
```

```graphql
# src/modules/posts/schema.graphql
type User {
  _id: String! # Shared fields must match exactly
}

type Post {
  _id: String!
  title: String!
  content: String!
  published: Boolean!
  owner: User! # Reference the shared type
}
```

The schemas will be merged, and the `User` type will contain all fields from both definitions.

#### Cross-Module Dependencies

Modules can reference shared types and access data from the auth gateway via `source`:

```typescript
// src/modules/posts/resolvers/AuthorizedUserQuery/posts.ts
import { createResolvers } from '../../axolotl.js';
import { prisma } from '@/src/db.js';

// Posts module adds the `posts` field to AuthorizedUserQuery
// The auth gateway already validated the token and returned the user as `source`
export default createResolvers({
  AuthorizedUserQuery: {
    posts: async ([source]) => {
      const user = source as { _id: string };
      return await prisma.post.findMany({ where: { authorId: user._id } });
    },
  },
});
```

> **CRITICAL:** Domain modules define resolvers for `AuthorizedUserQuery` / `AuthorizedUserMutation` fields. They **NEVER** define `Query.user` or `Mutation.user` — those gateway resolvers are owned exclusively by the auth module. See the `axolotl-resolvers` skill for the full gateway pattern.

When multiple modules add fields to the same type (e.g., both users and posts modules add fields to `AuthorizedUserQuery`), Axolotl's schema merger combines all fields at build time, and `mergeAxolotls` handles resolver merging at runtime.

#### Custom Scalars in Federation

Define scalars in each module that uses them:

```graphql
# src/modules/posts/schema.graphql
scalar Secret

type AuthorizedUserMutation {
  createPost(title: String!, content: String!, secret: Secret): String! @resolver
}
```

Scalar resolvers should be defined once in the main `axolotl.ts`:

```typescript
// src/axolotl.ts
import { Axolotl } from '@aexol/axolotl-core';
import { Models } from '@/src/models.js';

export const { adapter, createResolvers } = Axolotl(graphqlYogaAdapter)<
  Models<{ Secret: number }>, // Map custom scalar to TypeScript type
  unknown
>();
```

#### Subscriptions in Federation

Subscriptions work in federated setups, but each subscription field should only be defined in **one module**:

```typescript
// src/modules/users/resolvers/Subscription/countdown.ts
import { createResolvers, createSubscriptionHandler } from '@aexol/axolotl-core';

export default createResolvers({
  Subscription: {
    countdown: createSubscriptionHandler(async function* (input, { from }) {
      for (let i = from ?? 3; i >= 0; i--) {
        yield i;
        await new Promise((resolve) => setTimeout(resolve, 1000));
      }
    }),
  },
});
```

If multiple modules try to define the same subscription field, only the first one encountered will be used.

#### Directives Across Modules

Directives must be defined in each schema that uses them:

```graphql
directive @resolver on FIELD_DEFINITION
directive @auth(role: String!) on FIELD_DEFINITION
```

Directive implementations should be registered in each module's Axolotl instance that needs them.

---

### Federation Development Workflow

**Initial Setup:**

```bash
# Install dependencies
npm install

# Generate Prisma client
cd backend && npx prisma generate

# Generate all models and merged schema
cd backend && npx @aexol/axolotl build

# Generate resolver scaffolding (optional)
cd backend && npx @aexol/axolotl resolvers
```

**When to Regenerate:**

Run `cd backend && npx @aexol/axolotl build` when you:

- Add or modify any schema file
- Add new types or fields
- Add or remove federation modules
- Change `backend/axolotl.json` configuration

The CLI will regenerate:

1. Each submodule's `models.ts`
2. The merged `schema.graphql`
3. The root `models.ts`

**When to also run `prisma generate`:**

If any of the above changes also required Prisma schema modifications (new models, renamed fields, new relations), run `cd backend && npx prisma generate` after `axolotl build` to update the Prisma client types.

---

### Federation Best Practices

**Module Organization:**

- Organize by business domain (users, products, orders)
- Keep related types in the same module
- Use consistent directory structure across modules
- Make modules as independent as possible

**Avoid:**

- Creating modules for every single type
- Mixing unrelated concerns in one module
- Creating circular dependencies between modules

**Naming Conventions for Shared Types:**

```graphql
# Good: Both modules use "User"
type User {
  _id: String!
}

# Bad: Different names for same concept
type UserAccount {
  _id: String!
}
type UserProfile {
  _id: String!
}
```

**Module-specific types - prefix with domain:**

```graphql
type PostItem { ... }
type PostFilter { ... }
```

**Performance Considerations:**

- **Parallel resolver execution:** When multiple modules implement the same resolver, they execute in parallel using `Promise.all()`. Be aware of database connection limits and rate limiting.
- **Deep merge overhead:** Results are deep-merged using object spreading. Keep resolver return values focused and avoid deeply nested objects when possible.
- **Use DataLoader** for batching database queries across modules.

---

### Federation Troubleshooting

#### Schema Merge Conflicts

**Error:** `Federation conflict on Node.field pattern: User.email`

**Cause:** The same field on the same type has different definitions across modules.

**Solution:** Ensure field types match exactly:

```graphql
# users/schema.graphql
type User {
  email: String! # Required
}

# profile/schema.graphql
type User {
  email: String # Optional - CONFLICT!
}
```

Fix by making them identical in both files.

#### Type Generation Failures

**Error:** `Cannot find module '@/src/modules/users/models.js'`

**Solution:**

```bash
# Regenerate all models
cd backend && npx @aexol/axolotl build

# Check your tsconfig.json has correct path mappings
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

#### Resolver Not Found

**Causes:**

- Resolver not exported from module
- Not included in `mergeAxolotls` call
- Schema and resolver names don't match

**Solution:**

```typescript
// Ensure resolver is exported
export default createResolvers({
  Query: {
    user: async () => { ... }  // Must match schema field name exactly
  }
});

// Ensure it's merged
import usersResolvers from './modules/users/resolvers/resolvers.js';
export default mergeAxolotls(usersResolvers, ...otherResolvers);
```

#### Merged Resolver Returns Unexpected Data

**Issue:** Deep merge combines objects in unexpected ways.

**Solution:** Ensure resolvers return compatible object shapes. If modules return conflicting primitives, the last one wins. Use different field names to avoid conflicts.

---

### Axolotl vs Apollo Federation Comparison

| Feature                | Axolotl Micro-Federation | Apollo Federation            |
| ---------------------- | ------------------------ | ---------------------------- |
| **Deployment**         | Single service           | Multiple services            |
| **Schema Merging**     | Build-time               | Runtime with gateway         |
| **Type Splitting**     | Automatic deep merge     | Explicit `@key` directives   |
| **Resolver Execution** | Parallel within process  | Cross-service HTTP calls     |
| **Performance**        | Fast (in-process)        | Network overhead             |
| **Complexity**         | Simple config            | Gateway + federation service |
| **Use Case**           | Monorepo/single app      | Distributed microservices    |
| **Independence**       | Shared codebase          | Fully independent services   |

---

### Auth Gateway Module Pattern

The **auth module** (`src/modules/auth/`) is a dedicated federation module that owns the protected resolver gateway pattern. It exists to centralize authentication logic and prevent gateway resolver duplication across domain modules.

**Why a separate auth module?**

- **Single source of truth** for `Query.user` and `Mutation.user` gateway resolvers
- **Eliminates duplication** — without a separate auth module, multiple domain modules might define `Query.user`, causing merge conflicts or redundant auth checks
- **Clean separation** — domain modules (e.g., users, posts) focus purely on business logic; auth handles the gateway
- **Deep merge works cleanly** — auth returns the authenticated user object, which becomes `source` for child resolvers in all domain modules

**How it works:**

1. Auth module defines `Query.user` / `Mutation.user` in its schema with `@resolver`
2. Auth resolvers validate the token and return the authenticated user
3. Domain modules define fields on `AuthorizedUserQuery` / `AuthorizedUserMutation` (but NOT the gateway fields themselves)
4. `mergeAxolotls` combines everything — auth provides the gateway, domains provide the nested resolvers

---

### Running the Federation Example

```bash
# Navigate to the example
cd examples/yoga-federated

# Install dependencies (if not already done at root)
npm install

# Generate models
npm run models

# Run in development mode
npm run dev
```

Visit `http://localhost:4002/graphql` and try these operations:

```graphql
# Register a user
mutation Register {
  register(email: "user@example.com", password: "password")
}

# Login (cookie is set automatically by the server response)
mutation Login {
  login(email: "user@example.com", password: "password")
}

# Create a post
mutation CreatePost {
  user {
    createPost(title: "Learn Axolotl Federation", content: "A guide to federation patterns")
  }
}

# Query merged data (comes from both users and posts modules!)
query MyData {
  user {
    me {
      _id
      email
    }
    posts {
      _id
      title
      content
      published
    }
  }
}
```
