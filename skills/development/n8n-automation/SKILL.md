---
name: n8n-automation
description: Master skill for building production-ready n8n workflows. Combines 8 specialized skills for template search, expressions, MCP tools, patterns, validation, node configuration, JavaScript and Python code. Use when creating, validating, or debugging n8n workflows.
---

# n8n Automation Master Skill

Unified skill that orchestrates 8 specialized n8n skills to build production-ready workflows.

---

## 🔧 Sub-Skills Included

| Skill | Purpose | When Activated |
|-------|---------|----------------|
| [n8n-template-search](n8n-template-search/SKILL.md) | Template discovery | Finding workflow templates, examples |
| [n8n-mcp-tools-expert](n8n-mcp-tools-expert/SKILL.md) | MCP tool usage | Searching nodes, templates, validating |
| [n8n-workflow-patterns](n8n-workflow-patterns/SKILL.md) | Architectural patterns | Creating new workflows |
| [n8n-expression-syntax](n8n-expression-syntax/SKILL.md) | Expression syntax | $json, $node references |
| [n8n-validation-expert](n8n-validation-expert/SKILL.md) | Error handling | Validation failures, debugging |
| [n8n-node-configuration](n8n-node-configuration/SKILL.md) | Node parameters | Configuring complex nodes |
| [n8n-code-javascript](n8n-code-javascript/SKILL.md) | JavaScript code | Code node JS scripting |
| [n8n-code-python](n8n-code-python/SKILL.md) | Python code | Code node Python scripting |

---

## 🎯 Quick Reference

### Template Search (ALWAYS START HERE!)

**MCP Template Library** (2,709+ templates):
- `search_templates` - Find by keyword, nodes, task, metadata
- `get_template` - Get template JSON for deployment

**Web Template Discovery** (for recent/community templates):
- Load `n8n-template-search` skill for comprehensive web search
- Combines n8n.io, GitHub, forum sources

### MCP Tools Available

**Always Available (no n8n API needed):**
- `search_nodes` - Find nodes among 1,084+ available
- `get_node` - Get node documentation and config
- `validate_node` - Check node configuration
- `validate_workflow` - Validate complete workflow
- `search_templates` - Search 2,709 templates
- `get_template` - Get workflow template JSON

**Requires n8n API Key:**
- `n8n_create_workflow` - Deploy workflow
- `n8n_update_partial_workflow` - Update workflow
- `n8n_validate_workflow` - Validate by ID
- `n8n_deploy_template` - Deploy template directly

### nodeType Format (CRITICAL!)

```javascript
// For search/validate tools: SHORT prefix
"nodes-base.slack"
"nodes-base.httpRequest"
"nodes-langchain.agent"

// For workflow tools: FULL prefix  
"n8n-nodes-base.slack"
"n8n-nodes-base.httpRequest"
"@n8n/n8n-nodes-langchain.agent"
```

---

## 📋 Workflow Creation Process

### Step 0: Template Discovery (ALWAYS FIRST!) 🔍

**CRITICAL**: Before building anything, search for existing templates!

```javascript
// Quick keyword search (MCP - fastest)
search_templates({query: "slack notification"})

// Search by specific nodes
search_templates({
  searchMode: "by_nodes",
  nodeTypes: ["n8n-nodes-base.webhook", "n8n-nodes-base.slack"]
})

// Search by workflow pattern
search_templates({searchMode: "by_task", task: "webhook_processing"})

// Search by complexity/time
search_templates({
  searchMode: "by_metadata",
  complexity: "simple",
  maxSetupMinutes: 15
})

// If MCP insufficient: Web search (load n8n-template-search skill)
WebFetch({
  url: "https://n8n.io/workflows/?search=slack",
  prompt: "Find Slack notification templates with descriptions"
})
```

**Found suitable template?**
- ✅ YES → Deploy with `n8n_deploy_template({templateId: XXX, autoFix: true})`
- ❌ NO → Continue to Step 1 (Node Discovery)

### Step 1: Node Discovery (if no template fits)

```javascript
// Search for nodes
search_nodes({query: "slack", includeExamples: true})

// Get node details
get_node({nodeType: "nodes-base.slack", detail: "standard"})
```

### Step 3: Configuration & Validation

```javascript
// Quick validation
validate_node({nodeType: "nodes-base.slack", config: {...}, mode: "minimal"})

// Full validation
validate_node({nodeType: "nodes-base.slack", config: {...}, profile: "runtime"})
```

### Step 4: Build & Deploy

```javascript
// Create workflow
n8n_create_workflow({name: "...", nodes: [...], connections: {...}})

// Or deploy template directly
n8n_deploy_template({templateId: 2947, autoFix: true})
```

### Step 5: Validate Complete Workflow

```javascript
validate_workflow(workflowJson)
n8n_validate_workflow({id: "workflow-id"})
```

---

## ⚠️ Critical Warnings

### Never Trust Defaults

```javascript
// ❌ FAILS at runtime - missing required params
{resource: "message", operation: "post", text: "Hello"}

// ✅ WORKS - all parameters explicit
{resource: "message", operation: "post", select: "channel", channelId: "C123", text: "Hello"}
```

### IF Node Multi-Output Routing

```javascript
// Use branch parameter for IF nodes
{
  type: "addConnection",
  source: "If Node",
  target: "True Handler",
  sourcePort: "main",
  targetPort: "main",
  branch: "true"  // ← CRITICAL!
}
```

### addConnection Syntax

```javascript
// ❌ WRONG - object format
{type: "addConnection", connection: {source: {...}}}

// ✅ CORRECT - four separate strings
{
  type: "addConnection",
  source: "node-id",
  target: "target-id", 
  sourcePort: "main",
  targetPort: "main"
}
```

---

## 🔗 Expression Syntax Quick Reference

```javascript
// Access current item data
{{ $json.fieldName }}

// Access previous node
{{ $node["NodeName"].json.field }}

// Webhook data (CRITICAL: under .body)
{{ $json.body.fieldName }}

// Built-in variables
{{ $now }}      // Current datetime
{{ $env.KEY }}  // Environment variable
{{ $runIndex }} // Current run index
```

---

## 📊 Popular Nodes Reference

| Node | Type | Purpose |
|------|------|---------|
| Webhook | `n8n-nodes-base.webhook` | HTTP trigger |
| HTTP Request | `n8n-nodes-base.httpRequest` | API calls |
| Code | `n8n-nodes-base.code` | JS/Python scripts |
| IF | `n8n-nodes-base.if` | Conditional routing |
| Set | `n8n-nodes-base.set` | Data transformation |
| Slack | `n8n-nodes-base.slack` | Slack messaging |
| Gmail | `n8n-nodes-base.gmail` | Email automation |
| Google Sheets | `n8n-nodes-base.googleSheets` | Spreadsheet ops |
| AI Agent | `@n8n/n8n-nodes-langchain.agent` | AI automation |
| OpenAI Chat | `@n8n/n8n-nodes-langchain.lmChatOpenAi` | ChatGPT calls |

---

## 🔍 When to Load Sub-Skills

Read the specific sub-skill when:

| Situation | Load |
|-----------|------|
| Finding workflow templates/examples | [n8n-template-search](n8n-template-search/SKILL.md) |
| Writing expressions with `$json` | [n8n-expression-syntax](n8n-expression-syntax/SKILL.md) |
| Using MCP tools | [n8n-mcp-tools-expert](n8n-mcp-tools-expert/SKILL.md) |
| Designing workflow architecture | [n8n-workflow-patterns](n8n-workflow-patterns/SKILL.md) |
| Validation errors | [n8n-validation-expert](n8n-validation-expert/SKILL.md) |
| Configuring complex nodes | [n8n-node-configuration](n8n-node-configuration/SKILL.md) |
| Writing JavaScript in Code node | [n8n-code-javascript](n8n-code-javascript/SKILL.md) |
| Writing Python in Code node | [n8n-code-python](n8n-code-python/SKILL.md) |

---

## ✅ Validation Checklist

Before deploying any workflow:

- [ ] All nodes have unique IDs
- [ ] All required parameters configured
- [ ] Connections properly mapped
- [ ] Expressions use correct syntax
- [ ] Error handling implemented
- [ ] validate_workflow passed
- [ ] Tested in development first
