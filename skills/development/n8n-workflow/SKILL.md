---
name: n8n-workflow
description: Create, modify, and understand n8n automation workflows. Use when building n8n workflow JSON files, configuring nodes (HTTP Request, Code, IF, Merge, Webhook, Schedule), writing expressions with {{ $json }}, or implementing flow logic (conditionals, loops, error handling). Triggers for requests involving n8n, workflow automation, or node-based pipeline creation.
---

# n8n Workflow Creator

This skill provides guidance for creating valid n8n workflow JSON files.

## 🚨 Critical: Webhook Data Structure

**Most common mistake**: Webhook data is nested under `.body`, NOT at root!

```javascript
// ❌ WRONG - Returns undefined
{{ $json.email }}

// ✅ CORRECT - Webhook data is under .body
{{ $json.body.email }}
```

This applies to expressions AND Code nodes.

## Workflow Structure

```json
{
  "name": "Workflow Name",
  "nodes": [...],
  "connections": {...},
  "active": false,
  "settings": { "executionOrder": "v1" }
}
```

## The 5 Core Patterns

1. **Webhook Processing** - Webhook → Validate → Transform → Respond
2. **HTTP API Integration** - Trigger → HTTP Request → Transform → Action
3. **Database Operations** - Schedule → Query → Transform → Write → Verify
4. **AI Agent Workflow** - Trigger → AI Agent (Model + Tools) → Output
5. **Scheduled Tasks** - Schedule → Fetch → Process → Deliver → Log

See [patterns.md](references/patterns.md) for complete examples.

## Essential Nodes

| Node | Type | Use Case |
|------|------|----------|
| Manual Trigger | `n8n-nodes-base.manualTrigger` | Test execution |
| Schedule | `n8n-nodes-base.scheduleTrigger` | Cron-based runs |
| Webhook | `n8n-nodes-base.webhook` | HTTP endpoints |
| HTTP Request | `n8n-nodes-base.httpRequest` | API calls |
| Code | `n8n-nodes-base.code` | JavaScript/Python |
| Set | `n8n-nodes-base.set` | Modify/create fields |
| IF | `n8n-nodes-base.if` | Conditional branching |
| Merge | `n8n-nodes-base.merge` | Combine branches |
| Loop Over Items | `n8n-nodes-base.splitInBatches` | Batch processing |

See [nodes.md](references/nodes.md) for full configurations.

## Expression Syntax

Expressions use `{{ }}` syntax:

```javascript
{{ $json.fieldName }}              // Current item
{{ $json.body.email }}             // Webhook data (under .body!)
{{ $('NodeName').item.json.field }} // Other node's output
{{ $now }}                         // Current timestamp
```

**❌ Don't use `{{ }}` in:**
- Code nodes (use JavaScript directly)
- Webhook paths
- Credential fields

See [expressions.md](references/expressions.md) for advanced patterns.

## Code Node - Critical Rules

**ALWAYS return array with `json` property:**

```javascript
// ✅ CORRECT
const items = $input.all();
return items.map(item => ({ json: { ...item.json, processed: true } }));

// ✅ CORRECT - Single item
return [{ json: { result: 'success' } }];

// ❌ WRONG - No return
const data = $input.first();
// forgot return!

// ❌ WRONG - Object instead of array
return { json: { result: 'success' } };
```

**Best practices:**
- Validate input: `if (!items || items.length === 0) return [];`
- Use null checks: `item.json?.user?.email || 'default'`
- Try-catch for API calls
- Filter early, process late

## Common Gotchas

| Problem | Solution |
|---------|----------|
| Can't access webhook data | Use `$json.body.field`, not `$json.field` |
| Expression shows as text | Wrap in `{{ }}` |
| Unexpected node order | Check Settings → Execution Order (use v1) |
| Code node returns nothing | Add `return` statement |
| API returns 401/403 | Use Credentials section, not parameters |

## Connections Format

```json
"connections": {
  "Source Node": {
    "main": [[{ "node": "Target Node", "type": "main", "index": 0 }]]
  }
}
```

**IF node outputs:** `index: 0` = True, `index: 1` = False

## Best Practices

**✅ Do:**
- Use descriptive node names ("Fetch Users", not "HTTP Request 1")
- Set `onError: "continueRegularOutput"` for resilience
- Test incrementally, node by node
- Document complex workflows with notes
- Handle empty data cases

**❌ Don't:**
- Build workflows in one shot (iterate!)
- Skip error handling
- Hardcode credentials in parameters
- Use Code node when built-in nodes suffice
- Deploy without testing

## Reference Documentation

- [nodes.md](references/nodes.md) - Node configurations
- [expressions.md](references/expressions.md) - Expression syntax
- [patterns.md](references/patterns.md) - Workflow patterns
- [json-structure.md](references/json-structure.md) - JSON schema
