---
name: n8n-automation
description: Trigger and manage n8n workflows via webhooks and API. Use when you need to automate complex multi-step processes, integrate external services, or trigger workflows programmatically. Works with self-hosted n8n (free) or n8n cloud.
---

# n8n Automation

Trigger and manage n8n workflows programmatically.

## When to Use

- Need to trigger automated workflows from scripts/agents
- Want to integrate multiple services (Slack, Email, Database, etc.)
- Building complex automation pipelines
- Need visual workflow builder with code capabilities

## Prerequisites

**Self-hosted n8n (Free):**
```bash
# Docker (recommended)
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n

# Or npm
npm install n8n -g
n8n start
```

Access at: http://localhost:5678

## Quick Start

### 1. Create a Webhook Workflow in n8n

1. Open n8n UI
2. Add "Webhook" trigger node
3. Set HTTP Method to POST
4. Copy the webhook URL (e.g., `http://localhost:5678/webhook/xxx`)
5. Add your automation nodes
6. Activate the workflow

### 2. Trigger from Command Line

```bash
# Simple trigger
curl -X POST http://localhost:5678/webhook/your-webhook-id

# With JSON data
curl -X POST http://localhost:5678/webhook/your-webhook-id \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello", "user": "agent"}'
```

### 3. Trigger from Script

```bash
node skills/n8n-automation/scripts/trigger-webhook.js \
  --url "http://localhost:5678/webhook/xxx" \
  --data '{"key": "value"}'
```

## Available Scripts

### `scripts/trigger-webhook.js`
Trigger any n8n webhook with optional JSON payload.

### `scripts/n8n-api.js`
Full n8n API client for managing workflows programmatically.

### `scripts/workflow-templates/`
Ready-to-import workflow templates for common tasks.

## API Usage (Advanced)

n8n has a REST API for workflow management:

```bash
# List all workflows
curl http://localhost:5678/api/v1/workflows \
  -H "X-N8N-API-KEY: your-api-key"

# Execute a workflow
curl -X POST http://localhost:5678/api/v1/workflows/{id}/execute \
  -H "X-N8N-API-KEY: your-api-key"
```

## Common Workflow Patterns

### 1. Notification Pipeline
```
Webhook → IF (condition) → Slack/Email/Telegram
```

### 2. Data Processing
```
Webhook → HTTP Request → Transform → Database
```

### 3. Scheduled + Manual Trigger
```
Schedule/Webhook → Merge → Process → Output
```

### 4. Error Handling
```
Webhook → Try/Catch → On Error: Send Alert
```

## Tips

1. **Test webhooks**: Use "Test workflow" before activating
2. **Secure webhooks**: Add authentication header checks
3. **Use environment variables**: Store API keys in n8n settings
4. **Monitor executions**: Check execution history for debugging

## Self-Hosted vs Cloud

| Feature | Self-Hosted | Cloud |
|---------|-------------|-------|
| Cost | Free | $20+/mo |
| Setup | Manual | Instant |
| Data | Your server | n8n servers |
| Custom nodes | Yes | Limited |

## Limitations

- Webhook URLs change if workflow is recreated
- Self-hosted requires always-on server
- Complex workflows need learning curve
