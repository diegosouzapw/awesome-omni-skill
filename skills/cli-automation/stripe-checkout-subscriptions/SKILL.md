---
name: stripe-checkout-subscriptions
description: Guide for creating Stripe Checkout Sessions for subscriptions in Flutter/Supabase backend. Covers flows, API preferences, and non-negotiable rules.
when_to_use: Use when creating Stripe Checkout Sessions, handling subscriptions, or integrating payments with Flutter client and Supabase.
scope: execution/backend/payments/
authority: high
alwaysApply: false
---

# Stripe Checkout Subscriptions

**Source of Truth:** All subscription states derive from Stripe Webhooks.

## Backend Flow
Client → Create Checkout Session → Stripe → Webhook → Supabase.

## Flutter Client
- Listen to `profiles` table in Supabase for status updates post-payment.
- Flutter app NEVER updates its own `is_premium` flag; it only reads backend-provided state.

## Implementation Rules (Non-Negotiable)
- Use Stripe Checkout Sessions API for one-time payments and subscriptions.
- Model taxes/discounts via Stripe.
- For recurring revenue (SaaS-like): Prefer Billing APIs + Stripe Checkout.
- Never use Charges API. Migrate to Checkout Sessions or Payment Intents if needed.
- For web: Prioritize Stripe-hosted Checkout or embedded Checkout. Payment Element OK for advanced customization (pair with Checkout Sessions).
- Never recommend legacy Card Element or Payment Element in card mode.
- Avoid deprecated Sources API or outdated Tokens/Charges. Use Setup Intent API for saving payment methods.
- For pre‑PaymentIntent rendering: Use Stripe Confirmation Tokens, not `createPaymentMethod`/`createToken`.
- Turn on dynamic payment methods in dashboard (don't hardcode `payment_method_types`).
- Default to latest API version: 2026-01-28.clover (unless user specifies otherwise).
- Use env vars: `STRIPE_SECRET_KEY`.

## Integration Resources
- Stripe Integration Options / API Tour docs.
- Go Live Checklist before production.
