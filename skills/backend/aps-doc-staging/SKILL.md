---
name: aps-doc-staging
description: Expert documentation generation for staging transformation layers. Auto-detects SQL engine (Presto/Trino vs Hive), documents transformation rules, PII handling, deduplication strategies, and data quality rules. Use when documenting staging transformations.
---

# APS Staging Transformation Documentation Expert

Specialized skill for generating comprehensive documentation for staging transformation layers. Automatically detects SQL engines, extracts transformation rules, documents PII handling, and analyzes deduplication strategies.

## When to Use This Skill

Use this skill when:
- Documenting staging transformation workflows
- Creating documentation for data cleaning and standardization logic
- Documenting PII handling and security transformations
- Creating documentation for deduplication strategies
- Documenting data quality rules and validations
- Generating documentation for Presto/Trino or Hive transformations

**Example requests:**
```
"Document the staging transformation for customer events"
"Create staging layer documentation with transformation rules"
"Document PII handling in staging transformations"
"Generate staging documentation following this template: [Confluence URL]"
```

---

## 🚨 MANDATORY: Codebase Access Required

**WITHOUT codebase access = NO documentation. Period.**

**If no codebase access provided:**
```
I cannot create technical documentation without codebase access.

Required:
- Directory path to staging workflows
- Access to .dig, .sql, .yml files

Without access, I cannot extract real transformation SQL, PII logic, or table names.
Provide path: "Code is in /path/to/staging/"
```

**Before proceeding:**
1. Ask for codebase path if not provided
2. Use Glob to verify SQL files exist
3. STOP if cannot read files

**Documentation MUST contain:**
- Real transformation SQL from .sql files
- Actual PII hashing/masking logic
- Real table/column names
- Working SQL examples from code

**NO generic placeholders. Only real, extracted data.**
## REQUIRED Documentation Template

**Follow this EXACT structure (analyzed from production examples):**

```markdown
# Staging Transformation - {Engine} Engine

## Overview
**Engine**: {Presto/Trino or Hive}
**Architecture**: {Loop-based / Other}
**Processing Mode**: {Incremental / Full}
**Location**: {directory path}

### Key Characteristics
{List key features from actual workflow}

---

## Architecture Overview

### Directory Structure
{Actual directory tree from codebase}

### Core Components

#### 1. Main Workflow File
{Name and purpose}

**Key Features:**
- {Feature from actual .dig file}
- {Feature from actual .dig file}

**Workflow Phases:**
{Extract from actual workflow}

#### 2. Configuration File
{Name and structure from actual codebase}

**Configuration Structure:**
{Real YAML structure}

**Table Configuration Fields:**
{Document actual fields used}

#### 3. SQL Transformation Files
{Types: init, incremental, upsert - from actual codebase}

---

## Processing Flow

### Initial Load (First Run)
{Step-by-step from actual workflow}

### Incremental Load (Subsequent Runs)
{Step-by-step from actual workflow}

---

## Data Transformation Rules

{Document ACTUAL transformation rules from codebase}

### 1. Date/Timestamp Processing
{Real SQL examples from transformation files}

### 2. String Standardization
{Real SQL examples}

### 3. JSON Extraction
{Real examples if exists}

### 4. Email Processing
{Real examples if exists}

### 5. Phone Number Processing
{Real examples if exists}

### 6. Deduplication Logic
{Real ROW_NUMBER() or DISTINCT logic}

### 7. Metadata Columns
{Real source_system, load_timestamp columns}

---

## Table-Specific Transformation Rules

{If using reference table like staging_trnsfrm_rules:}

**Reference Table**: {database}.{table}
**Purpose**: {explain}

**Schema**: {real schema}

**How Used**: {explain how workflow reads these rules}

---

## Current Implementation

**Configured Tables**:
{List actual tables from config}

---

## How to Add New Source Tables

{Step-by-step with real examples}

---

## Monitoring & Troubleshooting

**Key Queries**:
{Real SQL for checking status, data quality}

**Common Issues**:
{Real issues and solutions}

---

## Best Practices

{List from actual production experience}

---

## Summary

{Brief recap of capabilities}
```

---

**Template Usage Notes:**
- Read actual workflows (.dig), configs (.yml), SQL files
- Extract REAL transformation logic from SQL
- Document REAL deduplication strategies
- Use actual table/column names from codebase
- Include working SQL examples
- NO placeholders - only real extracted data

## Summary

This skill generates production-ready staging documentation by:
- Reading actual .dig workflows, .yml configs, and .sql files
- Following the exact template structure shown above  
- Extracting real transformation rules from SQL
- Documenting actual deduplication logic
- Creating comprehensive documentation with working SQL examples

**Key capability:** Transforms staging codebase into professional Confluence documentation with all transformation rules documented.
