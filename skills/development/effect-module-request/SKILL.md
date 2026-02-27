---
name: effect-module-request
description: Guidance for `effect/Request` focused on APIs like of, fail, and succeed. Load after `effect-skill-router` when this module is the primary owner.
---

# Effect Module Request

## Owned scope

- Owns only `effect/Request`.
- Source of truth: `packages/effect/src/Request.ts`.

## What it is for

- The `Request` module provides a way to model requests to external data sources in a functional and composable manner. Requests represent descriptions of operations that can be batched, cached, and executed efficiently.

## API quick reference

- `of`
- `fail`
- `succeed`
- `Services`
- `failCause`
- `makeEntry`
- `isRequest`
- `Any`
- `Class`
- `Entry`
- `Error`
- `Result`
- `tagged`
- `Request`
- `Success`
- `complete`
- `Variance`
- `Constructor`
- Full API list: `references/api-reference.md`

## How to use it

- Start with constructor-style APIs to build values/services before composing operations.
- Use the reference docs to select the smallest API surface that solves your task.
- Validate behavior against existing tests before introducing new usage patterns.

## Starter example

```ts
import type { Request } from "effect";

// Define a request that fetches a user by ID
interface GetUser extends Request.Request<string, Error> {
  readonly _tag: "GetUser";
  readonly id: number;
}

// Define a request that fetches all users
interface GetAllUsers extends Request.Request<ReadonlyArray<string>, Error> {
  readonly _tag: "GetAllUsers";
}
```

## Common pitfalls

- Prefer explicit, typed combinators over ad-hoc casting or unchecked assumptions.

## Not covered here

- Adjacent modules in `effect/*` and `effect/unstable/*` are out of scope for this owner.

## Escalate to

- `effect-skill-router` for routing and ownership checks.

## Reference anchors

- Module source: `packages/effect/src/Request.ts`
- Representative tests: `packages/effect/test/Request.test.ts`
- API details: `references/api-reference.md`
- Usage notes: `references/usage-reference.md`
- Ownership mapping: `references/owner.md`
