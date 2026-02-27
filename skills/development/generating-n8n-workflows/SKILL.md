---
name: generating-n8n-workflows
description: Generates n8n workflow JSON files from user prompts for download and import. Use when user wants to create n8n automation, mentions workflow generation, or needs a .json file for n8n import.
---

# n8n Workflow Generator

Generates clean, minimal n8n workflow JSON files based on user requirements.

## When to use this skill

- User wants to create an n8n workflow
- User provides a prompt describing automation needs
- User needs a downloadable .json file for n8n import
- User mentions specific integrations (Slack, Email, HTTP, etc.)

## Workflow

- [ ] Parse user prompt for workflow requirements
- [ ] Identify trigger type and action nodes needed
- [ ] Generate minimal workflow JSON (no extra nodes)
- [ ] Validate JSON structure against n8n schema
- [ ] Provide download-ready .json file

## Instructions

### Step 1: Parse User Prompt

Extract from the user's description:
- **Trigger**: What starts the workflow? (webhook, schedule, manual, trigger-node)
- **Actions**: What should happen? (send email, HTTP request, database operation)
- **Integrations**: Which services? (Slack, Gmail, PostgreSQL, etc.)
- **Data flow**: How does data move between nodes?

### Step 2: Node Selection Rules

**CRITICAL: Generate ONLY necessary nodes. No extras.**

| User Needs | Minimal Nodes |
|------------|---------------|
| "Send Slack message daily" | Schedule Trigger → Slack |
| "HTTP webhook to save data" | Webhook → HTTP Request |
| "Email on form submit" | Webhook → Send Email |
| "Database backup weekly" | Schedule → Postgres → FTP |

**Forbidden extras:**
- No debug nodes
- No unnecessary Set nodes
- No extra HTTP requests
- No duplicate triggers
- No unused credentials placeholders

### Step 3: Generate Workflow JSON

Required structure:
```json
{
  "name": "Workflow Name",
  "nodes": [
    {
      "id": "uuid",
      "name": "Trigger",
      "type": "n8n-nodes-base.webhook",
      "position": [250, 300],
      "parameters": {}
    },
    {
      "id": "uuid",
      "name": "Action",
      "type": "n8n-nodes-base.slack",
      "position": [450, 300],
      "parameters": {}
    }
  ],
  "connections": {
    "Trigger": {
      "main": [[{"node": "Action", "type": "main", "index": 0}]]
    }
  }
}
```

### Step 4: Position Guidelines

- Start trigger at `[250, 300]`
- Each subsequent node: `+200` on X axis
- Keep Y at `300` for simple flows
- Use `[250, 200]` and `[250, 400]` for branches

### Step 5: Common Node Templates

**Webhook Trigger:**
```json
{
  "id": "{{uuid}}",
  "name": "Webhook",
  "type": "n8n-nodes-base.webhook",
  "typeVersion": 1,
  "position": [250, 300],
  "webhookId": "{{random-id}}",
  "parameters": {
    "httpMethod": "POST",
    "path": "{{unique-path}}",
    "responseMode": "responseNode"
  }
}
```

**HTTP Request:**
```json
{
  "id": "{{uuid}}",
  "name": "HTTP Request",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.1,
  "position": [450, 300],
  "parameters": {
    "method": "GET",
    "url": "",
    "sendBody": false
  }
}
```

**Slack:**
```json
{
  "id": "{{uuid}}",
  "name": "Slack",
  "type": "n8n-nodes-base.slack",
  "typeVersion": 2,
  "position": [450, 300],
  "parameters": {
    "operation": "post",
    "channel": "",
    "text": ""
  }
}
```

**Send Email:**
```json
{
  "id": "{{uuid}}",
  "name": "Send Email",
  "type": "n8n-nodes-base.emailSend",
  "typeVersion": 2,
  "position": [450, 300],
  "parameters": {
    "toEmail": "",
    "subject": "",
    "text": ""
  }
}
```

**Schedule Trigger:**
```json
{
  "id": "{{uuid}}",
  "name": "Schedule Trigger",
  "type": "n8n-nodes-base.scheduleTrigger",
  "typeVersion": 1.1,
  "position": [250, 300],
  "parameters": {
    "rule": {
      "interval": [
        {
          "field": "hours",
          "hoursInterval": 1
        }
      ]
    }
  }
}
```

**Code Node:**
```json
{
  "id": "{{uuid}}",
  "name": "Code",
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "position": [450, 300],
  "parameters": {
    "jsCode": "// Add your code here\nreturn items;"
  }
}
```

### Step 6: Connection Rules

Format connections array:
```json
{
  "connections": {
    "SourceNodeName": {
      "main": [
        [
          {
            "node": "TargetNodeName",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}
```

For multiple outputs, increment the inner array index.

## Scripts

Generate workflow using the helper:

```bash
python .agent/skills/generating-n8n-workflows/scripts/n8n_generator.py \
  --prompt "Send Slack message when webhook received" \
  --output workflow.json
```

## Examples

### Example 1: Simple Webhook to Slack

**User Prompt:** "When I send a webhook, post to Slack"

**Generated Nodes:** 2 (Webhook → Slack)

See: [examples/webhook-slack-example.json](examples/webhook-slack-example.json)

### Example 2: Daily Email Report

**User Prompt:** "Send daily email at 9 AM"

**Generated Nodes:** 2 (Schedule → Email)

See: [examples/schedule-email-example.json](examples/schedule-email-example.json)

### Example 3: HTTP API Call

**User Prompt:** "Call an API every hour and save response"

**Generated Nodes:** 2 (Schedule → HTTP Request)

See: [examples/schedule-http-example.json](examples/schedule-http-example.json)

## Validation Checklist

Before returning JSON:
- [ ] Only necessary nodes included
- [ ] All nodes have unique UUIDs
- [ ] All nodes have unique names
- [ ] Connections reference existing node names
- [ ] Positions are reasonable (no overlaps)
- [ ] JSON is valid (no trailing commas)
- [ ] No credential values hardcoded
- [ ] Workflow name is descriptive

## Resources

- [scripts/n8n_generator.py](scripts/n8n_generator.py) - Workflow generator script
- [examples/](examples/) - Sample workflow JSON files
- [Node Type Reference](https://docs.n8n.io/integrations/builtin/)
