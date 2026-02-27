---
name: bigquery-cli
description: Use when working with BigQuery from command line using bq tool, including querying data, loading/exporting tables, managing datasets, cost estimation with dry-run, or partitioning strategies
---

# BigQuery CLI (bq)

## Overview

The BigQuery CLI (`bq`) provides command-line access to Google BigQuery for data warehousing and analytics. **Core principle:** Always estimate costs with dry-run, use proper authentication, and leverage partitioning for scale.

## When to Use

- Running SQL queries on BigQuery
- Loading data from CSV/JSON/Avro/Parquet
- Exporting table data to GCS
- Managing datasets, tables, views
- Cost estimation and optimization
- Creating partitioned/clustered tables
- Viewing schemas and data samples

**When NOT to use:**
- When BigQuery UI is more appropriate (ad-hoc exploration)
- When using BigQuery client libraries (Python, Java, etc.)

## Authentication and Project Setup

**Before ANY bq operation:**
```bash
# Check authentication
gcloud auth list

# Login if needed
gcloud auth login

# Set default project
gcloud config set project PROJECT_ID

# Verify current project
gcloud config get-value project
```

## Command Structure

**Format:** `bq [--global_flags] <command> [--command_flags] [args]`

**Important global flags:**
- `--project_id=PROJECT` - Override default project
- `--dataset_id=DATASET` - Default dataset
- `--location=LOCATION` - Geographic location (us, eu, asia-northeast1)
- `--format=FORMAT` - Output format: pretty, sparse, prettyjson, json, csv
- `--quiet` / `-q` - Suppress status updates
- `--dry_run` - Validate without executing (queries only)

## Quick Reference

### Query Operations

| Task | Command | Key Flags |
|------|---------|-----------|
| Run query | `bq query 'SELECT ...'` | `--dry_run`, `--use_legacy_sql=false`, `--destination_table`, `--max_rows` |
| Estimate cost | `bq query --dry_run 'SELECT ...'` | Shows bytes to be processed |
| Save results | `bq query --destination_table=ds.table 'SELECT ...'` | `--append_table`, `--replace` |
| Parameterized query | `bq query --parameter='name:STRING:value' 'SELECT ...'` | Repeat `--parameter` for multiple |

### Data Loading

| Task | Command | Key Flags |
|------|---------|-----------|
| Load CSV | `bq load ds.table gs://bucket/file.csv schema` | `--skip_leading_rows=1`, `--autodetect`, `--field_delimiter` |
| Load JSON | `bq load --source_format=NEWLINE_DELIMITED_JSON ds.table file.json` | `--autodetect`, `--schema` |
| Load Parquet/Avro | `bq load --source_format=PARQUET ds.table gs://bucket/*.parquet` | Schema auto-detected |
| Replace table | `bq load --replace ds.table source schema` | Overwrites existing data |
| Append to table | `bq load --noreplace ds.table source schema` | Adds to existing data (default) |

### Data Export

| Task | Command | Key Flags |
|------|---------|-----------|
| Export to CSV | `bq extract --destination_format=CSV ds.table gs://bucket/file.csv` | `--field_delimiter`, `--print_header` |
| Export to JSON | `bq extract --destination_format=NEWLINE_DELIMITED_JSON ds.table gs://bucket/*.json` | Use wildcard for large files |
| Compressed export | `bq extract --compression=GZIP ds.table gs://bucket/*.json.gz` | GZIP, SNAPPY, or NONE |
| Export model | `bq extract -m ds.model gs://bucket/model` | For ML models |

### Resource Management

| Task | Command | Key Flags |
|------|---------|-----------|
| List datasets | `bq ls` or `bq ls PROJECT:` | `-a` for all (including hidden) |
| List tables | `bq ls DATASET` | `-m` for models, `-a` for all |
| List jobs | `bq ls -j PROJECT` | `--filter='state:RUNNING,PENDING'`, `--max_results` |
| Show table | `bq show ds.table` | `--schema`, `--format=prettyjson` |
| Show schema only | `bq show --schema ds.table` | Faster than full show |
| Preview data | `bq head ds.table` | `-n 100`, `-s 10` for offset |
| Show job | `bq show -j JOB_ID` | Check job status and details |

### Creating Resources

| Task | Command | Key Flags |
|------|---------|-----------|
| Create dataset | `bq mk DATASET` | `--location=us`, `--description` |
| Create table | `bq mk -t ds.table schema` | See schema format below |
| Create view | `bq mk --view='SELECT ...' ds.view` | SQL definition |
| Create materialized view | `bq mk --materialized_view='SELECT ...' ds.mview` | Auto-refreshed |
| Create partitioned table | `bq mk --table --time_partitioning_type=DAY ds.table schema` | See partitioning below |

### Deleting Resources

| Task | Command | Key Flags |
|------|---------|-----------|
| Delete table | `bq rm ds.table` | `-f` to skip confirmation |
| Delete dataset | `bq rm -r DATASET` | `-r` removes all tables, `-f` force |
| Delete model | `bq rm -m ds.model` | For ML models |
| Cancel job | `bq cancel JOB_ID` | Stops running query |

## Cost Estimation - ALWAYS USE DRY RUN

```bash
# ALWAYS check cost before running expensive queries
bq query --dry_run 'SELECT * FROM project.dataset.huge_table'

# Output shows bytes to be processed
# Cost = (Bytes / 1TB) × $6.25 (US, on-demand pricing)
# First 1 TB per month is free

# Add safety limit (in bytes)
bq query --maximum_bytes_billed=5000000000000 'SELECT ...'  # ~5TB limit
```

## Schema Format

**Inline (comma-separated):**
```
field1:STRING,field2:INTEGER,field3:FLOAT,field4:BOOLEAN,field5:TIMESTAMP
```

**With mode:**
```
field1:STRING:REQUIRED,field2:INTEGER:NULLABLE,field3:RECORD:REPEATED
```

**JSON file:**
```json
[
  {"name": "field1", "type": "STRING", "mode": "REQUIRED"},
  {"name": "field2", "type": "INTEGER", "mode": "NULLABLE"},
  {
    "name": "nested",
    "type": "RECORD",
    "fields": [
      {"name": "subfield", "type": "STRING"}
    ]
  }
]
```

**Common types:** STRING, INTEGER, FLOAT, BOOLEAN, TIMESTAMP, DATE, TIME, DATETIME, NUMERIC, BIGNUMERIC, BYTES, GEOGRAPHY, JSON, RECORD (nested), ARRAY

## Partitioning and Clustering

### Time-based Partitioning
```bash
# Partition by DATE/TIMESTAMP column
bq mk --table \
  --time_partitioning_type=DAY \
  --time_partitioning_field=event_date \
  --time_partitioning_expiration=2592000 \
  ds.events \
  event_id:STRING,event_date:DATE,data:STRING

# Require partition filter (prevents expensive full scans)
bq mk --table \
  --time_partitioning_type=DAY \
  --require_partition_filter=true \
  ds.events \
  schema.json
```

**Partition types:** DAY, HOUR, MONTH, YEAR

### Clustering
```bash
# Add clustering (up to 4 columns)
bq mk --table \
  --time_partitioning_type=DAY \
  --time_partitioning_field=event_date \
  --clustering_fields=user_id,region \
  ds.events \
  schema.json
```

**Benefits:** Faster queries, lower costs when filtering/grouping by clustered columns

## Output Formats

```bash
# Pretty table (default)
bq query --format=pretty 'SELECT ...'

# JSON (compact)
bq query --format=json 'SELECT ...'

# Pretty JSON (readable)
bq query --format=prettyjson 'SELECT ...'

# CSV with header
bq query --format=csv 'SELECT ...'

# Sparse table (simpler)
bq query --format=sparse 'SELECT ...'
```

## Common Workflows

### Cost Estimation → Query
```bash
# 1. Estimate cost
bq query --dry_run 'SELECT ...'

# 2. Review bytes to be processed
# Calculate: (bytes / 1099511627776) × $6.25

# 3. Run query if acceptable
bq query 'SELECT ...'

# OR add safety limit
bq query --maximum_bytes_billed=10000000000000 'SELECT ...'
```

### Load Data Pipeline
```bash
# 1. Check authentication and project
gcloud auth list
gcloud config get-value project

# 2. Create dataset if needed
bq ls | grep my_dataset || bq mk my_dataset

# 3. Load with autodetect (fast) or explicit schema (production)
bq load --autodetect --skip_leading_rows=1 \
  my_dataset.table \
  gs://bucket/data.csv

# 4. Verify
bq show my_dataset.table
bq head -n 10 my_dataset.table
```

### Export Large Table
```bash
# 1. Export with wildcard (required for >1GB)
bq extract \
  --compression=GZIP \
  --destination_format=NEWLINE_DELIMITED_JSON \
  dataset.large_table \
  'gs://bucket/export_*.json.gz'

# 2. Verify export
gsutil ls -lh gs://bucket/export_*

# 3. Check total size
gsutil du -sh gs://bucket/
```

### Create Partitioned Table for Scale
```bash
# 1. Create with partitioning and clustering
bq mk --table \
  --time_partitioning_type=DAY \
  --time_partitioning_field=event_date \
  --clustering_fields=user_id,event_type \
  --require_partition_filter=true \
  --time_partitioning_expiration=31536000 \
  dataset.events \
  event_id:STRING,user_id:STRING,event_type:STRING,event_date:DATE,data:JSON

# 2. Load initial data
bq load dataset.events gs://bucket/events_*.json

# 3. Query with partition filter (required)
bq query 'SELECT * FROM dataset.events WHERE event_date = "2025-01-20"'
```

## Best Practices

### Query Cost Optimization
1. **Always use --dry_run first** for queries scanning >1TB
2. **Add WHERE clauses** on partition columns
3. **SELECT specific columns**, not `SELECT *`
4. **Use --maximum_bytes_billed** as safety net
5. **Partition large tables** by date/timestamp
6. **Cluster by common filter columns**

### Loading Data
1. **Use --autodetect** for quick loads, explicit schema for production
2. **Load from GCS** (not local files) for large data
3. **Use appropriate source format**: Parquet/Avro > CSV/JSON
4. **Batch small files** into larger files before loading
5. **Set expiration** on staging tables

### Exporting Data
1. **Use wildcards** for large exports (>1GB limit per file)
2. **Compress exports** with GZIP or SNAPPY
3. **Use columnar formats** (Parquet, Avro) for analytics workflows
4. **Match regions** (BigQuery dataset and GCS bucket)

### Partitioning
1. **Partition by date/timestamp** for time-series data
2. **Use require_partition_filter** to prevent expensive scans
3. **Set partition expiration** to auto-delete old data
4. **Combine with clustering** for additional optimization
5. **Partition types**: DAY for most use cases, HOUR for high-volume

### Schema Design
1. **Use REQUIRED** for mandatory fields
2. **Use TIMESTAMP** over STRING for timestamps
3. **Use NUMERIC** for financial data (exact precision)
4. **Use JSON type** for flexible nested data (Standard SQL only)
5. **Nested records** better than wide tables with many columns

## Common Mistakes

| Mistake | Why It's Wrong | Correct Approach |
|---------|---------------|-----------------|
| No dry-run for large queries | Unexpected costs | Always `bq query --dry_run` first |
| Skipping authentication check | Commands fail | Run `gcloud auth list` before bq commands |
| SELECT * on huge tables | Scans all columns | Select only needed columns |
| Loading without --skip_leading_rows | Header becomes data | Use `--skip_leading_rows=1` for CSVs with headers |
| Single file for large exports | 1GB limit per file | Use wildcard: `gs://bucket/file_*.json.gz` |
| No partitioning on large tables | Expensive full scans | Use `--time_partitioning_type=DAY` |
| Legacy SQL (default in old versions) | Different syntax | Use `--use_legacy_sql=false` (or omit, it's default now) |
| Wrong dataset reference | Ambiguous table | Use fully qualified: `project.dataset.table` |
| No compression on exports | Larger GCS storage costs | Use `--compression=GZIP` |
| Forgetting location parameter | Cross-region costs | Match `--location` to dataset location |

## Red Flags - CHECK BEFORE RUNNING

- Running query without `--dry_run` on production data
- Using `SELECT *` on tables with >1TB
- Loading data without schema validation
- Exporting large table without wildcard
- Creating unpartitioned table for billions of rows
- No authentication check before commands
- Using wrong project (check `gcloud config get-value project`)
- Not matching dataset and GCS bucket regions

**All of these mean: Stop, review the command, fix the issue.**

## Dataset and Table References

**Fully qualified:**
```
project:dataset.table
# or
project.dataset.table
```

**Without project (uses default):**
```
dataset.table
```

**Partitioned table (specific date):**
```
dataset.table$20250120
```

**Legacy SQL (bracket notation):**
```
[project:dataset.table]
```

## Job Management

```bash
# List running jobs
bq ls -j --filter='state:RUNNING'

# Show job details
bq show -j JOB_ID

# Cancel long-running job
bq cancel JOB_ID

# Wait for job completion
bq wait JOB_ID
```

## Getting Help

```bash
# General help
bq help

# Command-specific help
bq help query
bq help load
bq help extract

# Show all flags
bq --helpfull
```

**When in doubt, check `bq help <command>` for exact flags and syntax.**

## Real-World Impact

**Cost savings:**
- Dry-run prevents accidental multi-thousand dollar queries
- Partitioning reduces scan costs by 10-100x
- Clustering adds 20-40% additional savings

**Performance:**
- Partitioning + clustering: queries 10-100x faster
- Proper schema: faster loads and queries
- Columnar formats (Parquet): 5-10x faster loads than CSV

**Common scenarios:**
- Ad-hoc analysis: `bq query --dry_run` → review → `bq query`
- Data pipelines: `bq load` from GCS → process → `bq extract`
- Cost monitoring: Always dry-run before production queries
- Scale: Partition + cluster tables with >100M rows
