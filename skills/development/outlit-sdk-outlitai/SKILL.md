---
name: outlit-sdk
description: Integrate Outlit SDK for customer context for agents. Triggers when users need to add Outlit to any web framework (React, Next.js, Vue, Nuxt, Svelte, Angular, Astro), server runtime (Node.js, Express, Fastify), desktop app (Tauri, Electron), or need help with Outlit event tracking, user identity, consent management, analytics migration, activation events, billing lifecycle, or troubleshooting existing Outlit installations.
---

# Outlit SDK Integration

Decision-tree-driven guide for integrating Outlit customer journey tracking. Detects what it can from the codebase, asks only what it must, and links to official docs for implementation details.

## Branching Check

Before anything else, check if Outlit is already installed:

1. Look for `@outlit/browser`, `@outlit/node`, or `outlit` (Rust crate) in `package.json` or `Cargo.toml`
2. If **not installed** -> go to [Phase 1: Quick Connect](#phase-1-quick-connect)
3. If **already installed** -> go to [Already Installed](#already-installed)

## Already Installed

Ask the user what they need help with:

- **Add event tracking** -> Run detection, then go to [Decision 7: Event Tracking](#decision-7-event-tracking). Fetch the relevant framework doc from the [Doc URL Map](#doc-url-map) for implementation patterns.
- **Add/change auth integration** -> Run detection, then go to [Decision 3: Auth & Identity](#decision-3-auth--identity). Fetch the framework doc and [identity resolution](https://docs.outlit.ai/concepts/identity-resolution) doc.
- **Add consent management** -> Go to [Decision 2: Consent Stance](#decision-2-consent-stance). Fetch the framework doc's consent section.
- **Add server-side tracking** -> Go to [Decision 1: App Type & SDK](#decision-1-app-type--sdk-package) for the server package, then fetch the [Node.js](https://docs.outlit.ai/tracking/server/nodejs) or [Rust](https://docs.outlit.ai/tracking/server/rust) doc.
- **Add billing/Stripe integration** -> Go to [Decision 6: Billing Integration](#decision-6-billing-integration). Fetch the [customer journey](https://docs.outlit.ai/concepts/customer-journey) doc.
- **Add activation tracking** -> Go to [Decision 5: Activation Event](#decision-5-activation-event). Fetch the [customer journey](https://docs.outlit.ai/concepts/customer-journey) doc.
- **Migrate from other analytics** -> Run detection, then go to [Decision 4: Existing Analytics](#decision-4-existing-analytics-strategy).
- **Debug/troubleshoot** -> Go to [Troubleshooting](#troubleshooting).

## Phase 1: Quick Connect

Goal: get events flowing in ~2 minutes so the user sees "Connected" on their Outlit onboarding screen. Zero user decisions required.

### Step 1: Detect Framework & Package Manager

Use glob/grep to check:

- **Framework:** Check `package.json` dependencies for `next`, `vue`, `nuxt`, `react`, `svelte`, `@sveltejs/kit`, `@angular/core`, `astro`, `express`, `fastify`. Check for `Cargo.toml` with `tauri` or `outlit`.
- **Package manager:** Check for `bun.lockb` (bun), `pnpm-lock.yaml` (pnpm), `yarn.lock` (yarn), `package-lock.json` (npm). Use the first match found.

### Step 2: Install SDK

Based on detected framework:

- Browser app (React, Next.js, Vue, Nuxt, Svelte, Angular, Astro) -> `@outlit/browser`
- Server app (Express, Fastify, Node.js) -> `@outlit/node`
- Tauri -> `outlit` Rust crate via `cargo add outlit`
- Electron -> `@outlit/browser`

Install using the detected package manager.

### Step 3: Add Public Key

Ask the user for their Outlit public key. They get it from **Outlit dashboard -> Settings -> Website Tracking** or from the onboarding screen.

Add to environment variables with the correct framework prefix:

| Framework | Env var |
|-----------|---------|
| Next.js | `NEXT_PUBLIC_OUTLIT_KEY` |
| Vite (Vue, Svelte, React+Vite) | `VITE_OUTLIT_KEY` |
| Create React App | `REACT_APP_OUTLIT_KEY` |
| Nuxt | `NUXT_PUBLIC_OUTLIT_KEY` |
| Angular | Add to `environment.ts` |
| Astro | `PUBLIC_OUTLIT_KEY` |
| Server apps | `OUTLIT_KEY` |

### Step 4: Minimal Setup

Fetch the framework-specific doc from the [Doc URL Map](#doc-url-map) and implement only the minimal setup — provider/init with just the `publicKey`, no auth, no consent, no custom events.

For React-based frameworks this means wrapping the app with `OutlitProvider`. For Vue it means installing the `OutlitPlugin`. For server apps it means creating an Outlit instance.

### Step 5: Verify Connection

Tell the user to:

1. Run their dev server
2. Open the app in a browser
3. Check the Outlit onboarding screen for the **"Connected"** badge
4. Or check DevTools -> Network for requests to `app.outlit.ai/api/i/v1/...` returning 200

Once connected, ask: **"Events are flowing. Ready to set up the full integration?"**

If yes -> continue to Phase 2. If no -> they can come back later (the skill will detect the existing install).

## Phase 2: Full Integration

Run the full detection first, then walk through each decision.

### Full Detection

Use grep/glob to detect all of the following before starting the decision tree:

| Signal | How to detect |
|--------|--------------|
| Auth provider | Deps: `@clerk/nextjs`, `@clerk/clerk-react`, `next-auth`, `@auth/core`, `@supabase/auth-helpers-nextjs`, `@supabase/ssr`, `@auth0/auth0-react`, `@auth0/nextjs-auth0`, `firebase`, `@firebase/auth` |
| Billing provider | Deps: `stripe`, `@stripe/stripe-js`, `@paddle/paddle-js`, `chargebee` |
| Existing analytics | Deps: `posthog-js`, `@posthog/node`, `@amplitude/analytics-browser`, `@amplitude/analytics-node`, `mixpanel-browser`, `@segment/analytics-next`, `@segment/analytics-node` |
| Analytics abstraction | Grep for files named `analytics.ts`, `analytics.js`, `tracking.ts`, `tracking.js` in `lib/`, `utils/`, `helpers/`, `services/` that import 2+ analytics libraries |
| EU/consent signals | Deps: `cookiebot`, `@onetrust/*`, `cookie-consent`, or grep for existing consent/cookie banner components |
| App type | Deps: `@tauri-apps/api`, `electron`, `react-native` |
| Activation patterns | Grep for onboarding routes/components, "first" resource creation handlers, invite flows |

Present a summary of what was detected to the user before proceeding.

### Decision 1: App Type & SDK Package

Auto-resolved from Phase 1 detection. If hybrid (e.g., Next.js with API routes that need server tracking), install both `@outlit/browser` and `@outlit/node`.

- Electron -> `@outlit/browser`
- Tauri -> `outlit` Rust crate
- React Native -> `@outlit/browser` (uses fingerprint instead of cookies)

### Decision 2: Consent Stance

| Detection | Recommendation |
|-----------|---------------|
| Existing CMP/cookie banner library | `autoTrack: false`, integrate with their existing CMP's consent callback to call `enableTracking()` |
| EU signals but no CMP | `autoTrack: false`, mention they need a consent solution but don't build one unless asked |
| No EU signals | `autoTrack: true` — simpler, uses cookies immediately |

**Always explain the tradeoff:** `autoTrack: true` starts tracking with cookies immediately. `autoTrack: false` waits until `enableTracking()` is called after user consent.

Fetch the relevant framework doc for consent implementation patterns.

### Decision 3: Auth & Identity

| Detection | Recommendation |
|-----------|---------------|
| React/Vue with OutlitProvider/OutlitPlugin | Pass `user` prop with `{ email, userId }` after auth resolves. No manual `identify()` needed. |
| Vanilla JS / script tag | Call `outlit.identify({ email, userId })` client-side right after auth completes |
| Server-only (Node/Rust) | Call `outlit.identify()` server-side for event attribution |

**Critical:** Client-side `identify()` (or the `user` prop) links the anonymous cookie/visitorId to a real person. This must happen after the auth flow completes. Server-side `identify()` is for attributing server events to a known user, but doesn't link browsing history.

If an auth provider was detected, fetch the framework doc and the [identity resolution](https://docs.outlit.ai/concepts/identity-resolution) doc for the specific auth provider pattern.

### Decision 4: Existing Analytics Strategy

| Detection | Recommendation |
|-----------|---------------|
| Existing analytics abstraction (e.g., `lib/analytics.ts` wrapping PostHog + Amplitude) | Add Outlit as another provider inside the existing abstraction |
| Scattered direct calls (e.g., `posthog.capture()` in 15 files) | Tell the user: "I found [N] direct [PostHog/Amplitude] calls across [N] files. Want me to create an analytics wrapper that calls both, or add Outlit calls alongside the existing ones?" |
| No existing analytics | Add Outlit directly, no wrapper needed |

The goal is minimal code changes. Don't reorganize their project.

### Decision 5: Activation Event

The activation event marks when a user first gets real value from the product. This is NOT just completing onboarding.

1. Scan the codebase for value-moment patterns:
   - First resource/project/item created
   - First core feature used (e.g., first message sent, first report generated)
   - First invite to a teammate
   - First successful integration/connection
2. If a clear value moment is found -> suggest it: "It looks like creating their first [X] could be your activation event — is that where users first get value?"
3. If only an onboarding flow is found -> mention it as a fallback: "I see an onboarding flow, but activation usually maps to when users get real value, not just completing setup. What action means a user has truly gotten value from your product?"
4. If nothing obvious -> ask: "What action means a user has gotten real value from your product? That's where `user.activate()` should go."

Fetch the [customer journey](https://docs.outlit.ai/concepts/customer-journey) doc for `user.activate()` implementation.

### Decision 6: Billing Integration

| Detection | Recommendation |
|-----------|---------------|
| Stripe in dependencies | Recommend the Stripe webhook integration — it automatically handles `customer.paid()`, `customer.trialing()`, `customer.churned()` based on Stripe events. Fetch the [customer journey](https://docs.outlit.ai/concepts/customer-journey) doc for webhook setup. |
| Other billing provider (Paddle, Chargebee) | Guide manual `customer.paid()` / `customer.trialing()` / `customer.churned()` calls in their existing webhook handlers |
| No billing provider detected | Skip. Mention it's available when they add billing. |

### Decision 7: Event Tracking

Pageviews are tracked automatically by default. For custom events:

1. Scan the codebase for existing analytics calls, user action handlers, form submissions, key button clicks
2. Suggest a list of events based on what the codebase reveals (e.g., `form_submitted`, `feature_used`, `item_created`, `search_performed`)
3. Ask the user to confirm, modify, or add to the list before implementing
4. Use `snake_case` for all event names

Fetch the framework doc for the `track()` API pattern.

## Doc URL Map

Fetch these docs as needed for implementation details. Always prefer linking to docs over hardcoding patterns.

| Topic | URL |
|-------|-----|
| Quickstart | `https://docs.outlit.ai/tracking/quickstart` |
| How tracking works | `https://docs.outlit.ai/tracking/how-it-works` |
| NPM package | `https://docs.outlit.ai/tracking/browser/npm` |
| React | `https://docs.outlit.ai/tracking/browser/react` |
| Next.js | `https://docs.outlit.ai/tracking/browser/nextjs` |
| Vue 3 | `https://docs.outlit.ai/tracking/browser/vue` |
| Nuxt | `https://docs.outlit.ai/tracking/browser/nuxt` |
| SvelteKit | `https://docs.outlit.ai/tracking/browser/sveltekit` |
| Angular | `https://docs.outlit.ai/tracking/browser/angular` |
| Astro | `https://docs.outlit.ai/tracking/browser/astro` |
| Script tag | `https://docs.outlit.ai/tracking/browser/script` |
| Calendar embeds | `https://docs.outlit.ai/tracking/browser/calendar-embeds` |
| Node.js | `https://docs.outlit.ai/tracking/server/nodejs` |
| Rust / Tauri | `https://docs.outlit.ai/tracking/server/rust` |
| Identity resolution | `https://docs.outlit.ai/concepts/identity-resolution` |
| Anonymous tracking | `https://docs.outlit.ai/concepts/anonymous-tracking` |
| Customer journey | `https://docs.outlit.ai/concepts/customer-journey` |
| MCP integration | `https://docs.outlit.ai/ai-integrations/mcp` |
| Ingest API | `https://docs.outlit.ai/api-reference/ingest` |
| API introduction | `https://docs.outlit.ai/api-reference/introduction` |
| Full docs index | `https://docs.outlit.ai/llms.txt` |

## Troubleshooting

### No events in dashboard

1. Verify the public key env var has the correct framework prefix (`NEXT_PUBLIC_`, `VITE_`, `REACT_APP_`, etc.)
2. Confirm the provider/init wraps the entire app or runs at startup
3. Check `autoTrack` setting — if `false`, `enableTracking()` must be called after consent
4. Check DevTools -> Network for requests to `app.outlit.ai/api/i/v1/...` — look for 200 responses
5. If requests are missing entirely, the SDK isn't initializing — check for errors in the console

### Events not linked to users

- `identify()` or the `user` prop must be called/set after the auth flow resolves
- Include both `email` and `userId` for reliable identity resolution
- Client-side identify links browsing history; server-side identify attributes server events

### Server-side events missing

- Always `await outlit.flush()` before the function returns, especially in serverless environments
- Verify `OUTLIT_KEY` env var is set in the server environment

### Race condition with async auth

Auth providers like Clerk and Auth0 load asynchronously. If `user.activate()` or `identify()` is called before the auth provider resolves, events get silently dropped. Ensure auth state is fully loaded before calling identity or stage methods.

## Key Principles

- **Minimal changes** — touch as few files as possible, add alongside existing code
- **Detect first, ask second** — auto-resolve what you can, only prompt for genuine decisions
- **Recommendations have reasoning** — when presenting a choice, lead with the recommendation and explain why
- **Client-side identify is critical** — this links anonymous browsing to a real user
- **Flush in serverless** — always `await outlit.flush()` before function exits
- **snake_case events** — `subscription_created` not `SubscriptionCreated`
- **Both IDs** — always provide both `email` and `userId` for identity resolution

## Installation

To add this skill to your Claude Code environment:

```sh
npx add-skill outlitai/outlit-agent-skills
# or
bunx add-skill outlitai/outlit-agent-skills
```
