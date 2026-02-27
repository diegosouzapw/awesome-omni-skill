---
name: workflow-builder
description: Design automation workflows and pipelines. Use when creating CI/CD, task automation, or process flows.
allowed-tools: Write, Bash
---

# Workflow Builder

**Version**: 1.0.0
**Purpose**: Design and implement automation workflows

---

## Workflow

### Step 1: Map Process

```
Input → Step 1 → Step 2 → ... → Output
```

### Step 2: Identify Triggers

| Trigger Type | Example |
|--------------|---------|
| Time | Every day at 9:00 |
| Event | File created |
| Manual | User command |
| Webhook | API call |

### Step 3: Design Steps

```yaml
workflow:
  name: "Daily Report"
  trigger: schedule("0 9 * * *")
  steps:
    - name: Gather Data
      action: fetch_metrics
    - name: Generate Report
      action: create_markdown
    - name: Send Notification
      action: send_slack
```

### Step 4: Implement

Choose platform:
- GitHub Actions
- Tasker (Android)
- Shortcuts (iOS)
- cron + shell

---

## Output

Provide:
1. Workflow diagram (Mermaid)
2. Implementation code
3. Setup instructions
