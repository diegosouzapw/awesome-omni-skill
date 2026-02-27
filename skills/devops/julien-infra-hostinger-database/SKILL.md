---
name: julien-infra-hostinger-database
description: Manage shared database instances on Hostinger VPS srv759970 - PostgreSQL, Redis, MongoDB operations. Use for database connections, backups, user management, performance checks, or troubleshooting database issues.
license: Apache-2.0
triggers:
  - database hostinger
  - postgresql vps
  - redis server
  - mongodb hostinger
  - db backup
  - database connection
allowed-tools:
  - Bash
  - Read
metadata:
  author: "Julien"
  version: "1.0.0"
  category: "infrastructure"
  keywords: ["database", "postgresql", "redis", "mongodb", "postgres"]
---

# Database Management - Hostinger VPS

Manage shared database instances (PostgreSQL, Redis, MongoDB) on srv759970.

## Overview

The server runs **shared database instances** used by multiple applications.

### Databases Available

| Database | Port | Container | Status | Used By |
|----------|------|-----------|--------|---------|
| **PostgreSQL** | 5432 | postgres-shared | ✅ Running | WordPress sites, apps |
| **Redis** | 6379 | redis-shared | ✅ Running | Caching, queues |
| **MongoDB** | 27017 | mongodb-shared | ✅ Running | Document storage |

## When to Use This Skill

Invoke automatically when:
- Database connection issues reported
- Need to create/manage database users
- Backup or restore operations needed
- Performance issues with databases
- Database queries or inspection needed
- User mentions PostgreSQL, Redis, MongoDB operations

## PostgreSQL Operations

### Connection

**From server**:
```bash
# Connect to PostgreSQL
ssh srv759970 'docker exec -it postgres-shared psql -U postgres'

# Connect to specific database
ssh srv759970 'docker exec -it postgres-shared psql -U postgres -d mydb'
```

**From application container**:
```
Host: postgres-shared
Port: 5432
User: postgres
Password: [set in environment]
```

**Connection string**:
```
postgresql://username:password@postgres-shared:5432/database_name
```

### List Databases

```bash
ssh srv759970 << 'EOF'
docker exec -it postgres-shared psql -U postgres -c "\l"
EOF
```

### Create Database

```bash
ssh srv759970 << 'EOF'
docker exec -it postgres-shared psql -U postgres << PSQL
CREATE DATABASE mydb;
CREATE USER myuser WITH ENCRYPTED PASSWORD 'mypassword';
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
PSQL
EOF
```

### Backup Database

```bash
# Backup single database
ssh srv759970 'docker exec postgres-shared pg_dump -U postgres mydb > /tmp/mydb_backup.sql'

# Download backup
scp srv759970:/tmp/mydb_backup.sql ./mydb_backup_$(date +%Y%m%d).sql

# Backup all databases
ssh srv759970 'docker exec postgres-shared pg_dumpall -U postgres > /tmp/all_dbs_backup.sql'
```

### Restore Database

```bash
# Upload backup
scp ./mydb_backup.sql srv759970:/tmp/

# Restore
ssh srv759970 'docker exec -i postgres-shared psql -U postgres mydb < /tmp/mydb_backup.sql'
```

### Check Database Size

```bash
ssh srv759970 << 'EOF'
docker exec -it postgres-shared psql -U postgres -c "
SELECT
    datname AS database,
    pg_size_pretty(pg_database_size(datname)) AS size
FROM pg_database
ORDER BY pg_database_size(datname) DESC;
"
EOF
```

### User Management

```bash
# List users
ssh srv759970 'docker exec -it postgres-shared psql -U postgres -c "\du"'

# Create user
ssh srv759970 << 'EOF'
docker exec -it postgres-shared psql -U postgres -c "
CREATE USER newuser WITH PASSWORD 'password123';
GRANT ALL PRIVILEGES ON DATABASE mydb TO newuser;
"
EOF

# Change password
ssh srv759970 'docker exec -it postgres-shared psql -U postgres -c "ALTER USER myuser PASSWORD '\''newpassword'\'';"'

# Drop user
ssh srv759970 'docker exec -it postgres-shared psql -U postgres -c "DROP USER username;"'
```

### Performance Monitoring

```bash
# Active connections
ssh srv759970 << 'EOF'
docker exec -it postgres-shared psql -U postgres -c "
SELECT count(*) as connections, datname
FROM pg_stat_activity
GROUP BY datname;
"
EOF

# Long-running queries
ssh srv759970 << 'EOF'
docker exec -it postgres-shared psql -U postgres -c "
SELECT pid, now() - query_start as duration, query
FROM pg_stat_activity
WHERE state = 'active' AND now() - query_start > interval '5 seconds'
ORDER BY duration DESC;
"
EOF

# Kill query
ssh srv759970 'docker exec -it postgres-shared psql -U postgres -c "SELECT pg_terminate_backend(PID);"'
```

## Redis Operations

### Connection

**From server**:
```bash
# Connect to Redis CLI
ssh srv759970 'docker exec -it redis-shared redis-cli'
```

**From application**:
```
Host: redis-shared
Port: 6379
Password: [if configured]
```

**Connection string**:
```
redis://redis-shared:6379
```

### Basic Commands

```bash
# Ping
ssh srv759970 'docker exec redis-shared redis-cli ping'

# Get info
ssh srv759970 'docker exec redis-shared redis-cli info'

# Memory usage
ssh srv759970 'docker exec redis-shared redis-cli info memory'

# Connected clients
ssh srv759970 'docker exec redis-shared redis-cli info clients'
```

### Key Operations

```bash
# List all keys (⚠️ use with caution in production)
ssh srv759970 'docker exec redis-shared redis-cli keys "*"'

# Count keys
ssh srv759970 'docker exec redis-shared redis-cli dbsize'

# Get key value
ssh srv759970 'docker exec redis-shared redis-cli get mykey'

# Set key value
ssh srv759970 'docker exec redis-shared redis-cli set mykey "value"'

# Delete key
ssh srv759970 'docker exec redis-shared redis-cli del mykey'

# Check key TTL
ssh srv759970 'docker exec redis-shared redis-cli ttl mykey'
```

### Databases in Redis

Redis supports multiple databases (0-15 by default):

```bash
# Select database
ssh srv759970 'docker exec redis-shared redis-cli -n 0'

# List keys in specific database
ssh srv759970 'docker exec redis-shared redis-cli -n 1 keys "*"'
```

**Database usage on srv759970**:
- DB 0: Default / General caching
- DB 1: WhisperX transcription queue
- DB 2: Session storage
- DB 3-15: Available

### Flush Redis (⚠️ Careful!)

```bash
# Flush current database
ssh srv759970 'docker exec redis-shared redis-cli flushdb'

# Flush ALL databases (⚠️ DANGER)
ssh srv759970 'docker exec redis-shared redis-cli flushall'
```

### Monitor Redis Activity

```bash
# Real-time monitoring
ssh srv759970 'docker exec redis-shared redis-cli monitor'

# Slow log
ssh srv759970 'docker exec redis-shared redis-cli slowlog get 10'
```

### Backup Redis

```bash
# Trigger save
ssh srv759970 'docker exec redis-shared redis-cli bgsave'

# Check save status
ssh srv759970 'docker exec redis-shared redis-cli lastsave'

# Backup RDB file
ssh srv759970 'docker cp redis-shared:/data/dump.rdb /tmp/redis_backup_$(date +%Y%m%d).rdb'
```

## MongoDB Operations

### Connection

**From server**:
```bash
# Connect to MongoDB shell
ssh srv759970 'docker exec -it mongodb-shared mongosh'

# Connect to specific database
ssh srv759970 'docker exec -it mongodb-shared mongosh mydb'
```

**From application**:
```
Host: mongodb-shared
Port: 27017
```

**Connection string**:
```
mongodb://mongodb-shared:27017/database_name
```

### List Databases

```bash
ssh srv759970 << 'EOF'
docker exec mongodb-shared mongosh --eval "show dbs"
EOF
```

### Create Database & Collection

```bash
ssh srv759970 << 'EOF'
docker exec mongodb-shared mongosh << MONGO
use mydb
db.createCollection("mycollection")
db.mycollection.insertOne({name: "test", created: new Date()})
MONGO
EOF
```

### Backup MongoDB

```bash
# Backup single database
ssh srv759970 'docker exec mongodb-shared mongodump --db=mydb --out=/tmp/mongo_backup'

# Backup all databases
ssh srv759970 'docker exec mongodb-shared mongodump --out=/tmp/mongo_backup_all'

# Download backup
ssh srv759970 'cd /tmp && tar -czf mongo_backup.tar.gz mongo_backup'
scp srv759970:/tmp/mongo_backup.tar.gz ./mongo_backup_$(date +%Y%m%d).tar.gz
```

### Restore MongoDB

```bash
# Upload backup
scp ./mongo_backup.tar.gz srv759970:/tmp/

# Extract and restore
ssh srv759970 << 'EOF'
cd /tmp
tar -xzf mongo_backup.tar.gz
docker exec mongodb-shared mongorestore --db=mydb /tmp/mongo_backup/mydb
EOF
```

### Database Statistics

```bash
# Database stats
ssh srv759970 << 'EOF'
docker exec mongodb-shared mongosh mydb --eval "db.stats()"
EOF

# Collection stats
ssh srv759970 << 'EOF'
docker exec mongodb-shared mongosh mydb --eval "db.mycollection.stats()"
EOF
```

### User Management

```bash
# Create user
ssh srv759970 << 'EOF'
docker exec mongodb-shared mongosh admin --eval '
db.createUser({
  user: "myuser",
  pwd: "mypassword",
  roles: [{role: "readWrite", db: "mydb"}]
})'
EOF

# List users
ssh srv759970 'docker exec mongodb-shared mongosh admin --eval "db.getUsers()"'
```

## Database Container Management

### Check Container Status

```bash
# All database containers
ssh srv759970 'docker ps --filter name=postgres-shared --filter name=redis-shared --filter name=mongodb-shared'

# Resource usage
ssh srv759970 'docker stats --no-stream postgres-shared redis-shared mongodb-shared'
```

### View Logs

```bash
# PostgreSQL logs
ssh srv759970 'docker logs postgres-shared --tail 50'

# Redis logs
ssh srv759970 'docker logs redis-shared --tail 50'

# MongoDB logs
ssh srv759970 'docker logs mongodb-shared --tail 50'

# Follow logs in real-time
ssh srv759970 'docker logs -f postgres-shared'
```

### Restart Databases

```bash
# Restart specific database
ssh srv759970 'docker restart postgres-shared'

# Restart all databases (⚠️ affects all apps)
ssh srv759970 'docker restart postgres-shared redis-shared mongodb-shared'
```

## Troubleshooting

### Connection Refused

**Symptoms**: Applications can't connect to database

**Checks**:
```bash
# 1. Is container running?
ssh srv759970 'docker ps | grep -E "postgres|redis|mongo"'

# 2. Is port exposed?
ssh srv759970 'docker port postgres-shared'

# 3. Can we connect from server?
ssh srv759970 'docker exec postgres-shared pg_isready'

# 4. Check network
ssh srv759970 'docker network inspect bridge'
```

**Solutions**:
- Restart container: `docker restart postgres-shared`
- Check docker-compose network configuration
- Verify connection strings in application

### Database Locked / Slow

**PostgreSQL**:
```bash
# Check for locks
ssh srv759970 << 'EOF'
docker exec postgres-shared psql -U postgres -c "
SELECT * FROM pg_locks pl
LEFT JOIN pg_stat_activity psa ON pl.pid = psa.pid
WHERE NOT granted;
"
EOF

# Kill blocking query
ssh srv759970 'docker exec postgres-shared psql -U postgres -c "SELECT pg_terminate_backend(PID);"'
```

**Redis**:
```bash
# Check slow queries
ssh srv759970 'docker exec redis-shared redis-cli slowlog get 10'

# Memory issues
ssh srv759970 'docker exec redis-shared redis-cli info memory'
```

### Out of Disk Space

```bash
# Check database sizes
ssh srv759970 'docker exec postgres-shared du -sh /var/lib/postgresql/data'
ssh srv759970 'docker exec redis-shared du -sh /data'
ssh srv759970 'docker exec mongodb-shared du -sh /data/db'

# PostgreSQL vacuum (reclaim space)
ssh srv759970 'docker exec postgres-shared psql -U postgres -d mydb -c "VACUUM FULL;"'
```

### Corrupted Database

**PostgreSQL**:
```bash
# Check database integrity
ssh srv759970 'docker exec postgres-shared pg_dump -U postgres mydb > /dev/null'

# Restore from backup if corrupted
# See backup/restore sections above
```

## Backup Strategy

### Automated Backups

Recommended backup schedule:
- **Daily**: PostgreSQL databases (10 PM)
- **Weekly**: MongoDB full backup (Sunday 2 AM)
- **Continuous**: Redis persistence (RDB every 15 min)

### Backup Location

```bash
/opt/backups/
├── postgres/
│   ├── daily/
│   └── weekly/
├── mongodb/
│   └── weekly/
└── redis/
    └── snapshots/
```

### Backup Scripts

See `scripts/backup-databases.sh` for automated backup script.

## Performance Best Practices

1. **PostgreSQL**:
   - Regular VACUUM
   - Index optimization
   - Connection pooling in apps

2. **Redis**:
   - Use appropriate data structures
   - Set expiry on keys
   - Monitor memory usage

3. **MongoDB**:
   - Create indexes for queries
   - Regular compaction
   - Monitor collection sizes

## Security Notes

- **PostgreSQL**: User passwords stored in docker-compose env
- **Redis**: No password by default (container network only)
- **MongoDB**: Authentication enabled in production mode
- **Network**: Databases only accessible within Docker network
- **Backups**: Should be encrypted if containing sensitive data

## Reference Files

- **references/postgresql.md** - Detailed PostgreSQL operations
- **references/redis.md** - Redis patterns and best practices
- **references/mongodb.md** - MongoDB operations guide
- **scripts/backup-databases.sh** - Automated backup script

## Quick Commands Reference

```bash
# PostgreSQL
ssh srv759970 'docker exec -it postgres-shared psql -U postgres'
ssh srv759970 'docker exec postgres-shared pg_dump -U postgres mydb > backup.sql'

# Redis
ssh srv759970 'docker exec redis-shared redis-cli ping'
ssh srv759970 'docker exec redis-shared redis-cli dbsize'

# MongoDB
ssh srv759970 'docker exec -it mongodb-shared mongosh'
ssh srv759970 'docker exec mongodb-shared mongodump --out=/tmp/backup'

# Container management
ssh srv759970 'docker restart postgres-shared'
ssh srv759970 'docker logs postgres-shared --tail 50'
ssh srv759970 'docker stats --no-stream postgres-shared'
```

## Important Notes

- **Shared instances**: Multiple apps use these databases - coordinate changes
- **Backups**: Always backup before major operations
- **Performance**: Monitor resource usage - databases can impact all apps
- **Network**: Databases communicate via Docker network, not localhost
- **Documentation**: See `docs/infrastructure/databases.md` for detailed config
