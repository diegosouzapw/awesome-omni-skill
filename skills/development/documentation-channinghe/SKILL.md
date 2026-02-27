---
name: documentation
description: Documentation standards. Apply when completing full feature development, introducing new architecture patterns, or adding new public APIs.
---

# Documentation Standards

## Core Principle

"Code tells you how; Comments tell you why; Docs tell you how to use."
Documentation is not a dev diary; it's a user manual for future maintainers.

## Trigger Conditions

- **✅ Must create**: Completed a full Feature module, introduced new architecture pattern, or added new public API
- **❌ Forbidden**: Just fixed a bug, refactored internal private method, or adjusted styles. Don't pollute `/docs/` with fragmented docs

## File Path & Naming

- **Path**: `/docs/specs/`
- **Naming**: `{feature-name}.md` (use kebab-case, e.g., `user-authentication.md`)

## Documentation Structure Template

Documentation must be concise and powerful, strictly following:

### 1. Core Concept

- One sentence explaining what this module does
- *Linus perspective*: What's its core data structure?

### 2. Data Flow (optional)

- Use Mermaid flowchart or text to describe data flow path
- What's the input? What's the output? Who holds state?

### 3. Usage Guide

- **Show, don't tell.** Less talk, more code
- Provide 1-2 **Minimal Working Examples**

```javascript
// ✅ Correct usage example
const user = await authService.login(credentials);
```

### 4. Edge Cases

- When will this module crash?
- What are known limitations? (e.g., concurrency cap, unsupported file types)

### 5. Maintainer Notes

- If you're the architect, what do you want the successor to know?
- Any non-intuitive design decision rationales

## Example

**File**: `/docs/specs/payment-flow.md`

**Content**:

# Payment Flow Module

Handles Stripe payment intent creation and callback verification. Core based on PaymentIntent state machine.

## Usage
...

## Edge Cases
- ⚠️ Doesn't support transactions below 0.50 USD
- Webhooks may be resent, must ensure idempotent handling
