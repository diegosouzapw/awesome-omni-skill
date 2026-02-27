---
name: bitrix24-agent
description: Design, implement, debug, and harden integrations between AI agents and Bitrix24 REST API (webhooks, OAuth 2.0, scopes, events, batch, limits, and REST 3.0). Use when asked to connect AI assistants/agents to Bitrix24, automate CRM/tasks/chats, process Bitrix24 events, choose an auth model, or resolve Bitrix24 API errors and performance issues.
---

# Bitrix24 Agent (Lean + Reliable)

Use this skill to deliver correct Bitrix24 integrations with minimal token usage.

## Default Mode: Lean

Apply these limits unless the user asks for deep detail:

- Load at most 2 reference files before first actionable step.
- Start from `references/packs.md`.
- Then open only one target file: `references/catalog-<pack>.md`.
- Open `references/chains-<pack>.md` only if user asks for workflow/chain.
- Open `references/bitrix24.md` only for auth architecture, limits, events reliability, or unknown errors.

Response format limits:

- Use concise output (goal + next action + one command).
- Do not retell documentation.
- Do not dump large JSON unless explicitly requested.
- Avoid repeating already provided guidance; return only delta.

## Routing Workflow

1. Determine intent:
- method call,
- troubleshooting,
- architecture decision,
- event/reliability setup.

Term normalization (product vocabulary):

- "collabs", "workgroups", "projects", "social network groups" -> `collab` (and `boards` for scrum).
- "Copilot", "CoPilot", "BitrixGPT", "AI prompts" -> `platform` (`ai.*`).
- "open lines", "contact center connectors", "line connectors" -> `comms` (`imopenlines.*`, `imconnector.*`).
- "feed", "live feed", "news feed" -> `collab` (`log.*`).
- "sites", "landing pages", "landing" -> `sites` (`landing.*`).
- "booking", "calendar", "work time", "time tracking" -> `services` (`booking.*`, `calendar.*`, `timeman.*`).
- "orders", "payments", "catalog", "products" -> `commerce` (`sale.*`, `catalog.*`).
- "consents", "consent", "e-signature", "sign" -> `compliance` (`userconsent.*`, `sign.*`).

2. Choose auth quickly:
- one portal/internal: incoming webhook.
- app/multi-portal/lifecycle features: OAuth.

3. Select minimal packs:
- default `core`.
- add only required packs: `comms`, `automation`, `collab`, `content`, `boards`, `commerce`, `services`, `platform`, `sites`, `compliance`, `diagnostics`.

4. Execute with guardrails:
- prefer `scripts/bitrix24_client.py` and `scripts/offline_sync_worker.py`,
- enforce allowlist + `--confirm-write` / `--confirm-destructive`,
- keep writes idempotent when possible.

5. Escalate to deep reference only on trigger:
- `WRONG_AUTH_TYPE`, `insufficient_scope`, `QUERY_LIMIT_EXCEEDED`, `expired_token`,
- offline event loss concerns,
- OAuth refresh race or tenant isolation issues.

## Quality Guardrails

- Never expose webhook/OAuth secrets.
- Scope and permissions must be least-privilege.
- No nested `batch`.
- Online events are not guaranteed delivery; use offline flow for no-loss processing.
- Prefer REST 3.0 where compatible; fallback to v2 where needed.

## Reference Loading Map

1. `references/packs.md` for pack and loading strategy.
2. `references/catalog-<pack>.md` for method shortlist.
3. `references/chains-<pack>.md` for implementation chains.
4. `references/bitrix24.md` only when deeper protocol detail is required.

Useful search shortcuts:

```bash
rg -n "^# Catalog|^# Chains" references/catalog-*.md references/chains-*.md
rg -n "WRONG_AUTH_TYPE|insufficient_scope|QUERY_LIMIT_EXCEEDED|expired_token" references/bitrix24.md
rg -n "offline|event\\.bind|event\\.offline|application_token" references/bitrix24.md
```

## Scripts

- `scripts/bitrix24_client.py`: method calls, packs, allowlist, confirmations, audit.
- `scripts/offline_sync_worker.py`: offline queue processing with retries and DLQ.
