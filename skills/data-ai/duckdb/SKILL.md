---
name: duckdb
description: "Use this skill when working with DuckDB for local data analysis, file ingestion, or data exploration. Covers reading CSV/Excel/Parquet/JSON files into DuckDB, SQL analytics on local data, data profiling, cleaning transformations, and export to various formats. Common phrases: \"analyze this CSV\", \"DuckDB query\", \"local data analysis\", \"read Excel in SQL\", \"profile this data\". Do NOT use for dbt model building (use dbt-transforms with DuckDB adapter) or cloud warehouse administration."
model:
  preferred: sonnet
  acceptable: [sonnet, opus]
  minimum: sonnet
  allow_downgrade: false
  reasoning_demand: medium
version: 1.0.0
---

# DuckDB Local Skill for Claude

Local-first SQL analytics on files. Read, profile, clean, and export data without a warehouse.

## When to Use This Skill

**Activate when:** Reading local files (CSV, Parquet, JSON, Excel) into DuckDB, profiling data quality, running SQL analytics on local data, cleaning/transforming datasets, exporting results to various formats.

**Don't use for:** Building dbt models (use dbt-transforms with DuckDB adapter), cloud warehouse administration, pipeline scheduling (use data-pipelines), Python DataFrame transformations (use python-data-engineering).

## Scope Constraints

- Local files only -- does not manage cloud warehouse connections or remote databases.
- No cloud warehouse administration (Snowflake, BigQuery, Redshift).
- Reference files loaded on demand -- do not pre-load multiple references.
- File paths must be validated before use; never execute user-provided paths without checking existence.

## Model Routing

| reasoning_demand | preferred | acceptable | minimum |
|-----------------|-----------|------------|---------|
| medium | Sonnet | Sonnet, Opus | Sonnet |

## Core Principles

1. **Local-first** -- Process data on the user's machine; no cloud credentials required.
2. **SQL-native** -- Use SQL for all transformations; avoid unnecessary Python wrappers.
3. **Schema inference** -- Let DuckDB auto-detect types; override only when inference fails.
4. **Zero config** -- Start with `import duckdb` or `duckdb` CLI; no server setup.
5. **Format-agnostic** -- Read CSV, Parquet, JSON, Excel through unified SQL interface.

## Default Engine Positioning

DuckDB is the default analytical engine for this skill suite. Assume DuckDB unless told otherwise.

**When to target a warehouse instead:**
- Data volume exceeds ~100GB or requires concurrent multi-user access
- Client provides warehouse access (Snowflake, BigQuery, Databricks)
- Production workload needs scheduling, monitoring, and SLA guarantees
- Regulatory requirements mandate data residency in a managed platform

When a warehouse is needed, DuckDB still serves as the local dev and validation target. Build and test locally, deploy to warehouse.

## Core Usage Patterns

| Pattern | Description | When to Use |
|---------|-------------|-------------|
| **DuckDB as dbt target** | `dbt-duckdb` adapter for local transforms | Prototyping, client demos, small datasets |
| **Standalone query engine** | Direct SQL on files, no framework | Ad-hoc analysis, data profiling, one-off cleaning |
| **DuckDB + DLT** | DLT loads files into DuckDB | Development pipeline with destination swapping |
| **Dev target, warehouse deploy** | DuckDB locally, Snowflake/BigQuery in prod | Full engagement lifecycle |

## Quick Start

```bash
pip install duckdb
```

### Python

```python
import duckdb

# Read a CSV and query it
result = duckdb.sql("SELECT * FROM read_csv_auto('data.csv') LIMIT 10")
result.show()
```

### CLI

```bash
duckdb -c "SELECT count(*) FROM read_csv_auto('data.csv')"
```

### Persistent Database

```python
con = duckdb.connect("my_analysis.duckdb")
con.sql("CREATE TABLE customers AS SELECT * FROM read_csv_auto('customers.csv')")
con.sql("SELECT count(*) FROM customers").show()
```

### Excel Files

```sql
INSTALL excel; LOAD excel;
SELECT * FROM read_xlsx('data.xlsx', sheet='Sheet1');
```

## File Ingestion

| Format | Function | Auto-Detect | Glob Support | Notes |
|--------|----------|-------------|--------------|-------|
| CSV/TSV | `read_csv_auto()` | delimiter, header, types | Yes | Most common; handles most encodings |
| Parquet | `read_parquet()` | schema from metadata | Yes | Fastest; preserves types perfectly |
| JSON | `read_json_auto()` | structure, nesting | Yes | Handles NDJSON and arrays |
| Excel | `read_xlsx()` (excel ext) | sheet, headers | No | Requires `INSTALL excel; LOAD excel;` (DuckDB >=1.2) |

### Basic Ingestion

```sql
-- CSV with auto-detection
SELECT * FROM read_csv_auto('customers.csv');

-- Parquet with glob
SELECT * FROM read_parquet('data/events/*.parquet');

-- JSON (newline-delimited)
SELECT * FROM read_json_auto('logs.ndjson', format='newline_delimited');

-- Multiple CSVs, union by column name
SELECT * FROM read_csv_auto('data/*.csv', union_by_name=true);
```

**For detailed ingestion options, see:** [File Ingestion Reference](references/file-ingestion.md)
**For Excel-specific patterns, see:** [Excel Specifics](references/excel-specifics.md)

## Data Profiling Patterns

### Column Statistics

```sql
SELECT
    column_name
    , column_type
    , count
    , null_percentage
    , approx_unique
    , min
    , max
FROM (SUMMARIZE SELECT * FROM read_csv_auto('data.csv'));
```

### NULL Rates and Cardinality

```sql
SELECT
    count(*) as total_rows
    , count(email) as email_present
    , round(100.0 * count(email) / count(*), 1) as email_pct
    , count(distinct email) as email_unique
    , count(*) - count(distinct email) as email_duplicates
FROM read_csv_auto('customers.csv');
```

### Value Distribution

```sql
-- Top 10 values for a column
SELECT status, count(*) as cnt, round(100.0 * count(*) / sum(count(*)) over (), 1) as pct
FROM read_csv_auto('orders.csv')
GROUP BY status
ORDER BY cnt DESC
LIMIT 10;
```

## Cleaning in SQL

Common transformations executed directly in DuckDB:

```sql
CREATE TABLE customers_clean AS
SELECT
    -- Dedup
    * EXCLUDE (row_num)
FROM (
    SELECT
        *
        , row_number() OVER (PARTITION BY email ORDER BY updated_at DESC) as row_num
    FROM read_csv_auto('customers.csv')
)
WHERE row_num = 1;
```

```sql
-- Type casting and trimming
SELECT
    trim(name) as name
    , lower(trim(email)) as email
    , try_cast(revenue as decimal(12,2)) as revenue
    , strptime(date_str, '%m/%d/%Y')::date as order_date
    , regexp_replace(phone, '[^0-9]', '', 'g') as phone_digits
FROM read_csv_auto('messy_data.csv');
```

## Export Patterns

```sql
-- CSV
COPY (SELECT * FROM customers_clean) TO 'output/customers.csv' (HEADER, DELIMITER ',');

-- Parquet (recommended for downstream analytics)
COPY (SELECT * FROM customers_clean) TO 'output/customers.parquet' (FORMAT PARQUET);

-- JSON (newline-delimited)
COPY (SELECT * FROM customers_clean) TO 'output/customers.json' (FORMAT JSON);
```

**For detailed export options, see:** [Export Patterns Reference](references/export-patterns.md)

## Performance Tips

| Setting | Command | Use When |
|---------|---------|----------|
| Memory limit | `SET memory_limit = '4GB';` | Large datasets on constrained machines |
| Threads | `SET threads = 4;` | Control parallelism |
| Temp directory | `SET temp_directory = '/tmp/duckdb';` | Spill to disk for large sorts |
| Progress bar | `SET enable_progress_bar = true;` | Long-running queries |

File format performance: **Parquet >> CSV >> JSON**. Convert to Parquet early for repeated queries.

**For detailed performance tuning, see:** [Performance Reference](references/performance.md)

## Security Posture

See [Security & Compliance Patterns](../shared-references/data-engineering/security-compliance-patterns.md) for the full framework.
See [Consulting Security Tier Model](../shared-references/data-engineering/security-tier-model.md) for tier definitions.

| Capability | Tier 1 (Schema-Only) | Tier 2 (Sampled) | Tier 3 (Full Access) |
|------------|----------------------|-------------------|---------------------|
| Read local files | Metadata only | Anonymized samples | Full data |
| Write/export files | Schema DDL only | Sample outputs | Full exports |
| Profiling | Column types/names | Stats on samples | Full profiling |
| Cleaning transforms | Compile/validate SQL | Execute on samples | Execute on full data |

- **Never** hardcode file paths containing credentials or sensitive data in scripts.
- **Never** commit data files to version control; use `.gitignore`.
- Validate file paths exist before reading; reject paths outside the working directory.

## Reference Files

Reference files loaded on demand:

- **[File Ingestion](references/file-ingestion.md)** -- CSV, Parquet, JSON parameters, glob patterns, schema inference, common issues
- **[Excel Specifics](references/excel-specifics.md)** -- Excel extension (read_xlsx), sheet selection, merged cells, date conversion, spatial fallback
- **[Export Patterns](references/export-patterns.md)** -- CSV/Parquet/JSON options, Python API export, partitioned writes
- **[Performance](references/performance.md)** -- Memory limits, threads, EXPLAIN ANALYZE, pragma settings, format comparison
