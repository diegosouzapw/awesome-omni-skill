---
name: n8n-builder
description: Expert n8n workflow builder that creates, deploys, and manages n8n workflows programmatically via the n8n REST API. Use when asked to create n8n workflows, automate n8n tasks, build automations, design workflow pipelines, connect services via n8n, or manage existing n8n workflows. Handles webhook flows, scheduled tasks, AI agents, database syncs, conditional logic, error handling, and any n8n node configuration.
---

# n8n Workflow Builder

## Setup

Requires two environment variables:
- `N8N_URL` — n8n instance URL (e.g. `https://your-n8n.example.com`)
- `N8N_API_KEY` — n8n API key (Settings → API → Create API Key)

## Workflow

1. **Understand the automation** — Clarify trigger (webhook/schedule/manual), data sources, processing logic, outputs, and error handling needs.

2. **Design the workflow JSON** — Build valid n8n workflow JSON following the schema in `references/workflow-schema.md`. Use patterns from `references/workflow-patterns.md` as templates.

3. **Deploy via API** — Use `scripts/n8n-api.sh create <file>` or pipe JSON to `scripts/n8n-api.sh create-stdin`.

4. **Activate** — Use `scripts/n8n-api.sh activate <workflow_id>` for trigger-based workflows.

5. **Verify** — List workflows to confirm deployment: `scripts/n8n-api.sh list`.

## API Script Reference

```bash
# List all workflows
scripts/n8n-api.sh list

# Create workflow from JSON file
scripts/n8n-api.sh create /tmp/workflow.json

# Create from stdin
echo '{"name":"Test",...}' | scripts/n8n-api.sh create-stdin

# Get, activate, deactivate, delete, execute
scripts/n8n-api.sh get <id>
scripts/n8n-api.sh activate <id>
scripts/n8n-api.sh deactivate <id>
scripts/n8n-api.sh delete <id>
scripts/n8n-api.sh execute <id>

# List credentials and tags
scripts/n8n-api.sh credentials
scripts/n8n-api.sh tags
```

## Building Workflow JSON

Every workflow needs: `name`, `nodes[]`, `connections{}`, `settings{}`.

Every node needs: `id`, `name`, `type`, `typeVersion`, `position`, `parameters`.

Connections use **source node display name** as key, mapping outputs to target nodes.

For full schema, node types, and expression syntax → read `references/workflow-schema.md`
For complete workflow examples (webhook, schedule, AI agent, DB sync, error handling) → read `references/workflow-patterns.md`

## Key Rules

- **Always set `"executionOrder": "v1"`** in settings
- **Node names must be unique** within a workflow
- **Node IDs must be unique** — use descriptive slugs like `webhook1`, `code1`
- **Position nodes** starting at `[250, 300]`, spacing ~200px horizontally
- **IF nodes** have two outputs: index 0 = true, index 1 = false
- **Webhook workflows** need `respondToWebhook` node if `responseMode` is `responseNode`
- **Credentials** must exist in n8n before activation — check with `scripts/n8n-api.sh credentials`
- **Test before activating** — use `scripts/n8n-api.sh execute <id>` for manual trigger workflows
- **Use `continueOnFail: true`** on risky HTTP/API nodes, then check for errors downstream

## Common Real Estate Workflows

- **Lead intake**: Webhook → validate → dedupe → insert DB → notify Slack/SMS
- **Call follow-up**: Schedule → query DB for completed calls → send SMS/email based on outcome
- **Drip campaign**: Schedule → query leads by stage → send stage-appropriate email/SMS
- **CRM sync**: Webhook → transform → update HubSpot/Salesforce + internal DB
- **Property alerts**: Schedule → scrape/API listings → filter new → notify leads
- **AI qualification**: Webhook → AI Agent (classify lead intent) → route to appropriate pipeline
