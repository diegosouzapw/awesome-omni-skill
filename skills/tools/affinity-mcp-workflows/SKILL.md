---
name: affinity-mcp-workflows
description: Use when working with Affinity CRM via MCP tools - find entities, manage workflows, log interactions, prepare briefings, find warm intros. Also use when user mentions "pipeline", "deals", "relationship strength", or wants to prepare for meetings.
---

# Affinity MCP Workflows

This skill covers the xaffinity MCP server tools, prompts, and resources for working with Affinity CRM.

## Prerequisites

The MCP server requires the xaffinity CLI to be installed:

```bash
pip install "affinity-sdk[cli]"
```

The CLI must be configured with an API key before the MCP server will work.

## ⚠️ REQUIRED WORKFLOW - Do Not Skip Steps

**You MUST complete steps 1-2 before running ANY queries or commands.**

Skipping these steps leads to incorrect or inefficient queries because:
- Command syntax may have changed since this skill was written
- New flags (like `--with-interaction-dates`) may exist that you don't know about
- You may use deprecated syntax that returns incomplete data
- The data model has nuances (e.g., `list export` vs `company ls`) that you'll miss

### Mandatory Pre-Flight Checklist

**Before proceeding to execute any commands:**

1. ✅ Read `xaffinity://data-model` using `read-xaffinity-resource`
2. ✅ Run `discover-commands` for your specific task
3. ✅ State what you learned from each step before continuing

**Example:**
```
"I read the data-model resource and learned that list entries have custom fields
accessed via fields.<Name>. I ran discover-commands for 'interaction' and found
that interaction ls supports --type all to fetch all types in one call, and
--days to limit the time range. Now I'll proceed with..."
```

## IMPORTANT: Write Operations Only After Explicit User Request

**Only use tools or prompts that modify CRM data when the user explicitly asks to do so.**

Write operations include:
- **Tools**: `execute-write-command`
- **Prompts**: `log-interaction-and-update-workflow`, `change-status`, `log-call`, `log-message`

Read-only operations (search, lookup, briefings) can be used proactively to help the user. But never create, update, or delete CRM records unless the user specifically requests it.

## Full Scan Protection

The MCP gateway enforces pagination limits to prevent unbounded data scans:

| Limit | Value | Description |
|-------|-------|-------------|
| Default | 1000 records | Applied when no `--max-results` specified |
| Maximum | 10000 records | Higher values are capped with a warning |
| `--all` flag | **BLOCKED** | Use `--max-results` or cursor pagination instead |

**Affected commands:** `list export`, `list ls`, `person ls`, `company ls`, `opportunity ls`, `note ls`, `reminder ls`, `interaction ls`, `field history`

**To fetch more than 10000 records:** Use cursor pagination with `--cursor` flag.

## Available Tools

### CLI Gateway (Primary Interface)

The CLI Gateway provides full access to the xaffinity CLI:

| Tool | Use Case |
|------|----------|
| `discover-commands` | Search CLI commands by keyword (e.g., "create person", "export list") |
| `execute-read-command` | Execute read-only CLI commands (get, search, list, export) |
| `execute-write-command` | **(write)** Execute write CLI commands (create, update, delete) |

**Usage pattern:**

1. **Discover** the right command: `discover-commands(query: "create person", category: "write")`
2. **Execute** it: `execute-write-command(command: "person create", argv: ["--first-name", "John", "--last-name", "Doe"])`

### Utility Tools

| Tool | Use Case |
|------|----------|
| `get-entity-dossier` | Comprehensive entity info (details, relationship strength, interactions, notes, list memberships) |
| `read-xaffinity-resource` | Access dynamic resources via `xaffinity://` URIs |

### Destructive Commands

Commands that delete data require double confirmation:

1. **Look up the entity first** using `execute-read-command` to show what will be deleted
2. **Ask the user in your response** by showing them the entity details and requesting confirmation
3. **Wait for user's next message** - do NOT proceed until they explicitly confirm
4. **Only after user confirms** should you execute with `confirm: true`

Example flow:
```
User: "Delete person 123"
You: execute-read-command(command: "person get", argv: ["123"])
You: "This will permanently delete John Smith (ID: 123, email: john@example.com).
      Type 'yes' to confirm deletion."
[Stop here and wait for user's response]

User: "yes"
You: execute-write-command(command: "person delete", argv: ["123"], confirm: true)
```

## Query vs CLI Commands: When to Use What

**Use `query` tool for:**
- Any operation needing **relationships** (persons at a company, companies for a person)
- Any operation needing **computed data** (interaction dates, unreplied messages)
- **Pipeline analysis** with aggregations or groupBy
- **Complex filtering** with AND/OR conditions
- **List entry operations** that need associated entities

**Use individual CLI commands for:**
- **Simple lookups**: `person get 123`, `company get 456`
- **Quick searches**: `person ls --query "John"`, `company ls --query "Acme"`
- **Metadata**: `list ls`, `field ls --list-id <id>`
- **Write operations**: All creates, updates, deletes

### Query Examples (Preferred for Complex Operations)

⚠️ **STOP: Did you complete the pre-flight checklist?** The syntax below may be outdated. Run `discover-commands` first to verify current syntax and available flags.

⚠️ **For queries with `expand` or `include`, ALWAYS use `dryRun: true` first** to see estimated API calls. These cause N+1 API calls (one per record) and can be slow or timeout.

```json
// STEP 1: Preview any expand/include query with dryRun first
{"query": {"from": "listEntries", "where": {"path": "listName", "op": "eq", "value": "Dealflow"}, "expand": ["interactionDates"], "limit": 100}, "dryRun": true}

// STEP 2: If API calls look reasonable (<200), run without dryRun
{"from": "listEntries", "where": {"path": "listName", "op": "eq", "value": "Dealflow"}, "expand": ["interactionDates"], "limit": 100}

// Pipeline with field values and unreplied email detection
{"from": "listEntries", "where": {"path": "listName", "op": "eq", "value": "Dealflow"}, "select": ["entityName", "fields.Status", "fields.Owner"], "expand": ["unreplied"]}

// Persons with their companies and interaction history summary
{"from": "persons", "where": {"path": "email", "op": "contains", "value": "@acme.com"}, "include": ["companies"], "expand": ["interactionDates"]}

// Pipeline summary by status (aggregation) - no expand, no dryRun needed
{"from": "listEntries", "where": {"path": "listName", "op": "eq", "value": "Dealflow"}, "groupBy": "fields.Status", "aggregate": {"count": {"count": true}}}

// List entries with associated persons and interactions (parameterized include)
{"from": "listEntries", "where": {"path": "listName", "op": "eq", "value": "Dealflow"}, "include": [{"interactions": {"limit": 50, "days": 180}}, "persons"]}
```

## Common CLI Commands

⚠️ **Reminder:** Run `discover-commands` first. The commands below are examples - actual syntax and flags may differ.

Use `discover-commands` to find commands, then `execute-read-command` or `execute-write-command` to run them.

### Search & Lookup (Simple Operations)

| Command | Use Case |
|---------|----------|
| `person ls --query "..."` | Quick search persons by name/email |
| `company ls --query "..."` | Quick search companies |
| `list ls` | List all Affinity lists |
| `field ls --list-id <id>` | Get field definitions and dropdown options |

**Note:** For list exports needing relationships or computed data, use `query` instead of `list export`.

### Entity Details

| Command | Use Case |
|---------|----------|
| `person get <id>` | Get person details |
| `company get <id>` | Get company details |
| `opportunity get <id>` | Get opportunity details |
| `relationship-strength ls --external-id <id>` | Get relationship strength for a person |
| `interaction ls --person-id <id> --type all` | Get all interactions (or use specific type: email, meeting, call, chat-message) |
| `field history <field-id> --person-id <id>` | Audit who changed a field and when. Use to track status changes or investigate field modifications. **Requires exactly one entity selector**: `--person-id`, `--company-id`, `--opportunity-id`, or `--list-entry-id` |

### Write Operations

| Command | Use Case |
|---------|----------|
| `interaction create --type call --person-id <id>` | Log a call/meeting/email |
| `note create --person-id <id> --content "..."` | Add a note |
| `entry field "<list>" <entryId> --set <field> <value>` | Update a field value |
| `person create --first-name "..." --last-name "..."` | Create a person |

## MCP Prompts (Guided Workflows)

These prompts provide guided multi-step workflows. Suggest them when appropriate.

**Note**: Prompts marked with (write) modify CRM data - only use when user explicitly requests.

| Prompt | Type | When to Suggest |
|--------|------|-----------------|
| `prepare-briefing` | read-only | User has upcoming meeting, needs context on a person/company |
| `pipeline-review` | read-only | User wants weekly/monthly pipeline review |
| `warm-intro` | read-only | User wants to find introduction path to someone |
| `interaction-brief` | read-only | Get interaction history summary for an entity |
| `log-interaction-and-update-workflow` | **write** | User explicitly asks to log a call/meeting and update pipeline |
| `change-status` | **write** | User explicitly asks to move a deal to new stage |
| `log-call` | **write** | User explicitly asks to log a phone call |
| `log-message` | **write** | User explicitly asks to log a chat/text message |

### How to Invoke Prompts

Prompts are invoked with arguments. Example:
- `prepare-briefing(entityName: "John Smith", meetingType: "demo")`
- `warm-intro(targetName: "Jane Doe", context: "partnership discussion")`
- `log-interaction-and-update-workflow(personName: "Alice", interactionType: "call", summary: "Discussed pricing")`

## Resources

Access dynamic data via `xaffinity://` URIs using `read-xaffinity-resource`:

| URI | Returns |
|-----|---------|
| `xaffinity://me` | Current authenticated user details |
| `xaffinity://me/person-id` | Current user's person ID in Affinity |
| `xaffinity://interaction-enums` | Valid interaction types and directions |
| `xaffinity://saved-views/{listId}` | Saved views available for a list |
| `xaffinity://field-catalogs/{listId}` | Field definitions for a list |
| `xaffinity://workflow-config/{listId}` | Workflow configuration for a list |

## Common Workflow Patterns

⚠️ **Before using any pattern below:** Complete the pre-flight checklist (read data-model, run discover-commands, state what you learned).

### Before a Meeting
1. Use `get-entity-dossier` for full context (relationship strength, recent interactions, notes)
2. **Or use**: `prepare-briefing` prompt for a guided flow

### After a Call/Meeting
1. Use `execute-write-command` with `interaction create` to log what happened
2. Use `query` to find list entry: `{"from": "listEntries", "where": {"and": [{"path": "listName", "op": "eq", "value": "Dealflow"}, {"path": "entityName", "op": "contains", "value": "Acme"}]}}`
3. Use `execute-write-command` with `entry field` if deal stage changed
4. **Or use**: `log-interaction-and-update-workflow` prompt

### Finding Warm Introductions
1. Use `execute-read-command` with `person ls` to locate target person
2. Use `execute-read-command` with `relationship-strength ls` for connection strength
3. **Or use**: `warm-intro` prompt for guided flow

### Pipeline Review
1. Use `query` with aggregation: `{"from": "listEntries", "where": {"path": "listName", "op": "eq", "value": "Dealflow"}, "groupBy": "fields.Status", "aggregate": {"count": {"count": true}}}`
2. Use `query` with expand for details: `{"from": "listEntries", "where": {"path": "listName", "op": "eq", "value": "Dealflow"}, "expand": ["interactionDates", "unreplied"]}`
3. **Or use**: `pipeline-review` prompt

### Updating Deal Status
1. Use `query` to find the entry: `{"from": "listEntries", "where": {"and": [{"path": "listName", "op": "eq", "value": "Dealflow"}, {"path": "entityName", "op": "contains", "value": "..."}]}}`
2. Use `execute-read-command` with `field ls --list-id` to see available statuses
3. Use `execute-write-command` with `entry field` to update
4. **Or use**: `change-status` prompt

## Tips

- **Entity types**: `person`, `company`, `opportunity`
- **Interaction types**: `call`, `meeting`, `email`, `chat_message`, `in_person`
- **Dossier is comprehensive**: `get-entity-dossier` returns relationship strength, interactions, notes, and list memberships in one call
- **Use names directly**: Most commands accept names instead of IDs (e.g., `person ls --query "John"`)
- **Finding entities in a list**: Use `query` with filters:
  ```json
  {"from": "listEntries", "where": {"and": [{"path": "listName", "op": "eq", "value": "Dealflow"}, {"path": "entityName", "op": "contains", "value": "Acme"}]}}
  ```
- **Output formats**: The `format` parameter controls result format:
  - `toon` (default for query): 40% fewer tokens, best for bulk queries
  - `markdown`: Best for LLM comprehension when analyzing data
  - `json`: Full structure with envelope - supports cursor pagination for large results
  - `csv`: For spreadsheet export
  - All formats support cursor pagination when results are truncated (use `nextCursor` to resume)

## Troubleshooting

If tools aren't working or returning unexpected results:

### Enable Debug Mode

```bash
# Enable (persistent, works with any MCP client)
mkdir -p ~/.config/xaffinity-mcp && touch ~/.config/xaffinity-mcp/debug

# Restart the MCP client (Claude Desktop: Cmd+Q, reopen)

# Disable when done
rm ~/.config/xaffinity-mcp/debug
```

### View Logs

**Claude Desktop**: `tail -f ~/Library/Logs/Claude/mcp-server-*.log`

Debug logs show component prefixes like `[xaffinity:tool:1.2.3]` to identify which component produced each message.

### Common Issues

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| Tools show old behavior after update | Cached MCP server process | Fully quit and restart Claude Desktop |
| API key errors | Key not configured | Run `xaffinity config setup-key` |
| CLI version errors | Outdated CLI | Run `pip install --upgrade affinity-sdk`
