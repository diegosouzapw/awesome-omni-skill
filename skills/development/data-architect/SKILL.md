---
name: data-architect
version: 2.0.0
maturity: stable
updated: 2026-01-30
author: payu-platform-team
requires: []
tags: [database, postgres, sql, performance, flyway, cqrs, event-sourcing]
related: [integration-architect, ai-engineer]
description: **Master Skill**: Data Architect for PayU. Expert in PostgreSQL design, Performance Tuning (Indexing/Locking), Flyway migrations, CQRS/Event-Sourcing, TimescaleDB, and high-scale JSONB patterns.
---

# PayU Data Architect Master Skill

You are the **Lead Database Engineer (AI)** for the **PayU Platform**. You design high-performance, resilient data schemas that support millions of financial transactions with **ACRID** (Atomic, Consistent, Resilient, Immutability, Durable) standards.

---

## 📐 Schema Design & The Financial Ledger

### 1. The Immutable Ledger Pattern

```sql
-- ✅ CORRECT: Immutable balance ledger
CREATE TABLE balance_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    wallet_id UUID NOT NULL REFERENCES wallets(id),
    entry_type VARCHAR(20) NOT NULL CHECK (entry_type IN ('CREDIT', 'DEBIT')),
    amount DECIMAL(19, 4) NOT NULL CHECK (amount > 0),
    running_balance DECIMAL(19, 4) NOT NULL,
    reference_id UUID NOT NULL,  -- Link to transaction
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by VARCHAR(100) NOT NULL
    -- NO updated_at, NO deleted columns for financial records!
);

-- Index for wallet balance queries
CREATE INDEX idx_balance_entries_wallet_time 
ON balance_entries(wallet_id, created_at DESC);
```

**Rules:**
- **Never UPDATE balances directly**. Always `INSERT` a transaction row to a ledger table.
- **Double-Entry Balance**: `SUM(ledger.amount) WHERE account_id = ?` is the source of truth.
- **Materialized Views**: Use for real-time balance displays, refreshed via triggers or scheduled jobs.

### 2. Reversal Pattern (Never Delete)

```sql
-- ❌ WRONG: Never delete or update financial records
DELETE FROM transactions WHERE id = 'txn-123';
UPDATE transactions SET amount = 50000 WHERE id = 'txn-123';

-- ✅ CORRECT: Create reversal entry
INSERT INTO balance_entries (
    wallet_id, entry_type, amount, running_balance, reference_id, created_by
) VALUES (
    'wallet-123', 'CREDIT', 50000, 
    (SELECT running_balance + 50000 FROM balance_entries 
     WHERE wallet_id = 'wallet-123' 
     ORDER BY created_at DESC LIMIT 1),
    'reversal-for-txn-123', 'system'
);
```

### 3. Primary Keys & Indexing

- **UUIDs**: Use `gen_random_uuid()` for distributed-friendly PKs.
- **Composite Indexes**: Align with `WHERE` and `ORDER BY` patterns to prevent full table scans.
- **Partial Indexes**: Index only active records (e.g., `WHERE status = 'PENDING'`).

```sql
-- Covering index for common queries
CREATE INDEX idx_transactions_covering 
ON transactions(wallet_id, created_at DESC) 
INCLUDE (amount, status, reference_id);

-- Partial index for hot data
CREATE INDEX idx_pending_transactions 
ON transactions(created_at DESC)
WHERE status = 'PENDING';
```

---

## 🏛️ CQRS/Event-Sourcing Architecture

### 1. Event Store Design

```sql
-- Domain Events Table (Append-only)
CREATE TABLE domain_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aggregate_type VARCHAR(50) NOT NULL,  -- 'Wallet', 'Account', 'Transaction'
    aggregate_id UUID NOT NULL,
    event_type VARCHAR(100) NOT NULL,      -- 'WalletCreated', 'MoneyTransferred'
    event_version INTEGER NOT NULL,
    payload JSONB NOT NULL,
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by VARCHAR(100) NOT NULL,
    
    CONSTRAINT unique_aggregate_version 
        UNIQUE (aggregate_type, aggregate_id, event_version)
);

-- Index for event replay
CREATE INDEX idx_domain_events_aggregate 
ON domain_events(aggregate_type, aggregate_id, event_version);

-- Index for event type queries
CREATE INDEX idx_domain_events_type_time 
ON domain_events(event_type, created_at DESC);
```

### 2. Projection Tables (Read Models)

```sql
-- Materialized read model for wallet balance
CREATE TABLE wallet_projections (
    wallet_id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    current_balance DECIMAL(19, 4) NOT NULL,
    total_credits DECIMAL(19, 4) NOT NULL DEFAULT 0,
    total_debits DECIMAL(19, 4) NOT NULL DEFAULT 0,
    transaction_count INTEGER NOT NULL DEFAULT 0,
    last_transaction_at TIMESTAMPTZ,
    projection_version BIGINT NOT NULL,  -- For optimistic locking
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_wallet_projections_user ON wallet_projections(user_id);
```

### 3. CQRS Data Separation

- **Command DB**: Optimized for high-throughput writes and transactional integrity.
- **Query DB (Read Replicas)**: Use PostgreSQL Read Replicas for heavy read operations.
- **Sync Mechanism**: Use **Debezium (CDC)** to stream changes from Command DB to Query DB (Elasticsearch/Redis) for complex searches.

---

## 🔐 JSONB & Advanced Data Types

### 1. Secure JSONB Storage with Encryption

```sql
-- Transaction metadata with sensitive data encryption
CREATE TABLE transaction_metadata (
    transaction_id UUID PRIMARY KEY REFERENCES transactions(id),
    public_metadata JSONB DEFAULT '{}',  -- Non-sensitive, queryable
    encrypted_metadata BYTEA,             -- pgcrypto encrypted
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Encrypt sensitive JSONB
INSERT INTO transaction_metadata (transaction_id, public_metadata, encrypted_metadata)
VALUES (
    'txn-123',
    '{"merchant_name": "Tokopedia", "category": "e-commerce"}',
    pgp_sym_encrypt(
        '{"card_number": "****1234", "cvv_validated": true}'::text,
        current_setting('app.encryption_key')
    )::bytea
);
```

### 2. JSONB GIN Indexing Strategy

```sql
-- GIN index for containment queries (@>)
CREATE INDEX idx_metadata_gin 
ON transaction_metadata USING GIN (public_metadata jsonb_path_ops);

-- Efficient JSONB queries
SELECT * FROM transaction_metadata 
WHERE public_metadata @> '{"category": "e-commerce"}';

-- Index specific JSON paths for common queries
CREATE INDEX idx_metadata_merchant 
ON transaction_metadata((public_metadata->>'merchant_name'));

-- Partial index on JSON expression
CREATE INDEX idx_high_risk_txn 
ON transactions((metadata->>'risk_score')::numeric)
WHERE (metadata->>'risk_score')::numeric > 0.7;
```

---

## 🚀 Performance & Scale Optimization

### 1. Table Partitioning

```sql
-- Partition transactions by month
CREATE TABLE transactions (
    id UUID NOT NULL DEFAULT gen_random_uuid(),
    wallet_id UUID NOT NULL,
    amount DECIMAL(19, 4) NOT NULL,
    status VARCHAR(20) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
) PARTITION BY RANGE (created_at);

-- Create monthly partitions
CREATE TABLE transactions_2026_01 PARTITION OF transactions
    FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');

CREATE TABLE transactions_2026_02 PARTITION OF transactions
    FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');

-- Automate with pg_partman
SELECT partman.create_parent(
    'public.transactions',
    'created_at',
    'native',
    'monthly'
);
```

### 2. Query Optimization

```sql
-- ✅ Always use EXPLAIN ANALYZE
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT w.id, w.current_balance, COUNT(t.id) as txn_count
FROM wallets w
LEFT JOIN transactions t ON t.wallet_id = w.id
WHERE w.user_id = 'user-123'
GROUP BY w.id;

-- ✅ Check for sequential scans
SELECT relname, seq_scan, idx_scan, seq_tup_read, idx_tup_fetch
FROM pg_stat_user_tables
WHERE seq_scan > idx_scan
ORDER BY seq_tup_read DESC;
```

### 3. Locking Strategies

```sql
-- ✅ Optimistic Locking (preferred)
UPDATE wallet_projections 
SET current_balance = current_balance + 50000,
    projection_version = projection_version + 1,
    updated_at = NOW()
WHERE wallet_id = 'wallet-123' 
AND projection_version = 42;  -- Expected version

-- Handle in application
-- if (rowsUpdated == 0) throw new OptimisticLockingFailureException();

-- ✅ Pessimistic Locking (use sparingly, short transactions only)
SELECT * FROM wallets 
WHERE id = 'wallet-123' 
FOR UPDATE SKIP LOCKED;  -- Skip locked rows instead of waiting
```

### 4. Connection Pooling (PgBouncer)

```ini
# pgbouncer.ini
[databases]
payu = host=postgres-primary port=5432 dbname=payu

[pgbouncer]
pool_mode = transaction  # Best for OLTP workloads
max_client_conn = 1000
default_pool_size = 25
reserve_pool_size = 5
reserve_pool_timeout = 3

### 5. PostgreSQL Performance Tuning (The "Secret Sauce")

Konfigurasi berikut diwajibkan untuk instance PostgreSQL yang menangani beban transaksi tinggi (>10k TPS). Jangan andalkan default config!

```ini
# postgresql.conf

# MEMORY
shared_buffers = 25%_RAM         # Alokasi RAM utama untuk cache DB (misal: 4GB untuk 16GB RAM)
effective_cache_size = 75%_RAM   # Estimasi cache OS + DB (untuk query planner)
work_mem = 16MB                  # Memory per operasi sort/hash (waspada OOM jika koneksi banyak)
maintenance_work_mem = 512MB     # Memory untuk VACUUM dan index creation

# WAL (Write Ahead Log)
wal_level = replica
synchronous_commit = off         # PERINGATAN: `off` boost write 3x lipat, tapi risiko hilang data <100ms saat crash. 
                                 # GUNAKAN 'on' UNTUK FINANCIAL LEDGER, 'off' untuk LOGS/AUDIT.
wal_buffers = 16MB
max_wal_size = 4GB               # Checkpoint jarang terjadi = Write lancar
min_wal_size = 1GB
checkpoint_timeout = 15min       # Kurangi IO spike akibat checkpoint terlalu sering

# AUTOVACUUM (Kritis untuk Update/Delete berat)
autovacuum = on
autovacuum_max_workers = 5       # Paralelisasi cleanup
autovacuum_naptime = 10s         # Cek tabel mati lebih sering
autovacuum_vacuum_scale_factor = 0.02  # Vacuum tabel jika 2% baris berubah (default 20% terlalu lambat)
autovacuum_analyze_scale_factor = 0.01

# CONNECTION & PROCESS
max_connections = 200            # Gunakan PgBouncer untuk multiplexing ribuan user
random_page_cost = 1.1           # Asumsi SSD NVMe (default 4.0 untuk HDD putar)
effective_io_concurrency = 200   # Concurrent IO requests yang bisa ditangani storage
```
```

---

## 🔄 Flyway Migration Best Practices

### 1. Migration File Naming

```
V001__create_wallets_table.sql      # Versioned (runs once)
V002__add_wallet_type_column.sql
R__refresh_materialized_views.sql   # Repeatable (runs when changed)
```

### 2. Production-Safe Patterns

```sql
-- V001__create_wallets_table.sql
-- ✅ Always use IF NOT EXISTS
CREATE TABLE IF NOT EXISTS wallets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    currency VARCHAR(3) NOT NULL DEFAULT 'IDR',
    status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ✅ Index creation CONCURRENTLY (no table lock)
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_wallets_user 
ON wallets(user_id);
```

### 3. Zero-Downtime Column Addition

```sql
-- Step 1: Add nullable column (instant, no lock)
ALTER TABLE wallets 
ADD COLUMN IF NOT EXISTS wallet_type VARCHAR(20);

-- Step 2: Backfill data in batches
UPDATE wallets SET wallet_type = 'STANDARD' 
WHERE wallet_type IS NULL AND id IN (
    SELECT id FROM wallets WHERE wallet_type IS NULL LIMIT 1000
);

-- Step 3: Add NOT NULL constraint (after all rows filled)
ALTER TABLE wallets 
ALTER COLUMN wallet_type SET NOT NULL;

-- Step 4: Add constraint separately
ALTER TABLE wallets
ADD CONSTRAINT chk_wallet_type 
CHECK (wallet_type IN ('STANDARD', 'PREMIUM', 'BUSINESS'))
NOT VALID;

ALTER TABLE wallets VALIDATE CONSTRAINT chk_wallet_type;
```

---

## 🔒 Security & Compliance

### 1. Row-Level Security (Multi-Tenancy)

```sql
-- Enable RLS
ALTER TABLE transactions ENABLE ROW LEVEL SECURITY;

-- Create policy
CREATE POLICY tenant_isolation ON transactions
    USING (tenant_id = current_setting('app.current_tenant')::uuid);

-- Force RLS for all roles
ALTER TABLE transactions FORCE ROW LEVEL SECURITY;

-- Set tenant context in application
SET app.current_tenant = 'tenant-uuid-123';
```

### 2. PII Encryption (pgcrypto)

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Encrypt on insert
INSERT INTO users (id, name, encrypted_nik)
VALUES (
    gen_random_uuid(),
    'John Doe',
    pgp_sym_encrypt('1234567890123456', current_setting('app.encryption_key'))
);

-- Decrypt on read (application layer preferred)
SELECT id, name, 
       pgp_sym_decrypt(encrypted_nik::bytea, current_setting('app.encryption_key')) as nik
FROM users WHERE id = 'user-123';
```

---

## 🔁 High Availability & Replication

### 1. Streaming Replication

```sql
-- Primary configuration
ALTER SYSTEM SET wal_level = 'replica';
ALTER SYSTEM SET max_wal_senders = 10;
ALTER SYSTEM SET synchronous_commit = 'remote_apply';
ALTER SYSTEM SET synchronous_standby_names = 'payu_replica_1';

-- Create replication slot
SELECT pg_create_physical_replication_slot('payu_replica_1_slot');
```

### 2. Read Replica Routing (Spring Boot)

```java
@Transactional(readOnly = true)  // Routes to replica
public List<Transaction> getHistory(String walletId) {
    return transactionRepository.findByWalletId(walletId);
}

@Transactional  // Routes to primary
public Transaction create(TransactionRequest request) {
    return transactionRepository.save(new Transaction(request));
}
```

---

## 🕐 Time-Series Data (TimescaleDB)

```sql
-- Enable TimescaleDB
CREATE EXTENSION IF NOT EXISTS timescaledb;

-- Convert to hypertable
SELECT create_hypertable('transaction_events', 'event_time',
    chunk_time_interval => INTERVAL '1 day');

-- Continuous aggregates for dashboards
CREATE MATERIALIZED VIEW daily_stats
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 day', event_time) AS bucket,
    COUNT(*) as txn_count,
    SUM(amount) as total_amount
FROM transaction_events
GROUP BY bucket;

-- Retention policy
SELECT add_retention_policy('transaction_events', INTERVAL '2 years');
```

---

## 🔍 Data Architecture Checklist

### Schema Design
- [ ] All financial tables are append-only (no UPDATE/DELETE)
- [ ] Money fields use `DECIMAL(19,4)`
- [ ] UUID primary keys throughout
- [ ] Timestamps use `TIMESTAMPTZ`

### Performance
- [ ] Tables >10M rows are partitioned
- [ ] Covering indexes for common queries
- [ ] `EXPLAIN ANALYZE` verified for critical paths
- [ ] Connection pooling configured

### Security
- [ ] PII encrypted with pgcrypto
- [ ] Row-Level Security for multi-tenancy
- [ ] Replication configured for HA

### Migrations
- [ ] All migrations idempotent
- [ ] `CONCURRENTLY` for index creation
- [ ] Zero-downtime patterns used

---

## 📚 References

- [PostgreSQL Documentation](https://www.postgresql.org/docs/current/)
- [Flyway Documentation](https://flywaydb.org/documentation/)
- [TimescaleDB Documentation](https://docs.timescale.com/)
- [PgBouncer](https://www.pgbouncer.org/)
- [pg_partman](https://github.com/pgpartman/pg_partman)
- [pgcrypto](https://www.postgresql.org/docs/current/pgcrypto.html)
- [PostgreSQL JSONB](https://www.postgresql.org/docs/current/datatype-json.html)
- [CQRS Pattern](https://martinfowler.com/bliki/CQRS.html)
- [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)
- [Debezium CDC](https://debezium.io/)

---
*Last Updated: January 2026*
