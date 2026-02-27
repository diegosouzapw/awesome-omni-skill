---
name: gitlab-ci
description: Initialize or update GitLab CI/CD pipelines for Go projects with comprehensive testing, coverage reporting, snapshot builds, and automated releases
---

# GitLab CI/CD Setup Skill

Automate Go project CI/CD with production-ready GitLab pipelines for testing, coverage reporting, snapshot builds, and releases.

## Overview

This skill provides a complete GitLab CI/CD setup for Go projects:

- **Unit Testing**: Automated test execution on every push
- **Coverage Reporting**: Test coverage tracking with configurable thresholds
- **Snapshot Builds**: Test release builds with GoReleaser on every push
- **Release Automation**: Production releases with Docker images on tag push
- **Docker Integration**: Multi-architecture container builds with GitLab Container Registry

The complete pipeline configuration is provided in `assets/.gitlab-ci.yml` with inline documentation.

## Prerequisites

Before using this skill, ensure:

1. **GitLab repository**: Project hosted on GitLab (gitlab.com or self-hosted)
2. **Go module**: Project uses Go modules (`go.mod` present)
3. **GoReleaser config** (for releases): `.goreleaser.yml` in repository root

## Pipeline Architecture

### Stages & Jobs

| Job | Stage | Trigger | Purpose | Requirements |
|-----|-------|---------|---------|--------------|
| **unit-tests** | test | Every push | Run test suite | DIND service |
| **coverage** | test | Every push (optional) | Coverage report & badge | DIND service |
| **build-snapshot** | build | Every push | Test release process | DIND, GoReleaser |
| **build-release** | release | Tags only (`v*`) | Production release | DIND, GoReleaser, `.goreleaser.yml` |

### Pipeline Triggers

- **Every push**: `unit-tests`, `build-snapshot` (validates changes don't break releases)
- **Tags only** (`v*`): `build-release` (creates production releases)
- **Docker-in-Docker**: All jobs use `docker:dind` service for container builds

## Instructions

### Step 1: Analyze Project Structure

Verify project setup:
```bash
# Check for required files
ls go.mod                # Required: Go module
ls .goreleaser.yml      # Required for releases
ls -la .gitlab-ci.yml   # Check existing pipeline
```

### Step 2: Copy Pipeline Configuration

Copy the template from this skill's assets:

```bash
# Copy pipeline configuration (well-documented with inline comments)
cp assets/.gitlab-ci.yml .gitlab-ci.yml
```

The asset file includes comprehensive inline documentation for all customization points.

### Step 3: Configure Taskfile (Recommended)

Create or update `Taskfile.yml` with CI tasks:

```yaml
version: '3'

tasks:
  test:
    desc: Run all tests
    cmds:
      - go generate ./...
      - go test -v ./...

  coverage:
    desc: Generate coverage report
    cmds:
      - go generate ./...
      - go test -coverpkg=./... -coverprofile=profile.cov ./...
      - go tool cover -func profile.cov

  snapshot:
    desc: Create snapshot build (test release)
    cmds:
      - goreleaser --snapshot --clean

  release:
    desc: Create production release
    cmds:
      - goreleaser --clean
```

### Step 4: Customize Pipeline

Update `.gitlab-ci.yml` based on your project needs. The asset file has detailed inline comments marking all customization points with `# CUSTOMIZE:` markers.

**Key customizations** (all documented in asset file):

| Setting | Location | Common Values |
|---------|----------|---------------|
| Go version | `image.name` in all jobs | `golang:1.24`, `golang:1.25.1` |
| Runner tags | `tags:` in all jobs | `gitlab-org-docker`, `docker`, `linux`, `shared` |
| GoReleaser version | `image.name` in build jobs | `goreleaser/goreleaser:v2.12.0` |
| Tag pattern | `build-release.only` | `tags`, `/^v\d+\.\d+\.\d+$/` |

**For coverage tracking**, uncomment the coverage job in `.gitlab-ci.yml` and customize exclusions:
```yaml
coverage:
  script:
    # Exclude packages from coverage report
    - sed -i '/cmd\//d' profile.cov              # Exclude cmd packages
    - sed -i '/internal\/mocks/d' profile.cov    # Exclude mocks
```

**For external Docker registries**, add variables in Settings → CI/CD → Variables:
- `EXTERNAL_CI_REGISTRY`: Registry URL (e.g., `docker.io`)
- `EXTERNAL_CI_REGISTRY_USER`: Username
- `EXTERNAL_CI_REGISTRY_PASSWORD`: Password (mark as protected & masked)

### Step 5: Configure GoReleaser

Ensure `.goreleaser.yml` uses GitLab variables for Docker images:

```yaml
dockers:
  - image_templates:
      - "{{ .Env.CI_REGISTRY_IMAGE }}:{{ .Version }}-amd64"
      - "{{ .Env.CI_REGISTRY_IMAGE }}:{{ .Version }}-arm64"
    dockerfile: Dockerfile
    use: buildx
    build_flag_templates:
      - "--platform=linux/amd64"
      - "--label=org.opencontainers.image.created={{.Date}}"
      - "--label=org.opencontainers.image.revision={{.FullCommit}}"

docker_manifests:
  - name_template: "{{ .Env.CI_REGISTRY_IMAGE }}:{{ .Version }}"
    image_templates:
      - "{{ .Env.CI_REGISTRY_IMAGE }}:{{ .Version }}-amd64"
      - "{{ .Env.CI_REGISTRY_IMAGE }}:{{ .Version }}-arm64"
  - name_template: "{{ .Env.CI_REGISTRY_IMAGE }}:latest"
    image_templates:
      - "{{ .Env.CI_REGISTRY_IMAGE }}:{{ .Version }}-amd64"
      - "{{ .Env.CI_REGISTRY_IMAGE }}:{{ .Version }}-arm64"
```

### Step 6: Configure GitLab Settings

Enable required GitLab features:

1. **Container Registry**: Settings → General → Visibility → Container Registry: Enabled

2. **CI/CD Variables** (if using external registry):
   - Settings → CI/CD → Variables → Add Variable
   - Add `EXTERNAL_CI_REGISTRY`, `EXTERNAL_CI_REGISTRY_USER`, `EXTERNAL_CI_REGISTRY_PASSWORD`

3. **Protected Tags** (recommended):
   - Settings → Repository → Protected tags
   - Protect `v*`: Allowed to create: Maintainers

### Step 7: Add Pipeline Badges (Optional)

Add badges to README.md:

```markdown
[![Pipeline](https://gitlab.com/OWNER/REPO/badges/main/pipeline.svg)](https://gitlab.com/OWNER/REPO/-/pipelines)
[![Coverage](https://gitlab.com/OWNER/REPO/badges/main/coverage.svg)](https://gitlab.com/OWNER/REPO/-/graphs/main/charts)
```