---
name: arktype
description: Expert knowledge for runtime validation in TypeScript using ArkType, a syntax-first validation library with TypeScript-like definitions, JIT compilation for 10x-100x performance over Zod, native recursion support, morphs for data transformation, and Standard Schema compatibility
last_updated: 2025-12-20T00:00:00Z
hash: 59e7d3a3f7777de4
---

# ArkType

ArkType is a runtime validation library for TypeScript that uses a **syntax-first approach**—definitions look exactly like TypeScript code. Unlike builder-pattern libraries (Zod, Yup), ArkType JIT-compiles schemas into optimized validators, achieving 10x-100x performance improvements.

## Core Principles

- **Write TypeScript syntax, not builder chains** - `"string >= 8"` instead of `.string().min(8)`
- **Zero drift between static and runtime types** - Automatic type inference from definitions
- **JIT compilation for performance** - Schemas compile to optimized JavaScript functions
- **Native recursion support** - Use Scopes for circular references without `lazy()` wrappers
- **Transform during validation** - Morphs allow data transformation in the validation pipeline
- **Standard Schema compliant** - Works with React Hook Form, TanStack Query, tRPC, and other modern libraries
- **Type-level performance matters** - Track both runtime speed AND TypeScript compiler instantiations
- **String definitions for simple types** - Use `type("string")` for primitives
- **Object definitions for structures** - Use `type({ name: "string" })` for objects
- **Scopes for complex models** - Use `scope({ ... })` for interconnected types

## Quick Reference

### Basic Validation

```typescript
import { type } from "arktype"

// Simple types
const email = type("string.email")
const age = type("number >= 18")
const tags = type("string[]")

// Object schemas
const user = type({
    name: "string",
    age: "number >= 18",
    "email?": "string.email",  // Optional field
    tags: "string[]"
})

// Validation
const { data, errors } = user({
    name: "Alice",
    age: 25,
    tags: ["typescript"]
})

if (errors) {
    console.error(errors.summary)
} else {
    console.log(data.name) // Fully typed
}
```

### Type Inference

```typescript
// Infer TypeScript type from ArkType definition
const userSchema = type({ name: "string", age: "number" })
type User = typeof userSchema.infer
// User = { name: string; age: number }
```

## Topics

### Core Concepts

- [Scopes and Recursion](./scopes-recursion.md) - Circular references and interconnected types
- [Generics](./generics.md) - Type functions for reusable schemas (e.g., `Paginated<T>`)
- [Morphs](./morphs.md) - Transform data during validation (string to Date, trimming, etc.)
- [Pattern Matching](./pattern-matching.md) - Type-safe switch statements with `.match()`

### Advanced Features

- [Constraints and Intersections](./constraints.md) - Ranges, regex, divisors, and type combinations
- [Custom Error Messages](./error-messages.md) - Configure errors at global, scope, or type level
- [Framework Integration](./framework-integration.md) - Hono, Fastify, and Standard Schema usage

### Performance and Migration

- [Benchmarking](./benchmarking.md) - Runtime and type-level performance testing with `@ark/attest`
- [Migrating from Zod](./zod-migration.md) - Syntax comparison and migration patterns

## Common Patterns

### Discriminated Unions (Automatic)

```typescript
// ArkType detects discriminants automatically
const response = type([
    { status: "'success'", data: "string" },
    "|",
    { status: "'error'", message: "string" }
])
```

### Constraints with Intersection

```typescript
// Email from specific domain
const staffEmail = type("string.email & /.*@company.com/")

// Even numbers under 100
const evenUnder100 = type("number % 2 & < 100")
```

### Data Transformation

```typescript
// Convert string to Date during validation
const dateSchema = type("string").morph((s) => new Date(s))

// Sanitize user input
const username = type("string > 0").morph(s => s.trim().toLowerCase())
```

## ArkType vs Zod

| Feature | ArkType | Zod |
|---------|---------|-----|
| **Syntax** | `"string >= 5"` | `z.string().min(5)` |
| **Performance** | JIT-compiled (10x-100x faster) | Interpreted |
| **Recursion** | Native via Scopes | Requires `z.lazy()` |
| **Inference** | `typeof schema.infer` | `z.infer<typeof schema>` |
| **Bundle Size** | ~40kB (zero deps) | ~13kB (zero deps) |

## When to Use ArkType

- **High-throughput APIs** - Performance critical validation (10k+ items)
- **Complex domain models** - Recursive data structures (folder trees, graphs)
- **TypeScript-first teams** - Prefer type syntax over builder patterns
- **Standard Schema adoption** - Need library compatibility
- **Type-level optimization** - Avoid "excessively deep instantiation" errors

## Resources

- [Official Docs](https://arktype.io)
- [GitHub](https://github.com/arktypeio/arktype)
- [Benchmarks](https://moltar.github.io/typescript-runtime-type-benchmarks)
- [Standard Schema](https://github.com/standard-schema/standard-schema)
