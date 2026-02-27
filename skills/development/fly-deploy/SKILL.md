---
name: fly-deploy
description: Deploy applications to Fly.io using flyctl. Handles project detection, fly.toml generation, secrets configuration, and deployment. Use when deploying apps to Fly.io, creating fly.toml files, debugging Fly.io deployment failures, or configuring Fly.io services.
---

# Fly.io Deployment

Deploy applications to Fly.io from source code or Docker images.

## Prerequisites

Ensure `flyctl` is installed and authenticated:

```bash
# Check installation
fly version

# Authenticate if needed
fly auth login
```

## Deployment Workflow

### 1. Detect Project Type

Examine the project root to determine the stack:

| Indicator | Type | Reference |
|-----------|------|-----------|
| `next.config.*` | Next.js | [references/nextjs.md](references/nextjs.md) |
| `Dockerfile` | Docker | [references/docker.md](references/docker.md) |
| `package.json` (no Next) | Node.js | [references/nodejs.md](references/nodejs.md) |
| `requirements.txt` / `pyproject.toml` | Python | [references/python.md](references/python.md) |
| `index.html` only | Static | [references/static.md](references/static.md) |

Read the appropriate reference file for framework-specific configuration.

### 2. Initialize or Configure

**New app:**
```bash
fly launch --no-deploy
```

This creates `fly.toml`. Review and adjust before deploying.

**Existing app (no fly.toml):**
```bash
fly launch --no-deploy
# Or create fly.toml manually using references/fly-toml.md
```

### 3. Set Secrets

Set environment variables that shouldn't be in fly.toml:

```bash
# Single secret
fly secrets set DATABASE_URL="postgres://..."

# Multiple secrets
fly secrets set KEY1=value1 KEY2=value2

# From .env file
cat .env | fly secrets import
```

Secrets trigger a redeploy. Use `--stage` to batch them:

```bash
fly secrets set --stage KEY1=value1
fly secrets set --stage KEY2=value2
fly secrets deploy
```

### 4. Deploy

```bash
fly deploy
```

**Common flags:**
- `--ha` — High availability (2+ machines, default)
- `--no-ha` — Single machine (dev/staging)
- `--strategy rolling|bluegreen|canary|immediate`
- `--wait-timeout 5m` — Extend for slow builds

### 5. Verify

```bash
# Check status
fly status

# View logs
fly logs

# Open in browser
fly open
```

## Troubleshooting

If deployment fails, consult [references/troubleshooting.md](references/troubleshooting.md) for common errors and fixes.

**Quick checks:**
1. Health check passing? Check `internal_port` matches app's listen port
2. App starting? Check `fly logs` for crash loops
3. Build failing? Check Dockerfile or buildpack compatibility

## Configuration Reference

For detailed fly.toml options, see [references/fly-toml.md](references/fly-toml.md).

**Key settings:**
- `primary_region` — Where machines deploy by default
- `[http_service]` — HTTP/HTTPS configuration
- `[env]` — Non-sensitive environment variables
- `[[vm]]` — CPU/memory sizing
