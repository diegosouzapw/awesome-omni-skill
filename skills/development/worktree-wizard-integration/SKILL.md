---
name: worktree-wizard-integration
description: This skill should be used when the user asks to "set up worktree-wizard", "integrate worktree-wizard", "add worktree support", "create docker-compose for worktrees", "add wt labels", "configure hot-reload for Docker", "set up volume mounts", "isolate ports per worktree", "onboard project to worktree-wizard", or needs guidance on wt.base-port labels, WT_* env var patterns, slot-based port isolation, dev-mode Dockerfiles, or hot-reload configurations per framework.
version: 0.1.0
---

# worktree-wizard Integration Knowledge

## Purpose

Provide the domain knowledge needed to correctly integrate worktree-wizard into any project. This includes docker-compose conventions, env var patterns, volume mount strategies, hot-reload configurations, and justfile setup.

## Core Conventions

### Port Isolation via Labels

Add a `wt.base-port` label to every service needing port isolation. worktree-wizard adds the slot number (1-9) to compute the host port per worktree.

```yaml
services:
  api:
    labels:
      wt.base-port: "8000"
    ports:
      - "${WT_API_PORT:-8000}:8000"   # host:container
```

**Rules:**
- Label value is the base port (slot 0 / main repo)
- Container-internal port never changes — only host port offsets
- Env var naming: `WT_{SERVICE_NAME_UPPER}_PORT` (hyphens/dots become underscores)
- Always provide `:-default` fallback so slot 0 works without env vars
- Services with multiple ports: use `wt.base-port` for the primary port only; additional ports use manual env var patterns

### Data Isolation

To opt in to data isolation for a service, add a `wt.data-dir` label. This creates isolated data directories per slot.

```yaml
  db:
    labels:
      wt.data-dir: "/var/lib/postgresql/data"
    volumes:
      - ${WT_DB_DATA:-./.docker-data/db}:/var/lib/postgresql/data
```

### Internal Networking

Services communicate by **service name** over Docker's internal network. Port offsets are irrelevant for inter-service communication:

```yaml
  backend:
    environment:
      - DATABASE_URL=postgresql://postgres:postgres@db:5432/app
```

Never use `localhost` or offset ports for service-to-service connections.

## Volume Mounts for Hot-Reload

Mount source code into the container so host edits are visible immediately:

```yaml
  backend:
    volumes:
      - ./backend:/app        # source code
```

For Node.js services, preserve container `node_modules` with an anonymous volume:

```yaml
  frontend:
    volumes:
      - ./frontend:/app
      - /app/node_modules     # prevents host node_modules from shadowing
```

## Hot-Reload per Framework

For per-framework Dockerfile patterns, dev commands, and configuration details, consult `references/stack-patterns.md`.

**Quick reference:**

| Framework | Dev Command | Watches |
|-----------|------------|---------|
| FastAPI | `uvicorn main:app --reload --host 0.0.0.0` | `*.py` |
| Django | `python manage.py runserver 0.0.0.0:8000` | `*.py` |
| Flask | `flask run --reload --host 0.0.0.0` | `*.py` |
| Express | `nodemon index.js` or `tsx watch` | `*.js/*.ts` |
| Vite | `vite --host 0.0.0.0 --port 3000` | `src/`, `index.html` |
| Next.js | `next dev --hostname 0.0.0.0 --port 3000` | `pages/`, `app/` |
| Go (Air) | `air` | `*.go` |
| Rust (cargo-watch) | `cargo watch -x run` | `*.rs` |
| Rails | `rails server -b 0.0.0.0` | `app/`, `config/` |
| Spring Boot | `./mvnw spring-boot:run` | `src/` (with devtools) |

## Justfile Setup

Import worktree.just from the project justfile:

```just
import "worktree-wizard/worktree.just"
```

Override config vars **before** the import if needed:

```just
wt-health-timeout := "90"
import "worktree-wizard/worktree.just"
```

## .wt-required-tools

List CLI tools the project needs beyond git/just/docker:

```
# .wt-required-tools
node
psql
```

## Post-Setup Hook

For custom logic after worktree creation (migrations, seed data):

```bash
#!/bin/bash
# scripts/wt-post-setup.sh
echo "Running migrations..."
docker compose exec -T db psql -U postgres -c "CREATE DATABASE app;" 2>/dev/null || true
```

Must be executable: `chmod +x scripts/wt-post-setup.sh`

## .gitignore Additions

Always add:
```
.worktrees/
.docker-data/
.env*.local
```

## Project Shapes

### Single Service
One backend or microservice with optional database:
- 1 app service + 0-2 infrastructure services (db, cache)
- Volume mount at project root: `./:/app` or `./src:/app/src`

### Monolith (Backend + Frontend)
Separate subdirectories for backend and frontend:
- Backend service: volume `./backend:/app`
- Frontend service: volume `./frontend:/app` + `/app/node_modules`
- Each gets its own `wt.base-port`
- Frontend references backend via service name internally, via `WT_BACKEND_PORT` for browser access

## Additional Resources

### Reference Files
- **`references/stack-patterns.md`** — Dockerfile templates, dev commands, and compose service blocks per framework (Python, Node, Go, Rust, Ruby, Java)
