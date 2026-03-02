---
name: bronze-layer-setup
description: End-to-end Bronze layer creation for testing and demos. Creates table DDLs, generates fake data with Faker, copies from existing sources, and configures Asset Bundle jobs. Covers Unity Catalog compliance, Change Data Feed, automatic liquid clustering, and governance metadata. Use when setting up Bronze layer tables, creating test/demo data, rapid prototyping Medallion Architecture, or bootstrapping a new Databricks project. For Faker-specific patterns (corruption rates, function signatures, provider examples), load the faker-data-generation skill.
metadata:
  author: prashanth subrahmanyam
  version: "2.0"
  domain: bronze
  role: orchestrator
  pipeline_stage: 2
  pipeline_stage_name: bronze
  reads:
    - context/*.csv
  next_stages:
    - silver-layer-setup
  workers:
    - faker-data-generation
  common_dependencies:
    - databricks-asset-bundles
    - databricks-expert-agent
    - databricks-python-imports
    - databricks-table-properties
    - schema-management-patterns
    - naming-tagging-standards
    - databricks-autonomous-operations
  last_verified: "2026-02-07"
  volatility: low
  upstream_sources: []  # Internal setup patterns
---

# Bronze Layer Setup

Create Bronze layer tables with test data for rapid prototyping of Medallion Architecture.

## When to Use

- Setting up Bronze layer tables for a new project
- Creating test/demo data for Silver/Gold layer development
- Rapid prototyping of Medallion Architecture
- Bootstrapping a Databricks project with realistic test data
- Copying data from existing sources to a new Bronze schema

**For Faker-specific patterns** (corruption rates, function signatures, provider examples), the `faker-data-generation` worker skill is loaded at Step 4 via the Mandatory Skill Dependencies table below.

## Core Philosophy

The Bronze layer in this approach is optimized for **testing, demos, and rapid prototyping**:

- Quick setup with realistic test data
- Faker data generation as the primary method
- Unity Catalog compliance (proper governance metadata)
- Change Data Feed enabled for downstream Silver/Gold testing
- Automatic liquid clustering for query optimization
- Flexible data sources (generate, copy, or reference existing)
- **NOT for production ingestion** (use separate ingestion pipelines for that)

### 🔴 Non-Negotiable Defaults (Applied to EVERY Bronze Table and Job)

These defaults are ALWAYS applied. There are NO exceptions, NO overrides, NO alternative options.

| Default | Value | Applied Where | NEVER Do This Instead |
|---------|-------|---------------|----------------------|
| **Serverless** | `environments:` block with `environment_key` | Every job YAML | ❌ NEVER define `job_clusters:` or `existing_cluster_id:` |
| **Environments V4** | `environment_version: "4"` | Every job's `environments.spec` | ❌ NEVER omit or use older versions |
| **Auto Liquid Clustering** | `CLUSTER BY AUTO` | Every `CREATE TABLE` DDL | ❌ NEVER use `CLUSTER BY (col1, col2)` or `PARTITIONED BY` |
| **Change Data Feed** | `'delta.enableChangeDataFeed' = 'true'` | Every table's TBLPROPERTIES | ❌ NEVER omit (required for Silver streaming) |
| **Auto-Optimize** | `'delta.autoOptimize.optimizeWrite' = 'true'` | Every table's TBLPROPERTIES | ❌ NEVER omit |
| **notebook_task** | `notebook_task:` with `base_parameters:` | Every task in job YAML | ❌ NEVER use `python_task:` or CLI-style `parameters:` |

```sql
-- ✅ CORRECT: Every Bronze table DDL MUST include
CREATE TABLE IF NOT EXISTS {catalog}.{schema}.{table_name} (
    ...
)
USING DELTA
CLUSTER BY AUTO          -- 🔴 MANDATORY
TBLPROPERTIES (
    'delta.enableChangeDataFeed' = 'true',          -- 🔴 MANDATORY
    'delta.autoOptimize.optimizeWrite' = 'true',    -- 🔴 MANDATORY
    'delta.autoOptimize.autoCompact' = 'true',      -- 🔴 MANDATORY
    'layer' = 'bronze'
)
```

```yaml
# ✅ CORRECT: Every Bronze job MUST include
environments:
  - environment_key: "default"
    spec:
      environment_version: "4"   # 🔴 MANDATORY
tasks:
  - task_key: setup_tables
    environment_key: default     # 🔴 MANDATORY on every task
    notebook_task:               # 🔴 MANDATORY (never python_task)
      notebook_path: ../src/setup_tables.py
      base_parameters:           # 🔴 MANDATORY (never CLI-style parameters)
        catalog: ${var.catalog}
```

## Quick Start (30 minutes)

**What You'll Create:**
1. `setup_tables.py` - DDL definitions for all Bronze tables
2. `generate_dimensions.py` - Faker-based dimension data generator
3. `generate_facts.py` - Faker-based fact data generator (with FK integrity)
4. `bronze_setup_job.yml` + `bronze_data_generator_job.yml` - Asset Bundle jobs

**Deployment Commands (run when ready — NOT auto-executed by this skill):**
```bash
# 1. Deploy setup job
databricks bundle deploy -t dev

# 2. Create tables
databricks bundle run bronze_setup_job -t dev

# 3. Generate data (dimensions -> facts in sequence)
databricks bundle run bronze_data_generator_job -t dev
```

**Key Decisions:**
- **Data Source:** Faker (recommended) | Existing tables | External copy
- **Record Counts:** Dimensions: 100-200 | Facts: 1,000-10,000
- **Tables Needed:** 5-10 tables (dimensions + facts)

**Output:** Bronze Delta tables with Change Data Feed enabled, ready for Silver layer testing

## Working Memory Management

This orchestrator spans 5 steps (Step 6 is user-triggered). To maintain coherence without context pollution:

**After each step, persist a brief summary note** capturing:
- **Step 1 output:** Requirements filled — project name, entity list, data source approach, record counts
- **Step 2 output:** Data source decision (Faker / existing / external), rationale
- **Step 3 output:** DDL file path (`setup_tables.py`), count of tables defined, any schema deviations
- **Step 4 output:** Generator file paths, Faker config decisions (providers, FK integrity strategy)
- **Step 5 output:** Job YAML file paths, `databricks.yml` sync status
- **Step 6 output (if user-triggered):** Deployment results, row counts per table, CDF verification status

**What to keep in working memory:** Only the current step's context, the table list from Step 1, and the previous step's summary note. Discard intermediate outputs (full DDL strings, generated DataFrames) — they are on disk and reproducible.

## Workflow

### Step 1: Gather Requirements (15 min)

Fill in the requirements template: [references/requirements-template.md](references/requirements-template.md)

- Project name, entity list (5-10 tables), data source approach
- Domain taxonomy, data classification, record counts
- Business/technical ownership

### Step 2: Choose Data Source Approach

Three approaches detailed in [references/data-source-approaches.md](references/data-source-approaches.md):

| Approach | Best For | Time |
|---|---|---|
| **A: Schema CSV + Faker** (recommended) | Create Bronze tables matching customer's source schema from `context/*.csv`, then populate with Faker-generated data | 30-45 min |
| **B: Existing tables** | Data already exists in Databricks — read it directly | 15-20 min |
| **C: External copy** | Copy data from sample datasets, CSVs, databases, or other workspaces | 20-30 min |

**Approach A (Schema CSV + Faker)** is the standard approach for this framework. It reads the customer's source schema CSV from `context/` to create Bronze DDLs with the **exact same table structure**, then generates Faker data matching those column types and FK relationships. This ensures Bronze faithfully represents the customer's source system.

```python
# ✅ CORRECT: Read schema CSV to build Bronze DDLs
import csv
from pathlib import Path

def extract_tables_from_schema_csv(csv_path: Path) -> dict:
    """Extract table definitions from customer schema CSV."""
    from collections import defaultdict
    tables = defaultdict(list)
    with open(csv_path) as f:
        reader = csv.DictReader(f)
        for row in reader:
            tables[row["table_name"]].append({
                "name": row["column_name"],
                "type": row.get("full_data_type", row.get("data_type", "STRING")),
                "nullable": row.get("is_nullable", "YES") == "YES",
                "comment": row.get("comment", ""),
            })
    return dict(tables)

# Extract tables from customer schema
schema_tables = extract_tables_from_schema_csv(Path("context/Wanderbricks_Schema.csv"))

# Generate DDL for each table — names and types come from CSV, never hardcoded
for table_name, columns in schema_tables.items():
    col_defs = ", ".join(f"{c['name']} {c['type']}" for c in columns)
    ddl = f"CREATE TABLE IF NOT EXISTS {{catalog}}.{{schema}}.{table_name} ({col_defs}) ..."
```

```python
# ❌ WRONG: Hardcoding table definitions instead of extracting from schema CSV
tables = {
    "bookings": ["booking_id BIGINT", "user_id BIGINT", ...],  # ❌ Might be incomplete
    "users": ["user_id BIGINT", "email STRING", ...],           # ❌ Might have wrong types
}
```

### Step 3: Create Table DDLs (30 min)

Use the setup script template: [scripts/setup_tables.py](scripts/setup_tables.py)

**File structure to create:**
```
src/{project}_bronze/
├── __init__.py                # Package initialization
├── setup_tables.py            # Table DDL definitions
├── generate_dimensions.py     # Generate dimension data with Faker
├── generate_facts.py          # Generate fact data with Faker
└── copy_from_source.py        # Optional: Copy from existing source
```

**Critical DDL rules:**
- `CLUSTER BY AUTO` on all tables (never specify columns manually)
- `delta.enableChangeDataFeed = true` (required for Silver)
- Standard audit columns: `ingestion_timestamp`, `source_file`
- Mark tables as `data_purpose = testing_demo`, `is_production = false`

### Step 4: Generate or Load Data (30-45 min)

**Option A (Faker):** Use the `faker-data-generation` skill for patterns.
- Generate dimensions first (for FK integrity)
- Generate facts with references to dimension keys
- Use seeded Faker for reproducibility

**Option B/C (Copy):** Use the copy script template: [scripts/copy_from_source.py](scripts/copy_from_source.py)

### Step 5: Configure Asset Bundle Jobs (15 min)

Use the job templates:
- [assets/templates/bronze-setup-job.yaml](assets/templates/bronze-setup-job.yaml) - Table creation job
- [assets/templates/bronze-data-generator-job.yaml](assets/templates/bronze-data-generator-job.yaml) - Data generation job

---

### 🛑 STOP — Artifact Creation Complete

**Steps 1–5 are complete.** All files (DDLs, generators, job YAMLs) have been created. **Do NOT proceed to Step 6 unless the user explicitly requests deployment.**

Report what was created and ask the user if they want to deploy and run.

---

### Step 6: Deploy & Validate (15 min) — USER-TRIGGERED ONLY

> **This step is executed ONLY when the user explicitly requests deployment.** Do not auto-execute.

Run validation queries: [references/validation-queries.md](references/validation-queries.md)

## Critical Rules

### Required TBLPROPERTIES

Every Bronze table must include:

```sql
TBLPROPERTIES (
    'delta.enableChangeDataFeed' = 'true',
    'delta.autoOptimize.optimizeWrite' = 'true',
    'delta.autoOptimize.autoCompact' = 'true',
    'layer' = 'bronze',
    'source_system' = '{source}',
    'domain' = '{domain}',
    'entity_type' = '{dimension|fact}',
    'contains_pii' = '{true|false}',
    'data_classification' = '{confidential|internal|public}',
    'business_owner' = '{team}',
    'technical_owner' = 'Data Engineering',
    'data_purpose' = 'testing_demo',
    'is_production' = 'false'
)
```

### Table Naming Convention

- Dimensions: `bronze_{entity}_dim` (e.g., `bronze_store_dim`, `bronze_product_dim`)
- Facts: `bronze_{entity}` (e.g., `bronze_transactions`, `bronze_inventory`)
- Date dimension: `bronze_date_dim` (SQL-generated, not Faker)

### Data Generation Order

1. **Dimensions first** - Create master data tables
2. **Date dimension** - Generated via SQL SEQUENCE (not Faker)
3. **Facts last** - Load dimension keys for FK integrity

## Mandatory Skill Dependencies

**MANDATORY: Read each skill below using the Read tool BEFORE writing any code for the indicated step. Do NOT generate these patterns from memory.**

| Step | Read Skill (MANDATORY) | What It Provides |
|------|------------------------|------------------|
| All steps | `data_product_accelerator/skills/common/databricks-expert-agent/SKILL.md` | Core extraction principle: extract names from source, never hardcode |
| Step 3 (DDLs) | `data_product_accelerator/skills/common/databricks-table-properties/SKILL.md` | Bronze TBLPROPERTIES, `CLUSTER BY AUTO`, governance metadata |
| Step 3 (DDLs) | `data_product_accelerator/skills/common/schema-management-patterns/SKILL.md` | `CREATE SCHEMA IF NOT EXISTS`, Predictive Optimization |
| Step 4 (Data) | `data_product_accelerator/skills/bronze/01-faker-data-generation/SKILL.md` | Faker corruption patterns, function signatures, provider examples |
| Step 5 (Jobs) | `data_product_accelerator/skills/common/databricks-asset-bundles/SKILL.md` | Serverless job YAML, Environments V4, `notebook_task`, `base_parameters` |
| Step 5 (Jobs) | `data_product_accelerator/skills/common/databricks-python-imports/SKILL.md` | Pure Python import patterns for notebook code sharing |
| Step 6 (if user-triggered) | `data_product_accelerator/skills/common/databricks-autonomous-operations/SKILL.md` | Deploy → Poll → Diagnose → Fix → Redeploy loop when jobs fail |

**NEVER do these without FIRST reading the corresponding skill:**
- NEVER write `TBLPROPERTIES` without reading `databricks-table-properties`
- NEVER write Faker generators without reading `faker-data-generation`
- NEVER write Asset Bundle YAML without reading `databricks-asset-bundles`
- NEVER write `CREATE SCHEMA` without reading `schema-management-patterns`

## Reference Files

- **[references/requirements-template.md](references/requirements-template.md)** - Fill-in template for project requirements, entity list, data classification, ownership
- **[references/data-source-approaches.md](references/data-source-approaches.md)** - Detailed patterns for all 3 data source approaches (Faker, Existing, Copy)
- **[references/validation-queries.md](references/validation-queries.md)** - Validation SQL queries and implementation checklist

## Scripts

- **[scripts/setup_tables.py](scripts/setup_tables.py)** - Template table creation notebook with DDL patterns, audit columns, governance metadata
- **[scripts/copy_from_source.py](scripts/copy_from_source.py)** - Template copy-from-source notebook for Approach B/C

## Asset Templates

- **[assets/templates/bronze-setup-job.yaml](assets/templates/bronze-setup-job.yaml)** - Asset Bundle job for table creation
- **[assets/templates/bronze-data-generator-job.yaml](assets/templates/bronze-data-generator-job.yaml)** - Asset Bundle job for Faker data generation with dependency chain

## Pipeline Progression

**Previous stage:** `gold/00-gold-layer-design` → Gold layer design must be complete so that the target dimensional model is understood before creating Bronze tables

**Next stage:** After completing the Bronze layer, proceed to:
- **`silver/00-silver-layer-setup`** — Set up Silver layer DLT pipelines with data quality rules

---

## Post-Completion: Skill Usage Summary (MANDATORY)

**After completing all steps of this orchestrator, output a Skill Usage Summary reflecting what you ACTUALLY did — not a pre-written summary.**

### What to Include

1. Every skill `SKILL.md` or `references/` file you read (via the Read tool), in the order you read them
2. Which step you were in when you read it
3. Whether it was a **Worker**, **Common**, **Cross-domain**, or **Reference** file
4. A one-line description of what you specifically used it for in this session

### Format

| # | Step | Skill / Reference Read | Type | What It Was Used For |
|---|------|----------------------|------|---------------------|
| 1 | Step N | `path/to/SKILL.md` | Worker / Common / Reference | One-line description |

### Summary Footer

End with:
- **Totals:** X worker skills, Y common skills, Z reference files read across N steps
- **Skipped:** List any skills from the dependency table above that you did NOT need to read, and why (e.g., "step not applicable", "user skipped", "no issues encountered")
- **Unplanned:** List any skills you read that were NOT listed in the dependency table (e.g., for troubleshooting, edge cases, or user-requested detours)

---

## References

- [Unity Catalog](https://docs.databricks.com/unity-catalog/)
- [Delta Lake](https://docs.databricks.com/delta/)
- [Asset Bundles](https://docs.databricks.com/dev-tools/bundles/)
- [Faker Library](https://faker.readthedocs.io/)
