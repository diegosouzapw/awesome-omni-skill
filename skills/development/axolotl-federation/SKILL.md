---
name: axolotl-federation
description: Axolotl micro-federation architecture - config, schema merging, mergeAxolotls, cross-module dependencies, best practices, and troubleshooting
---

## Micro-Federation

- Each domain has its own `schema.graphql`, `models.ts`, `axolotl.ts`, and `resolvers/`
- `axolotl build` merges schemas into a single supergraph
- Each module has its own generated `models.ts` for local type safety
- All modules built and deployed together (not distributed microservices)

---

## Module Adapter

All modules use `graphqlYogaWithContextAdapter<AppContext>()` for typed context. Only root passes a context builder.

```typescript
// every module's axolotl.ts — type-only, no context builder
import { graphqlYogaWithContextAdapter } from '@aexol/axolotl-graphql-yoga';
import type { AppContext } from '@/src/context.js';
const yogaAdapter = graphqlYogaWithContextAdapter<AppContext>();
export const { createResolvers } = Axolotl(yogaAdapter)<Models, unknown>();

// root src/axolotl.ts — WITH context builder (runs once at startup)
const yogaAdapter = graphqlYogaWithContextAdapter<AppContext>(async (initial) => {
  // verifyAuth here — runs per-request
  return { ...initial, authUser };
});
export const { createResolvers, adapter } = Axolotl(yogaAdapter)<Models, Scalars>();
```

---

## Schema Merging Rules

- **Types merged by name** — fields from all modules combined; conflicting field types cause build failure
- **Root types auto-merged** — `Query`, `Mutation`, `Subscription` fields combined across modules
- **Shared field definitions must match exactly**

---

## No `extend type` — Use `type` Declarations

**Never use `extend type`.** Axolotl merges by name, not SDL extension.

```graphql
# ✅ CORRECT — plain type in each module
# src/modules/users/schema.graphql
type AuthorizedUserQuery {
  me: User @resolver
}

# src/modules/posts/schema.graphql
type AuthorizedUserQuery {
  posts: [Post!]! @resolver
}
# → merged: AuthorizedUserQuery has both `me` and `posts`

# ❌ WRONG
extend type AuthorizedUserQuery {
  posts: [Post!]! @resolver
}
```

Applies to ALL shared types: `Query`, `Mutation`, `AuthorizedUserQuery`, `AuthorizedUserMutation`, and any shared domain types.

---

## Sharing Types Across Modules

Modules can declare the same type name — shared fields must match exactly, unique fields are merged:

```graphql
# users/schema.graphql         # posts/schema.graphql
type User {                     type User {
  _id: String!                    _id: String!  # must match
  email: String!                }
}                               type Post { owner: User! }
# → merged User has _id + email
```

---

## `mergeAxolotls` Behavior

- **Non-overlapping resolvers** — combined into one map
- **Overlapping resolvers** — executed in parallel, results deep-merged
- **Subscriptions** — first-wins; define each in exactly ONE module

```typescript
import { mergeAxolotls } from '@aexol/axolotl-core';
export default mergeAxolotls(authResolvers, usersResolvers /*, postsResolvers */);
```

---

## Auth Gateway & Cross-Module Access

- Auth module exclusively owns `Query.user` / `Mutation.user` — never duplicate in domain modules
- Domain modules add fields to `AuthorizedUserQuery` / `AuthorizedUserMutation`
- Domain resolvers access auth via `context.authUser` (set by context builder):

```typescript
export default createResolvers({
  AuthorizedUserQuery: {
    posts: async ([, , context]) => {
      return prisma.post.findMany({ where: { authorId: context.authUser!._id } });
    },
  },
});
```

---

## When to Run `axolotl build`

Run `cd backend && npx @aexol/axolotl build` after any schema/type/field change or federation config change.

---

## Custom Scalars

Declare `scalar Secret` in each module schema that uses it. Map once in root `axolotl.ts`:

```typescript
Axolotl(yogaAdapter)<Models<{ Secret: number }>, Scalars>();
```

See `axolotl-server` skill for `createScalars` implementation.
