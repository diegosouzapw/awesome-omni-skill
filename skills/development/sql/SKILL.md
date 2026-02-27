---
name: "SQL"
description: "Execute these commands after EVERY implementation (see AGENT_AUTOMATION module for full workflow)."
version: "1.0.0"
category: "languages"
author: "Rulebook"
tags: ["languages", "language"]
dependencies: []
conflicts: []
---
<!-- SQL:START -->
# SQL Project Rules

## Agent Automation Commands

**CRITICAL**: Execute these commands after EVERY implementation (see AGENT_AUTOMATION module for full workflow).

```bash
# Complete quality check sequence:
sqlfluff lint migrations/  # SQL linting
sqlfluff format --check migrations/  # Format check
# Run migration tests (project-specific)
# Run pgTAP tests if configured

# Migration validation:
flyway validate           # Flyway migrations
# OR: liquibase validate  # Liquibase migrations
```

## SQL Configuration

**CRITICAL**: Use modern SQL standards with linting and testing.

- **Standard**: SQL:2016 or later
- **Linter**: SQLFluff or SQL Lint
- **Testing**: pgTAP (PostgreSQL), utPLSQL (Oracle), tSQLt (SQL Server)
- **Version Control**: All DDL and migrations tracked

## Code Quality Standards

### Mandatory Quality Checks

**CRITICAL**: After implementing ANY change, you MUST run these commands in order.

**IMPORTANT**: These commands MUST match your GitHub Actions workflows to prevent CI/CD failures!

```bash
# Pre-Commit Checklist (MUST match .github/workflows/*.yml)

# 1. Format check (matches workflow)
sqlfluff format --check .

# 2. Lint (MUST pass with no warnings - matches workflow)
sqlfluff lint .

# 3. Run all tests (MUST pass 100% - matches workflow)
# PostgreSQL:
psql -d testdb -f tests/test_suite.sql

# SQL Server:
sqlcmd -S localhost -d testdb -i tests/test_suite.sql

# 4. Validate migrations (matches workflow)
# Check migration order and dependencies
flyway validate
# or: liquibase validate

# If ANY fails: ❌ DO NOT COMMIT - Fix first!
```

**Why This Matters:**
- Running different commands locally than in CI causes database deployment failures
- Example: Using `sqlfluff fix` locally but `sqlfluff format --check` in CI = failure
- Example: Missing migration validation = broken database deployments

### Style Guide

Example `.sqlfluff` configuration:
```ini
[sqlfluff]
dialect = postgres
templater = jinja
sql_file_exts = .sql,.sql.j2
max_line_length = 100
indent_unit = space
indented_joins = True
indented_using_on = True

[sqlfluff:rules]
capitalisation.keywords = upper
capitalisation.identifiers = lower
capitalisation.functions = upper
```

### Testing Example (pgTAP)

```sql
BEGIN;
SELECT plan(3);

-- Test table exists
SELECT has_table('users', 'users table should exist');

-- Test function
SELECT is(
    calculate_total(100, 0.1),
    110.0::numeric,
    'calculate_total should apply 10% markup'
);

-- Test constraint
SELECT col_is_pk('users', 'id', 'id should be primary key');

SELECT * FROM finish();
ROLLBACK;
```

### Migration Best Practices

```sql
-- migrations/001_create_users.sql
-- IDEMPOTENT: Can run multiple times safely

-- Create table if not exists
CREATE TABLE IF NOT EXISTS users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create index if not exists
CREATE INDEX IF NOT EXISTS idx_users_email ON users(email);

-- Add column if not exists (PostgreSQL 9.6+)
DO $$
BEGIN
    IF NOT EXISTS (
        SELECT 1 FROM information_schema.columns
        WHERE table_name = 'users' AND column_name = 'updated_at'
    ) THEN
        ALTER TABLE users ADD COLUMN updated_at TIMESTAMP WITH TIME ZONE;
    END IF;
END $$;
```

<!-- SQL:END -->

