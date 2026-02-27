---
name: postkit
description: PostgreSQL-native identity, configuration, metering, and job queues. SQL functions that work with any language or driver. Use when working with user management, sessions, permissions, access control, login/logout, MFA, password resets, relationship-based access, versioned configuration, prompts, usage tracking, quotas, billing periods, or background jobs in PostgreSQL. Covers authn (user/session management), authz (ReBAC permissions), config (versioned key-value storage), meter (usage tracking with reservations), and queue (job scheduling with retries and dead letters).
---

## Setup (REQUIRED FIRST STEP)

Clone the repository if not already present:

```bash
[ -d postkit ] || git clone https://github.com/varunchopra/postkit.git
```

Then build the dist files:

```bash
cd postkit && make build && cd ..
```

Install the SQL schema on the user's database:

```bash
psql $DATABASE_URL -f postkit/dist/postkit.sql
# Or individual modules: postkit/dist/authn.sql, postkit/dist/authz.sql, postkit/dist/config.sql, postkit/dist/meter.sql, postkit/dist/queue.sql
```

For the optional Python SDK:

```bash
pip install -e "./postkit/sdk[binary]"
```

## Requirements

**All languages** (SQL functions):
- PostgreSQL 14+
- Schema installed from `dist/` files

**Python SDK only** (optional wrapper):
- `psycopg>=3.1.0` (not psycopg2)
- Python 3.11+

## Instructions

1. Read `postkit/AGENTS.md` for multi-tenancy, hashing, login flows, and module API maps
2. Read the relevant module docs based on what the user needs:
   - Authentication (users/sessions): `postkit/docs/authn/`
   - Authorization (permissions): `postkit/docs/authz/`
   - Configuration (versioned key-value): `postkit/docs/config/`
   - Metering (usage tracking, quotas): `postkit/docs/meter/`
   - Job Queues (scheduling, retries): `postkit/docs/queue/`

## Documentation Map

| What you need | Where to look |
|---------------|---------------|
| Integration guide | `postkit/AGENTS.md` |
| authn SQL reference | `postkit/docs/authn/sql.md` |
| authn Python SDK | `postkit/docs/authn/sdk.md` |
| authz SQL reference | `postkit/docs/authz/sql.md` |
| authz Python SDK | `postkit/docs/authz/sdk.md` |
| config SQL reference | `postkit/docs/config/sql.md` |
| config Python SDK | `postkit/docs/config/sdk.md` |
| meter SQL reference | `postkit/docs/meter/sql.md` |
| meter Python SDK | `postkit/docs/meter/sdk.md` |
| queue SQL reference | `postkit/docs/queue/sql.md` |
| queue Python SDK | `postkit/docs/queue/sdk.md` |
