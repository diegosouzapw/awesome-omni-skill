---
name: catalyzed
description: Help with Catalyzed API - datasets, tables, pipelines, queries, vector search. Use when building Catalyzed integrations.
version: "1.0.0"
allowed-tools: WebFetch, Read, Write, Edit, Bash
---

# Catalyzed API Assistant

You are helping a developer integrate with the Catalyzed data platform.

## Live Documentation (fetched at load time)

!`curl -sf https://docs.catalyzed.ai/llms.txt 2>/dev/null`

## Quick Reference (fallback if above is empty)

**Base URL:** `https://api.catalyzed.ai`
**Docs:** `https://docs.catalyzed.ai`
**Auth:** `Authorization: Bearer <API_TOKEN>`

### Core Endpoints

| Resource | Method | Endpoint | Description |
|----------|--------|----------|-------------|
| **Teams** | GET | /teams | List user's teams |
| **Teams** | GET | /teams/:teamId | Get team by ID |
| **Datasets** | GET | /datasets | List datasets |
| **Datasets** | POST | /datasets | Create dataset |
| **Datasets** | GET | /datasets/:datasetId | Get dataset by ID |
| **Datasets** | PUT | /datasets/:datasetId | Update dataset |
| **Datasets** | DELETE | /datasets/:datasetId | Delete dataset |
| **Tables** | GET | /dataset-tables | List tables |
| **Tables** | POST | /dataset-tables | Create table with schema |
| **Tables** | GET | /dataset-tables/:tableId | Get table by ID |
| **Tables** | PUT | /dataset-tables/:tableId | Update table metadata |
| **Tables** | DELETE | /dataset-tables/:tableId | Delete table |
| **Rows** | POST | /dataset-tables/:tableId/rows?mode=append | Insert rows |
| **Rows** | POST | /dataset-tables/:tableId/rows?mode=upsert | Upsert rows |
| **Rows** | POST | /dataset-tables/:tableId/rows?mode=delete | Delete rows by PK |
| **Queries** | POST | /dataset-tables/:tableId/query | Execute SQL query |
| **Schema** | GET | /dataset-tables/:tableId/schema | Get table schema |
| **Pipelines** | GET | /pipelines | List pipelines |
| **Pipelines** | POST | /pipelines | Create pipeline |
| **Pipelines** | GET | /pipelines/:pipelineId | Get pipeline by ID |
| **Pipelines** | PUT | /pipelines/:pipelineId | Update pipeline |
| **Pipelines** | POST | /pipelines/:pipelineId/trigger | Trigger pipeline execution |
| **Executions** | GET | /pipeline-executions | List executions |
| **Executions** | GET | /pipeline-executions/:executionId | Get execution status |
| **Knowledge Bases** | GET | /knowledge-bases | List knowledge bases |
| **Knowledge Bases** | POST | /knowledge-bases | Create knowledge base |
| **Knowledge Bases** | GET | /knowledge-bases/:kbId | Get knowledge base |
| **Knowledge Bases** | POST | /knowledge-bases/:kbId/sources | Add source (file or table) |
| **Knowledge Bases** | POST | /knowledge-bases/:kbId/query | Vector search query |
| **Files** | POST | /files | Upload file |
| **Files** | GET | /files/:fileId | Get file metadata |
| **API Tokens** | GET | /api-tokens | List API tokens |
| **API Tokens** | POST | /api-tokens | Create API token |

### Authentication

All API requests require a Bearer token in the Authorization header:

```bash
curl -H "Authorization: Bearer cat_YOUR_API_TOKEN" \
  https://api.catalyzed.ai/teams
```

API tokens are created via the Catalyzed dashboard or API. Tokens have a `cat_` prefix.

### Common Patterns

**Create a dataset and table:**
```bash
# Create dataset
curl -X POST https://api.catalyzed.ai/datasets \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"teamId": "...", "name": "My Dataset"}'

# Create table with schema
curl -X POST https://api.catalyzed.ai/dataset-tables \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "datasetId": "...",
    "tableName": "events",
    "fields": [
      {"name": "id", "arrowType": "utf8", "nullable": false},
      {"name": "timestamp", "arrowType": "timestamp[us]", "nullable": false},
      {"name": "data", "arrowType": "utf8", "nullable": true}
    ],
    "primaryKeyColumns": ["id"]
  }'
```

**Insert data:**
```bash
curl -X POST "https://api.catalyzed.ai/dataset-tables/:tableId/rows?mode=append" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '[
    {"id": "1", "timestamp": "2025-01-01T00:00:00Z", "data": "hello"},
    {"id": "2", "timestamp": "2025-01-02T00:00:00Z", "data": "world"}
  ]'
```

**Query data:**
```bash
curl -X POST https://api.catalyzed.ai/dataset-tables/:tableId/query \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"sql": "SELECT * FROM events WHERE timestamp > '\''2025-01-01'\''", "maxRows": 100}'
```

### Fetching Detailed Documentation

When the user needs details on a specific topic, use WebFetch to get the markdown version:

- Quickstart: `https://docs.catalyzed.ai/getting-started/quickstart.md`
- Querying Data: `https://docs.catalyzed.ai/guides/querying-data.md`
- Ingesting Data: `https://docs.catalyzed.ai/guides/ingesting-data.md`
- Pipelines: `https://docs.catalyzed.ai/concepts/pipelines.md`
- Tables: `https://docs.catalyzed.ai/concepts/tables.md`
- Knowledge Bases: `https://docs.catalyzed.ai/concepts/knowledge-bases.md`
- Vector Search: `https://docs.catalyzed.ai/guides/vector-search.md`
- Full index: `https://docs.catalyzed.ai/llms.txt`

For code examples, see [examples.md](examples.md).
For full API reference, see [api-reference.md](api-reference.md).
