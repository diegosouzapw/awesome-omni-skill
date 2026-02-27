---
name: docker-database
description: Configure database containers with security, persistence, and health checks
---

# Docker Database Configuration Skill

## Overview

This skill configures database containers following best practices for:
- Security (passwords, network isolation)
- Persistence (named volumes)
- Health checks
- Initialization scripts
- Backup and restore

## Supported Databases

- PostgreSQL
- MySQL/MariaDB
- MongoDB
- Redis

## Process

### 1. Consult Documentation

Read `05-databases.md` for:
- Image versions
- Environment variables
- Health check configurations
- Volume paths
- Initialization patterns

### 2. Generate Configuration

## PostgreSQL Configuration

```yaml
services:
  postgres:
    image: postgres:16-alpine
    container_name: postgres
    restart: unless-stopped
    shm_size: 256mb
    environment:
      POSTGRES_USER: ${DB_USER:-appuser}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME:-appdb}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    ports:
      - "127.0.0.1:5432:5432"
    networks:
      - database
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER:-appuser} -d ${DB_NAME:-appdb}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

volumes:
  postgres_data:

networks:
  database:
    internal: true
```

## MySQL Configuration

```yaml
services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}
    volumes:
      - mysql_data:/var/lib/mysql
      - ./init-scripts:/docker-entrypoint-initdb.d
    ports:
      - "127.0.0.1:3306:3306"
    command: >
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_unicode_ci
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p${MYSQL_ROOT_PASSWORD}"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mysql_data:
```

## MongoDB Configuration

```yaml
services:
  mongodb:
    image: mongo:7
    container_name: mongodb
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD}
      MONGO_INITDB_DATABASE: ${MONGO_DB}
    volumes:
      - mongo_data:/data/db
      - mongo_config:/data/configdb
    ports:
      - "127.0.0.1:27017:27017"
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mongo_data:
  mongo_config:
```

## Redis Configuration

```yaml
services:
  redis:
    image: redis:7-alpine
    container_name: redis
    restart: unless-stopped
    command: >
      redis-server
      --appendonly yes
      --maxmemory 256mb
      --maxmemory-policy allkeys-lru
      --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    ports:
      - "127.0.0.1:6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "${REDIS_PASSWORD}", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  redis_data:
```

## Connection Strings

### PostgreSQL
```
DATABASE_URL=postgresql://user:password@db:5432/dbname
```

### MySQL
```
DATABASE_URL=mysql://user:password@db:3306/dbname
```

### MongoDB
```
MONGO_URI=mongodb://user:password@mongodb:27017/dbname?authSource=admin
```

### Redis
```
REDIS_URL=redis://:password@redis:6379
```

## Backup Commands

```bash
# PostgreSQL
docker compose exec -T db pg_dumpall -c -U appuser > backup.sql

# MySQL
docker compose exec -T db mysqldump -u root -p"$PASS" --all-databases > backup.sql

# MongoDB
docker compose exec -T db mongodump --archive --gzip > backup.gz

# Redis (if persistence enabled)
docker compose exec redis redis-cli -a "$PASS" BGSAVE
```

## Restore Commands

```bash
# PostgreSQL
docker compose exec -T db psql -U appuser -d appdb < backup.sql

# MySQL
docker compose exec -T db mysql -u root -p"$PASS" < backup.sql

# MongoDB
docker compose exec -T db mongorestore --archive --gzip < backup.gz
```

## Best Practices

1. **Always use named volumes** for data persistence
2. **Bind to localhost only** (127.0.0.1) unless external access needed
3. **Use internal networks** to isolate database traffic
4. **Implement health checks** for dependency management
5. **Store credentials in .env** or Docker secrets
6. **Use initialization scripts** for schema setup
7. **Regular backups** before any risky operations
