---
name: aps-doc-ingestion
description: Expert documentation generation for ingestion layers. Automatically detects connector types (REST API, Database, File, Streaming), documents authentication patterns, rate limiting strategies, and incremental load patterns. Use when documenting data source ingestion workflows.
---

# APS Ingestion Documentation Expert

Specialized skill for generating comprehensive documentation for ingestion layers. Automatically detects and documents connector-specific patterns, authentication methods, rate limiting, and incremental strategies.

## When to Use This Skill

Use this skill when:
- Documenting a new data source ingestion workflow
- Creating documentation for REST API connectors (Salesforce, HubSpot, etc.)
- Documenting database ingestion (MySQL, PostgreSQL, BigQuery, etc.)
- Documenting file-based ingestion (S3, GCS, SFTP, etc.)
- Documenting streaming ingestion (Kafka, Kinesis, etc.)
- Creating parent-child documentation for multiple data sources

**Example requests:**
```
"Document the Klaviyo ingestion workflow"
"Create documentation for Salesforce API ingestion"
"Document all data sources in the ingestion layer"
"Generate ingestion documentation following this template: [Confluence URL]"
```

---

## 🚨 MANDATORY: Codebase Access Required

**WITHOUT codebase access = NO documentation. Period.**

**If no codebase access provided:**
```
I cannot create technical documentation without codebase access.

Required:
- Directory path to ingestion workflows
- Access to .dig, .yml configuration files

Without access, I cannot extract real table names, connectors, or incremental logic.
Provide path: "Code is in /path/to/ingestion/"
```

**Before proceeding:**
1. Ask for codebase path if not provided
2. Use Glob to verify files exist
3. STOP if cannot read files

**Documentation MUST contain:**
- Real connector names from .dig files
- Actual table names from datasources.yml
- Real incremental fields and schedules
- Working examples from actual configs

**NO generic placeholders. Only real, extracted data.**

---

## Layer-Specific Intelligence

### Auto-Detection Capabilities

This skill automatically detects and documents:

#### 1. Connector Type Detection

**REST API Connectors:**
```yaml
Detects from configuration:
- endpoint URLs (https://api.example.com/v1/...)
- HTTP methods (GET, POST, PUT)
- Pagination patterns (offset, cursor, page number)
- Response format (JSON, XML)

Documents:
- API endpoint structure
- Request/response examples
- Pagination strategy
- Response handling
```

**Database Connectors:**
```yaml
Detects from configuration:
- JDBC connection strings
- Query-based ingestion patterns
- Incremental query logic
- Connection parameters

Documents:
- Connection configuration
- Source queries
- Data type mappings
- Isolation levels
```

**File-Based Connectors:**
```yaml
Detects from configuration:
- S3/GCS bucket paths
- File patterns (*.csv, *.json, *.parquet)
- Compression formats (gzip, zip, snappy)
- File naming conventions

Documents:
- Bucket/path structure
- File format specifications
- Decompression logic
- File processing order
```

**Streaming Connectors:**
```yaml
Detects from configuration:
- Kafka topics/consumer groups
- Kinesis streams
- Partition strategies
- Offset management

Documents:
- Topic/stream configuration
- Consumer settings
- Checkpoint mechanisms
- Backpressure handling
```

#### 2. Authentication Pattern Detection

**OAuth 2.0:**
```yaml
Detects:
- Token endpoint URLs
- Client ID references
- Scope definitions
- Token refresh logic

Documents (securely):
- Authentication flow
- Token lifecycle
- Scope requirements
- Refresh strategy
(WITHOUT exposing secrets)
```

**API Key Authentication:**
```yaml
Detects:
- API key header names
- Key rotation patterns
- Rate limit tiers

Documents:
- Header configuration
- Key rotation schedule
- Usage tier limits
```

**Basic Authentication:**
```yaml
Detects:
- Username/password references
- Credential storage patterns

Documents:
- Authentication method
- Credential management
```

**Service Account / JWT:**
```yaml
Detects:
- Service account files
- JWT token generation
- Key expiration

Documents:
- Service account setup
- Token generation process
- Key rotation policy
```

#### 3. Rate Limiting Strategy Detection

```yaml
Detects from workflow:
- Request throttling (requests per second/minute)
- Retry backoff strategies (exponential, linear)
- Concurrent request limits
- Circuit breaker patterns

Documents:
- Rate limit thresholds
- Backoff algorithm
- Retry configuration
- Concurrent connection limits
```

#### 4. Incremental Load Pattern Detection

**Timestamp-Based:**
```yaml
Detects:
- updated_at, modified_at, created_at fields
- Timestamp comparison logic
- Watermark tracking

Documents:
- Incremental field name
- Timestamp format
- Watermark storage
- Lookback window
```

**Sequence-Based:**
```yaml
Detects:
- Auto-increment ID fields
- Sequence tracking
- Max ID queries

Documents:
- Sequence field name
- High-water mark logic
- Gap handling
```

**Full Reload:**
```yaml
Detects:
- No incremental field
- Full table scans
- Truncate-and-load patterns

Documents:
- Full reload schedule
- Data volume considerations
- Performance impact
```

## REQUIRED Documentation Template

**Follow this EXACT structure (analyzed from production examples):**

### For Parent Ingestion Page:

```markdown
## Overview
{Brief description of ingestion layer}

### Project Structure
{Directory tree from actual codebase}

## Main Ingestion Runner
**Workflow File**: ingestion_runner.dig
{Schedule, tasks, parallelization}

## Database Configuration
{Table with databases and purposes}

## Monitoring and Logging
{SQL queries for status checks}

## Individual Source Documentation
{Links to child pages}
```

### For Individual Source (Child Page):

```markdown
# {Source} Ingestion

## Overview
**Workflow Files:**
- {source}_ingest_inc.dig - Incremental
- {source}_ingest_hist.dig - Historical (if exists)

{Description}

**Data Source Type**: {type}
**Connector**: {connector name}
**Source System**: {system}
**Target Database**: {database}

---

## Configuration Files
{Table with file types and purposes}

---

## Active Tables (Incremental)
{Table with all incremental tables from datasources.yml}

## Active Tables (Historical)
{Table with all historical tables - if exists}

---

## Incremental Workflow Process

### Step 1: Log Ingestion Start
{Code snippet from workflow}

### Step 2: Setup Table and Time
{Explain create table logic + get last time logic}

### Step 3: Load Incremental Data
{Code snippet + query example}

### Step 4: Log Ingestion Success
{Code snippet}

---

## Historical Workflow Process
{Similar steps for historical if exists}

---

## Parallelization
{Explain _parallel settings and concurrency}

---

## Error Handling
{_error block from workflows}

---

## Authentication
{td_authentication_id reference}

---

## Data Flow Diagram
{Simple text diagram showing source → target}

---

## Incremental Logic
{Explain first run vs subsequent runs}

---

## Timestamp Format
{Document actual format from configs}

---

## Monitoring and Troubleshooting
{SQL queries for checking status, errors}

---

## Key Features
{Bullet list of main capabilities}

---

## Adding New Tables
{Step-by-step guide with real examples}

---

## Configuration Reference
{Sample datasource config + load config}

---

## Summary
{Brief recap of workflow capabilities}
```

---

## Summary

This skill generates production-ready ingestion documentation by:
- Reading actual .dig workflows and .yml configs from codebase
- Following the exact template structure shown above
- Extracting real table names, incremental fields, connectors
- Creating comprehensive, accurate documentation with working examples

**Key capability:** Transforms codebase into professional Confluence documentation.
