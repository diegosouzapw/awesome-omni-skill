---
name: add-doc
description: Add new documentation sources to .claude/doc-sources.toml for Context7 access
---

# Add Documentation Source Skill

Add new tool documentation sources to enable real-time documentation queries via the doc-query skill.

## When to Use This Skill

**Invoke this skill when:**

1. **Adopting a new tool** - After creating/updating ADR for tool selection
2. **User mentions needing docs** - "We should add X documentation"
3. **Encountering undocumented tool** - Query fails with "Unknown source"
4. **Following CLAUDE.md workflow** - "Enable documentation" step

## What This Skill Does

- Validates source name and aliases don't conflict
- Adds TOML entry to `.claude/doc-sources.toml`
- Makes source immediately available to doc-query skill
- Maintains configuration consistency

## Instructions

### Step 1: Identify Context7 Library ID

Before adding a source, you need the Context7 library identifier:

1. Visit https://context7.com to browse available libraries
2. Look for official documentation sources (e.g., `websites/python_3_14`)
3. Note the exact library ID format: `websites/LIBRARY_ID`

**Common patterns:**
- Official sites: `websites/python_3_14`, `websites/code_claude`
- GitHub projects: `github_repos/OWNER/REPO`

### Step 2: Run add-source Script

```bash
.claude/skills/add-doc/scripts/add-source <name> <context7_id> <description> [options]
```

**Parameters:**
- `name`: Source name (lowercase, hyphens for spaces)
- `context7_id`: Context7 library identifier
- `description`: Brief description (in quotes)
- `--tokens N`: Optional default token limit (default: 2500)
- `--alias A1,A2`: Optional comma-separated aliases

**Examples:**
```bash
# Basic usage
.claude/skills/add-doc/scripts/add-source ruff websites/ruff "Ruff Python linter and formatter"

# With custom tokens
.claude/skills/add-doc/scripts/add-source mypy websites/mypy "Mypy static type checker" --tokens 3000

# With aliases
.claude/skills/add-doc/scripts/add-source pytest websites/pytest "Pytest testing framework" --alias py.test,test
```

### Step 3: Verify Addition

```bash
.claude/skills/doc-query/scripts/list-sources
```

The new source should appear in the list.

### Step 4: Test Query

```bash
.claude/skills/doc-query/scripts/query <name> "test query"
```

Verify documentation is retrieved successfully.

## Error Handling

**If source already exists:**
```
Error: Source 'ruff' already exists
```
Solution: Use a different name or update existing source manually.

**If alias conflicts:**
```
Error: Alias 'py' already used by 'python'
```
Solution: Choose different aliases.

**If Context7 ID not found:**
```
Error: Library not found: websites/nonexistent
```
Solution: Verify the library ID at https://context7.com.

## Manual Addition (Alternative)

If you prefer to edit manually:

1. Copy template:
   ```bash
   cat .claude/skills/add-doc/templates/source-entry.toml
   ```

2. Edit `.claude/doc-sources.toml` directly:
   ```toml
   [sources.ruff]
   context7_id = "websites/ruff"
   description = "Ruff Python linter and formatter"
   default_tokens = 2000
   aliases = ["ruff-lint"]
   ```

3. Verify with `list-sources`

## Integration with Other Skills

**Typical workflow (from CLAUDE.md):**

1. **Document decision** - Create/update ADR for tool adoption
   ```
   Use adr skill to create ADR-000X-adopt-ruff.md
   ```

2. **Enable documentation** - Add to doc-sources.toml
   ```
   Use add-doc skill to add ruff documentation source
   ```

3. **Query as needed** - Access documentation during development
   ```
   Use doc-query skill to query "ruff configuration"
   ```

## Token Efficiency

- Script runs locally (no AI tokens)
- Validation prevents configuration errors
- Only queries Context7 when doc-query is invoked
- Cost: 0 Sonnet tokens (shell script execution)

## Configuration Format

The script generates TOML entries in this format:

```toml
[sources.TOOL_NAME]
context7_id = "websites/LIBRARY_ID"
description = "Tool description"
default_tokens = 2500              # Optional
aliases = ["alias1", "alias2"]     # Optional
```

See [source-entry.toml template](templates/source-entry.toml) for annotated example.

## Technical Details

**Implementation:**
- Python 3.12+ stdlib only (tomllib, pathlib)
- TOML validation and conflict detection
- Atomic file updates (read, validate, append)
- Modern type hints (`str | None`, `list[str]`)

**See:** [ADR-0003](../../../docs/adrs/adr-0003-use-context7-for-documentation-access.md)

## Examples

### Example 1: Adding ruff

**Scenario:** Team adopts ruff for linting/formatting

**Steps:**
```bash
# 1. Add documentation source
.claude/skills/add-doc/scripts/add-source ruff websites/ruff "Ruff Python linter and formatter" --tokens 2000 --alias ruff-lint

# 2. Verify
.claude/skills/doc-query/scripts/list-sources

# 3. Test query
.claude/skills/doc-query/scripts/query ruff "configuration options"
```

**Output:**
```
✓ Added documentation source: ruff
  Context7 ID: websites/ruff
  Description: Ruff Python linter and formatter
  Default tokens: 2000
  Aliases: ruff-lint

Updated: /path/to/.claude/doc-sources.toml
```

### Example 2: Adding mypy

**Scenario:** Need type checking documentation

**Steps:**
```bash
.claude/skills/add-doc/scripts/add-source mypy websites/mypy "Mypy static type checker for Python" --alias typecheck
```

**Query later:**
```bash
.claude/skills/doc-query/scripts/query mypy "strict mode"
# Or using alias:
.claude/skills/doc-query/scripts/query typecheck "strict mode"
```

### Example 3: Adding pytest

**Scenario:** Adding test framework documentation

**Steps:**
```bash
.claude/skills/add-doc/scripts/add-source pytest websites/pytest "Pytest testing framework" --tokens 3000 --alias py.test,test
```

**Why 3000 tokens:** Pytest has extensive documentation; higher limit reduces truncation.

## Maintenance

**Adding new fields to config:**
Edit template in `templates/source-entry.toml` and update `format_toml_entry()` in add-source script.

**Removing sources:**
Manually edit `.claude/doc-sources.toml` to remove unwanted sections.

**Updating sources:**
Edit `.claude/doc-sources.toml` directly or remove and re-add with add-source.
