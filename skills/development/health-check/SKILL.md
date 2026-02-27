---
name: health-check
description: Check health status of all running services and dependencies. Use when verifying services are running, debugging connectivity issues, or monitoring system status.
argument-hint: [service]
allowed-tools: Bash, Read, Glob
---

# Service Health Checker

Check the health and status of all services, databases, and dependencies.

## Arguments
- `$ARGUMENTS`: Specific service name (optional - checks all if not specified)

## Services Configuration

| Service | Type | Health Endpoint | Port |
|---------|------|-----------------|------|
| eruditiontx-services-mvp | FastAPI | `/health`, `/v1/health` | 8000 |
| mathmatterstx-services | FastAPI | `/health`, `/v1/health` | 8002 |
| eruditiontx-client-mvp | React | `/` | 3000 |
| notaryo.ph | Next.js | `/api/health` | 3000 |
| MongoDB | Database | Connection test | 27017 |
| PostgreSQL | Database | Connection test | 5432 |
| Redis | Cache | `PING` | 6379 |
| MinIO | Storage | `/minio/health/live` | 9000 |

## Health Check Commands

### HTTP Services

```bash
# Basic health check
curl -sf http://localhost:$PORT/health && echo "OK" || echo "FAILED"

# With timeout
curl -sf --max-time 5 http://localhost:$PORT/health

# Get response details
curl -sf -w "\nStatus: %{http_code}\nTime: %{time_total}s\n" http://localhost:$PORT/health

# Check multiple endpoints
for port in 8000 8002 3000; do
    echo "Checking port $port..."
    curl -sf --max-time 5 http://localhost:$port/health && echo " OK" || echo " FAILED"
done
```

### Database Services

**MongoDB:**
```bash
# Check connection
mongosh --eval "db.adminCommand('ping')" --quiet
# or
mongosh mongodb://localhost:27017/test --eval "db.runCommand({ping: 1})"
```

**PostgreSQL:**
```bash
# Check connection
pg_isready -h localhost -p 5432
# or with psql
psql -h localhost -U postgres -c "SELECT 1" > /dev/null 2>&1 && echo "OK" || echo "FAILED"
```

**Redis:**
```bash
redis-cli ping
# Should return "PONG"
```

### Process Status

**Systemd Services:**
```bash
systemctl is-active erudition-service
systemctl status erudition-service --no-pager
```

**Docker Containers:**
```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
docker inspect --format='{{.State.Health.Status}}' $CONTAINER
```

**PM2 Processes:**
```bash
pm2 status
pm2 show $APP_NAME
```

### Port Checks

```bash
# Check if port is listening
lsof -i :$PORT
# or
netstat -tlnp | grep $PORT
# or on macOS
lsof -nP -iTCP:$PORT | grep LISTEN
```

## Comprehensive Health Check

Run all checks at once:

```bash
echo "=== Service Health Check ==="
echo ""

# HTTP Services
echo "HTTP Services:"
for service in "8000:eruditiontx-api" "8002:mathmatterstx-api" "3000:frontend"; do
    port=$(echo $service | cut -d: -f1)
    name=$(echo $service | cut -d: -f2)
    status=$(curl -sf --max-time 3 http://localhost:$port/health && echo "UP" || echo "DOWN")
    printf "  %-20s %s\n" "$name" "$status"
done

echo ""
echo "Databases:"
# MongoDB
mongo_status=$(mongosh --eval "db.adminCommand('ping')" --quiet 2>/dev/null && echo "UP" || echo "DOWN")
printf "  %-20s %s\n" "MongoDB" "$mongo_status"

# PostgreSQL
pg_status=$(pg_isready -h localhost -p 5432 2>/dev/null && echo "UP" || echo "DOWN")
printf "  %-20s %s\n" "PostgreSQL" "$pg_status"

# Redis
redis_status=$(redis-cli ping 2>/dev/null | grep -q PONG && echo "UP" || echo "DOWN")
printf "  %-20s %s\n" "Redis" "$redis_status"

echo ""
echo "Docker Containers:"
docker ps --format "  {{.Names}}: {{.Status}}" 2>/dev/null || echo "  Docker not running"
```

## Output Format

```
Health Check Report
Generated: [timestamp]

Services:
  Service                 Status    Response Time
  ---------------------   ------    -------------
  eruditiontx-api         UP        45ms
  mathmatterstx-api       UP        32ms
  frontend                UP        28ms

Databases:
  Database                Status    Connection
  ---------------------   ------    ----------
  MongoDB                 UP        localhost:27017
  PostgreSQL              DOWN      Connection refused
  Redis                   UP        localhost:6379

Infrastructure:
  Component               Status
  ---------------------   ------
  Docker                  Running (5 containers)
  Nginx                   Running

Issues Found:
  - PostgreSQL is not running
  - Port 5432 not listening

Recommendations:
  1. Start PostgreSQL: brew services start postgresql
  2. Check PostgreSQL logs: tail -f /usr/local/var/log/postgresql.log
```

## Alerting

If critical services are down:

```bash
# Send alert (example with curl to Slack webhook)
if ! curl -sf http://localhost:8000/health > /dev/null; then
    curl -X POST -H 'Content-type: application/json' \
        --data '{"text":"ALERT: eruditiontx-api is DOWN"}' \
        $SLACK_WEBHOOK_URL
fi
```

## Troubleshooting

If a service is down:

1. **Check if process is running**: `ps aux | grep $SERVICE`
2. **Check port binding**: `lsof -i :$PORT`
3. **Check logs**: `/logs $SERVICE error`
4. **Restart service**: `sudo systemctl restart $SERVICE`
