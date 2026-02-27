---
name: container-registry
description: "Container registry management patterns covering tagging strategy, immutability, retention policies, vulnerability scanning, multi-arch builds, and CI/CD integration. Internal reference for agents managing infrastructure."
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Knowledge Pack: Container Registry Management

## Purpose

Provide production-grade container registry management patterns for tagging, security scanning, retention, and CI/CD integration across major registry providers.

---

## 1. Tagging Strategy

### Primary Pattern

```
{registry}/{namespace}/{service}:{semver}-{git-sha-short}
```

### Tag Types

| Tag Type | Pattern | Example | Use Case |
|---|---|---|---|
| **Release** | `{semver}-{sha}` | `1.2.3-abc1234` | Production deployments |
| **Branch** | `{branch}-{sha}` | `main-abc1234` | Pre-release / staging |
| **PR** | `pr-{number}-{sha}` | `pr-42-abc1234` | Pull request previews |
| **Latest** | `latest` | `latest` | Dev only (never in prod) |

### Tagging Rules

- Every image MUST include the short Git SHA for traceability.
- Semantic version tags MUST be immutable (never re-push the same tag).
- The `latest` tag is mutable and MUST NOT be used in production manifests.
- Branch tags are mutable within a branch lifecycle and suitable for staging.
- Use OCI annotations for metadata rather than encoding all info in the tag.

### Example CI Tag Generation

```bash
#!/usr/bin/env bash
set -euo pipefail

REGISTRY="registry.example.com"
SERVICE="my-service"
SHA_SHORT="$(git rev-parse --short=7 HEAD)"

# Release build (on tag push)
if [[ "${GITHUB_REF}" =~ ^refs/tags/v ]]; then
  VERSION="${GITHUB_REF#refs/tags/v}"
  IMAGE="${REGISTRY}/${SERVICE}:${VERSION}-${SHA_SHORT}"
  docker tag "${SERVICE}:build" "${IMAGE}"
  docker push "${IMAGE}"
fi

# Branch build (on push to main)
if [[ "${GITHUB_REF}" == "refs/heads/main" ]]; then
  IMAGE="${REGISTRY}/${SERVICE}:main-${SHA_SHORT}"
  docker tag "${SERVICE}:build" "${IMAGE}"
  docker push "${IMAGE}"
fi

# PR build
if [[ "${GITHUB_REF}" =~ ^refs/pull/ ]]; then
  PR_NUMBER="${GITHUB_REF#refs/pull/}"
  PR_NUMBER="${PR_NUMBER%/merge}"
  IMAGE="${REGISTRY}/${SERVICE}:pr-${PR_NUMBER}-${SHA_SHORT}"
  docker tag "${SERVICE}:build" "${IMAGE}"
  docker push "${IMAGE}"
fi
```

---

## 2. Immutability

### Configuration per Registry

#### Amazon ECR

```bash
# Enable tag immutability on repository
aws ecr put-image-tag-mutability \
  --repository-name my-service \
  --image-tag-mutability IMMUTABLE

# Or via Terraform
resource "aws_ecr_repository" "my_service" {
  name                 = "my-service"
  image_tag_mutability = "IMMUTABLE"

  image_scanning_configuration {
    scan_on_push = true
  }

  encryption_configuration {
    encryption_type = "AES256"
  }
}
```

#### Azure Container Registry (ACR)

```bash
# Enable tag locking on specific tag
az acr repository update \
  --name myregistry \
  --image my-service:1.0.0-abc1234 \
  --write-enabled false
```

```hcl
# Terraform
resource "azurerm_container_registry" "main" {
  name                = "myregistry"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  sku                 = "Premium"
  admin_enabled       = false

  retention_policy {
    days    = 30
    enabled = true
  }
}
```

#### Google Artifact Registry (GAR)

```bash
# Artifact Registry does not natively support tag immutability.
# Use Binary Authorization to enforce image policies.
gcloud artifacts repositories create my-repo \
  --repository-format=docker \
  --location=us-central1 \
  --description="Production images"
```

#### Harbor

```bash
# Enable tag immutability via Harbor API or UI
# Project > Configuration > Tag Immutability > Add Rule
# Scope: repositories matching "**"
# Tag: matching "v*" (semver tags)
```

---

## 3. Retention Policies

### Standard Policy

| Rule | Value | Rationale |
|---|---|---|
| Keep last N tagged images | 10 | Rollback window for production |
| Auto-delete untagged manifests | After 7 days | Clean up intermediate builds |
| Keep release tags | Indefinitely | Audit trail for versioned releases |
| Delete PR tags | After 14 days | Short-lived preview environments |

#### Amazon ECR Lifecycle Policy

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Remove untagged images after 7 days",
      "selection": {
        "tagStatus": "untagged",
        "countType": "sinceImagePushed",
        "countUnit": "days",
        "countNumber": 7
      },
      "action": {
        "type": "expire"
      }
    },
    {
      "rulePriority": 2,
      "description": "Remove PR images after 14 days",
      "selection": {
        "tagStatus": "tagged",
        "tagPatternList": ["pr-*"],
        "countType": "sinceImagePushed",
        "countUnit": "days",
        "countNumber": 14
      },
      "action": {
        "type": "expire"
      }
    },
    {
      "rulePriority": 3,
      "description": "Keep only last 10 branch images",
      "selection": {
        "tagStatus": "tagged",
        "tagPatternList": ["main-*"],
        "countType": "imageCountMoreThan",
        "countNumber": 10
      },
      "action": {
        "type": "expire"
      }
    }
  ]
}
```

```hcl
# Terraform
resource "aws_ecr_lifecycle_policy" "my_service" {
  repository = aws_ecr_repository.my_service.name
  policy     = file("${path.module}/ecr-lifecycle-policy.json")
}
```

#### Azure ACR Retention

```bash
# Purge untagged manifests older than 7 days
az acr run --cmd "acr purge \
  --filter 'my-service:.*' \
  --untagged \
  --ago 7d" \
  --registry myregistry /dev/null

# Schedule purge as ACR Task (runs daily)
az acr task create \
  --name purge-untagged \
  --registry myregistry \
  --cmd "acr purge --filter '.*:.*' --untagged --ago 7d" \
  --schedule "0 2 * * *" \
  --context /dev/null
```

#### Google Artifact Registry Cleanup

```bash
# Use gcr-cleaner or Artifact Registry cleanup policies
gcloud artifacts repositories set-cleanup-policies my-repo \
  --location=us-central1 \
  --policy=policy.json
```

```json
{
  "cleanupPolicies": [
    {
      "id": "delete-untagged",
      "action": { "type": "Delete" },
      "condition": {
        "tagState": "UNTAGGED",
        "olderThan": "604800s"
      }
    },
    {
      "id": "keep-minimum-versions",
      "action": { "type": "Keep" },
      "mostRecentVersions": {
        "keepCount": 10
      }
    }
  ]
}
```

---

## 4. Vulnerability Scanning

### Trivy Integration

```yaml
# .github/workflows/scan.yaml
name: Container Scan
on:
  push:
    branches: [main]
  pull_request:

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t my-service:scan .

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: my-service:scan
          format: table
          exit-code: 1              # Fail the build on findings
          severity: CRITICAL,HIGH   # Block on critical and high
          ignore-unfixed: true      # Skip vulnerabilities without fixes
          vuln-type: os,library

      - name: Run Trivy SBOM generation
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: my-service:scan
          format: cyclonedx
          output: sbom.json
          vuln-type: os,library

      - name: Upload SBOM
        uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: sbom.json
```

### Blocking Policy for Critical CVEs

```yaml
# .trivyignore — explicitly acknowledge accepted risks
# Each entry must have a comment with justification and expiry date

# CVE-2024-XXXXX: No fix available; mitigated by network policy. Expires 2025-06-01.
CVE-2024-XXXXX
```

### CI Gate Decision Table

| Severity | PR Build | Main Branch | Release Tag |
|---|---|---|---|
| Critical | Block | Block | Block |
| High | Warn | Block | Block |
| Medium | Info | Warn | Warn |
| Low | Ignore | Info | Info |

---

## 5. Multi-Arch Builds

### Docker Buildx Setup

```bash
# Create and bootstrap a multi-platform builder
docker buildx create --name multiarch --driver docker-container --use
docker buildx inspect --bootstrap
```

### Build and Push Multi-Arch Image

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --tag registry.example.com/my-service:1.0.0-abc1234 \
  --push \
  --cache-from type=registry,ref=registry.example.com/my-service:cache \
  --cache-to type=registry,ref=registry.example.com/my-service:cache,mode=max \
  .
```

### GitHub Actions Multi-Arch Build

```yaml
# .github/workflows/build.yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: docker/setup-qemu-action@v3

      - uses: docker/setup-buildx-action@v3

      - uses: docker/login-action@v3
        with:
          registry: registry.example.com
          username: ${{ secrets.REGISTRY_USERNAME }}
          password: ${{ secrets.REGISTRY_PASSWORD }}

      - uses: docker/build-push-action@v6
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: true
          tags: |
            registry.example.com/my-service:${{ github.sha }}
            registry.example.com/my-service:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

## 6. CI/CD Integration

### Complete Pipeline: Build, Scan, Push, Deploy

```yaml
# .github/workflows/deploy.yaml
name: Build and Deploy
on:
  push:
    branches: [main]
    tags: ["v*"]

env:
  REGISTRY: registry.example.com
  SERVICE: my-service

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      digest: ${{ steps.build.outputs.digest }}
    steps:
      - uses: actions/checkout@v4

      - uses: docker/setup-buildx-action@v3

      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ secrets.REGISTRY_USERNAME }}
          password: ${{ secrets.REGISTRY_PASSWORD }}

      # Generate tags based on event type
      - name: Generate image metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.SERVICE }}
          tags: |
            type=semver,pattern={{version}}-{{sha}}
            type=ref,event=branch,suffix=-{{sha}}
            type=ref,event=pr,prefix=pr-,suffix=-{{sha}}

      # Build and push
      - name: Build and push
        id: build
        uses: docker/build-push-action@v6
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  scan:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ needs.build.outputs.image-tag }}
          format: table
          exit-code: 1
          severity: CRITICAL,HIGH
          ignore-unfixed: true

  deploy:
    needs: [build, scan]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' || startsWith(github.ref, 'refs/tags/v')
    environment:
      name: ${{ startsWith(github.ref, 'refs/tags/v') && 'production' || 'staging' }}
    steps:
      - uses: actions/checkout@v4

      - name: Update image tag in manifests
        run: |
          cd k8s/overlays/${{ startsWith(github.ref, 'refs/tags/v') && 'prod' || 'staging' }}
          kustomize edit set image \
            ${{ env.REGISTRY }}/${{ env.SERVICE }}=${{ needs.build.outputs.image-tag }}

      - name: Commit and push manifest update
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add .
          git commit -m "deploy: update ${{ env.SERVICE }} to ${{ needs.build.outputs.image-tag }}"
          git push
```

### Pipeline Flow Diagram

```
[Push / Tag] --> [Build Image] --> [Scan with Trivy] --> [Push to Registry]
                                        |
                                   Block if CRITICAL
                                        |
                                  [Update Manifests] --> [GitOps Sync]
                                                              |
                                                    ArgoCD / Flux deploys
```
