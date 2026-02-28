---
name: airtable-connect
description: "Connect to any Airtable base by name. Load when user mentions 'airtable', 'connect airtable', 'setup airtable', 'query [base-name]', 'add to [table]', 'airtable bases', or any base name from persistent context. Meta-skill that discovers workspace, caches schemas, and routes to appropriate operations."
version: "1.0"
---

# Airtable Connect

**Meta-skill for complete Airtable workspace integration.**

## Purpose

Enable natural language interaction with ANY Airtable base. User says "query my Projects base" or "add a record to CRM" and it just works - no manual API calls, no remembering base IDs, no schema lookups.

---

## Shared Resources

This skill uses `airtable-master` shared library. Load references as needed:

| Resource | When to Load |
|----------|--------------|
| `airtable-master/scripts/check_airtable_config.py` | Always first (pre-flight) |
| `airtable-master/references/setup-guide.md` | If config check fails |
| `airtable-master/references/error-handling.md` | On any API errors |
| `airtable-master/references/api-reference.md` | For API details |

---

## First-Time User Setup

**If user has never used Airtable integration before:**

1. Run config check with JSON to detect setup state:
   ```bash
   python 00-system/skills/airtable/airtable-master/scripts/check_airtable_config.py --json
   ```

2. Parse the `ai_action` field in JSON output:
   - `prompt_for_api_key` → Guide user to get PAT, add to .env
   - `run_setup_wizard` → Run interactive wizard
   - `proceed_with_warning` → Partial config, warn but continue
   - `proceed_with_operation` → All good, continue

3. If setup needed, help user:
   - Tell them: "Airtable needs a Personal Access Token (PAT)"
   - Link: https://airtable.com/create/tokens
   - Scopes needed: `data.records:read`, `data.records:write`, `schema.bases:read`
   - **Write directly to `.env`** when user provides token
   - Re-verify with config check

**Setup triggers**: "setup airtable", "connect airtable", "configure airtable"

---

## Workflow 0: Config Check (ALWAYS FIRST)

Every workflow MUST start with config validation:

```bash
python 00-system/skills/airtable/airtable-master/scripts/check_airtable_config.py --json
```

**Parse `ai_action` from JSON:**
- **`proceed_with_operation`**: Fully configured, continue
- **`proceed_with_warning`**: API works but no bases (warn user to add bases to PAT)
- **`prompt_for_api_key`**: Need API key, guide user through setup
- **`run_setup_wizard`**: Run setup wizard

**If not configured:**
1. Tell user: "Airtable integration needs to be set up first."
2. Either guide them manually OR run: `python 00-system/skills/airtable/airtable-master/scripts/setup_airtable.py`
3. Restart workflow after setup complete

---

## Workflow 1: Discover Bases

**Triggers**: "connect airtable", "sync airtable", "discover bases", "what bases", "refresh airtable"

**Purpose**: Find all accessible bases in user's Airtable workspace and cache schemas.

**Steps**:
1. Run config check (Workflow 0)
2. Run discovery script:
   ```bash
   python 00-system/skills/airtable/airtable-master/scripts/discover_bases.py
   ```
3. Script outputs:
   - Number of bases found
   - Base names and IDs
   - Creates/updates: `01-memory/integrations/airtable-bases.yaml`
4. Show user summary of discovered bases
5. Confirm context file saved

**First-time flow**: If `airtable-bases.yaml` doesn't exist, discovery runs automatically.

---

## Workflow 2: Query Records

**Triggers**: "query [base]", "find in [table]", "search [base]", "show [table]", "list records"

**Purpose**: Query any base/table by name with optional filters.

**Steps**:
1. Run config check (Workflow 0)
2. Load context: Read `01-memory/integrations/airtable-bases.yaml`
   - If file doesn't exist → Run Workflow 1 (Discover) first
3. Match base name (fuzzy):
   - User says "Projects" → matches "Client Projects", "My Projects", etc.
   - If multiple matches → Show disambiguation prompt
   - If no match → Suggest running discovery
4. Run query:
   ```bash
   python 00-system/skills/airtable/airtable-master/scripts/query_records.py \
     --base <base_id> --table <table_name> [--filter "..."] [--sort ...] [--limit N]
   ```
5. Format and display results using field types from cached schema
6. Offer follow-up actions: "Want to add a record?" / "Query with different filters?"

**Filter Syntax**:
- `--filter "Status = Active"`
- `--filter "Priority = High"`
- `--filter "{Field} contains Design"`

---

## Workflow 3: Create Record

**Triggers**: "add to [table]", "create in [base]", "new [item] in [table]"

**Purpose**: Create a new record in any table with field validation.

**Steps**:
1. Run config check (Workflow 0)
2. Load context and match base/table (same as Workflow 2)
3. Load schema for target table from context file
4. Prompt user for required fields based on schema:
   - Show field name + type + options (for single/multiple select)
   - Validate input against field type
5. Run create:
   ```bash
   python 00-system/skills/airtable/airtable-master/scripts/manage_records.py create \
     --base <base_id> --table <table_name> \
     --fields '{"Name": "...", "Status": "..."}'
   ```
6. Confirm creation with record ID
7. Offer: "Add another?" / "View in Airtable?"

---

## Workflow 4: Update Record

**Triggers**: "update [record]", "edit [record]", "change [field] to [value]"

**Purpose**: Modify fields of an existing record.

**Steps**:
1. Run config check (Workflow 0)
2. Identify record:
   - By record ID if known
   - By search in table: `python query_records.py --filter "Name contains [search]"`
3. Show current field values
4. Accept changes from user
5. Run update:
   ```bash
   python 00-system/skills/airtable/airtable-master/scripts/manage_records.py update \
     --base <base_id> --table <table_name> --record <record_id> \
     --fields '{"Status": "Done", "Priority": "High"}'
   ```
6. Confirm changes with updated record

---

## Workflow 5: Delete Record

**Triggers**: "delete [record]", "remove [record]"

**Purpose**: Delete a record from a table.

**Steps**:
1. Run config check (Workflow 0)
2. Identify record (by ID or search)
3. Confirm with user: "Are you sure you want to delete [record name]?"
4. Run delete:
   ```bash
   python 00-system/skills/airtable/airtable-master/scripts/manage_records.py delete \
     --base <base_id> --table <table_name> --record <record_id>
   ```
5. Confirm deletion

---

## Workflow 6: Batch Operations

**Triggers**: "bulk update", "update multiple", "batch create"

**Purpose**: Create, update, or delete multiple records at once (max 10 per batch).

**Steps**:
1. Run config check (Workflow 0)
2. Collect records to process
3. Run batch operation:
   ```bash
   python 00-system/skills/airtable/airtable-master/scripts/manage_records.py batch-create \
     --base <base_id> --table <table_name> \
     --records '[{"fields": {...}}, {"fields": {...}}]'
   ```
4. Report results (success/failure counts)

**Note**: Airtable limits batch operations to 10 records per request.

---

## Context File Format

**Location**: `01-memory/integrations/airtable-bases.yaml`

```yaml
---
last_synced: 2025-12-11T12:00:00
bases:
  - id: "appXXXXXXXXXXXXXX"
    name: "Client Projects"
    permission_level: "create"
    tables:
      - id: "tblXXXXXXXXXXXXXX"
        name: "Projects"
        fields:
          - name: "Name"
            type: "singleLineText"
          - name: "Status"
            type: "singleSelect"
            options: ["Not Started", "In Progress", "Complete"]
          - name: "Priority"
            type: "singleSelect"
            options: ["Low", "Medium", "High"]
          - name: "Due Date"
            type: "date"
  - id: "appYYYYYYYYYYYYYY"
    name: "CRM"
    permission_level: "edit"
    tables:
      - id: "tblYYYYYYYYYYYYYY"
        name: "Contacts"
        fields: [...]
---

# Airtable Bases Context

Auto-generated by airtable-connect skill.
Run "refresh airtable" to update.
```

---

## Fuzzy Matching Logic

When user says a base/table name:

1. **Exact match**: "Client Projects" → finds "Client Projects"
2. **Case-insensitive**: "client projects" → finds "Client Projects"
3. **Partial match**: "Projects" → finds "Client Projects", "Personal Projects"
4. **Word match**: "crm" → finds "Sales CRM", "CRM Database"

**Disambiguation prompt** (if multiple matches):
```
Found 3 bases matching "Projects":
1. Client Projects
2. Personal Projects
3. Project Templates

Which one? (enter number or be more specific)
```

---

## Error Handling

| Error | Action |
|-------|--------|
| No bases found | Check PAT has base access at https://airtable.com/create/tokens |
| Base not in context | Run discovery: "refresh airtable" |
| Permission denied | Verify PAT has correct scopes |
| Rate limited (429) | Wait and retry with exponential backoff |
| Invalid field | Show valid options from cached schema |

Load `airtable-master/references/error-handling.md` for detailed troubleshooting.

---

## Examples

**First-time setup**:
```
User: "connect to airtable"
AI: [Runs config check, detects no API key]
    Airtable integration needs setup. I need your Personal Access Token (PAT).

    Get one at: https://airtable.com/create/tokens
    Required scopes: data.records:read, data.records:write, schema.bases:read

    Paste your PAT here (starts with 'pat.'):

User: pat.xxxxx...
AI: [Writes to .env, re-runs config check]
    ✅ Connected! Found 3 bases: Client Projects, CRM, Tasks
    What would you like to do?
```

**Query by name**:
```
User: "query my Projects for active items"
AI: Querying "Client Projects" → "Projects" table...
    Found 12 active items:
    1. Website Redesign (High priority, due Jan 15)
    2. Q1 Marketing Plan (Medium priority, due Jan 30)
    ...
```

**Create record**:
```
User: "add a new task to my Tasks base"
AI: Creating in "Tasks" base → "Tasks" table.
    Required fields:
    - Task Name: [your input]
    - Status: Not Started / In Progress / Complete
    - Due Date: [optional, format: YYYY-MM-DD]

    What's the task name?
```

---

*Airtable Connect v1.0 - Meta-skill for complete Airtable workspace integration*
