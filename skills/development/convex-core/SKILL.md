---
name: convex-core
description: Core Convex development guidelines - functions, validators, schema, queries, mutations, and database patterns
globs:
  - "convex/**/*.ts"
  - "convex/schema.ts"
triggers:
  - convex
  - query
  - mutation
  - schema
  - validator
  - index
  - database
  - ctx.db
  - defineTable
  - defineSchema
---

# Convex Core Development Guide

Complete guidelines for Convex functions, validators, schema, queries, mutations, and database patterns.

---

# Function Syntax (REQUIRED)

ALWAYS use the new function syntax:

```typescript
import { query } from "./_generated/server";
import { v } from "convex/values";

export const myFunction = query({
  args: {
    // Arguments with validators
  },
  returns: v.null(), // Return validator REQUIRED
  handler: async (ctx, args) => {
    // Function body
  },
});
```

---

# Validators Reference

| Type | Validator | Notes |
|------|-----------|-------|
| Id | `v.id(tableName)` | Document ID |
| Null | `v.null()` | Use instead of undefined |
| Int64 | `v.int64()` | NOT v.bigint() (deprecated) |
| Float64 | `v.number()` | |
| Boolean | `v.boolean()` | |
| String | `v.string()` | Max 1MB UTF-8 |
| Bytes | `v.bytes()` | Max 1MB |
| Array | `v.array(values)` | Max 8192 elements |
| Object | `v.object({...})` | Max 1024 entries |
| Record | `v.record(keys, values)` | Dynamic keys |
| Optional | `v.optional(validator)` | |
| Union | `v.union(v1, v2, ...)` | |
| Literal | `v.literal("value")` | For discriminated unions |

**NOT SUPPORTED:** `v.map()`, `v.set()`, `v.bigint()`

---

# Function Registration

## Public Functions
```typescript
import { query, mutation, action } from "./_generated/server";
import { api } from "./_generated/api";
// Reference: api.filename.functionName
```

## Internal Functions (Private)
```typescript
import { internalQuery, internalMutation, internalAction } from "./_generated/server";
import { internal } from "./_generated/api";
// Reference: internal.filename.functionName
```

## Critical Rules
- ALWAYS include `args` and `returns` validators
- If no return value, use `returns: v.null()`
- File-based routing: `convex/users.ts` → `api.users.functionName`

---

# Schema Definition

```typescript
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  users: defineTable({
    name: v.string(),
    email: v.string(),
    role: v.optional(v.union(v.literal("admin"), v.literal("user"))),
  })
    .index("by_email", ["email"])
    .index("by_role", ["role"]),

  messages: defineTable({
    userId: v.id("users"),
    content: v.string(),
    channelId: v.id("channels"),
  })
    .index("by_channel", ["channelId"])
    .index("by_user_and_channel", ["userId", "channelId"]),
});
```

## System Fields (Automatic)
- `_id`: `v.id(tableName)`
- `_creationTime`: `v.number()`

---

# Index Rules (CRITICAL)

```typescript
// WRONG - Will cause error!
.index("by_creation_time", ["_creationTime"])  // Built-in, don't add
.index("by_author_and_time", ["author", "_creationTime"])  // _creationTime is automatic

// CORRECT
.index("by_author", ["author"])  // _creationTime added automatically
.index("by_channel_and_author", ["channelId", "authorId"])
```

- Index names describe all fields: `by_field1_and_field2`
- Query order must match index order
- Never include `_creationTime` in index definition

---

# Query Patterns

## Using Indexes (REQUIRED)
```typescript
// CORRECT - Use withIndex
const messages = await ctx.db
  .query("messages")
  .withIndex("by_channel", (q) => q.eq("channelId", channelId))
  .order("desc")
  .take(10);

// WRONG - Never use filter()
const messages = await ctx.db
  .query("messages")
  .filter((q) => q.eq(q.field("channelId"), channelId))  // BAD!
  .collect();
```

## Get by ID
```typescript
const user = await ctx.db.get(userId);
if (!user) throw new Error("User not found");
```

## Ordering
```typescript
.order("asc")   // Ascending (default)
.order("desc")  // Descending
```

## Collecting Results
```typescript
.collect()     // Get all results
.take(n)       // Get first n results
.first()       // Get first result or null
.unique()      // Get single result, throws if multiple
```

---

# Mutations

```typescript
// Insert - returns Id
const id = await ctx.db.insert("users", { name: "Alice", email: "alice@example.com" });

// Patch - partial update
await ctx.db.patch(userId, { name: "Bob" });

// Replace - full replacement
await ctx.db.replace(userId, { name: "Bob", email: "bob@example.com" });

// Delete
await ctx.db.delete(userId);
```

## Delete Pattern (No .delete() on queries)
```typescript
const items = await ctx.db
  .query("items")
  .withIndex("by_user", (q) => q.eq("userId", userId))
  .collect();

for (const item of items) {
  await ctx.db.delete(item._id);
}
```

---

# Function Calling

```typescript
// From mutation or action
const user = await ctx.runQuery(api.users.get, { id: userId });
await ctx.runMutation(internal.users.update, { id: userId, name });

// From action only
await ctx.runAction(internal.ai.generate, { prompt });
```

## Type Annotation for Same-File Calls
```typescript
export const f = query({
  args: { name: v.string() },
  returns: v.string(),
  handler: async (ctx, args) => "Hello " + args.name,
});

export const g = query({
  args: {},
  returns: v.null(),
  handler: async (ctx) => {
    const result: string = await ctx.runQuery(api.example.f, { name: "Bob" });
    return null;
  },
});
```

---

# Pagination

```typescript
import { paginationOptsValidator } from "convex/server";

export const list = query({
  args: {
    paginationOpts: paginationOptsValidator,
    channelId: v.id("channels"),
  },
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .order("desc")
      .paginate(args.paginationOpts);
  },
});
// Returns: { page, isDone, continueCursor }
```

---

# Full Text Search

```typescript
// Schema
defineTable({
  body: v.string(),
  channel: v.string(),
}).searchIndex("search_body", {
  searchField: "body",
  filterFields: ["channel"],
})

// Query
const results = await ctx.db
  .query("messages")
  .withSearchIndex("search_body", (q) =>
    q.search("body", "hello").eq("channel", "#general")
  )
  .take(10);
```

---

# System Limits

| Limit | Value |
|-------|-------|
| Function args/returns | 8 MiB |
| Array elements | 8192 |
| Object entries | 1024 |
| Document size | 1 MiB |
| Query/Mutation timeout | 1 second |
| DB read per query | 8 MiB / 16384 docs |
| DB write per mutation | 8 MiB / 8192 docs |

---

# TypeScript Types

```typescript
import { Id, Doc } from "./_generated/dataModel";

// Use Id<> for document IDs
function getUser(userId: Id<"users">): Promise<Doc<"users"> | null>

// Record with Id keys
const map: Record<Id<"users">, string> = {};

// Arrays with explicit types
const items: Array<{ id: Id<"items">; name: string }> = [];
```
