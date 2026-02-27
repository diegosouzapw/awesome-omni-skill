---
name: effect-module-resource
description: Guidance for `effect/Resource` focused on APIs like get, isResource, and auto. Load after `effect-skill-router` when this module is the primary owner.
---

# Effect Module Resource

## Owned scope

- Owns only `effect/Resource`.
- Source of truth: `packages/effect/src/Resource.ts`.

## What it is for

- Module-specific APIs and usage patterns for Effect programs.

## API quick reference

- `get`
- `isResource`
- `auto`
- `manual`
- `refresh`
- `Resource`
- Full API list: `references/api-reference.md`

## How to use it

- Treat stateful APIs as synchronization boundaries and keep updates atomic.
- Use the reference docs to select the smallest API surface that solves your task.
- Validate behavior against existing tests before introducing new usage patterns.

## Starter example

```ts
import { Resource } from "effect/Resource";

const value = Resource.get();
```

## Common pitfalls

- Prefer explicit, typed combinators over ad-hoc casting or unchecked assumptions.

## Not covered here

- Adjacent modules in `effect/*` and `effect/unstable/*` are out of scope for this owner.

## Escalate to

- `effect-skill-router` for routing and ownership checks.

## Reference anchors

- Module source: `packages/effect/src/Resource.ts`
- Representative tests: `packages/effect/test/Resource.test.ts`
- API details: `references/api-reference.md`
- Usage notes: `references/usage-reference.md`
- Ownership mapping: `references/owner.md`
