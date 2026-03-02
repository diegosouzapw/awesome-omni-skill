---
name: asyncpg-detection
description: This skill should be used when the user asks to "detect asyncpg usage", "find asyncpg patterns", "scan for asyncpg imports", or "identify asyncpg database code in FastAPI projects". It automatically scans Python files to identify asyncpg imports, connection patterns, and query execution methods that need conversion to SQLAlchemy.
version: 1.0.0
---

# AsyncPG Detection for FastAPI Projects

This skill provides comprehensive detection of asyncpg usage patterns in FastAPI applications, identifying all code that needs to be converted to SQLAlchemy with asyncpg engine support.

## Detection Overview

Scan FastAPI projects for asyncpg patterns including imports, connection management, queries, transactions, and error handling. Generate detailed reports with line numbers and conversion recommendations.

## Core Detection Patterns

### Import Detection
Look for these import statements:
- `import asyncpg`
- `from asyncpg import`
- `import asyncpg as pg`
- `from asyncpg import Connection, Pool`

### Connection Patterns
Identify these asyncpg connection approaches:
- `asyncpg.connect()` calls
- `asyncpg.create_pool()` usage
- Manual connection string parsing
- Environment-based connection configuration

### Query Patterns
Detect these asyncpg execution methods:
- `connection.fetch()` for SELECT queries
- `connection.execute()` for INSERT/UPDATE/DELETE
- `connection.fetchval()` for single values
- `connection.fetchrow()` for single rows
- `connection.iter()` for result iteration

## Usage Instructions

To detect asyncpg usage in your FastAPI project:

1. **Run comprehensive scan**: Use the `/convert-asyncpg-to-sqlalchemy` command to scan all Python files in the project
2. **Analyze detection results**: Review the generated report for files containing asyncpg code
3. **Prioritize conversion**: Focus on files with the most asyncpg usage first
4. **Check for complex patterns**: Look for nested connections, transactions, and error handling that may require special attention

## Reporting Format

The detection generates reports with:
- **File list**: All files containing asyncpg imports
- **Pattern analysis**: Specific asyncpg methods found
- **Complexity assessment**: Files requiring manual intervention
- **Conversion recommendations**: Suggested SQLAlchemy equivalents

## Additional Resources

### Reference Files
- **`references/patterns-mapping.md`** - Complete asyncpg to SQLAlchemy pattern mapping
- **`references/complex-cases.md`** - Handling of complex asyncpg scenarios
- **`references/supabase-specific.md`** - Supabase-specific asyncpg patterns

### Examples
- **`examples/detection-report.md`** - Sample detection output
- **`examples/fastapi-project-structure.md`** - Example FastAPI project with asyncpg usage