---
name: database-design
description: Designs database schemas, data models, relationships, indexes, and migrations for relational, NoSQL, time-series, and warehouse databases. Covers normalization, denormalization, ETL optimization, event sourcing, star schema, and performance tuning. Trigger keywords: schema, table, column, migration, ERD, normalize, denormalize, index, foreign key, primary key, constraint, relationship, SQL, DDL, data model, database design, data warehouse, star schema, snowflake schema, time-series, event sourcing, dimension table, fact table, ETL, data pipeline, OLAP, OLTP.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

# Database Design

## Overview

This skill focuses on designing efficient, scalable, and maintainable database schemas and data models. It covers:

- **OLTP Systems**: Relational databases (PostgreSQL, MySQL) with normalization and transactional integrity
- **OLAP Systems**: Data warehouses with star/snowflake schemas for analytics
- **NoSQL**: Document stores (MongoDB), key-value (Redis), wide-column (Cassandra)
- **Time-Series**: Specialized databases for metrics and events (TimescaleDB, InfluxDB)
- **Event Sourcing**: Append-only event stores for audit and temporal queries
- **Data Pipelines**: Schema design considerations for ETL/ELT workflows

This skill incorporates data modeling expertise for both operational and analytical workloads.

## Instructions

### 1. Understand Data Requirements

- Identify entities and their attributes
- Map relationships between entities (one-to-one, one-to-many, many-to-many)
- Determine data access patterns (read vs write heavy, query patterns)
- Estimate data volumes, growth rate, and retention requirements
- Distinguish OLTP (transactional) vs OLAP (analytical) needs

### 2. Design Schema

**For OLTP (Transactional Systems)**:

- Normalize to 3NF to eliminate redundancy
- Define primary keys (surrogate vs natural)
- Establish foreign key relationships with appropriate cascade rules
- Choose appropriate data types for storage efficiency
- Plan for NULL handling and default values
- Add CHECK constraints for data integrity

**For OLAP (Data Warehouses)**:

- Design star schema (central fact table with dimension tables)
- Or snowflake schema (normalized dimensions) if cardinality is high
- Create slowly changing dimensions (SCD Type 1, 2, or 3)
- Denormalize for query performance
- Add surrogate keys for dimension tables
- Design fact tables with foreign keys to dimensions and measure columns

**For Time-Series**:

- Use timestamp as primary key component
- Partition by time ranges (day, week, month)
- Design for append-only writes
- Consider downsampling and aggregation tables
- Use appropriate retention policies

**For Event Sourcing**:

- Store events as immutable append-only records
- Include event type, aggregate ID, timestamp, payload
- Design projections for read models
- Plan for event versioning and schema evolution

### 3. Optimize for Performance

- Design indexes for query patterns (WHERE, JOIN, ORDER BY, GROUP BY)
- Consider covering indexes to avoid table lookups
- Use partial indexes for filtered queries
- Plan denormalization for read-heavy workloads
- Design partitioning strategy for large tables (range, hash, list)
- Add materialized views for expensive aggregations
- Design for concurrent access (optimistic vs pessimistic locking)

### 4. Plan Migrations

- Create reversible migrations with UP and DOWN scripts
- Handle data transformations safely (backfill, defaults)
- Plan for zero-downtime deployments (expand/contract pattern)
- Version control all schema changes
- Test migrations on production-like data volumes
- Document breaking changes and migration dependencies

### 5. Consider ETL/Data Pipeline Impact

- Design schemas that support efficient bulk loading
- Add staging tables for incremental updates
- Include audit columns (created_at, updated_at, loaded_at)
- Plan for change data capture (CDC) if needed
- Design idempotent upsert operations
- Consider schema evolution and backward compatibility

## Best Practices

1. **Choose Appropriate Types**: Use correct data types for storage efficiency (INT vs BIGINT, VARCHAR vs TEXT, DECIMAL vs FLOAT)
2. **Index Wisely**: Index columns used in WHERE, JOIN, ORDER BY, GROUP BY, but avoid over-indexing (write cost)
3. **Normalize First**: Start normalized (3NF) for OLTP, denormalize strategically for OLAP or read-heavy workloads
4. **Use Constraints**: Enforce data integrity at database level (PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK, NOT NULL)
5. **Plan for Scale**: Consider sharding, partitioning, and replication early for high-volume tables
6. **Document Schemas**: Maintain ERD, data dictionary, and relationship diagrams
7. **Test Migrations**: Always test on production-like data volumes and monitor performance
8. **Audit Everything**: Add created_at, updated_at, created_by for accountability
9. **Version Events**: For event sourcing, include schema version in event payload
10. **Optimize for Cardinality**: High-cardinality columns benefit from indexes, low-cardinality may not
11. **Separate Reads from Writes**: For high-scale systems, consider CQRS pattern with separate read/write models
12. **Design for Idempotency**: Ensure ETL operations can safely retry without duplicates

## Examples

### Example 1: E-Commerce Schema (PostgreSQL)

```sql
-- Users table with proper constraints
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at TIMESTAMPTZ,

    CONSTRAINT email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

-- Products with proper indexing
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    sku VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL CHECK (price >= 0),
    stock_quantity INTEGER NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
    category_id UUID REFERENCES categories(id),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes for common queries
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_price ON products(price);
CREATE INDEX idx_products_name_search ON products USING gin(to_tsvector('english', name));

-- Orders with proper relationships
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    total_amount DECIMAL(10, 2) NOT NULL,
    shipping_address JSONB NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT valid_status CHECK (status IN ('pending', 'paid', 'shipped', 'delivered', 'cancelled'))
);

CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created ON orders(created_at DESC);

-- Order items junction table
CREATE TABLE order_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id UUID NOT NULL REFERENCES products(id),
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10, 2) NOT NULL,

    UNIQUE(order_id, product_id)
);
```

### Example 2: Migration Script

```sql
-- Migration: Add customer loyalty program
-- Version: 20240115_001

BEGIN;

-- Add loyalty tier to users
ALTER TABLE users
ADD COLUMN loyalty_tier VARCHAR(20) DEFAULT 'bronze',
ADD COLUMN loyalty_points INTEGER DEFAULT 0;

-- Add constraint for valid tiers
ALTER TABLE users
ADD CONSTRAINT valid_loyalty_tier
CHECK (loyalty_tier IN ('bronze', 'silver', 'gold', 'platinum'));

-- Create points history table
CREATE TABLE loyalty_points_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    points_change INTEGER NOT NULL,
    reason VARCHAR(100) NOT NULL,
    reference_type VARCHAR(50),
    reference_id UUID,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_loyalty_history_user ON loyalty_points_history(user_id);
CREATE INDEX idx_loyalty_history_created ON loyalty_points_history(created_at DESC);

COMMIT;

-- Rollback script (save separately)
-- BEGIN;
-- DROP TABLE IF EXISTS loyalty_points_history;
-- ALTER TABLE users DROP COLUMN IF EXISTS loyalty_points;
-- ALTER TABLE users DROP COLUMN IF EXISTS loyalty_tier;
-- COMMIT;
```

### Example 3: MongoDB Document Design

```javascript
// User document with embedded addresses
{
  _id: ObjectId("..."),
  email: "user@example.com",
  profile: {
    name: "John Doe",
    avatar_url: "https://..."
  },
  addresses: [
    {
      type: "shipping",
      street: "123 Main St",
      city: "Boston",
      state: "MA",
      zip: "02101",
      is_default: true
    }
  ],
  preferences: {
    newsletter: true,
    notifications: {
      email: true,
      push: false
    }
  },
  created_at: ISODate("2024-01-15T10:00:00Z")
}

// Indexes
db.users.createIndex({ email: 1 }, { unique: true });
db.users.createIndex({ "addresses.zip": 1 });
db.users.createIndex({ created_at: -1 });
```

### Example 4: Data Warehouse Star Schema (PostgreSQL)

```sql
-- Dimension: Date (conformed dimension)
CREATE TABLE dim_date (
    date_key INTEGER PRIMARY KEY,  -- YYYYMMDD format
    full_date DATE NOT NULL,
    day_of_week INTEGER,
    day_name VARCHAR(10),
    month INTEGER,
    month_name VARCHAR(10),
    quarter INTEGER,
    year INTEGER,
    is_weekend BOOLEAN,
    is_holiday BOOLEAN
);

-- Dimension: Product
CREATE TABLE dim_product (
    product_key SERIAL PRIMARY KEY,  -- Surrogate key
    product_id VARCHAR(50) NOT NULL,  -- Natural key from source
    product_name VARCHAR(255) NOT NULL,
    category VARCHAR(100),
    subcategory VARCHAR(100),
    brand VARCHAR(100),
    unit_cost DECIMAL(10, 2),

    -- SCD Type 2 columns for tracking changes
    effective_date DATE NOT NULL,
    expiration_date DATE,
    is_current BOOLEAN DEFAULT TRUE,

    UNIQUE(product_id, effective_date)
);

CREATE INDEX idx_dim_product_current ON dim_product(product_id) WHERE is_current = TRUE;

-- Dimension: Customer
CREATE TABLE dim_customer (
    customer_key SERIAL PRIMARY KEY,
    customer_id VARCHAR(50) NOT NULL,
    customer_name VARCHAR(255),
    customer_segment VARCHAR(50),
    region VARCHAR(100),
    country VARCHAR(100),

    -- SCD Type 1 (overwrite) for most attributes
    -- Use SCD Type 2 if you need to track segment changes
    effective_date DATE NOT NULL,
    expiration_date DATE,
    is_current BOOLEAN DEFAULT TRUE
);

-- Fact: Sales (central fact table)
CREATE TABLE fact_sales (
    sale_id BIGSERIAL PRIMARY KEY,
    date_key INTEGER NOT NULL REFERENCES dim_date(date_key),
    product_key INTEGER NOT NULL REFERENCES dim_product(product_key),
    customer_key INTEGER NOT NULL REFERENCES dim_customer(customer_key),

    -- Measures (additive facts)
    quantity INTEGER NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,
    discount_amount DECIMAL(10, 2) DEFAULT 0,
    tax_amount DECIMAL(10, 2) NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    cost_amount DECIMAL(10, 2) NOT NULL,

    -- Degenerate dimensions (attributes with no separate dimension table)
    order_number VARCHAR(50),
    transaction_time TIMESTAMP NOT NULL
);

-- Indexes for typical analytical queries
CREATE INDEX idx_fact_sales_date ON fact_sales(date_key);
CREATE INDEX idx_fact_sales_product ON fact_sales(product_key);
CREATE INDEX idx_fact_sales_customer ON fact_sales(customer_key);
CREATE INDEX idx_fact_sales_composite ON fact_sales(date_key, product_key, customer_key);

-- Materialized view for pre-aggregated monthly sales
CREATE MATERIALIZED VIEW mv_monthly_sales AS
SELECT
    d.year,
    d.month,
    p.category,
    c.region,
    SUM(f.quantity) AS total_quantity,
    SUM(f.total_amount) AS total_revenue,
    SUM(f.cost_amount) AS total_cost,
    SUM(f.total_amount - f.cost_amount) AS total_profit
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_customer c ON f.customer_key = c.customer_key
GROUP BY d.year, d.month, p.category, c.region;

CREATE INDEX idx_mv_monthly_sales ON mv_monthly_sales(year, month, category);
```

### Example 5: Time-Series Database (TimescaleDB)

```sql
-- Create hypertable for metrics
CREATE TABLE metrics (
    time TIMESTAMPTZ NOT NULL,
    device_id VARCHAR(50) NOT NULL,
    metric_name VARCHAR(100) NOT NULL,
    value DOUBLE PRECISION NOT NULL,
    tags JSONB,

    PRIMARY KEY (time, device_id, metric_name)
);

-- Convert to hypertable with 1-day chunks
SELECT create_hypertable('metrics', 'time', chunk_time_interval => INTERVAL '1 day');

-- Create indexes for common query patterns
CREATE INDEX idx_metrics_device_time ON metrics(device_id, time DESC);
CREATE INDEX idx_metrics_name_time ON metrics(metric_name, time DESC);
CREATE INDEX idx_metrics_tags ON metrics USING gin(tags);

-- Compression policy (compress chunks older than 7 days)
ALTER TABLE metrics SET (
    timescaledb.compress,
    timescaledb.compress_segmentby = 'device_id, metric_name'
);

SELECT add_compression_policy('metrics', INTERVAL '7 days');

-- Retention policy (drop chunks older than 90 days)
SELECT add_retention_policy('metrics', INTERVAL '90 days');

-- Continuous aggregate for hourly rollups
CREATE MATERIALIZED VIEW metrics_hourly
WITH (timescaledb.continuous) AS
SELECT
    time_bucket('1 hour', time) AS hour,
    device_id,
    metric_name,
    AVG(value) AS avg_value,
    MAX(value) AS max_value,
    MIN(value) AS min_value,
    COUNT(*) AS count
FROM metrics
GROUP BY hour, device_id, metric_name;

-- Refresh policy for continuous aggregate
SELECT add_continuous_aggregate_policy('metrics_hourly',
    start_offset => INTERVAL '3 hours',
    end_offset => INTERVAL '1 hour',
    schedule_interval => INTERVAL '1 hour');
```

### Example 6: Event Sourcing Pattern (PostgreSQL)

```sql
-- Event store (append-only)
CREATE TABLE events (
    event_id BIGSERIAL PRIMARY KEY,
    aggregate_id UUID NOT NULL,
    aggregate_type VARCHAR(100) NOT NULL,
    event_type VARCHAR(100) NOT NULL,
    event_version INTEGER NOT NULL,
    payload JSONB NOT NULL,
    metadata JSONB,
    occurred_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    -- For optimistic concurrency control
    sequence_number INTEGER NOT NULL,

    CONSTRAINT unique_sequence UNIQUE(aggregate_id, sequence_number)
);

-- Indexes for event replay
CREATE INDEX idx_events_aggregate ON events(aggregate_id, sequence_number);
CREATE INDEX idx_events_type_time ON events(event_type, occurred_at);
CREATE INDEX idx_events_occurred ON events(occurred_at DESC);

-- Snapshots for performance (optional, reduces replay cost)
CREATE TABLE snapshots (
    snapshot_id BIGSERIAL PRIMARY KEY,
    aggregate_id UUID NOT NULL,
    aggregate_type VARCHAR(100) NOT NULL,
    sequence_number INTEGER NOT NULL,
    state JSONB NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT unique_snapshot UNIQUE(aggregate_id, sequence_number)
);

CREATE INDEX idx_snapshots_aggregate ON snapshots(aggregate_id, sequence_number DESC);

-- Projection (read model) - materialized view of event stream
CREATE TABLE account_balances (
    account_id UUID PRIMARY KEY,
    current_balance DECIMAL(15, 2) NOT NULL,
    last_event_sequence INTEGER NOT NULL,
    updated_at TIMESTAMPTZ NOT NULL
);

-- Example: Event handlers update projections
-- (In application code, not database triggers for better testability)
-- When AccountCredited event occurs:
--   UPDATE account_balances SET current_balance = current_balance + amount

-- Query to rebuild projection from events
CREATE OR REPLACE FUNCTION rebuild_account_balance(p_account_id UUID)
RETURNS DECIMAL AS $$
DECLARE
    v_balance DECIMAL(15, 2) := 0;
BEGIN
    SELECT COALESCE(SUM(
        CASE
            WHEN event_type = 'AccountCredited' THEN (payload->>'amount')::DECIMAL
            WHEN event_type = 'AccountDebited' THEN -(payload->>'amount')::DECIMAL
            ELSE 0
        END
    ), 0)
    INTO v_balance
    FROM events
    WHERE aggregate_id = p_account_id
    AND aggregate_type = 'Account'
    ORDER BY sequence_number;

    RETURN v_balance;
END;
$$ LANGUAGE plpgsql;
```

### Example 7: ETL Staging Pattern (PostgreSQL)

```sql
-- Staging table for incremental loads
CREATE TABLE staging_orders (
    order_id VARCHAR(50) PRIMARY KEY,
    customer_id VARCHAR(50),
    order_date TIMESTAMPTZ,
    total_amount DECIMAL(10, 2),
    status VARCHAR(20),

    -- ETL metadata
    source_system VARCHAR(50),
    extracted_at TIMESTAMPTZ NOT NULL,
    loaded_at TIMESTAMPTZ DEFAULT NOW(),
    batch_id VARCHAR(100),

    -- For change detection
    source_hash VARCHAR(64),
    is_processed BOOLEAN DEFAULT FALSE
);

-- Production table
CREATE TABLE orders (
    order_id VARCHAR(50) PRIMARY KEY,
    customer_id VARCHAR(50),
    order_date TIMESTAMPTZ,
    total_amount DECIMAL(10, 2),
    status VARCHAR(20),

    -- Audit columns
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    source_system VARCHAR(50),

    -- For CDC
    source_hash VARCHAR(64),
    version INTEGER DEFAULT 1
);

-- Merge staging into production (idempotent upsert)
CREATE OR REPLACE FUNCTION merge_orders()
RETURNS INTEGER AS $$
DECLARE
    v_rows_affected INTEGER;
BEGIN
    -- Insert new records
    INSERT INTO orders (order_id, customer_id, order_date, total_amount, status, source_system, source_hash)
    SELECT order_id, customer_id, order_date, total_amount, status, source_system, source_hash
    FROM staging_orders s
    WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.order_id = s.order_id)
    AND NOT is_processed;

    GET DIAGNOSTICS v_rows_affected = ROW_COUNT;

    -- Update changed records
    UPDATE orders o
    SET
        customer_id = s.customer_id,
        order_date = s.order_date,
        total_amount = s.total_amount,
        status = s.status,
        updated_at = NOW(),
        source_hash = s.source_hash,
        version = o.version + 1
    FROM staging_orders s
    WHERE o.order_id = s.order_id
    AND o.source_hash != s.source_hash
    AND NOT s.is_processed;

    GET DIAGNOSTICS v_rows_affected = v_rows_affected + ROW_COUNT;

    -- Mark staging records as processed
    UPDATE staging_orders SET is_processed = TRUE WHERE NOT is_processed;

    RETURN v_rows_affected;
END;
$$ LANGUAGE plpgsql;
```
