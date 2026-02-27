---
name: add-datalake-consumer
description: Adds an event consumer that writes to Azure Data Lake (Parquet) following BI_SALES_RISK plan. Creates events/consumers/[Name]DataLakeCollector.ts subscribing to RabbitMQ, building Parquet rows, writing to /path_prefix/year=YYYY/month=MM/day=DD/. Use when adding DataLakeCollector in logging or similar “event to Data Lake” pipelines.
---

# Add Data Lake Consumer

Event consumer that subscribes to RabbitMQ and writes to **Azure Data Lake** (Parquet). Pattern: logging’s DataLakeCollector for `risk.evaluated` (BI_SALES_RISK_IMPLEMENTATION_PLAN §3.5, §9.1). **BI Sales Risk:** Paths and Parquet columns MUST match `documentation/requirements/BI_SALES_RISK_DATA_LAKE_LAYOUT.md` (§2.1 risk.evaluated, §2.2 ml_outcomes, §4 config).

## 1. Consumer

**Path:** `src/events/consumers/[Name]DataLakeCollector.ts`

- `EventConsumer` with `queue`, `exchange: coder_events`, `bindings`: e.g. `['risk.evaluated','ml.prediction.completed','opportunity.updated','forecast.generated']`.
- Handler: map event to row. For risk.evaluated use columns in Data Lake Layout §2.1. Build path: `{path_prefix}/year={YYYY}/month={MM}/day={DD}/...` (Layout §1).
- Write via `@azure/storage-blob` (BlockBlob) or `@azure/storage-blob` + `parquetjs` (or Arrow) for Parquet. Buffer/batch by time or count if needed.
- Config: `data_lake.connection_string`, `data_lake.container`, `data_lake.path_prefix` (e.g. `/risk_evaluations`).

## 2. Config

**config/default.yaml:**
```yaml
data_lake:
  connection_string: ${DATA_LAKE_CONNECTION_STRING}
  container: ${DATA_LAKE_CONTAINER:-risk}
  path_prefix: ${DATA_LAKE_PATH_PREFIX:-/risk_evaluations}

rabbitmq:
  url: ${RABBITMQ_URL}
  exchange: coder_events
  queue: [module]_data_lake
  bindings:
    - risk.evaluated
    - ml.prediction.completed
    # ...
```

**config/schema.json:** add `data_lake` with `connection_string`, `container`, `path_prefix`.

## 3. Server

In `server.ts`: `await dataLakeCollector.start()` after RabbitMQ connect.

## 4. Checklist

- [ ] Consumer in `events/consumers/`, subscribe to RabbitMQ (no Azure Service Bus)
- [ ] Path: `{path_prefix}/year=.../month=.../day=.../`; format Parquet
- [ ] Config: `data_lake.*` and schema; `rabbitmq` queue and bindings
- [ ] Start collector in server
