---
name: stripe-api-versioning-and-upgrades
description: Upgrading Stripe API versions, SDKs, Stripe.js, and mobile SDKs. Covers versioning rules and upgrade checklist.
when_to_use: Use when updating Stripe API/SDK versions, testing version changes, or migrating integrations.
scope: execution/backend/payments/
authority: high
alwaysApply: false
---

# Stripe API Versioning & Upgrades

**Latest API version:** 2026-01-28.clover. Default to latest unless user specifies otherwise.

## API Versioning Basics
- Date-based versions (e.g., `2026-01-28.clover`, `2025-08-27.basil`).
- **Backward-compatible:** New resources/params, webhook types (no code changes).
- **Breaking:** Field renames/removals, behavior changes (review changelog).

## Server-Side SDKs
- **Dynamic languages** (Node.js/Python/Ruby/PHP): Set `apiVersion` globally or per-request.
  ```javascript
  const stripe = require('stripe')(key, { apiVersion: '2026-01-28.clover' });
  ```
