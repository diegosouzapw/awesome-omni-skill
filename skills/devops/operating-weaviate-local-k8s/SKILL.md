---
name: operating-weaviate-local-k8s
description: Deploys, upgrades, and manages local Weaviate clusters in Kind (Kubernetes in Docker) using the weaviate-local-k8s tool. Determines optimal cluster configuration from requirements including version, replicas, modules, authentication, and features. Use when deploying Weaviate for testing, development, bug reproduction, CI/CD integration, or when the user describes a cluster scenario to set up.
---

# Operating Weaviate Local K8s Clusters

Deploy and manage local Weaviate clusters using the `local-k8s.sh` tool. This skill translates deployment requirements into the correct environment variables and commands.

`$WEAVIATE_LOCAL_K8S_DIR` refers to the weaviate-local-k8s repository root. Set it or `cd` into the repo before running commands.

## Quick Reference

```bash
# Setup
WEAVIATE_VERSION="1.28.0" ./local-k8s.sh setup

# Upgrade
WEAVIATE_VERSION="1.29.0" ./local-k8s.sh upgrade

# Clean (destructive, all data lost)
./local-k8s.sh clean
```

## Decision Guide

Given a scenario, determine which env vars to set:

### Cluster Size

| Scenario | WORKERS | REPLICAS |
|----------|---------|----------|
| Quick test / dev | 0 (default) | 1 (default) |
| Multi-node / HA | 2 | 3 |
| Stress / performance | 4 | 5 |

Rule: `WORKERS >= REPLICAS - 1` (control-plane counts as a node).

### Feature Selection

| Need | Env vars |
|------|----------|
| RBAC auth | `RBAC=true` |
| OIDC auth | `OIDC=true RBAC=true` |
| Dynamic users | `DYNAMIC_USERS=true RBAC=true` |
| S3 backups | `ENABLE_BACKUP=true` |
| Tenant offloading | `S3_OFFLOAD=true` |
| Usage metrics | `USAGE_S3=true ENABLE_RUNTIME_OVERRIDES=true` |
| Modules | `MODULES="text2vec-transformers"` |
| Fast startup (no monitoring) | `OBSERVABILITY=false` |
| Custom Helm chart | `HELM_BRANCH="main"` |
| Local images (offline/rate limits) | `./local-k8s.sh setup --local-images` |
| Custom Weaviate env vars | `VALUES_INLINE="--set env.VAR_NAME=value"` (see VALUES_INLINE section) |
| Custom Helm values | `VALUES_INLINE="--set key=value"` (see VALUES_INLINE section) |
| Test from local source | Build image + `--local-images` (see Build from Local Source) |

### Timeout Estimation

```
timeout = (REPLICAS * 100s) + modules(1200s) + backup(100s) + observability(100s) + dash0(100s)
```

Override: `HELM_TIMEOUT="20m"`

## Common Deployment Patterns

### Basic Single-Node

```bash
WEAVIATE_VERSION="1.28.0" ./local-k8s.sh setup
```

Verify: `curl localhost:8080/v1/meta`

### Multi-Node HA Cluster

```bash
WORKERS=2 REPLICAS=3 WEAVIATE_VERSION="1.28.0" ./local-k8s.sh setup
```

Verify:
```bash
curl localhost:8080/v1/nodes | jq '.nodes[] | {name, status}'
curl localhost:8080/v1/cluster/statistics | jq '{synchronized, count: (.statistics | length)}'
```

### With RBAC

```bash
WEAVIATE_VERSION="1.28.0" RBAC=true ./local-k8s.sh setup
```

Default: user=`admin-user`, key=`admin-key` (root). Test: `curl -H "Authorization: Bearer admin-key" localhost:8080/v1/meta`

**Note**: With just `RBAC=true`, only static API key users (configured in Helm values) are available. To create/manage users at runtime via API or CLI (`weaviate-cli create user`), add `DYNAMIC_USERS=true`:

```bash
WEAVIATE_VERSION="1.28.0" RBAC=true DYNAMIC_USERS=true ./local-k8s.sh setup
```

### With Modules

```bash
WEAVIATE_VERSION="1.28.0" MODULES="text2vec-transformers" HELM_TIMEOUT="20m" ./local-k8s.sh setup
```

Module images are large (~1-5GB). Timeout auto-increases by 1200s.

### With Backup (MinIO S3)

```bash
WEAVIATE_VERSION="1.28.0" ENABLE_BACKUP=true ./local-k8s.sh setup
```

MinIO on port 9000. Credentials: `aws_access_key` / `aws_secret_key`.

When both `ENABLE_BACKUP=true` and `USAGE_S3=true` are enabled, MinIO serves both purposes with separate buckets:
- `weaviate-backups/` — backup data
- `weaviate-usage/` — usage/billing metrics at `billing/{node-name}/{timestamp}.json` (uploaded every ~60s)

Browse usage data: `kubectl exec -n weaviate minio -- mc alias set local http://localhost:9000 aws_access_key aws_secret_key && kubectl exec -n weaviate minio -- mc ls --recursive local/weaviate-usage/`

### Full-Featured

```bash
WORKERS=2 REPLICAS=3 WEAVIATE_VERSION="1.28.0" \
  MODULES="text2vec-transformers,generative-openai" \
  RBAC=true ENABLE_BACKUP=true S3_OFFLOAD=true \
  ENABLE_RUNTIME_OVERRIDES=true OBSERVABILITY=true \
  HELM_TIMEOUT="20m" ./local-k8s.sh setup
```

### Minimal (Fastest Startup)

```bash
WEAVIATE_VERSION="1.28.0" OBSERVABILITY=false ./local-k8s.sh setup
```

### Build and Deploy from Local Weaviate Source

Test code changes by building a Docker image from a local Weaviate source checkout and deploying it to the Kind cluster. Run the build from the Weaviate source directory, then deploy from the weaviate-local-k8s directory.

**Step 1: Build the image** (from the Weaviate source directory)

```bash
# Generate the image tag from the current branch/commit
IMAGE_TAG=$(./tools/dev/image-tag.sh)

# Build and load into local Docker daemon
DOCKER_BUILDKIT=1 docker buildx build --load \
  -t semitechnologies/weaviate:${IMAGE_TAG} \
  -f Dockerfile .
```

The tag format is `<version>-dev-<short-sha>[.arm64]` (e.g., `1.36.0-dev-957b003.arm64`). The `tools/dev/image-tag.sh` script in the Weaviate repository generates it automatically.

**Important**: Use `docker buildx build --load` (not `make weaviate-image`) because `make weaviate-image` uses the buildx container driver which does NOT load the image into the local Docker daemon. The `--load` flag ensures the image is available to Kind.

**Step 2: Deploy with the local image** (from the weaviate-local-k8s directory)

```bash
WEAVIATE_VERSION="${IMAGE_TAG}" ./local-k8s.sh setup --local-images
```

The `--local-images` flag loads images from the local Docker daemon into the Kind cluster and sets `imagePullPolicy=Never`, preventing Kubernetes from trying to pull from a remote registry.

**Full example** (assuming both repos are sibling directories):

```bash
# Build from Weaviate source
cd ../weaviate
IMAGE_TAG=$(./tools/dev/image-tag.sh)
DOCKER_BUILDKIT=1 docker buildx build --load \
  -t semitechnologies/weaviate:${IMAGE_TAG} \
  -f Dockerfile .

# Deploy from weaviate-local-k8s
cd ../weaviate-local-k8s
WEAVIATE_VERSION="${IMAGE_TAG}" WORKERS=2 REPLICAS=3 ./local-k8s.sh setup --local-images
```

## Port Mapping

With `EXPOSE_PODS=true` (default), each replica gets dedicated ports:

| Type | Service | Pod 0 | Pod 1 | Pod N |
|------|---------|-------|-------|-------|
| HTTP | 8080 | 8081 | 8082 | 8080+(N+1) |
| gRPC | 50051 | 50052 | 50053 | 50051+(N+1) |
| Metrics | 2112 | 2113 | 2114 | 2112+(N+1) |
| Profiler | 6060 | 6061 | 6062 | 6060+(N+1) |

Additional: Prometheus=9091, Grafana=3000, Keycloak=9090, MinIO=9000.

## Verification Workflow

```bash
# 1. Pods running
kubectl get pods -n weaviate

# 2. Nodes healthy
curl localhost:8080/v1/nodes | jq '.nodes[] | {name, status}'

# 3. Raft synchronized (multi-node)
curl localhost:8080/v1/cluster/statistics | jq '{synchronized, count: (.statistics | length)}'

# 4. With RBAC
curl -H "Authorization: Bearer admin-key" localhost:8080/v1/meta
```

## Upgrade Workflow

**IMPORTANT**: The upgrade command needs the same env vars used during setup. Env vars like `OBSERVABILITY`, `ENABLE_BACKUP`, `USAGE_S3`, etc. are NOT remembered from setup — they must be passed again on every upgrade. Omitting them can cause failures (e.g., missing CRDs, misconfigured Helm values).

```bash
# Standard upgrade (rolling, non-destructive)
# Pass the same env vars used during setup
WEAVIATE_VERSION="1.29.0" OBSERVABILITY=false ./local-k8s.sh upgrade

# Upgrade with StatefulSet deletion (for major version jumps or adding new features)
WEAVIATE_VERSION="1.29.0" DELETE_STS=true ./local-k8s.sh upgrade
```

Use `DELETE_STS=true` when:
- Major version jumps with schema changes
- Adding new features to an existing cluster (e.g., adding `USAGE_S3=true` or `ENABLE_BACKUP=true`) — this avoids Helm duplicate env var conflicts

The Kind cluster is NOT recreated. Only Weaviate pods are updated.

## Runtime Overrides

When `ENABLE_RUNTIME_OVERRIDES=true`, a ConfigMap named `weaviate-runtime-overrides` is created (polled every 30s). Set override values via `VALUES_INLINE` using the pattern:

```
--set runtime_overrides.values.<snake_case_key>=<value>
```

The key is always the snake_case form of the Weaviate config name (e.g., `query_slow_log_enabled` → `QUERY_SLOW_LOG_ENABLED`).

```bash
# Example: Object TTL
ENABLE_RUNTIME_OVERRIDES=true \
  VALUES_INLINE="--set runtime_overrides.values.objects_ttl_delete_schedule=@every\ 10s --set runtime_overrides.values.objects_ttl_batch_size=100" \
  WEAVIATE_VERSION="1.36.0" ./local-k8s.sh setup
```

### Available Override Keys

**Operational**: `async_replication_disabled`, `autoschema_enabled`, `default_quantization`, `inverted_sorter_disabled`, `maximum_allowed_collections_count`, `objects_ttl_delete_schedule`, `objects_ttl_batch_size`, `objects_ttl_pause_every_no_batches`, `objects_ttl_pause_duration`, `operational_mode`, `query_slow_log_enabled`, `query_slow_log_threshold`, `replica_movement_minimum_async_wait`, `replicated_indices_request_queue_enabled`, `revectorize_check_disabled`, `tenant_activity_read_log_level`, `tenant_activity_write_log_level`

**Raft**: `raft_drain_sleep`, `raft_timeouts_multiplier`

**Usage Tracking**: `usage_gcs_bucket`, `usage_gcs_prefix`, `usage_policy_version`, `usage_s3_bucket`, `usage_s3_prefix`, `usage_scrape_interval`, `usage_shard_jitter_interval`, `usage_verify_permissions`

**OIDC**: `authentication_oidc_certificate`, `authentication_oidc_client_id`, `authentication_oidc_groups_claim`, `authentication_oidc_issuer`, `authentication_oidc_jwks_url`, `authentication_oidc_scopes`, `authentication_oidc_skip_client_id_check`, `authentication_oidc_username_claim`

Verify: `kubectl get configmap -n weaviate weaviate-runtime-overrides -o yaml`

## Dependencies Between Features

1. `USAGE_S3=true` requires `ENABLE_RUNTIME_OVERRIDES=true`
2. `DASH0=true` requires `DASH0_TOKEN`
3. `AUTH_CONFIG` file must exist when provided
4. `DOCKER_CONFIG` must be absolute path (no `~`)
5. `rbac` and `admin_list` are mutually exclusive in auth configs

## VALUES_INLINE: Custom Helm Overrides

`VALUES_INLINE` passes raw Helm `--set` flags appended last to the Helm command, giving them the **highest precedence**.

**CRITICAL SYNTAX**: Each value requires its own `--set` prefix. Do NOT use commas to separate multiple values.

```bash
# CORRECT - each value has its own --set prefix
VALUES_INLINE="--set env.MY_VAR=value1 --set env.OTHER_VAR=value2"

# WRONG - do NOT use comma-separated values without --set
# VALUES_INLINE="env.MY_VAR=value1,env.OTHER_VAR=value2"    # WILL NOT WORK
# VALUES_INLINE="--set env.MY_VAR=value1,env.OTHER_VAR=value2"  # WILL NOT WORK
```

### Passing Custom Weaviate Environment Variables

Use `--set env.VARIABLE_NAME=value` to inject env vars into Weaviate pods. The `env.` prefix maps to the `env:` section in the Helm chart's `values.yaml`.

```bash
# Single env var
VALUES_INLINE="--set env.LOG_LEVEL=debug" \
  WEAVIATE_VERSION="1.28.0" ./local-k8s.sh setup

# Multiple env vars
VALUES_INLINE="--set env.LAZY_LOAD_SHARD_COUNT_THRESHOLD=100 --set env.LAZY_LOAD_SHARD_SIZE_THRESHOLD_GB=0.1" \
  WEAVIATE_VERSION="1.28.0" ./local-k8s.sh setup

# Env vars combined with other Helm values
VALUES_INLINE="--set env.QUERY_DEFAULTS_LIMIT=50 --set resources.requests.memory=4Gi --set resources.limits.memory=8Gi" \
  WEAVIATE_VERSION="1.28.0" ./local-k8s.sh setup
```

### Other Common VALUES_INLINE Uses

```bash
# Resource limits
VALUES_INLINE="--set resources.requests.memory=4Gi --set resources.limits.memory=8Gi"

# Storage class
VALUES_INLINE="--set storage.storageClassName=fast-ssd"

# Image registry
VALUES_INLINE="--set image.registry=cr.weaviate.io --set image.pullPolicy=Always"

# LSM access strategy
VALUES_INLINE="--set env.PERSISTENCE_LSM_ACCESS_STRATEGY=pread"
```

### Verify Env Vars Were Applied

```bash
kubectl exec -n weaviate weaviate-0 -- env | grep MY_VAR
```

## Helm Values Precedence

Precedence from lowest to highest:
1. `AUTH_CONFIG` file (applied via `-f`, lowest precedence)
2. `values-override.yaml` (applied via `-f`, overrides AUTH_CONFIG)
3. Generated `--set` flags (from script logic, override all `-f` files)
4. `VALUES_INLINE` `--set` flags (appended last, highest precedence)

Note: Helm `--set` flags always override `-f` files regardless of position on the command line.

## Detailed References

- **Environment variables**: See [references/environment-variables.md](references/environment-variables.md) for complete env var reference (40+ variables)
- **Deployment patterns**: See [references/deployment-patterns.md](references/deployment-patterns.md) for advanced Helm configuration and scenarios
- **Authentication**: See [references/auth-config.md](references/auth-config.md) for RBAC, OIDC, custom AUTH_CONFIG, and dynamic users
- **Modules**: See [references/modules-config.md](references/modules-config.md) for module system, custom modules, and image management
- **Observability**: See [references/observability-config.md](references/observability-config.md) for Prometheus, Grafana, Dash0, and debugging
- **Troubleshooting**: See [references/troubleshooting.md](references/troubleshooting.md) for common issues and solutions
- **CLI integration**: See [references/cli-integration.md](references/cli-integration.md) for weaviate-cli configuration
