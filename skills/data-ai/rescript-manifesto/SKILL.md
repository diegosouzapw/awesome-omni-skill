---
name: rescript-manifesto
description: Type-safe ReScript patterns for sound type systems. Auto-triggers when working with ReScript (.res) files or discussing ReScript/ReasonML architecture. Enforces phantom types, branded IDs, state machines, domain primitives, exhaustive matching, and parse-don't-validate patterns. Use for any ReScript code generation, review, or refactoring.
---

# ReScript Sound Type Manifesto

Apply these principles when generating or reviewing ReScript code.

## Core Rules

### 1. Domain-Typed Primitives
Never use raw `string`, `int`, `float` for domain concepts:

```rescript
// ❌ BAD
let userId: int = 123
let email: string = "test@example.com"

// ✅ GOOD  
type userId = UserId(int)
type email = Email(string)
```

### 2. Make Impossible States Impossible
Use variants instead of multiple booleans/options:

```rescript
// ❌ BAD
type request = {isLoading: bool, data: option<t>, error: option<e>}

// ✅ GOOD
type request<'data, 'error> =
  | Idle
  | Loading  
  | Success('data)
  | Failed('error)
```

### 3. No Default Values
Never conflate "unknown" with valid values:

```rescript
// ❌ BAD: 0 for unknown
let balance = 0.0

// ✅ GOOD: Explicit optionality
let balance: option<Money.t> = None
```

### 4. Parse, Don't Validate
Transform untyped data to typed data once, at boundaries:

```rescript
// ❌ BAD: Validate repeatedly
if isValidEmail(str) { sendEmail(str) }

// ✅ GOOD: Parse once
switch Email.make(str) {
| Ok(email) => sendEmail(email)  // email is Email.t, guaranteed valid
| Error(e) => handleError(e)
}
```

### 5. Exhaustive Matching
Never use wildcards (`_`) for domain variants:

```rescript
// ❌ BAD: Hides future bugs
switch status {
| Active => "active"
| _ => "other"
}

// ✅ GOOD: Compiler catches new variants
switch status {
| Active => "active"
| Pending => "pending"  
| Cancelled => "cancelled"
}
```

### 6. Errors as Values
Use `result<'a, 'e>` not exceptions:

```rescript
// ❌ BAD
let divide = (a, b) => if b == 0 { raise(DivByZero) } else { a / b }

// ✅ GOOD
let divide = (a, b): result<int, [#DivByZero]> =>
  b == 0 ? Error(#DivByZero) : Ok(a / b)
```

### 7. Shared Types Across Boundaries
Backend and frontend share domain types from a common package. Types are contracts.

## Opaque Types Pattern

When validation needed at construction:

```rescript
module Email: {
  type t
  let make: string => result<t, [#InvalidFormat]>
  let toString: t => string
} = {
  type t = string
  let make = s => s->String.includes("@") ? Ok(s) : Error(#InvalidFormat)
  let toString = t => t
}
```

## State Machine Pattern

Model states as types, transitions as functions:

```rescript
type pending = {orderId: orderId}
type paid = {orderId: orderId, paidAt: timestamp}
type shipped = {orderId: orderId, paidAt: timestamp, shippedAt: timestamp}

let pay: pending => paid
let ship: paid => shipped  // Can ONLY ship paid orders
```

## Quick Checklist

Before writing any type:

| Check | If Yes |
|-------|--------|
| Using raw `string`/`int`/`float`? | Wrap in domain type |
| 2+ related booleans? | Convert to variant |
| 2+ related options? | Convert to variant |
| Using 0, "", [] as placeholder? | Use `option` or loading variant |
| Type exists in both FE and BE? | Move to shared package |

## Full Reference

For detailed patterns, examples, and philosophy: see [references/MANIFESTO.md](references/MANIFESTO.md)
