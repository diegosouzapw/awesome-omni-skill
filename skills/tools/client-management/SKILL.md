---
name: client-management
description: Manage client relationships using the local CRM — add contacts, track deals through the pipeline, schedule follow-ups, log interactions, and generate reports. All data stored locally in SQLite.
alwaysApply: false
---

# Client Management

You have access to the local-crm MCP tools for managing contacts, deals, follow-ups, and interactions. Use these to help the user maintain their business relationships effectively.

## Available Tools

### Contacts
- **crm_create_contact** — Add a new contact (name, email, phone, company, role, tags, notes)
- **crm_search_contacts** — Find contacts by keyword (searches name, email, company, tags, notes)
- **crm_get_contact** — Get a contact's full details by ID
- **crm_list_contacts** — Browse contacts, optionally filtered by company or tag
- **crm_update_contact** — Update a contact's details
- **crm_delete_contact** — Permanently delete a contact and all related data

### Deals (Pipeline)
- **crm_create_deal** — Create a deal linked to a contact (title, amount, stage)
- **crm_move_deal** — Move a deal to a different pipeline stage
- **crm_list_deals** — List deals, optionally filtered by stage or contact

Pipeline stages in order: **lead** → **qualified** → **proposal** → **negotiation** → **won** / **lost**

### Interactions
- **crm_log_interaction** — Record a call, email, meeting, note, or message with a contact
- **crm_list_interactions** — View interaction history for a contact or deal

### Follow-Ups
- **crm_schedule_follow_up** — Schedule a follow-up with a due date
- **crm_complete_follow_up** — Mark a follow-up as done
- **crm_list_follow_ups** — View pending, overdue, or completed follow-ups

### Reports
- **crm_pipeline_report** — Pipeline overview with deals by stage and values
- **crm_dashboard** — High-level CRM dashboard with key metrics

## Workflow Patterns

### New Client Onboarding
1. `crm_create_contact` — Add the client with company, role, and tags
2. `crm_create_deal` — Create a deal if there's a business opportunity
3. `crm_log_interaction` — Log the initial meeting or call
4. `crm_schedule_follow_up` — Set a reminder for next steps

### Daily CRM Check
1. `crm_dashboard` — Quick overview of key metrics
2. `crm_list_follow_ups` with overdue=true — Check for missed follow-ups
3. `crm_list_follow_ups` with status=pending — Review upcoming tasks
4. Act on follow-ups → log interactions → schedule new follow-ups

### Pipeline Review
1. `crm_pipeline_report` — See deals by stage with total values
2. `crm_list_deals` with specific stages — Drill into stuck deals
3. `crm_move_deal` — Update stages as deals progress

### Finding a Contact
1. `crm_search_contacts` — Search by name, email, company, or tags
2. `crm_get_contact` — Get full details by ID
3. `crm_list_interactions` — Review interaction history
4. `crm_list_deals` with contact_id — See their deals

## Guidelines

- **Always confirm before destructive operations** — deletion is permanent
- **Use tags for categorization** — e.g. "client,vip,tech" or "lead,startup,ai"
- **Log interactions consistently** — the more data you track, the more useful the CRM becomes
- **Check follow-ups regularly** — overdue follow-ups mean missed opportunities
- **All data is local** — stored in SQLite at the plugin's data directory. No cloud sync.
