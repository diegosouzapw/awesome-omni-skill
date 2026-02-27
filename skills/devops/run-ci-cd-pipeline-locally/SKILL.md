---
name: Run CI/CD Pipeline Locally
description: Run the Wavecraft CI checks locally. Prefer the native `cargo xtask` commands for speed; use Docker + `act` only when validating GitHub Actions workflows or Linux-specific behavior.
---

# Skill: Run GitHub Actions CI/CD pipeline Locally

Use this skill to test CI workflows locally before pushing to GitHub. **Preferred path:** run the native `cargo xtask` commands (fast, no Docker). Use Docker + `act` only when you need to validate workflow YAML or Linux-specific behavior.

## Workflow parity requirement ✅

**Keep an xtask mirror for every workflow file** under `.github/workflows/`. If a workflow changes (or a new one is added), update or add an xtask so local runs stay fast and consistent. Use Docker + `act` only for workflow/YAML-specific checks or Linux-only behavior.

| Workflow file             | Preferred local mirror          | Notes                                                                             |
| ------------------------- | ------------------------------- | --------------------------------------------------------------------------------- |
| `ci.yml`                  | `cargo xtask ci-check`          | Full lint + test suite                                                            |
| `template-validation.yml` | `cargo xtask validate-template` | CLI template generation validation                                                |
| `release.yml`             | `cargo xtask release`           | Build/sign/notarize pipeline                                                      |
| `cli-release.yml`         | `cargo xtask release`           | Mirror release steps where applicable; add a dedicated xtask if workflow diverges |
| `continuous-deploy.yml`   | `cargo xtask release`           | Use release pipeline for parity; add a publish-focused xtask if needed            |

## Preferred: Native xtask (fast, no Docker)

These commands mirror CI checks without container overhead and are the recommended way to validate locally.

```bash
# Pre-push validation (lint + tests)
cargo xtask ci-check

# Template validation (mirrors template-validation.yml)
cargo xtask validate-template

# Optional targeted runs
cargo xtask lint
cargo xtask test
```

## Prerequisites

- **Rust toolchain** (stable, with clippy + rustfmt)
- **Node.js** + **npm** (for UI checks)

## Docker-based (only when needed)

Use Docker + `act` when validating GitHub Actions workflow changes, Linux-specific behavior, or job orchestration.

**⚠️ Agent Handoff:** If you don't have terminal execution tools available, delegate to an agent who does (Coder or Tester) to run the Docker/act commands below.

### Prerequisites (Docker path)

- **Docker Desktop** must be installed and running
- **act** must be installed: `brew install act`
- **Wavecraft custom CI image** (recommended): See the “Building the Custom Image” section below

### Quick Start for Wavecraft (Docker)

Wavecraft provides a custom Docker image with all dependencies pre-installed:

```bash
# Build the custom image (one-time setup)
docker build --platform linux/amd64 -t wavecraft-ci:latest .github/skills/run-ci-pipeline-locally/

# Run a specific job
act -j check-engine -W .github/workflows/ci.yml \
    --container-architecture linux/amd64 \
    -P ubuntu-latest=wavecraft-ci:latest \
    --pull=false \
    --artifact-server-path ./tmp/act-artifacts
```

### Available CI Jobs

| Job              | Description                  | Local Testing |
| ---------------- | ---------------------------- | ------------- |
| `check-ui`       | Prettier, ESLint, TypeScript | ✅ Works      |
| `test-ui`        | Vitest unit tests            | ✅ Works      |
| `prepare-engine` | Build UI + compile Rust      | ✅ Works      |
| `check-engine`   | cargo fmt + clippy           | ✅ Works      |
| `test-engine`    | cargo test                   | ✅ Works      |

### Run Full Pipeline Locally (Docker)

```bash
# Run entire CI workflow
act -W .github/workflows/ci.yml \
    --container-architecture linux/amd64 \
    -P ubuntu-latest=wavecraft-ci:latest \
    --pull=false \
    --artifact-server-path ./tmp/act-artifacts
```

## Building the Custom Image

The custom image includes:

- Ubuntu 22.04
- Node.js 20.x
- Rust stable with rustfmt + clippy
- GTK, WebKit, X11 development libraries

```bash
# Build for linux/amd64 (required for Apple Silicon Macs)
docker build --platform linux/amd64 -t wavecraft-ci:latest \
    .github/skills/run-ci-pipeline-locally/

# Verify the image
docker images | grep wavecraft-ci
```

**Note:** Build takes ~5-10 minutes on first run.

## Commands Reference (Docker)

**Important:** Do not combine `act` commands with `head` or `tail` — follow the logs directly.

### List Available Jobs

```bash
act -l
```

### Run Specific Workflow

```bash
act -W .github/workflows/ci.yml
```

### Run Specific Job

```bash
act -j <job-name>
```

### Run on Specific Event

```bash
act push
act pull_request
```

### Dry Run (Preview Without Executing)

```bash
act -n
```

### Pass Secrets

```bash
# Use GitHub CLI token
act -s GITHUB_TOKEN="$(gh auth token)"

# Use secrets file (.secrets with KEY=value format)
act --secret-file .secrets
```

## Key Flags for Wavecraft

| Flag                                   | Purpose                                     |
| -------------------------------------- | ------------------------------------------- |
| `--container-architecture linux/amd64` | Required on Apple Silicon Macs              |
| `-P ubuntu-latest=wavecraft-ci:latest` | Use custom image with dependencies          |
| `--pull=false`                         | Use local image, don't pull from Docker Hub |
| `-W .github/workflows/ci.yml`          | Specify workflow file                       |
| `-j <job-name>`                        | Run specific job                            |
| `--artifact-server-path <dir>`         | Enable local artifact upload/download       |

## Limitations (Docker)

### Artifact Upload/Download

By default, `actions/upload-artifact` and `actions/download-artifact` fail locally because they require GitHub's artifact server. Use the `--artifact-server-path` flag to enable a local artifact server:

```bash
act ... --artifact-server-path /tmp/act-artifacts
```

Artifacts will be stored in the specified directory and can be downloaded by subsequent jobs.

### Caching Differences

- `act` doesn't share cache with GitHub Actions
- First local run compiles everything from scratch
- Subsequent runs with `--reuse` can be faster

### Network Dependencies

Some steps require network access:

- `cargo` downloads crates
- `npm` downloads packages
- Git repos are cloned

## Debugging Failed Runs

```bash
# Verbose output
act -v

# Very verbose output
act -vv

# Keep containers running after failure for inspection
act --reuse

# Interactive shell in the container
docker exec -it <container-id> bash
```

## Common Issues

| Issue                             | Solution                                            |
| --------------------------------- | --------------------------------------------------- |
| Docker not running                | Start Docker Desktop                                |
| `--container-architecture` errors | Ensure Docker supports linux/amd64 emulation        |
| Image pull timeout                | Use `--pull=false` with local image                 |
| Permission denied on files        | Check Docker volume mounts                          |
| Job uses `macos-latest`           | Skip job or test manually                           |
| Missing Linux packages            | Rebuild custom image                                |
| Rust compilation slow             | Expected on first run; use `--reuse` for subsequent |
| `ACTIONS_RUNTIME_TOKEN` error     | Add `--artifact-server-path /tmp/act-artifacts`     |

## Updating the Custom Image

If CI dependencies change (new apt packages, Rust version, etc.):

1. Edit the Dockerfile at `.github/skills/run-ci-pipeline-locally/Dockerfile`
2. Rebuild: `docker build --platform linux/amd64 -t wavecraft-ci:latest .github/skills/run-ci-pipeline-locally/`
3. Test: `act -j check-engine ... --pull=false`
