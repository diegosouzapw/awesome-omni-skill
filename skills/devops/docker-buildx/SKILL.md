---
name: docker-buildx
description: "Docker Buildx — multi-platform builds, BuildKit mounts, Bake, cache backends, CI integration. Use when building container images. NOT for GPU runtime or Compose."
---

# Docker Buildx

Buildx is Docker's extended build system powered by BuildKit. It replaces `docker build` with multi-platform support, advanced caching, and distributed build capabilities.

## Builders

Builders are BuildKit instances that execute builds. Choose based on needed features:

| Driver | Isolation | Multi-platform | Cache Export | Use Case |
|--------|-----------|----------------|--------------|----------|
| `docker` | None (default) | ❌ | ❌ | Basic local builds |
| `docker-container` | Container | ✅ | ✅ | Local dev with advanced features |
| `kubernetes` | Pod | ✅ | ✅ | CI/CD on K8s clusters |
| `remote` | External | ✅ | ✅ | Shared team builders |

```bash
# Create a docker-container builder (most common for local dev)
docker buildx create --name mybuilder --driver docker-container --use

# Kubernetes builder — runs BuildKit as a pod
docker buildx create --name k8s-builder --driver kubernetes \
  --driver-opt namespace=buildkit,replicas=3,rootless=true

# Remote builder — connect to existing BuildKit daemon
docker buildx create --name remote --driver remote tcp://buildkit.internal:1234

# Inspect and bootstrap
docker buildx inspect --bootstrap
docker buildx ls
```

## Multi-Platform Builds

Build images for multiple architectures in a single command:

```bash
# Build for amd64 + arm64, push to registry
docker buildx build --platform linux/amd64,linux/arm64 \
  -t registry.example.com/app:latest --push .

# QEMU emulation setup (one-time, for cross-arch on single machine)
docker run --privileged --rm tonistiigi/binfmt --install all

# Verify registered emulators
cat /proc/sys/fs/binfmt_misc/qemu-aarch64
```

**Native cross-compilation** (faster than QEMU — use when possible):

```dockerfile
# syntax=docker/dockerfile:1
FROM --platform=$BUILDPLATFORM golang:1.22 AS build
ARG TARGETOS TARGETARCH
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
# Cross-compile natively — no emulation needed
RUN CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH go build -o /app .

FROM gcr.io/distroless/static
COPY --from=build /app /app
ENTRYPOINT ["/app"]
```

Key variables injected by BuildKit: `BUILDPLATFORM`, `TARGETPLATFORM`, `TARGETOS`, `TARGETARCH`, `TARGETVARIANT`.

## BuildKit Mounts

### Cache Mounts

Persist package manager caches across builds — dramatic speedup for repeated builds:

```dockerfile
# Python — cache pip downloads and wheels
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

# Apt — cache both lists and archives
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt/lists,sharing=locked \
    apt-get update && apt-get install -y build-essential

# Go — cache module downloads and build cache
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    go build -o /app .

# npm — cache npm store
RUN --mount=type=cache,target=/root/.npm \
    npm ci --production

# Rust — cache cargo registry and build artifacts
RUN --mount=type=cache,target=/usr/local/cargo/registry \
    --mount=type=cache,target=/app/target \
    cargo build --release
```

> `sharing=locked` prevents concurrent access — use for apt/dpkg. Default `sharing=shared` is fine for pip/npm/go.

### Secret Mounts

Access credentials at build time without baking into layers:

```dockerfile
# Mount a secret file
RUN --mount=type=secret,id=aws,target=/root/.aws/credentials \
    aws s3 cp s3://bucket/model.bin /app/model.bin

# Use as environment variable
RUN --mount=type=secret,id=token \
    TOKEN=$(cat /run/secrets/token) && \
    curl -H "Authorization: Bearer $TOKEN" https://api.example.com/data > /app/data.json
```

```bash
# Pass secrets at build time
docker buildx build --secret id=aws,src=$HOME/.aws/credentials \
  --secret id=token,src=.env.token -t myapp .
```

### SSH Mounts

Clone private repos without copying keys into the image:

```dockerfile
RUN --mount=type=ssh git clone git@github.com:org/private-repo.git
```

```bash
docker buildx build --ssh default=$SSH_AUTH_SOCK -t myapp .
```

### Bind Mounts

Mount files from build context or other build stages without copying:

```dockerfile
# Mount requirements separately — avoids COPY cache invalidation on code changes
RUN --mount=type=bind,source=requirements.txt,target=/tmp/req.txt \
    --mount=type=cache,target=/root/.cache/pip \
    pip install -r /tmp/req.txt
```

## Cache Backends

Control where BuildKit stores and retrieves layer caches:

| Backend | Speed | Shared | CI-Friendly | Best For |
|---------|-------|--------|-------------|----------|
| `local` | ⚡ Fast | ❌ | ❌ | Local dev |
| `inline` | ⚡ Fast | ✅ | ✅ | Simple registry cache (limited to pushed layers) |
| `registry` | Medium | ✅ | ✅ | Team sharing, multi-stage caching |
| `gha` | Medium | ✅ | ✅ | GitHub Actions (uses GHA cache API) |
| `s3` | Slower | ✅ | ✅ | Cross-CI, persistent, large caches |

```bash
# Registry cache — full build cache, separate from image
docker buildx build \
  --cache-from type=registry,ref=registry.example.com/app:buildcache \
  --cache-to type=registry,ref=registry.example.com/app:buildcache,mode=max \
  -t registry.example.com/app:latest --push .

# Inline cache — embedded in image metadata (simpler, but only caches pushed layers)
docker buildx build \
  --cache-from type=registry,ref=registry.example.com/app:latest \
  --cache-to type=inline \
  -t registry.example.com/app:latest --push .

# GitHub Actions cache
docker buildx build \
  --cache-from type=gha --cache-to type=gha,mode=max \
  -t myapp .

# S3 cache (good for cross-CI or self-hosted runners)
docker buildx build \
  --cache-from type=s3,region=us-east-1,bucket=buildcache,name=myapp \
  --cache-to type=s3,region=us-east-1,bucket=buildcache,name=myapp,mode=max \
  -t myapp .

# Local directory cache
docker buildx build \
  --cache-from type=local,src=/tmp/buildcache \
  --cache-to type=local,dest=/tmp/buildcache,mode=max \
  -t myapp .
```

> `mode=max` caches all layers (including intermediate stages). Default `mode=min` only caches layers in the final image.

## Bake

Declarative build definitions — like `docker-compose.yml` but for builds. Define targets, variables, and matrices in HCL, JSON, or Compose files.

### HCL Example (`docker-bake.hcl`)

```hcl
variable "REGISTRY" {
  default = "ghcr.io/myorg"
}
variable "TAG" {
  default = "latest"
}

group "default" {
  targets = ["app", "worker"]
}

target "app" {
  dockerfile = "Dockerfile"
  context    = "."
  target     = "production"
  tags       = ["${REGISTRY}/app:${TAG}"]
  platforms  = ["linux/amd64", "linux/arm64"]
  cache-from = ["type=registry,ref=${REGISTRY}/app:buildcache"]
  cache-to   = ["type=registry,ref=${REGISTRY}/app:buildcache,mode=max"]
}

target "worker" {
  inherits   = ["app"]
  dockerfile = "Dockerfile.worker"
  tags       = ["${REGISTRY}/worker:${TAG}"]
}
```

### Matrix Builds

```hcl
variable "PYTHON_VERSIONS" {
  default = ["3.10", "3.11", "3.12"]
}

target "python-matrix" {
  name       = "python-${replace(ver, ".", "-")}"
  matrix     = { ver = PYTHON_VERSIONS }
  dockerfile = "Dockerfile"
  args       = { PYTHON_VERSION = ver }
  tags       = ["registry.example.com/app:py${ver}"]
}
```

### Using Bake

```bash
# Build all targets in default group
docker buildx bake

# Build specific target
docker buildx bake app

# Override variables
docker buildx bake --set "*.platform=linux/amd64" --set TAG=v1.2.3

# Print resolved build plan (dry run)
docker buildx bake --print

# Use specific file
docker buildx bake -f docker-bake.hcl -f docker-bake.override.hcl
```

## Multi-Stage Optimization

### Layer Ordering (Most → Least Stable)

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.12-slim AS base

# 1. System deps (rarely change)
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt/lists,sharing=locked \
    apt-get update && apt-get install -y --no-install-recommends libpq-dev

# 2. Python deps (change occasionally)
COPY requirements.txt .
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --no-compile -r requirements.txt

# 3. Application code (changes frequently)
COPY . .

FROM base AS test
RUN pytest

FROM base AS production
USER nonroot
ENTRYPOINT ["python", "-m", "app"]
```

### Wheel Pre-Building (Separate Build + Runtime)

```dockerfile
FROM python:3.12 AS wheels
RUN --mount=type=cache,target=/root/.cache/pip \
    --mount=type=bind,source=requirements.txt,target=/tmp/req.txt \
    pip wheel -r /tmp/req.txt -w /wheels

FROM python:3.12-slim AS runtime
COPY --from=wheels /wheels /wheels
RUN pip install --no-index --find-links=/wheels /wheels/*.whl && rm -rf /wheels
COPY . /app
```

## Build Args, Targets, and Build Contexts

```bash
# Build a specific stage
docker buildx build --target test -t myapp:test .

# Pass build args
docker buildx build --build-arg PYTHON_VERSION=3.12 -t myapp .

# Additional build contexts — reference other Dockerfiles or directories
docker buildx build \
  --build-context shared=../shared-lib \
  --build-context configs=docker-image://alpine:latest \
  -t myapp .
```

```dockerfile
# Reference additional build context
COPY --from=shared /utils.py /app/utils.py
COPY --from=configs /etc/alpine-release /app/
```

## CI Integration

### GitHub Actions

```yaml
# .github/workflows/build.yml
name: Build
on: push

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v6
        with:
          push: true
          platforms: linux/amd64,linux/arm64
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### Bake in CI

```yaml
      - uses: docker/bake-action@v5
        with:
          files: docker-bake.hcl
          targets: default
          push: true
          set: |
            *.cache-from=type=gha
            *.cache-to=type=gha,mode=max
```

## .dockerignore

Reduces build context size and prevents cache busting from irrelevant file changes:

```gitignore
# VCS
.git
**/.git

# Dependencies (rebuilt in container)
node_modules
__pycache__
*.pyc
.venv
vendor

# Build artifacts
dist
build
*.egg-info

# IDE / OS
.vscode
.idea
*.swp
.DS_Store

# Docker
Dockerfile*
docker-compose*
docker-bake*
.dockerignore

# CI/CD
.github
.gitlab-ci.yml

# Docs / Tests (unless needed in image)
docs
tests
*.md
LICENSE
```

> Large `.git` directories are the #1 cause of slow context transfers. Always exclude `.git`.

## Debugging Builds

```bash
# Verbose output — see every RUN command's output
docker buildx build --progress=plain -t myapp .

# Force rebuild — ignore all caches
docker buildx build --no-cache -t myapp .

# Inspect a specific stage interactively
docker buildx build --target build -t myapp:debug .
docker run -it --rm myapp:debug /bin/sh

# Export build to local directory (inspect filesystem without running)
docker buildx build --output type=local,dest=./build-output .

# Export as tarball
docker buildx build --output type=tar,dest=./image.tar .

# Build with metadata output
docker buildx build --metadata-file build-metadata.json -t myapp .
cat build-metadata.json | jq .
```

## Cross-References

- [github-actions](../github-actions/) — CI/CD pipelines using docker/build-push-action
- [harbor](../harbor/) — Push multi-arch images to Harbor registry
- [helm](../helm/) — Build container images for Helm-deployed applications
- [gpu-operator](../gpu-operator/) — Build CUDA-based training/inference images requiring GPU base layers
- [skills/uv](../skills/uv/) — Use uv for fast Python dependency installation in Dockerfiles

## References

- [Docker Buildx docs](https://docs.docker.com/build/buildx/)
- [BuildKit Dockerfile reference](https://docs.docker.com/reference/dockerfile/) — mount syntax, multi-stage
- [Bake reference](https://docs.docker.com/build/bake/reference/)
- [Cache backends](https://docs.docker.com/build/cache/backends/)
- [Multi-platform builds](https://docs.docker.com/build/building/multi-platform/)
- [GitHub Actions docker/build-push-action](https://github.com/docker/build-push-action)
- [BuildKit source](https://github.com/moby/buildkit)
- [`troubleshooting.md`](references/troubleshooting.md) — Builder instance issues, multi-platform failures, cache problems, and registry errors
