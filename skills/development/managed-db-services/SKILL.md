---
name: managed-db-services
version: 1.0.0
min_doctl_version: "1.82.0"
description: Configure DigitalOcean Managed MySQL, MongoDB, Valkey, Kafka, and OpenSearch for App Platform. Use when setting up non-PostgreSQL databases, configuring trusted sources, or troubleshooting database connectivity.
related_skills: [designer, networking]
deprecated: false
---

# Managed Database Services Skill

Configure DigitalOcean Managed MySQL, MongoDB, Valkey (Redis), Kafka, and OpenSearch for App Platform applications.

## Quick Decision

```
Which database engine?
├── PostgreSQL    → Use the postgres skill instead
├── MySQL         → See reference/mysql.md
├── MongoDB       → See reference/mongodb.md
├── Valkey/Redis  → See reference/valkey.md
├── Kafka         → See reference/kafka.md (⚠️ trusted sources limitations)
└── OpenSearch    → See reference/opensearch.md
```

> **Tip**: For complex multi-step deployments, use the **planner** skill. For an overview of all skills, see [root SKILL.md](../../SKILL.md).

---

## Critical Constraints

| Constraint | Impact |
|------------|--------|
| Dev databases | PostgreSQL only — MySQL/MongoDB/Kafka/OpenSearch require `production: true` |
| Build-time DB access | ❌ Trusted sources block build phase — use PRE_DEPLOY job for migrations |
| Kafka trusted sources | IP-based only (`ip_addr:`); app-based (`app:`) NOT supported |
| OpenSearch logging | ❌ NOT supported with trusted sources enabled |
| MongoDB db_name | Cannot contain capital letters in app spec |

---

## Trusted Sources Quick Reference

| Network Mode | Rule Type | Supported Engines |
|--------------|-----------|-------------------|
| Public | `app:$APP_ID` | MySQL, MongoDB, Valkey, OpenSearch |
| Public | `app:$APP_ID` | ❌ Kafka (not supported) |
| VPC | `ip_addr:<vpc-cidr>` | All engines |
| VPC | `app:$APP_ID` | ❌ None (app rules whitelist public IP only) |

> **VPC deployments**: Use VPC CIDR (`ip_addr:10.126.0.0/20`) — simpler than per-app IPs.
>
> See [networking skill - Trusted Sources](../networking/SKILL.md#trusted-sources-the-big-picture) for complete configuration.

---

## Bindable Variables (All Engines)

```yaml
databases:
  - name: db                      # Component name (used in ${db.VAR_NAME})
    engine: <ENGINE>              # MYSQL, MONGODB, REDIS, KAFKA, OPENSEARCH
    production: true              # REQUIRED for bindable variables
    cluster_name: my-cluster      # Must match existing cluster name
    db_name: myappdb              # Database within cluster (where applicable)
    db_user: myappuser            # User created via doctl
```

| Variable | Description |
|----------|-------------|
| `${db.DATABASE_URL}` | Full connection string (PUBLIC hostname only!) |
| `${db.HOSTNAME}` | Database host (PUBLIC hostname only!) |
| `${db.PORT}` | Database port |
| `${db.USERNAME}` | Database user |
| `${db.PASSWORD}` | Database password (auto-populated) |
| `${db.DATABASE}` | Database name |
| `${db.CA_CERT}` | CA certificate for TLS |

> **VPC Note**: Bindable variables return PUBLIC hostnames even with VPC enabled. For private endpoints, add separate `*_PRIVATE_*` environment variables with hardcoded private hostnames.

---

## Engine Quick Reference

| Engine | App Spec | Port | Protocol | Key Notes |
|--------|----------|------|----------|-----------|
| MySQL | `MYSQL` | 25060 | `mysql://...?ssl-mode=REQUIRED` | [Full guide](reference/mysql.md) |
| MongoDB | `MONGODB` | 27017 | `mongodb+srv://...?tls=true&authSource=admin` | [Full guide](reference/mongodb.md) |
| Valkey | `REDIS` | 25061 | `rediss://` (with SSL) | [Full guide](reference/valkey.md) |
| Kafka | `KAFKA` | 9093 | SASL/SCRAM-SHA-256 | [Full guide](reference/kafka.md) |
| OpenSearch | `OPENSEARCH` | 25060 | `https://` with basic auth | [Full guide](reference/opensearch.md) |

---

## Quick Start: MySQL

```bash
# 1. Create cluster + user
doctl databases create my-mysql --engine mysql --region nyc3 --size db-s-1vcpu-2gb --version 8
CLUSTER_ID=$(doctl databases list --format ID,Name --no-header | grep my-mysql | awk '{print $1}')
doctl databases db create $CLUSTER_ID myappdb
doctl databases user create $CLUSTER_ID myappuser

# 2. Add to trusted sources
APP_ID=$(doctl apps list --format ID,Spec.Name --no-header | grep my-app | awk '{print $1}')
doctl databases firewalls append $CLUSTER_ID --rule app:$APP_ID

# 3. Reference in app spec
```

```yaml
databases:
  - name: db
    engine: MYSQL
    production: true
    cluster_name: my-mysql
    db_name: myappdb
    db_user: myappuser

services:
  - name: api
    envs:
      - key: DATABASE_URL
        scope: RUN_TIME
        value: ${db.DATABASE_URL}
```

**Full guide**: See [mysql.md](reference/mysql.md)

---

## Quick Start: MongoDB

```bash
doctl databases create my-mongo --engine mongodb --region nyc3 --size db-s-1vcpu-2gb --version 7
CLUSTER_ID=$(doctl databases list --format ID,Name --no-header | grep my-mongo | awk '{print $1}')
doctl databases user create $CLUSTER_ID myappuser
doctl databases firewalls append $CLUSTER_ID --rule app:$APP_ID
```

```yaml
databases:
  - name: db
    engine: MONGODB
    production: true
    cluster_name: my-mongo
    db_user: myappuser

services:
  - name: api
    envs:
      - key: MONGODB_URI
        scope: RUN_TIME
        value: ${db.DATABASE_URL}
```

**Full guide**: See [mongodb.md](reference/mongodb.md)

---

## Quick Start: Valkey

```bash
doctl databases create my-valkey --engine redis --region nyc3 --size db-s-1vcpu-2gb --version 7
CLUSTER_ID=$(doctl databases list --format ID,Name --no-header | grep my-valkey | awk '{print $1}')
doctl databases firewalls append $CLUSTER_ID --rule app:$APP_ID
```

```yaml
databases:
  - name: cache
    engine: REDIS
    production: true
    cluster_name: my-valkey

services:
  - name: api
    envs:
      - key: REDIS_URL
        scope: RUN_TIME
        value: ${cache.DATABASE_URL}
```

**Full guide**: See [valkey.md](reference/valkey.md)

---

## Quick Start: Kafka

> **Warning**: Kafka does NOT support `app:$APP_ID` trusted source rules. Use VPC + IP-based rules or disable trusted sources.

```bash
doctl databases create my-kafka --engine kafka --region nyc3 --size db-s-2vcpu-4gb --version 3.7
CLUSTER_ID=$(doctl databases list --format ID,Name --no-header | grep my-kafka | awk '{print $1}')
doctl databases topics create $CLUSTER_ID my-topic --partition-count 3 --replication-factor 2
```

```yaml
databases:
  - name: kafka
    engine: KAFKA
    production: true
    cluster_name: my-kafka

services:
  - name: api
    envs:
      - key: KAFKA_BROKER
        scope: RUN_TIME
        value: ${kafka.HOSTNAME}:${kafka.PORT}
      - key: KAFKA_USERNAME
        scope: RUN_TIME
        value: ${kafka.USERNAME}
      - key: KAFKA_PASSWORD
        scope: RUN_TIME
        value: ${kafka.PASSWORD}
      - key: KAFKA_CA_CERT
        scope: RUN_TIME
        value: ${kafka.CA_CERT}
```

**Full guide**: See [kafka.md](reference/kafka.md)

---

## Quick Start: OpenSearch

> **Warning**: Logging to OpenSearch requires trusted sources to be disabled.

```bash
doctl databases create my-opensearch --engine opensearch --region nyc3 --size db-s-2vcpu-4gb --version 2
CLUSTER_ID=$(doctl databases list --format ID,Name --no-header | grep my-opensearch | awk '{print $1}')
doctl databases user create $CLUSTER_ID myappuser
doctl databases firewalls append $CLUSTER_ID --rule app:$APP_ID
```

```yaml
databases:
  - name: search
    engine: OPENSEARCH
    production: true
    cluster_name: my-opensearch
    db_user: myappuser

services:
  - name: api
    envs:
      - key: OPENSEARCH_URL
        scope: RUN_TIME
        value: https://${search.USERNAME}:${search.PASSWORD}@${search.HOSTNAME}:${search.PORT}
```

**Full guide**: See [opensearch.md](reference/opensearch.md)

---

## Common doctl Commands

```bash
# List all database clusters
doctl databases list

# Get cluster details
doctl databases get <cluster-id>

# Create user (DO manages password)
doctl databases user create <cluster-id> <username>

# List users
doctl databases user list <cluster-id>

# Create database within cluster
doctl databases db create <cluster-id> <db-name>

# Get connection details
doctl databases connection <cluster-id>

# Trusted sources (firewall)
doctl databases firewalls append <cluster-id> --rule app:<app-id>
doctl databases firewalls list <cluster-id>
```

---

## Quick Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| "Connection refused" | App not in trusted sources | `doctl databases firewalls append <cluster-id> --rule app:<app-id>` |
| "Access denied" | User permissions not set | Grant permissions via SQL or recreate user |
| Bindable vars empty | Missing `production: true` | Add `production: true` to database block |
| SSL required | Connection string missing SSL | Add `?ssl-mode=REQUIRED` (MySQL), `?tls=true` (MongoDB), use `rediss://` (Valkey) |
| Kafka connection fails | Using `app:` rule | Kafka only supports `ip_addr:` rules — use VPC or disable TS |

---

## Reference Files

- **[mysql.md](reference/mysql.md)** — Connection pools, user privileges, password encryption
- **[mongodb.md](reference/mongodb.md)** — User roles, authSource configuration
- **[valkey.md](reference/valkey.md)** — Eviction policies, SSL protocol
- **[kafka.md](reference/kafka.md)** — SASL auth, SSL cert handling, Schema Registry
- **[opensearch.md](reference/opensearch.md)** — ACLs, logging limitations

---

## When to Use Postgres Skill Instead

Use the **postgres skill** for:
- Schema isolation (multi-tenant)
- Complex permission management
- Multiple apps sharing one cluster
- Connection pool configuration

This skill is for straightforward single-database setups with MySQL, MongoDB, Valkey, Kafka, or OpenSearch.

---

## Integration with Other Skills

| Skill | Integration |
|-------|-------------|
| **designer** | Generates `databases:` block in app spec |
| **deployment** | No additional secrets needed — bindable vars handle credentials |
| **networking** | VPC + trusted sources configuration |
| **troubleshooting** | Debug container for connectivity testing |

---

## Documentation Links

- [Managed Databases Overview](https://docs.digitalocean.com/products/databases/)
- [MySQL](https://docs.digitalocean.com/products/databases/mysql/)
- [MongoDB](https://docs.digitalocean.com/products/databases/mongodb/)
- [Redis/Valkey](https://docs.digitalocean.com/products/databases/redis/)
- [Kafka](https://docs.digitalocean.com/products/databases/kafka/)
- [OpenSearch](https://docs.digitalocean.com/products/databases/opensearch/)
- [doctl databases reference](https://docs.digitalocean.com/reference/doctl/reference/databases/)
