---
name: bosun
description: "Get started with Bosun -- what it is, how to set it up, and how to use it"
---

Guide the user through getting started with **Bosun** and help them understand how it works. Use resource files for deep dives into specific topics.

## What is Bosun?

Bosun is a single-binary CLI tool that brings GitOps workflows to homelab Docker Compose deployments. Write short manifest files, bosun generates Docker Compose + Traefik + Gatus configs. Push to git, bosun reconciles your server automatically. No Kubernetes required.

**The pitch:** Write a 10-line manifest, get a fully configured service with routing, monitoring, health checks, and compose orchestration -- all deployed automatically when you push to git.

## Nautical Terminology

Everything uses nautical/Below Deck naming:

| Term | Meaning |
|------|---------|
| **Yacht** | Your server running Docker Compose |
| **Crew** | Your containers |
| **Captain** | GitHub (gives orders via webhooks) |
| **Manifest** | Service definitions (what to deploy) |
| **Provisions** | Reusable config templates (how to deploy) |
| **Radio** | Webhook/tunnel for receiving push events |
| **Bosun** | The CLI tool itself (receives orders, deploys crew) |

## Prerequisites

Before setting up Bosun, ensure the following are installed:

- **Docker + Docker Compose v2** -- `docker compose version` to verify
- **Git** -- for repository operations
- **SOPS + Age** -- for secret encryption (`age-keygen` to generate keys, `sops` CLI for encryption)
- **Go 1.24+** -- only if building from source
- **Linux or macOS** -- tested on Unraid, Debian, Ubuntu, macOS
- **Tailscale** (optional) -- for webhook tunnel via `bosun radio`

## Installation

Pick one method:

```bash
# Quick install (recommended) -- downloads, verifies checksum, installs to /usr/local/bin
curl -fsSL https://raw.githubusercontent.com/cameronsjo/bosun/main/scripts/install.sh | bash

# Via Go
go install github.com/cameronsjo/bosun/cmd/bosun@latest

# From source
git clone https://github.com/cameronsjo/bosun.git
cd bosun && make build    # output at build/bosun
```

Update an existing install:

```bash
bosun update              # Update to latest release
bosun update --check      # Check for updates without installing
```

## Initial Setup

1. **Generate an Age encryption key** (if not already done):
   ```bash
   age-keygen -o ~/.config/sops/age/keys.txt
   ```

2. **Create `.sops.yaml`** in the project root with the public key from step 1.

3. **Initialize the project:**
   ```bash
   bosun init              # Interactive setup wizard
   bosun init --yes        # Non-interactive (for CI/CD)
   bosun init my-homelab   # Initialize in a new directory
   ```

4. **Run pre-flight checks:**
   ```bash
   bosun doctor
   ```
   Validates Docker, Git, SOPS, Age key, manifest directory, and webhook status.

5. **Start your services:**
   ```bash
   bosun yacht up
   ```

## Command Groups at a Glance

| Group | Commands | Purpose |
|-------|----------|---------|
| **Setup** | `init`, `doctor`, `update` | Project creation, health checks, updates |
| **Yacht** | `yacht up/down/restart/status` | Manage Docker Compose stacks |
| **Crew** | `crew list/logs/inspect/restart` | Manage individual containers |
| **Manifest** | `provision`, `provisions`, `create`, `lint`, `render`, `chart` | Build and validate service configs |
| **GitOps** | `daemon`, `reconcile`, `trigger`, `drift` | Automated deployment pipeline |
| **Radio** | `radio test/status` | Webhook and tunnel connectivity |
| **Alerts** | `alert status/test` | Deploy notification testing and status |
| **Diagnostics** | `status`, `log`, `doctor`, `validate` | Health monitoring and debugging |
| **Emergency** | `mayday`, `overboard`, `restore` | Error triage, rollback, force-remove, backup restore |
| **Utilities** | `update`, `completion` | Self-update, shell completions |

For complete command details with all flags and examples, see `@resources/commands.md`.

## Core Workflow

**For manual deployments:**

```bash
bosun provision mystack        # Render manifest -> compose/traefik/gatus files
bosun provision mystack -n     # Preview without writing (dry run)
bosun yacht up                 # Start/update services
bosun crew list                # Verify containers are running
bosun drift --live             # Check for drift between config and running state
```

**For automated GitOps:**

```bash
bosun daemon                   # Run long-lived daemon (webhook + polling)
# ... push changes to git ...
# Bosun automatically: pulls repo -> decrypts secrets -> templates configs -> deploys
bosun trigger                  # Or manually trigger reconciliation
bosun drift                    # Check for drift between deploys
```

For full GitOps setup details, see `@resources/gitops.md`.

## Writing Service Manifests

A manifest describes what to deploy. Provisions describe how. Bosun renders both into Docker Compose, Traefik, and Gatus configs.

**Minimal example** (a web app with routing and monitoring):

```yaml
# manifest/services/myapp.yml
name: myapp
provisions: [container, reverse-proxy, monitoring]
config:
  image: ghcr.io/org/myapp:latest
  port: 3000
  subdomain: myapp
  domain: example.com
```

**Full example** (web app with Postgres sidecar, health checks, and homepage entry):

```yaml
name: norish
provisions: [container, healthcheck, homepage, reverse-proxy, monitoring]
config:
  image: ghcr.io/norishapp/norish:latest
  port: 3000
  subdomain: recipes
  domain: example.com
  group: Apps
  icon: mdi-food
  description: Recipe manager
services:
  postgres:
    version: 17
    db: norish
    db_password: "{{ $secrets.apps.norish.db_password }}"
```

For the complete manifest schema, variable interpolation rules, and provision types, see `@resources/manifests.md`.

## Configuration

Bosun finds its project root by searching upward from the current directory for:
- `bosun.yaml` or `bosun.yml` config file
- `bosun/` directory with `docker-compose.yml`
- `manifest/` or `manifests/` directory

For the full config file reference and environment variables, see `@resources/configuration.md`.

## Common Tasks

| Task | Command |
|------|---------|
| Start all services | `bosun yacht up` |
| Start specific services | `bosun yacht up traefik redis` |
| Check container health | `bosun crew list` |
| View container logs | `bosun crew logs <name> -f` |
| Preview rendered config | `bosun provision mystack -n` |
| Show diff before applying | `bosun provision mystack -d` |
| Run one-shot deploy | `bosun reconcile` |
| Dry-run deploy | `bosun reconcile -n` |
| Check for drift | `bosun drift --live` |
| View recent errors | `bosun mayday` |
| Rollback to snapshot | `bosun mayday -r interactive` |
| Force-remove stuck container | `bosun overboard <name>` |
| Scaffold new service | `bosun create webapp myapp` |
| Validate manifests | `bosun lint` |
| Check system health | `bosun status` |
| Restore from backup | `bosun restore` or `bosun restore -l` |
| Test alert providers | `bosun alert test` |
| Set up shell completions | `bosun completion bash \| sudo tee /etc/bash_completion.d/bosun` |

## Troubleshooting Quick Reference

| Problem | Fix |
|---------|-----|
| "project root not found" | Ensure you're inside a bosun project, or use `bosun --root /path` |
| "connect to docker" | Check Docker is running (`docker ps`), check socket permissions |
| "sops decrypt failed" | Verify `sops --version`, check age key at `~/.config/sops/age/keys.txt` |
| "docker compose: command not found" | Install Docker Compose v2: `docker compose version` |
| SSH failures (remote deploy) | Test `ssh user@host exit`, check `ssh-add -l` |

Run `bosun doctor` to diagnose most setup issues.

## Pirate Mode (Easter Egg)

All commands have nautical aliases. Run `bosun yarr` to see them:

| Command | Alias |
|---------|-------|
| `yacht` | `hoist` |
| `crew` | `scallywags` |
| `provision` | `plunder`, `loot`, `forge` |
| `radio` | `parrot` |
| `status` | `bridge` |
| `doctor` | `checkup` |
| `mayday` | `mutiny` |
| `overboard` | `plank` |
| `init` | `christen` |

## Key Project Files

When working on Bosun itself:

| Path | Purpose |
|------|---------|
| `cmd/bosun/main.go` | CLI entry point |
| `internal/cmd/` | Cobra command definitions (one file per command) |
| `internal/manifest/` | YAML rendering engine for service manifests |
| `internal/reconcile/` | GitOps engine (clone, decrypt, template, deploy) |
| `internal/docker/` | Docker SDK wrapper |
| `internal/config/` | Configuration loading and project discovery |
| `Makefile` | Build, test, lint, CI targets (`make help` for full list) |

## Development

```bash
make build              # Build binary -> build/bosun
make test               # Run tests
make test-cover         # Run tests with coverage report
make lint               # Run linter (requires golangci-lint)
make ci                 # Run full CI pipeline locally via Dagger
make run ARGS="doctor"  # Run without building
```

## Resource Files

For deeper information on specific topics:

- `@resources/commands.md` -- Every CLI command with flags, examples, and behavior
- `@resources/manifests.md` -- Manifest system: schemas, provisions, stacks, interpolation
- `@resources/gitops.md` -- GitOps pipeline: daemon, reconciliation, webhooks, drift
- `@resources/configuration.md` -- Config file reference, environment variables, project structure
