---
name: marketing_automation
description: Automated multi-step marketing campaigns with email, SMS, and server action workflows triggered by audience behavior.
metadata:
  author: Claude Agent
  version: "19.0"
  source: "Odoo 19.0 Official Documentation"
  extracted: "2026-02-18"
---

# marketing_automation — Odoo 19.0 Skill Reference

## Overview

The Marketing Automation application enables users to build automated campaign workflows that execute sequences of emails, SMS messages, and server actions based on predefined triggers, durations, and audience filters. Campaigns target specific record models (Lead/Opportunity, Contact, Mailing Contact, Event Registration, etc.) and react to participant behavior (opened, clicked, replied, bounced) to branch into child activities. Used by marketing teams to automate drip campaigns, lead nurturing, double opt-in flows, and internal CRM actions.

## Key Concepts

- **Campaign**: A workflow of activities executed against a target audience defined by model and filter rules. States: Draft, Running, Stopped.
- **Target**: The Odoo model the campaign operates on (e.g., Lead/Opportunity, Mailing Contact, Contact, Event Registration).
- **Filter / Domain**: Configurable rules that narrow the target audience. Supports "Match all" / "Match any" logic with nested conditions.
- **Unicity based on**: Field used to deduplicate records (typically Email).
- **Record**: A database entry matching the campaign's target + filter criteria.
- **Participant**: A record that has been engaged by the campaign (entered the workflow).
- **Activity**: A single step in the workflow — Email, SMS, or Server Action.
- **Child Activity**: An activity triggered by a parent activity's outcome (opened, not opened, clicked, replied, bounced, etc.).
- **Trigger**: Defines when an activity executes — at the beginning of the workflow, or based on a parent activity's event.
- **Trigger Type**: The specific event that fires the trigger (e.g., Mail: opened, Mail: not clicked, SMS: bounced, beginning of workflow).
- **Expiry Duration**: Optional time limit after which a pending activity is cancelled.
- **Activity Filter**: Additional domain rules applied to a specific activity (and its children), narrowing the campaign's overall audience.
- **Applied Filter**: Read-only combined view of campaign filter + activity filter.
- **Campaign Template**: Pre-built campaign configurations (6 templates: Tag Hot Contacts, Welcome Flow, Double Opt-in, Commercial Prospection, Schedule Calls, Prioritize Hot Leads).
- **Double Opt-in**: Campaign template that sends a confirmation email to new mailing list contacts; clicking confirms consent and adds them to a "Confirmed contacts" list via a server action.
- **Trace**: A record of an activity execution for a participant (Processed, Scheduled, Rejected).

## Core Workflows

### 1. Create a Campaign from Scratch

1. Navigate to Marketing Automation app, click **New**.
2. Set the **Target** model (e.g., Lead/Opportunity).
3. Set **Unicity based on** (typically Email).
4. Configure **Filter** rules to define the audience. Check `# record(s)` count.
5. Optionally enable **Include archived** to include archived records.
6. Click **Add new activity** in the Workflow section.
7. Configure the activity: Activity Name, Activity Type (Email/SMS/Server Action), mail/SMS template or server action.
8. Set **Trigger**: interval number + unit (Hours/Days/Weeks/Months) + trigger type (beginning of workflow).
9. Optionally set **Expiry Duration** and **Activity Filter**.
10. Click **Save & Close**.
11. Add child activities by hovering over **Add child activity** beneath a parent, selecting a trigger type (Opened, Not Opened, Clicked, Replied, Bounced, etc.).

### 2. Use a Campaign Template

1. Open Marketing Automation app — template cards display when no campaigns exist.
2. Click a template card (e.g., Double Opt-in).
3. Review pre-configured Target, Filter, and Workflow activities.
4. Customize email templates via the Templates smart button.
5. Test and start the campaign.

### 3. Test a Campaign

1. Open the campaign, click **Launch a Test**.
2. Select or create a test contact in the popup.
3. Click **Launch** — the test page shows the workflow.
4. Click the **Play** button beside each activity to execute it.
5. Monitor results in real-time. Child activities appear indented after parent execution.
6. Status moves to **Completed** when all activities finish. Click **Stop** to end early.

### 4. Run and Stop a Campaign

1. On the campaign form, click **Start** to launch to the full target audience.
2. Status changes to **Running**. Smart buttons populate: Templates, Clicks, Tests, Participants.
3. Activity blocks display real-time metrics: Success/Rejected counts, Sent, Clicked%, Replied%, Bounced%.
4. To stop, click **Stop** — status changes to **Stopped**.
5. Restarting after modifications prompts an **Update** warning; existing participants may re-enter the workflow.

### 5. Analyze Campaign Metrics

1. View activity-level metrics in each workflow block (Graph tab for line chart, Filter tab for domain details).
2. Click stats on the DETAILS line (Sent, Clicked, Replied, Bounced) to see individual records.
3. Navigate to Reporting > **Link Tracker** for URL click analytics.
4. Navigate to Reporting > **Traces** for activity execution history (Processed, Scheduled, Rejected).
5. Navigate to Reporting > **Participants** for participant overview across campaigns.

## Technical Reference

### Models

| Model | Description |
|-------|-------------|
| `marketing.campaign` | Campaign record |
| `marketing.activity` | Workflow activity |
| `marketing.participant` | Campaign participant |
| `marketing.trace` | Activity execution trace |

### Key Fields (marketing.campaign)

- `name` — Campaign name
- `model_id` — Target model
- `unique_field_id` — Unicity field (for deduplication)
- `domain` — Filter domain
- `state` — draft, running, stopped
- `include_archived` — Include archived records

### Key Fields (marketing.activity)

- `name` — Activity name
- `activity_type` — email, action, sms
- `parent_id` — Parent activity (for child activities)
- `trigger_type` — beginning, act, mail_open, mail_not_open, mail_reply, mail_not_reply, mail_click, mail_not_click, mail_bounce, sms_click, sms_not_click, sms_bounce
- `interval_number` — Trigger delay number
- `interval_type` — hours, days, weeks, months
- `validity_duration` — Expiry enabled flag
- `validity_duration_number` — Expiry delay number
- `validity_duration_type` — Expiry delay unit
- `domain` — Activity-specific filter

### Activity Types

| Type | Template Field | Trigger Types |
|------|---------------|---------------|
| Email | `mail_template_id` | Opened, Not Opened, Replied, Not Replied, Clicked, Not Clicked, Bounced |
| Server Action | `server_action_id` | Add Another Activity only |
| SMS | `sms_template_id` | Clicked, Not Clicked, Bounced |

### Server Action Types

Update Record, Create Activity, Send Email, Send SMS, Add/Remove Followers, Create Record, Execute Code, Send Webhook Notification, Execute Existing Actions.

### Key Menu Paths

- `Marketing Automation > Campaigns` — Campaign dashboard
- `Marketing Automation > Configuration > Favorite Filters` — Saved filter management
- `Marketing Automation > Reporting > Link Tracker` — URL click analytics
- `Marketing Automation > Reporting > Traces` — Activity execution logs
- `Marketing Automation > Reporting > Participants` — Participant analytics

## API / RPC Patterns

<!-- TODO: not found in docs -->

## Version Notes (19.0)

<!-- TODO: not found in docs — no explicit 18.0 vs 19.0 breaking changes documented -->

## Common Pitfalls

- **Installing Marketing Automation also installs Email Marketing**: It is a dependency. Additionally, install CRM and SMS Marketing for full feature access.
- **Changing the Target model invalidates existing activities**: Never modify the Target field after activities have been added to the workflow.
- **Test campaigns should be run on production databases**: Duplicate/trial databases have limited email sending capabilities; tests may not send properly.
- **Restarting a stopped campaign can re-engage previous participants**: After modifications, clicking Start again prompts a warning. Participants who completed the original workflow may be reintroduced.
- **Double Opt-in: any link click triggers the server action**: The child "Add to list" server action cannot distinguish between different URLs in the email (except `/unsubscribe_from_list`). Include only one call-to-action link besides unsubscribe.
