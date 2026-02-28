---
name: highlevel
version: 1.2.0
description: "Connect your AI assistant to GoHighLevel CRM via the official API v2. Manage contacts, conversations, calendars, pipelines, invoices, payments, workflows, and 30+ endpoint groups through natural language. Includes interactive setup wizard and 100+ pre-built, safe API commands. Python 3.6+ stdlib only — zero external dependencies."
tags: [gohighlevel, crm, api, contacts, conversations, calendars, opportunities, invoices, workflows, automation]
author: Ty Shane
homepage: https://launchmyopenclaw.com
metadata:
  author: Ty Shane
  version: "1.2.0"
  clawdbot:
    emoji: 🦞
    homepage: "https://launchmyopenclaw.com"
    requires:
      env:
        - HIGHLEVEL_TOKEN
        - HIGHLEVEL_LOCATION_ID
compatibility: "Requires Python 3.6+ (stdlib only, no pip installs). Requires two environment variables: HIGHLEVEL_TOKEN and HIGHLEVEL_LOCATION_ID."
---

# GoHighLevel API Skill

> **Turn your AI assistant into a GoHighLevel command center.** Search contacts, send messages, book appointments, manage pipelines, create invoices, schedule social posts — across all 39 GHL API v2 endpoint groups, using plain English.

**Don't have GoHighLevel yet?** Start with the free 5-Day AI Employee Challenge and build a fully automated system:
👉 [**Start the 5-Day AI Employee Challenge**](https://gohighlevel.com/5-day-challenge?fp_ref=369ai)

## Requirements

| Requirement | Details |
|-------------|---------|
| **Runtime** | Python 3.6+ (uses only standard library: `urllib`, `json`, `os`, `re`, `sys`, `time`) |
| **External packages** | **None** — zero `pip install` required |
| **Environment variables** | `HIGHLEVEL_TOKEN` (Primary — your Private Integration bearer token) |
| | `HIGHLEVEL_LOCATION_ID` (your sub-account Location ID) |
| **Network access** | HTTPS to `services.leadconnectorhq.com` only |

**Base URL**: `https://services.leadconnectorhq.com`
**Required Headers**: `Authorization: Bearer $HIGHLEVEL_TOKEN` + `Version: 2021-07-28`
**Rate Limits**: 100 requests/10 seconds burst, 200K/day per location

## Security Design

All API functions use pre-defined endpoint paths — there is no arbitrary HTTP request capability. Every user-supplied ID is validated against a strict alphanumeric regex (`^[a-zA-Z0-9_-]{1,128}$`) before being included in any URL path, preventing path traversal and injection. The scripts use only Python's built-in `urllib.request` for all network calls. No shell commands, no external binaries, no file writes outside of stdout.

## Setup — `/highlevel-setup`

If the user says "set up highlevel", "connect my GHL", or `/highlevel-setup`, run the setup wizard:

```bash
python3 scripts/setup-wizard.py
```

The wizard automatically: checks environment variables → guides Private Integration creation → tests the connection → pulls first 5 contacts as a quick win.

### Manual Setup (if wizard can't run)

#### Step 1: Create a Private Integration (NOT the old API Keys method)
1. Log into **app.gohighlevel.com**
2. Switch to your **Sub-Account** (recommended for single-location use)
3. Click **Settings** (bottom-left gear icon)
4. Select **Private Integrations** in the left sidebar
   - If not visible, enable it first: Settings → Labs → toggle Private Integrations ON
5. Click **"Create new Integration"**
6. Enter a name (e.g., "Claude AI Assistant") and description
7. **Grant only the scopes you need** (least-privilege recommended):

   | Use case | Recommended scopes |
   |----------|--------------------|
   | Contact management only | `contacts.readonly`, `contacts.write` |
   | Contacts + messaging | Above + `conversations.readonly`, `conversations.write`, `conversations/message.write` |
   | Full CRM (contacts, calendar, pipeline) | Above + `calendars.readonly`, `calendars.write`, `opportunities.readonly`, `opportunities.write` |
   | Adding workflows & invoices | Above + `workflows.readonly`, `invoices.readonly`, `invoices.write` |
   | Read-only reporting | `contacts.readonly`, `opportunities.readonly`, `calendars.readonly`, `invoices.readonly`, `locations.readonly` |

   You can always add more scopes later in Settings → Private Integrations → Edit without regenerating the token.

8. Click Create → **Copy the token IMMEDIATELY** — it is shown only once and cannot be retrieved later

#### Agency vs Sub-Account Integrations

| Feature | Agency Integration | Sub-Account Integration |
|---------|-------------------|------------------------|
| Created at | Agency Settings → Private Integrations | Sub-Account Settings → Private Integrations |
| Access scope | Agency + all sub-accounts (pass `locationId`) | Single location only |
| Available scopes | All scopes including `locations.write`, `oauth.*`, `saas.*`, `snapshots.*`, `companies.readonly` | Sub-account scopes only |
| Best for | Multi-location management, SaaS configurator | Single client integrations (recommended default) |

> **Recommendation:** Start with a Sub-Account integration and the minimum scopes you need. You can upgrade to Agency-level later if you need multi-location access.

### Step 2: Get Your Location ID
1. While in the sub-account, go to **Settings** → **Business Info** (or **Business Profile**)
2. The **Location ID** is displayed in the General Information section
3. Alternative: check the URL bar — it's the ID after `/location/` in `app.gohighlevel.com/v2/location/{LOCATION_ID}/...`

### Step 3: Set Environment Variables
```bash
export HIGHLEVEL_TOKEN="your-private-integration-token"
export HIGHLEVEL_LOCATION_ID="your-location-id"
```

### Step 4: Test Connection
Run `python3 scripts/ghl-api.py test_connection` — should return location name and status.

After successful setup, pull 5 contacts as a quick win to confirm everything works.

## Helper Script

`scripts/ghl-api.py` — Executable Python script (stdlib only) with built-in retry logic, pagination, input validation, and error handling.

**Core Commands:**
| Command | Description |
|---------|-------------|
| `test_connection` | Verify token + location ID work |
| `search_contacts [query]` | Search by name, email, or phone |
| `get_contact [id]` | Get full contact details |
| `create_contact [json]` | Create new contact |
| `update_contact [id] [json]` | Update contact fields |
| `list_opportunities` | List pipeline opportunities |
| `list_conversations` | List recent conversations |
| `send_message [contactId] [message]` | Send SMS/email |
| `list_calendars` | List all calendars |
| `get_free_slots [calendarId] [startDate] [endDate]` | Available booking slots |
| `list_workflows` | List all workflows |
| `add_to_workflow [contactId] [workflowId]` | Enroll contact in workflow |
| `list_invoices` | List invoices |
| `list_products` | List products |
| `list_forms` | List forms |
| `list_campaigns` | List campaigns |
| `get_location_details` | Get location info |
| `list_location_tags` | List location tags |
| `list_courses` | List courses/memberships |

All functions are safe, pre-defined endpoints. No arbitrary request capability.

## Complete API v2 Coverage (39 Endpoint Groups)

The skill provides safe, specific functions for all major GHL operations. Each function maps to a specific, allowed API endpoint with validated parameters.

| # | Group | Base Path | Key Operations | Scope Prefix |
|---|-------|-----------|----------------|-------------|
| 1 | **Contacts** | `/contacts/` | CRUD, search, upsert, tags, notes, tasks, bulk ops | `contacts` |
| 2 | **Conversations** | `/conversations/` | Search, messages (SMS/email/WhatsApp/FB/IG/chat), recordings | `conversations` |
| 3 | **Calendars** | `/calendars/` | CRUD, free slots, groups, resources, appointments | `calendars` |
| 4 | **Opportunities** | `/opportunities/` | CRUD, search, pipelines, stages, status, followers | `opportunities` |
| 5 | **Workflows** | `/workflows/` | List workflows, enroll/remove contacts | `workflows` |
| 6 | **Campaigns** | `/campaigns/` | List campaigns (read-only) | `campaigns` |
| 7 | **Invoices** | `/invoices/` | CRUD, send, void, record payment, Text2Pay, schedules, estimates | `invoices` |
| 8 | **Payments** | `/payments/` | Orders, transactions, subscriptions, coupons, providers | `payments` |
| 9 | **Products** | `/products/` | CRUD, prices, collections, reviews, store stats | `products` |
| 10 | **Locations** | `/locations/` | Get/update location, custom fields, custom values, tags, templates | `locations` |
|    | | | **Custom Fields CRUD:** | |
|    | | | `GET /locations/{id}/customFields` — List | |
|    | | | `POST /locations/{id}/customFields` — Create | |
|    | | | `PUT /locations/{id}/customFields/{fid}` — Update | |
|    | | | `DELETE /locations/{id}/customFields/{fid}` — Delete | |
|    | | | **Custom Values CRUD:** | |
|    | | | `GET /locations/{id}/customValues` — List | |
|    | | | `POST /locations/{id}/customValues` — Create | |
|    | | | `PUT /locations/{id}/customValues/{vid}` — Update | |
|    | | | `DELETE /locations/{id}/customValues/{vid}` — Delete | |
|    | | | **Tags CRUD:** | |
|    | | | `GET /locations/{id}/tags` — List | |
|    | | | `POST /locations/{id}/tags` — Create | |
|    | | | `PUT /locations/{id}/tags/{tid}` — Update | |
|    | | | `DELETE /locations/{id}/tags/{tid}` — Delete | |
| 11 | **Users** | `/users/` | CRUD, filter by email/role | `users` |
| 12 | **Forms** | `/forms/` | List forms, get submissions | `forms` |
| 13 | **Surveys** | `/surveys/` | List surveys, get submissions | `surveys` |
| 14 | **Funnels** | `/funnels/` | List funnels, pages, redirects | `funnels` |
| 15 | **Social Planner** | `/social-media-posting/` | Posts CRUD, accounts, CSV import, categories, stats | `socialplanner` |
| 16 | **Blogs** | `/blogs/` | Create/update posts, categories, authors | `blogs` |
| 17 | **Email** | `/emails/` | Templates CRUD, scheduled emails | `emails` |
| 18 | **Media** | `/medias/` | Upload, list, delete files | `medias` |
| 19 | **Trigger Links** | `/links/` | CRUD trigger links | `links` |
| 20 | **Businesses** | `/businesses/` | CRUD businesses | `businesses` |
| 21 | **Companies** | `/companies/` | Get company details (Agency) | `companies` |
| 22 | **Custom Objects** | `/objects/` | Schema CRUD, record CRUD | `objects` |
| 23 | **Associations** | `/associations/` | CRUD associations and relations | `associations` |
| 24 | **Proposals/Docs** | `/proposals/` | Documents, contracts, templates | `documents_contracts` |
| 25 | **Snapshots** | `/snapshots/` | List, status, share links (Agency) | `snapshots` |
| 26 | **SaaS** | `/saas/` | Subscription mgmt, plans, bulk ops (Agency $497) | `saas` |
| 27 | **Courses** | `/courses/` | Import courses/memberships | `courses` |
| 28 | **Voice AI** | `/voice-ai/` | Call logs, agent CRUD, actions, goals | `voice-ai` |
| 29 | **Phone System** | `/phone-system/` | Phone numbers, number pools | `phonenumbers` |
| 30 | **Custom Menus** | `/custom-menus/` | CRUD custom menu links (Agency) | `custom-menu-link` |
| 31 | **OAuth** | `/oauth/` | Token exchange, installed locations | `oauth` |
| 32 | **Marketplace** | `/marketplace/` | Installations, billing, charges | `marketplace` |
| 33 | **Conversation AI** | `/conversation-ai/` | AI chatbot configuration | — |
| 34 | **Knowledge Base** | `/knowledge-base/` | Knowledge base for AI features | — |
| 35 | **AI Agent Studio** | `/agent-studio/` | Custom AI agent CRUD | — |
| 36 | **Brand Boards** | `/brand-boards/` | Brand board management | — |
| 37 | **Store** | `/store/` | E-commerce store management | — |
| 38 | **LC Email** | `/lc-email/` | Email infrastructure (ISV) | — |
| 39 | **Custom Fields** | `/locations/:id/customFields/` | Custom field CRUD | `locations/customFields` |

## Reference Docs (load on demand)

For detailed endpoint paths, parameters, and examples for each group:

- `references/contacts.md` — Contact CRUD, search, tags, notes, tasks, bulk operations
- `references/conversations.md` — Messaging across all channels, recordings, transcriptions
- `references/calendars.md` — Calendar CRUD, free slots, appointments, groups, resources
- `references/opportunities.md` — Pipeline management, stages, status updates
- `references/invoices-payments.md` — Invoices, payments, orders, subscriptions, products
- `references/locations-users.md` — Location settings, custom fields/values, users, tags
- `references/social-media.md` — Social planner posts, accounts, OAuth connections
- `references/forms-surveys-funnels.md` — Forms, surveys, funnels, trigger links
- `references/advanced.md` — Custom objects, associations, snapshots, SaaS, Voice AI, blogs, courses
- `references/troubleshooting.md` — Common errors, rate limits, token rotation, debugging

## Important Notes

- **Private Integrations are required** — the old Settings → API Keys method is deprecated/EOL
- **Token rotation**: Tokens don't auto-expire but GHL recommends 90-day rotation. Unused tokens auto-expire after 90 days inactivity
  - **"Rotate and expire later"** — new token generated, old token stays active for 7-day grace period
  - **"Rotate and expire now"** — old token invalidated immediately (use for compromised credentials)
  - You can edit scopes without regenerating the token
- **OAuth tokens** (marketplace apps only): Access tokens expire in 24 hours (86,399s); refresh tokens last up to 1 year
- Agency tokens can access sub-account data by passing `locationId` parameter
- **Rate limits are per-resource** — each sub-account independently gets 100/10s burst + 200K/day. SaaS endpoints: 10 req/sec global
- All list endpoints default to 20 records, max 100 per page via `limit` param
- Use cursor pagination with `startAfter` / `startAfterId` for large datasets
- Monitor rate limits via response headers: `X-RateLimit-Limit-Daily`, `X-RateLimit-Daily-Remaining`, `X-RateLimit-Max`, `X-RateLimit-Remaining`, `X-RateLimit-Interval-Milliseconds`
- **$497 Agency Pro plan** required for: SaaS Configurator, Snapshots, full agency management APIs

## Webhook Events

50+ webhook event types for real-time notifications. Key events: `ContactCreate`, `ContactDelete`, `ContactTagUpdate`, `InboundMessage`, `OutboundMessage`, `OpportunityCreate`, `OpportunityStageUpdate`, `OpportunityStatusUpdate`, appointment events, payment events, form submission events. Webhooks continue firing even if access token expires. Config is per marketplace app.
Docs: https://marketplace.gohighlevel.com/docs/webhook/WebhookIntegrationGuide

## Official SDKs & Developer Resources

- **Node.js**: `@gohighlevel/api-client` (npm) — supports `privateIntegrationToken` config, auto 401 retry
- **Python**: `gohighlevel-api-client` (PyPI) — session storage, auto token refresh, webhook middleware
- **PHP SDK** also available
- All SDKs use `apiVersion: '2021-07-28'`
- **OpenAPI Specs**: https://github.com/GoHighLevel/highlevel-api-docs
- **API Docs**: https://marketplace.gohighlevel.com/docs/
- **Developer Slack**: https://developers.gohighlevel.com/join-dev-community

---

### Built by Ty Shane

[🌐 LaunchMyOpenClaw.com](https://launchmyopenclaw.com) • [🌐 MyFBLeads.com](https://myfbleads.com)
[▶️ YouTube @10xcoldleads](https://youtube.com/@10xcoldleads) • [📘 Facebook](https://facebook.com/ty.shane.howell.2025) • [💼 LinkedIn](https://linkedin.com/in/ty-shane/)
📧 ty@10xcoldleads.com

**No GoHighLevel account yet?** → [Start the free 5-Day AI Employee Challenge](https://gohighlevel.com/5-day-challenge?fp_ref=369ai)
