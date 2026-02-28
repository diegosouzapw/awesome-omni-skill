---
name: plugin-builder
description: Build Cursor plugin scaffolds with optional Runlayer MCP integration. Use this skill when the user wants to create a new Cursor plugin, scaffold plugin skills/commands/rules/hooks, or set up MCP connector configurations. Triggers include "create a plugin", "build a plugin", "scaffold a plugin", "new plugin for [domain]", or any task involving Cursor plugin development.
---

# Cursor Plugin Builder

Build Cursor plugin scaffolds. Optionally integrates with Runlayer MCP for tool discovery.

## Before You Start: Check Runlayer MCP (Optional)

1. Check if Runlayer MCP tools are available (e.g., `mcp__runlayer__list_servers`)
2. **If available**: call `mcp__runlayer__list_servers(scope="accessible")` to discover available integrations
3. **If NOT available**: proceed without Runlayer integration. The plugin can still include rules, skills, hooks, agents, and commands.

## Directory Context

1. Run `ls -la` to understand the current directory structure
2. Check if this is already a plugin marketplace (look for `.cursor-plugin/marketplace.json`)
3. If it's a marketplace, new plugins should be created as subdirectories
4. **Always confirm the target location with the user before creating files**

## Workflow

### Step 1: Gather Requirements

Use AskQuestion to collect structured input:

**Question 1: Plugin basics**
- Plugin name (kebab-case slug, e.g., `customer-support`)
- 1-2 sentence description

**Question 2: Components to include** (multi-select)
- Rules (.mdc files) -- persistent AI guidance
- Skills (SKILL.md) -- specialized agent capabilities
- Commands (.md) -- explicit user-invoked actions
- Hooks (hooks.json) -- automation on agent events
- Agents (.md) -- custom agent behaviors
- MCP servers (.mcp.json) -- external tool integrations

**Question 3: If skills selected -- automation preference**
- "Fully automated" -- Skills trigger automatically based on context
- "Manual triggers only" -- Commands require explicit invocation
- "Both" -- Skills for automation, Commands as explicit fallback

**Question 4: If MCP tools needed -- tool categories** (from Runlayer if available)
- Present discovered servers grouped by category (CRM, ticketing, analytics, etc.)
- If no Runlayer, ask which external services to integrate

### Step 2: Generate Plugin Structure

```
<plugin-slug>/
├── .cursor-plugin/
│   └── plugin.json
├── rules/                    # if rules selected
│   └── <rule-name>.mdc
├── skills/                   # if skills selected
│   └── <skill-name>/
│       └── SKILL.md
├── commands/                 # if commands selected
│   └── <command-name>.md
├── agents/                   # if agents selected
│   └── <agent-name>.md
├── hooks/                    # if hooks selected
│   └── hooks.json
├── .mcp.json                 # if MCP tools needed
└── README.md
```

### File Templates

**plugin.json:**
```json
{
  "name": "<plugin-slug>",
  "description": "<1-2 sentence description>",
  "version": "1.0.0",
  "author": { "name": "<author>" },
  "keywords": ["<relevant>", "<tags>"]
}
```

**Rule (.mdc):**
```markdown
---
description: <What this rule enforces>
alwaysApply: true
---

## <Rule Title>

- Rule point 1
- Rule point 2
```

For glob-scoped rules (apply only to matching files):
```markdown
---
description: <What this rule enforces>
globs:
  - "**/*.ts"
  - "**/*.tsx"
---
```

**SKILL.md:**
```markdown
---
name: <skill-name>
description: <When this skill should activate. Be specific about triggers and contexts.>
---

# <Skill Title>

## When to Use
<Specific triggers and contexts>

## Workflow
<Step-by-step guidance>

## Best Practices
<Domain-specific guidance>
```

**Command (.md):**
```markdown
---
name: <command-name>
description: <What this command does>
---

# <Command Title>

<Instructions for executing this command>

## Usage
Invoke via `/plugin-name:command-name`

## Workflow
<Step-by-step execution>
```

**Agent (.md):**
```markdown
---
name: <agent-name>
description: <Brief description of the agent's purpose>
---

# <Agent Title>

<System prompt and behavioral instructions for the agent>
```

**hooks.json:**
```json
{
  "hooks": {
    "<hookEvent>": [{ "command": "./scripts/<script>.sh" }]
  }
}
```

Available hook events: `sessionStart`, `sessionEnd`, `preToolUse`, `postToolUse`, `beforeShellExecution`, `afterShellExecution`, `beforeMCPExecution`, `afterMCPExecution`, `beforeReadFile`, `afterFileEdit`, `beforeSubmitPrompt`

**.mcp.json (only if MCP tools needed):**
```json
{
  "mcpServers": {
    "<server-name>": {
      "url": "<server-url>"
    }
  }
}
```

If Runlayer is available, use URLs from `mcp__runlayer__list_servers` results. Never guess URLs.

### Step 3: Generate CONNECTORS.md (if MCP tools used)

```markdown
# Connectors

| Category | Available Servers |
|----------|-------------------|
| CRM | Salesforce, HubSpot |
| Ticketing | Zendesk, Jira |

## Configuration
Configure MCP servers in `.mcp.json` based on which services you use.
```

## Plugin Examples

### Security Plugin
- **Rules:** secrets-hygiene, code-review-standards
- **Skills:** vulnerability-scan, dependency-audit
- **Hooks:** afterFileEdit (lint), beforeShellExecution (validate)

### DevOps Plugin
- **Skills:** incident-response, deployment-review
- **Commands:** /deploy-status, /incident-summary
- **MCP:** Monitoring (Datadog), CI/CD (GitHub Actions)

### Customer Support Plugin
- **Skills:** ticket-triage, response-drafting
- **Commands:** /escalate, /draft-response
- **MCP:** Ticketing (Zendesk), CRM (Salesforce)

## Testing

After generating, instruct the user to test locally:

1. Install the plugin from the local directory in Cursor
2. Try invoking skills by describing relevant tasks
3. Try commands via `/plugin-name:command-name`
4. Verify rules apply by checking agent behavior in matching contexts

## Submission

When ready to publish:

1. Ensure `.cursor-plugin/plugin.json` manifest is valid
2. All rules/skills/agents/commands have proper frontmatter
3. Push to a public Git repository
4. Submit at [cursor.com/marketplace/publish](https://cursor.com/marketplace/publish)
