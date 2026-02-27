---
name: deck-cli
description: Manage Kong Gateway and Konnect declaratively using decK CLI. Use when working with Kong Gateway configurations, syncing state, converting OpenAPI specs to Kong config, validating configurations, or automating Kong deployments. Handles declarative configuration files, API lifecycle automation, and Kong entity management.
compatibility: Requires decK CLI installed. Works with Kong Gateway (on-prem/cloud) and Kong Konnect.
metadata:
  author: Kong
  version: "1.0"
  deck_commands: gateway,file
---

# decK CLI Agent Skill

## Overview

decK is Kong's declarative configuration tool for API Lifecycle Automation. It allows you to manage Kong Gateway and Konnect configurations as code.

**Use this skill when:**
- Configuring or updating Kong Gateway declaratively
- Converting OpenAPI specs to Kong configuration
- Syncing local configuration to Kong Gateway/Konnect
- Validating Kong configurations
- Managing Kong entities (Services, Routes, Plugins, Consumers, etc.)
- Setting up CI/CD for Kong configurations
- Comparing local vs. live Kong state

## Quick Reference

### Common Commands

| Command | Purpose |
|---------|---------|
| `deck gateway sync` | Apply local config to Kong Gateway |
| `deck gateway diff` | Compare local config with Kong state |
| `deck gateway dump` | Export current Kong state to file |
| `deck file validate` | Validate configuration file locally |
| `deck file openapi2kong` | Convert OpenAPI to Kong config |
| `deck file render` | Render final configuration |

## Getting Started

### 1. Check decK Installation

```bash
deck version
```

If decK is not installed, guide the user to install it via:
- **macOS**: `brew install deck`
- **Linux**: Download from GitHub releases
- **Docker**: `docker run kong/deck`

### 2. Verify Connectivity

For Kong Gateway (default: localhost:8001):
```bash
deck gateway ping
```

For Konnect:
```bash
deck gateway ping --konnect-token $KONNECT_TOKEN --konnect-control-plane-name <cp-name>
```

## Core Workflows

### Workflow 1: Export Current Kong Configuration

Export everything:
```bash
deck gateway dump -o kong.yaml
```

Export specific workspace:
```bash
deck gateway dump --workspace dev -o dev-workspace.yaml
```

Export with tags:
```bash
deck gateway dump --select-tag production -o production.yaml
```

### Workflow 2: Apply Configuration Changes

**Preview changes first (dry run):**
```bash
deck gateway diff -s kong.yaml
```

**Apply changes:**
```bash
deck gateway sync -s kong.yaml
```

**Apply without deleting (add/update only):**
```bash
deck gateway apply -s kong.yaml
```

### Workflow 3: Convert OpenAPI to Kong Configuration

```bash
deck file openapi2kong -s openapi.yaml -o kong-config.yaml
```

With additional options:
```bash
deck file openapi2kong \
  -s openapi.yaml \
  -o kong-config.yaml \
  --select-tag api-v1 \
  --uuid-base my-api
```

### Workflow 4: Validate Configuration

**Local validation (no Kong connection needed):**
```bash
deck file validate -s kong.yaml
```

**Validate against live Kong Gateway:**
```bash
deck gateway validate -s kong.yaml
```

### Workflow 5: Lint Configuration

Check for best practices and potential issues:
```bash
deck file lint -s kong.yaml
```

## Configuration Files

### Basic Structure

decK uses YAML files with Kong entity definitions:

```yaml
_format_version: "3.0"
_konnect:
  control_plane_name: my-control-plane

services:
- name: example-service
  url: http://example.com
  routes:
  - name: example-route
    paths:
    - /api
    methods:
    - GET
    - POST
  plugins:
  - name: rate-limiting
    config:
      minute: 100
      policy: local

plugins:
- name: cors
  config:
    origins:
    - "*"
```

### Multiple Files

Split configuration across files:
```bash
deck gateway sync -s kong.yaml -s plugins.yaml -s routes.yaml
```

Merge files for rendering:
```bash
deck file merge -o combined.yaml kong.yaml plugins.yaml routes.yaml
```

## Authentication Configuration

### Kong Gateway (On-Prem)

Using Admin API URL:
```bash
deck gateway sync --kong-addr http://localhost:8001 -s kong.yaml
```

With RBAC token:
```bash
deck gateway sync \
  --kong-addr https://admin.kong.example.com \
  --headers "Kong-Admin-Token:$KONG_ADMIN_TOKEN" \
  -s kong.yaml
```

### Kong Konnect

Using personal access token:
```bash
export KONNECT_TOKEN=kpat_xxx...
deck gateway sync \
  --konnect-token $KONNECT_TOKEN \
  --konnect-control-plane-name production \
  -s kong.yaml
```

Using configuration file (`~/.deck.yaml`):
```yaml
konnect-token: kpat_xxx...
konnect-control-plane-name: production
```

## Advanced Features

### Working with Tags

**Select specific tags:**
```bash
deck gateway sync --select-tag team-a --select-tag production -s kong.yaml
```

**Add tags to all entities:**
```bash
deck file add-tags -s kong.yaml -o tagged.yaml team-b,staging
```

**Remove tags:**
```bash
deck file remove-tags -s kong.yaml -o cleaned.yaml old-tag
```

### Workspace Management (Enterprise)

**Sync to specific workspace:**
```bash
deck gateway sync --workspace production -s kong.yaml
```

**Dump workspace:**
```bash
deck gateway dump --workspace staging -o staging.yaml
```

### De-duplicate Plugin Configuration

Reference shared plugin configs:
```yaml
_format_version: "3.0"

plugins:
- name: rate-limiting
  _plugin_config_reference: global-rate-limit

_plugin_configs:
  global-rate-limit:
    minute: 100
    hour: 5000
```

### Using Environment Variables

Reference environment variables in config files:
```yaml
services:
- name: api-service
  url: $(BACKEND_URL)
  routes:
  - name: api-route
    paths:
    - /api
```

Render with values:
```bash
export BACKEND_URL=http://api.example.com
deck file render -s kong.yaml -o rendered.yaml
```

## File Conversion & Transformation

### Convert Kong 2.x to 3.x Format

```bash
deck file convert \
  --from kong-gateway-2.x \
  --to kong-gateway-3.x \
  --input-file kong-2x.yaml \
  --output-file kong-3x.yaml
```

### Convert to Kubernetes Ingress Controller

```bash
deck file kong2kic -s kong.yaml -o k8s-manifests/
```

### Convert to Terraform (Konnect Only)

```bash
deck file kong2tf -s kong.yaml -o terraform/
```

### Patch Configuration

Update specific values:
```bash
deck file patch -s kong.yaml -o patched.yaml \
  --selector "services[*].retries" \
  --value "5"
```

### Add Plugins

Add plugin to all services:
```bash
deck file add-plugins -s kong.yaml -o with-plugins.yaml \
  --selector "services[*]" \
  rate-limiting minute=100
```

### Apply Namespace

Prefix routes by path:
```bash
deck file namespace -s kong.yaml -o namespaced.yaml \
  --path-prefix /v1
```

## CI/CD Integration

### Basic Pipeline Example

```bash
#!/bin/bash
# 1. Validate locally
deck file validate -s kong.yaml || exit 1

# 2. Lint for best practices
deck file lint -s kong.yaml || exit 1

# 3. Preview changes
deck gateway diff -s kong.yaml

# 4. Require approval, then sync
deck gateway sync -s kong.yaml
```

### GitHub Actions

```yaml
- name: Setup decK
  uses: kong/setup-deck@v1
  with:
    deck-version: 1.38.1

- name: Validate Configuration
  run: deck file validate -s kong.yaml

- name: Sync to Kong
  run: deck gateway sync -s kong.yaml
  env:
    KONNECT_TOKEN: ${{ secrets.KONNECT_TOKEN }}
```

## Troubleshooting

### Common Issues

**Connection errors:**
```bash
# Test connectivity
deck gateway ping --verbose

# Check Admin API endpoint
curl http://localhost:8001/
```

**Format validation errors:**
- Ensure `_format_version` is set correctly (`3.0` for Kong 3.x)
- Check YAML syntax with a validator
- Run `deck file validate` for detailed errors

**Entity conflicts:**
- Use `deck gateway diff` to see what will change
- Use `--select-tag` to limit scope
- Check for duplicate names or IDs

**Authentication issues:**
- Verify token is valid and not expired
- Check RBAC permissions for the token
- Ensure control plane name is correct (Konnect)

### Debug Mode

Enable verbose output:
```bash
deck gateway sync -s kong.yaml --verbose 3
```

## Best Practices

1. **Always diff before sync:** Use `deck gateway diff` to preview changes
2. **Use tags for segmentation:** Organize configs by team, environment, or API
3. **Version control:** Keep decK files in Git
4. **Environment-specific configs:** Use separate files for dev/staging/prod
5. **Validate early:** Run `deck file validate` before committing
6. **Test in non-prod first:** Validate changes in lower environments
7. **Use workspace isolation:** Leverage workspaces for multi-tenancy
8. **Back up before major changes:** Run `deck gateway dump` first

## Important Notes

- decK manages Kong Gateway state declaratively - it will **delete** entities not in your config file (unless using `apply` instead of `sync`)
- Always use `--select-tag` in production to limit the scope of operations
- decK does not manage:
  - Kong configuration properties (kong.conf)
  - Key-auth credentials (stored separately for security)
  - Some Enterprise-only features (requires special flags)

## Quick Checklist

Before running `deck gateway sync`:
- [ ] Run `deck file validate -s <file>`
- [ ] Run `deck gateway diff -s <file>` to preview
- [ ] Backup current state: `deck gateway dump -o backup.yaml`
- [ ] Verify authentication works: `deck gateway ping`
- [ ] Consider using `--dry-run` flag first
- [ ] Have rollback plan ready

## Next Steps

For detailed command reference and advanced scenarios, consult:
- [Official decK documentation](https://docs.konghq.com/deck/)
- [decK GitHub repository](https://github.com/Kong/deck)
- The `references/` directory in this skill for command-specific details
