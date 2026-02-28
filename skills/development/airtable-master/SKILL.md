---
name: airtable-master
description: Shared resource library for Airtable integration skills. DO NOT load directly - provides common references (setup, API docs, error handling, field types) and scripts used by airtable-connect, airtable-query, and airtable-sync.
---

# Airtable Master

**This is NOT a user-facing skill.** It's a shared resource library.

## Purpose

Provides shared resources to eliminate duplication across:
- `airtable-connect` - Connect to bases, discover schema
- `airtable-query` - Query records with filters
- `airtable-sync` - Import/export records

**Instead of loading this skill**, users invoke the specific skill above.

---

## Architecture: DRY Principle

**Problem solved:** Multiple Airtable skills would have duplicated content (setup instructions, API docs, error handling).

**Solution:** Extract shared content into `references/` and `scripts/`.

**Result:** Single source of truth, reduced context per skill.

---

## Shared Resources

### references/

**[setup-guide.md](references/setup-guide.md)** - Complete setup wizard
- Creating Personal Access Token (PAT)
- Configuring .env file
- Selecting scopes
- Validation steps

**[api-reference.md](references/api-reference.md)** - API patterns
- Base URL and headers
- Key endpoints (bases, tables, records)
- Pagination
- Batch operations
- Rate limits

**[error-handling.md](references/error-handling.md)** - Troubleshooting
- HTTP error codes (401, 403, 404, 422, 429)
- Recovery patterns
- Debugging tips

**[field-types.md](references/field-types.md)** - Field type reference
- All 20+ Airtable field types
- Read and write formats
- Type-specific validation

### scripts/

#### Configuration & Setup

**[check_airtable_config.py](scripts/check_airtable_config.py)** - Pre-flight validation
```bash
python check_airtable_config.py [--verbose] [--json]
```
| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--verbose` / `-v` | No | False | Show detailed output |
| `--json` | No | False | Output structured JSON for AI consumption |

Exit codes: 0=configured, 1=partial (no bases), 2=not configured

**When to Use:** Run this FIRST before any Airtable operation. Use to validate PAT is configured, test API connection, check base access, or diagnose authentication issues.

---

**[setup_airtable.py](scripts/setup_airtable.py)** - Interactive setup wizard
```bash
python setup_airtable.py [--non-interactive] [--api-key KEY]
```
| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--non-interactive` | No | False | Skip prompts (requires API key) |
| `--api-key` | No | - | API key for non-interactive mode |

Default: Runs interactively. Guides through PAT creation, tests connection, saves to `.env`, auto-runs base discovery.

**When to Use:** Use when Airtable integration needs initial setup, when check_airtable_config.py returns exit code 2, or when user needs to reconfigure credentials or switch workspaces.

---

**[discover_bases.py](scripts/discover_bases.py)** - Base discovery (GET /meta/bases)
```bash
python discover_bases.py [--refresh] [--json] [--with-schema]
```
| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--refresh` | No | False | Force re-discovery even if cache exists |
| `--json` | No | False | Output as JSON only (no progress messages) |
| `--with-schema` | No | False | Also fetch table schemas (slower) |

Saves to: `01-memory/integrations/airtable-bases.yaml`

**When to Use:** Use when user asks "what bases do I have", "list my airtable bases", "show tables", after adding new bases, or when base/table name resolution fails (--refresh to update cache).

---

#### Record Operations

**[query_records.py](scripts/query_records.py)** - Query records from table (GET /{baseId}/{tableIdOrName})
```bash
python query_records.py --base BASE --table TABLE [--filter FORMULA] [--fields FIELDS] [--view VIEW] [--sort FIELD] [--limit N] [--json] [--verbose]
```
| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--base` | **Yes** | - | Base ID (appXXX) or name |
| `--table` | **Yes** | - | Table ID (tblXXX) or name |
| `--filter` | No | - | Airtable formula filter |
| `--fields` | No | - | Comma-separated field names to retrieve |
| `--view` | No | - | View ID or name |
| `--sort` | No | - | Sort field (prefix with `-` for descending) |
| `--limit` | No | - | Max records to return |
| `--json` | No | False | Output as JSON |
| `--verbose` / `-v` | No | False | Show debug info |

**Filter Formula Examples:**
```bash
--filter "{Status}='Active'"
--filter "AND({Status}='Active', {Priority}='High')"
--filter "SEARCH('john', LOWER({Name}))"
--filter "{Due Date} < TODAY()"
```

**When to Use:** Use when user wants to read records from a table, search with filters, list data from Airtable, or retrieve specific records. Supports Airtable formula syntax for complex filters.

---

**[manage_records.py](scripts/manage_records.py)** - CRUD operations (POST/PATCH/PUT/DELETE /{baseId}/{tableIdOrName})
```bash
# Create record(s)
python manage_records.py create --base BASE --table TABLE --data JSON [--file FILE] [--typecast] [--json] [--verbose]

# Update record(s)
python manage_records.py update --base BASE --table TABLE --record RECID --data JSON [--file FILE] [--typecast] [--replace] [--json] [--verbose]

# Delete record(s)
python manage_records.py delete --base BASE --table TABLE --record RECID [--file FILE] [--json] [--verbose]
```

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `action` | **Yes** | - | Action: `create`, `update`, `delete` (positional) |
| `--base` | **Yes** | - | Base ID (appXXX) or name |
| `--table` | **Yes** | - | Table ID (tblXXX) or name |
| `--record` | No* | - | Record ID (for update/delete single record) |
| `--data` | No* | - | JSON data for fields |
| `--file` | No* | - | JSON file with record(s) data |
| `--typecast` | No | False | Enable automatic type conversion |
| `--replace` | No | False | Replace mode for update (PUT vs PATCH) |
| `--json` | No | False | Output as JSON |
| `--verbose` / `-v` | No | False | Show debug info |

*For create: `--data` or `--file` required
*For update: `--record` + `--data` or `--file` required
*For delete: `--record` or `--file` required

**Usage Examples:**
```bash
# Create a single record
python manage_records.py create --base "My CRM" --table "Contacts" \
    --data '{"Name": "John Doe", "Email": "john@example.com"}'

# Create multiple records from file
python manage_records.py create --base appXXX --table tblYYY --file records.json

# Update a record
python manage_records.py update --base "Tasks" --table "Tasks" \
    --record recXXX --data '{"Status": "Done"}'

# Delete a record
python manage_records.py delete --base "Tasks" --table "Tasks" --record recXXX

# Batch operations with typecast
python manage_records.py create --base "CRM" --table "Leads" \
    --file leads.json --typecast
```

**Batch Limits:** Max 10 records per API call (script handles batching automatically)

**When to Use:** Use `create` when user wants to add new records, `update` to modify existing records (PATCH=partial, PUT=replace), `delete` to remove records. Supports batch operations from JSON files.

---

## Intelligent Error Detection Flow

When an Airtable skill fails due to missing configuration, the AI should:

### Step 1: Run Config Check with JSON Output

```bash
python 00-system/skills/airtable/airtable-master/scripts/check_airtable_config.py --json
```

### Step 2: Parse the `ai_action` Field

The JSON output includes an `ai_action` field that tells the AI what to do:

| ai_action | What to Do |
|-----------|------------|
| `proceed_with_operation` | Config OK, continue with the original operation |
| `proceed_with_warning` | Partial config (API works but no bases accessible) |
| `prompt_for_api_key` | Ask user: "I need your Airtable API key. Get one at https://airtable.com/create/tokens" |
| `run_setup_wizard` | Run: `python 00-system/skills/airtable/airtable-master/scripts/setup_airtable.py` |

### Step 3: Help User Fix Issues

If `ai_action` is `prompt_for_api_key`:

1. Tell user: "Airtable integration needs setup. I need your Personal Access Token (PAT)."
2. Show them: "Get one at: https://airtable.com/create/tokens"
3. Guide them through scopes: "Add scopes: data.records:read, data.records:write, schema.bases:read"
4. Ask: "Paste your Airtable PAT here (starts with 'pat.'):"
5. Once they provide it, **write directly to `.env`**:
   ```
   # Edit .env file to add:
   AIRTABLE_API_KEY=pat.their_token_here
   ```
6. Re-run config check to verify

### JSON Output Structure

```json
{
  "status": "not_configured",
  "exit_code": 2,
  "ai_action": "prompt_for_api_key",
  "missing": [
    {"item": "AIRTABLE_API_KEY", "required": true, "location": ".env"}
  ],
  "fix_instructions": [...],
  "env_template": "AIRTABLE_API_KEY=pat.YOUR_TOKEN_HERE",
  "setup_wizard": "python 00-system/skills/airtable/airtable-master/scripts/setup_airtable.py"
}
```

---

## How Skills Reference This

Each skill loads shared resources **only when needed** (progressive disclosure):

**airtable-connect** uses:
- `check_airtable_config.py` (validate before connection)
- `discover_bases.py` (find and cache bases)
- `api-reference.md` (schema endpoints)
- `error-handling.md` (troubleshooting)

**airtable-query** uses:
- `check_airtable_config.py` (validate before query)
- `query_records.py` (list/filter records)
- `api-reference.md` (query patterns)
- `field-types.md` (interpret results)

**airtable-sync** uses:
- `check_airtable_config.py` (validate before sync)
- `manage_records.py` (CRUD operations)
- `api-reference.md` (batch patterns)
- `error-handling.md` (recovery)

---

## Usage Example

**User says:** "query my Projects base in Airtable"

**What happens:**
1. AI loads `airtable-connect` (NOT airtable-master)
2. `airtable-connect` SKILL.md says: "Run check_airtable_config.py first"
3. AI executes: `python 00-system/skills/airtable/airtable-master/scripts/check_airtable_config.py --json`
4. If exit code 2, AI prompts user for API key and helps them set up
5. If exit code 0, AI executes: `python 00-system/skills/airtable/airtable-master/scripts/query_records.py --base "Projects"`
6. If errors occur, AI loads: `airtable-master/references/error-handling.md`

**airtable-master is NEVER loaded directly** - it's just a resource library.

---

## Key Differences from Notion

| Aspect | Airtable | Notion |
|--------|----------|--------|
| Auth | Personal Access Token (PAT) | Integration Token |
| Rate Limit | 5 req/s per base | 3 req/s |
| Batch Size | 10 records | Variable |
| Pagination | Offset-based | Cursor-based |
| Field Types | 20+ types | 20+ property types |

---

**Version**: 1.3
**Created**: 2025-12-11
**Updated**: 2025-12-11
**Status**: Production Ready

**Changelog**:
- v1.3: Added "When to Use" sections to all 5 scripts for AI routing guidance
- v1.2: Added comprehensive script argument documentation with usage examples and argument tables
- v1.1: Added Intelligent Error Detection Flow with `--json` support, added setup_airtable.py
- v1.0: Initial release
