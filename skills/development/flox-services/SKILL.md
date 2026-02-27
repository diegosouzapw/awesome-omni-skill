---
name: flox-services
description: Running services and background processes in Flox environments. Use for service configuration, network services, logging, database setup, and service debugging.
---

# Flox Services Guide

## Running Services in Flox Environments

- Start with `flox activate --start-services` or `flox activate -s`
- Define `is-daemon`, `shutdown.command` for background processes
- Keep services running using `tail -f /dev/null`
- Use `flox services status/logs/restart` to manage (must be in activated env)
- Service commands don't inherit hook activations; explicitly source/activate what you need

## Core Commands

```bash
flox activate -s                # Start services
flox services status            # Check service status
flox services logs <service>    # View service logs
flox services restart <service> # Restart a service
flox services stop <service>    # Stop a service
```

## Network Services Pattern

Always make host/port configurable via vars:

```toml
[services.webapp]
command = '''exec app --host "$APP_HOST" --port "$APP_PORT"'''

[vars]
APP_HOST = "0.0.0.0"  # Network-accessible
APP_PORT = "8080"
```

## Service Logging Pattern

Always pipe to `$FLOX_ENV_CACHE/logs/` for debugging:

```toml
[services.myapp]
command = '''
  mkdir -p "$FLOX_ENV_CACHE/logs"
  exec app 2>&1 | tee -a "$FLOX_ENV_CACHE/logs/app.log"
'''
```

## Python venv Pattern for Services

Services must activate venv independently:

```toml
[services.myapp]
command = '''
  [ -f "$FLOX_ENV_CACHE/venv/bin/activate" ] && \
    source "$FLOX_ENV_CACHE/venv/bin/activate"
  exec python-app "$@"
'''
```

Or use venv Python directly:

```toml
[services.myapp]
command = '''exec "$FLOX_ENV_CACHE/venv/bin/python" app.py'''
```

## Using Packaged Services

Override package's service by redefining with same name.

## Database Service Examples

### PostgreSQL

```toml
[services.postgres]
command = '''
  mkdir -p "$FLOX_ENV_CACHE/postgres"
  if [ ! -d "$FLOX_ENV_CACHE/postgres/data" ]; then
    initdb -D "$FLOX_ENV_CACHE/postgres/data"
  fi
  exec postgres -D "$FLOX_ENV_CACHE/postgres/data" \
    -k "$FLOX_ENV_CACHE/postgres" \
    -h "$POSTGRES_HOST" \
    -p "$POSTGRES_PORT"
'''
is-daemon = true

[vars]
POSTGRES_HOST = "localhost"
POSTGRES_PORT = "5432"
POSTGRES_USER = "myuser"
POSTGRES_DB = "mydb"
```

### Redis

```toml
[services.redis]
command = '''
  mkdir -p "$FLOX_ENV_CACHE/redis"
  exec redis-server \
    --bind "$REDIS_HOST" \
    --port "$REDIS_PORT" \
    --dir "$FLOX_ENV_CACHE/redis"
'''
is-daemon = true

[vars]
REDIS_HOST = "127.0.0.1"
REDIS_PORT = "6379"
```

### MongoDB

```toml
[services.mongodb]
command = '''
  mkdir -p "$FLOX_ENV_CACHE/mongodb"
  exec mongod \
    --dbpath "$FLOX_ENV_CACHE/mongodb" \
    --bind_ip "$MONGODB_HOST" \
    --port "$MONGODB_PORT"
'''
is-daemon = true

[vars]
MONGODB_HOST = "127.0.0.1"
MONGODB_PORT = "27017"
```

## Web Server Examples

### Node.js Development Server

```toml
[services.dev-server]
command = '''
  exec npm run dev -- --host "$DEV_HOST" --port "$DEV_PORT"
'''

[vars]
DEV_HOST = "0.0.0.0"
DEV_PORT = "3000"
```

### Python Flask/FastAPI

```toml
[services.api]
command = '''
  source "$FLOX_ENV_CACHE/venv/bin/activate"
  exec python -m uvicorn main:app \
    --host "$API_HOST" \
    --port "$API_PORT" \
    --reload
'''

[vars]
API_HOST = "0.0.0.0"
API_PORT = "8000"
```

### Simple HTTP Server

```toml
[services.web]
command = '''exec python -m http.server "$WEB_PORT"'''

[vars]
WEB_PORT = "8000"
```

## Environment Variable Convention

Use variables like `POSTGRES_HOST`, `POSTGRES_PORT` to define where services run.

These store connection details *separately*:
- `*_HOST` is the hostname or IP address (e.g., `localhost`, `db.example.com`)
- `*_PORT` is the network port number (e.g., `5432`, `6379`)

This pattern ensures users can override them at runtime:
```bash
POSTGRES_HOST=db.internal POSTGRES_PORT=6543 flox activate -s
```

Use consistent naming across services so the meaning is clear to any system or person reading the variables.

## Service with Shutdown Command

```toml
[services.myapp]
command = '''exec myapp start'''
is-daemon = true

[services.myapp.shutdown]
command = '''myapp stop'''
```

## Dependent Services

Services can wait for other services to be ready:

```toml
[services.db]
command = '''exec postgres -D "$FLOX_ENV_CACHE/postgres"'''
is-daemon = true

[services.api]
command = '''
  # Wait for database
  until pg_isready -h localhost -p 5432; do
    sleep 1
  done
  exec python -m uvicorn main:app
'''

[vars]
POSTGRES_HOST = "localhost"
POSTGRES_PORT = "5432"
```

## Service Health Checks

```toml
[services.api]
command = '''
  # Health check function
  health_check() {
    curl -sf http://localhost:8000/health > /dev/null
  }

  exec python -m uvicorn main:app --host 0.0.0.0 --port 8000
'''
```

## Best Practices

- Log service output to `$FLOX_ENV_CACHE/logs/`
- Test activation with `flox activate -- <command>` before adding to services
- When debugging services, run the exact command from manifest manually first
- Always make host/port configurable via vars for network services
- Use `exec` to replace the shell process with the service command
- Services must activate venv inside service command, not rely on hook activation
- Use `is-daemon = true` for background processes that should detach

## Debugging Service Issues

### Check Service Status
```bash
flox services status
```

### View Service Logs
```bash
flox services logs myservice
```

### Run Service Command Manually
```bash
flox activate
# Copy the exact command from manifest and run it
```

### Check if Service is Listening
```bash
# Check if port is open
lsof -i :8000
netstat -an | grep 8000

# Test connection
curl http://localhost:8000
nc -zv localhost 8000
```

## Common Pitfalls

### Services Don't Preserve State
Services see fresh environment (no preserved state between restarts). Store persistent data in `$FLOX_ENV_CACHE`.

### Service Commands Don't Inherit Hook Activations
Explicitly source/activate what you need inside the service command.

### Forgetting to Create Directories
Always `mkdir -p` for data directories in service commands.

### Port Conflicts
Use configurable ports via variables to avoid conflicts with other services.

## Related Skills

- **flox-environments** - Environment basics and package installation
- **flox-sharing** - Composing environments with shared services
- **flox-containers** - Running services in containers
