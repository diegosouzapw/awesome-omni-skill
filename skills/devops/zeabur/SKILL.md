---
name: zeabur
description: "Zeabur cloud platform assistant for deployment, management, and optimization. Use when: (1) Deploying applications to Zeabur (Git, Docker, local upload, templates), (2) Managing Zeabur services via CLI (npx zeabur) or GraphQL API, (3) Configuring domains, environment variables, networking, or volumes, (4) Troubleshooting deployment failures, connectivity issues, or build errors, (5) Optimizing Zeabur costs, performance, or high availability architecture, (6) Working with Zeabur templates (YAML spec creation and deployment), (7) Setting up CI/CD pipelines with Zeabur, (8) Managing databases and prebuilt services on Zeabur, (9) Any mention of zeabur, zeabur.yaml, zbpack, zeabur.app, or zeabur CLI commands. Triggers: zeabur, deploy to zeabur, zeabur cli, zeabur api, zeabur template, zbpack, zeabur.app, zeabur domain, zeabur variable, zeabur service."
---

# Zeabur Assistant

## Tool Priority

When both CLI and GraphQL API (or MCP) can accomplish a task, prefer CLI (`npx zeabur`). Fall back to GraphQL API when:
- CLI command forces interactive selection that cannot be bypassed (e.g., `project delete`, `service ls` without working context)
- The operation requires custom query fields not available via CLI output
- CLI command fails or returns insufficient information

Priority order: **CLI > GraphQL API > MCP**

**CLI interactive limitation**: Many CLI commands ignore `--id`/`--name` flags and force interactive project/service selection, which fails in non-interactive environments. When this happens, fall back to GraphQL API immediately — do not retry the same CLI command.

## Core Concepts

Project, Service, Environment, and Deployment follow standard cloud platform semantics. Key platform-specific concepts:

| Concept | Description |
|---------|-------------|
| **zbpack** | Auto-detect build tool. Identifies language/framework, generates Dockerfile. |
| **Region** | Deployment location. Common: `aws-us-west`, `aws-ap-east` (HK), `aws-ap-northeast` (JP/TW), `gcp-asia-east` (TW). Selected at project creation, cannot be changed later. |

## Deployment Decision Tree

```
User wants to deploy →
├─ Has GitHub repo?
│  └─ Yes → GitHub integration (auto CI/CD on push)
├─ Has Dockerfile or Docker image?
│  ├─ Image on registry → Docker image deploy
│  └─ Dockerfile in repo → Dockerfile deploy (auto-detected)
├─ Local source code only?
│  └─ `npx zeabur deploy` (interactive) or Dashboard drag-drop
├─ Multi-service stack (app + db + cache)?
│  └─ Template YAML (define all services in one spec)
└─ Simple function (single endpoint)?
   └─ Functions (serverless JS/Python)
```

## Authentication

Always check auth status first with `npx zeabur auth status`.

If not logged in, ask the user which method to use:

1. **API Token** — User provides a token (from Dashboard > Settings > API Keys). Login via `npx zeabur auth login --token <TOKEN>`. This enables all CLI commands without browser interaction. If CLI token login fails, fall back to GraphQL API with `Authorization: Bearer {TOKEN}` header.
2. **Interactive CLI login** — Run `npx zeabur auth login`. Requires browser interaction from the user.

**Token expiry**: If previously authenticated commands start returning 401/unauthorized, the token may have expired. Re-run `npx zeabur auth status` to confirm, then re-authenticate. See [references/troubleshooting.md](references/troubleshooting.md) for detailed auth troubleshooting.

## CLI Quick Reference

All commands via `npx zeabur`. Add `-i=false` to disable interactive mode (for CI/CD).

```bash
npx zeabur auth login              # Authenticate
npx zeabur context set project     # Set working project
npx zeabur deploy                  # Deploy from local directory
```

For complete command reference with all flags and examples, read [references/cli-commands.md](references/cli-commands.md).

## API

GraphQL endpoint: `https://api.zeabur.com/graphql`
Auth: `Authorization: Bearer {TOKEN}` (generate at Dashboard > Settings > API Keys)

For complete API reference including queries, mutations, REST upload API, and WebSocket terminal, read [references/api-reference.md](references/api-reference.md).

## Configuration

For complete configuration details (zbpack.json, template YAML, environment variables, predefined variables, volumes), read [references/deployment.md](references/deployment.md).

### Key Points

- **Build config**: `zbpack.json` in project root — set `build_command`, `start_command`, `app_dir`, `plan_type`
- **Env vars**: Set via Dashboard, CLI, or template. Reference syntax: `${VAR_NAME}`. Predefined: `${PORT}` (8080), `${ZEABUR_WEB_URL}`, `${PASSWORD}`
- **Override build**: `ZBPACK_IGNORE_DOCKERFILE=true`, `ZBPACK_DOCKERFILE_NAME=custom.Dockerfile`

## Networking

| Type | Pattern |
|------|---------|
| Public HTTP | Bind domain via Dashboard/CLI. Auto-SSL. |
| Auto domain | `service-name.zeabur.app` |
| Custom domain | CNAME to zeabur. Apex domain: use Cloudflare CNAME flattening or A record. |
| Private (internal) | `[service-name].zeabur.internal` — same project only, no bandwidth cost. |
| Non-HTTP (TCP) | Assigned hostname + port (e.g., `hkg1.clusters.zeabur.com:34567`) |

**Important**: With Cloudflare, do NOT use Full (Strict) SSL mode.

## Data Management

- **Volumes**: Mount persistent storage at a path. $0.20/GB/month. Note: volumes disable zero-downtime restart.
- **Config Editor**: Edit config files at absolute paths (e.g., `/etc/nginx/nginx.conf`). Free.
- **Backups**: Online backup for PostgreSQL/MySQL/MariaDB/MongoDB. Auto-backup daily. 7-day retention.

For template YAML format, zbpack.json config, and deployment details, read [references/deployment.md](references/deployment.md).

## Troubleshooting

For common issues and diagnostic steps, read [references/troubleshooting.md](references/troubleshooting.md).

### Quick Diagnostic Checklist

1. **Auth failure** → `npx zeabur auth status` — re-authenticate if token expired
2. **Build failure** → `npx zeabur deployment log -t=build` — check language detection, dependencies
3. **Service unreachable** → verify domain binding, port ($PORT env var), health check
4. **DB connection fail** → use private hostname (`*.zeabur.internal`), check exposed variables
5. **Env var not working** → check variable precedence, restart service
6. **SSL error** → Cloudflare users: switch from Full (Strict) to Full

## Common Recipes

For step-by-step real-world scenarios (Next.js + PostgreSQL, monorepo deploy, staging/production setup, CI/CD pipeline, Docker image deploy), read [references/recipes.md](references/recipes.md).

## Operations

For cost optimization, high availability architecture, and MCP integration setup, read [references/operations.md](references/operations.md).
