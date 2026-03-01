---
name: ci-cd-workflows
description: Guide for GitHub Actions workflows, test orchestration, parallel testing,
  adapter builds, releases, and CI/CD configuration. Use when working with .github/workflows/,
  versions.json, or troubleshooting CI issues.
version: 1.0.0
tags:
  - ci-cd
  - github-actions
  - workflows
  - testing
  - deployment
  - release
  - orchestration
---

# CI/CD Workflows

**Purpose:** Guide development, modification, and troubleshooting of AIDB's GitHub Actions CI/CD infrastructure

This skill provides practical guidance for working with workflows and configuration. Complete documentation lives in `docs/developer-guide/ci-cd.md`.

## Related Skills

- **testing-strategy** - Test architecture and patterns
- **dev-cli-development** - Local testing with dev-cli
- **adapter-development** - Adapter build processes

## System Overview

**For complete architecture**, see `docs/developer-guide/ci-cd.md` and [architecture.md](resources/architecture.md).

### Workflow Organization

**Note:** All workflow files are now in `.github/workflows/` root directory (GitHub Actions requirement). Workflows use category prefixes for organization:

**Release Workflows** (`release-*.yaml`)

- `release-pr.yaml` - PR-based release orchestration (validate, test, build, matrix, adapters, upload)
- `release-publish.yaml` - Publish draft release on PR merge

**Testing Workflows** (`test-*.yaml`)

- `test-parallel.yaml` - Main test orchestrator (10-15 min, runs everything including frameworks). Called by `release-pr.yaml` or via manual dispatch. Builds adapters and Docker images as part of its flow.

**Adapter Workflows** (`adapter-*.yaml`)

- `adapter-build.yaml` - Build debug adapters for multiple platforms
- `adapter-build-act.yaml` - ACT local testing workflow (for local development with act)

**Maintenance Workflows** (`maintenance-*.yaml`)

- *(Currently none - Dependabot PRs are merged manually)*

**Reusable Workflows** (`*.yaml`)

- `load-versions.yaml` - Dynamic version loading
- `test-suite.yaml` - Generic test runner
- `test-frameworks.yaml` - Framework testing
- `build-adapters.yaml` - Adapter build orchestration
- `build-docker.yaml` - Docker image builds

### Single Source of Truth

**versions.json** (repo root):

- Infrastructure versions (Python, Node, Java) with docker_tag fields
- Adapter versions and repositories (debugpy, vscode-js-debug, java-debug)
- Global packages (build tools: pip, setuptools, wheel, typescript, ts-node)
- Platform/architecture matrix
- GitHub Actions configuration

All workflows dynamically load versions via `load-versions.yaml` - zero hardcoded versions.

See `docs/developer-guide/ci-cd.md` for configuration overview.

### Version Management Strategy

- **versions.json** - Infrastructure versions (Python, Node, Java) and adapter versions updated manually
- **pyproject.toml** - Application dependencies managed by Dependabot (creates PRs automatically)

### Dependabot Integration

Dependabot creates PRs directly against `main`, but **no workflows run** on these PRs:

1. **Target Branch**: All Dependabot PRs target `main`
1. **No CI Triggers**: `release-pr.yaml` skips Dependabot PRs (`github.actor != 'dependabot[bot]'`)
1. **Manual Management**: PRs are reviewed, typically retargeted to release branches, and merged manually
1. **Ecosystems**: pip (daily), github-actions (weekly), npm (weekly)

**Configuration**: `.github/dependabot.yaml`

## Common Tasks

### Running Tests

**On GitHub:**

- Release PRs to `main` → triggers `release-pr.yaml` which calls `test-parallel.yaml`
- Manual dispatch via Actions UI (workflow_dispatch) with options:
  - suite: all, cli, shared, common, logging, mcp, core, frameworks, launch, ci_cd
  - `skip_coverage`: Skip coverage reporting for faster execution
  - `debug_logging`: Enable TRACE-level logging (equivalent to `dev-cli -vvv`)

**Debug logging:** Enable trace-level debugging via `debug_logging` input (mirrors `dev-cli -vvv`):

- `gh workflow run test-parallel.yaml -f debug_logging=true`
- Sets: `AIDB_LOG_LEVEL=TRACE`, `AIDB_ADAPTER_TRACE=1`, `AIDB_CONSOLE_LOGGING=1`

**Test artifacts:** All jobs upload logs to `test-logs-{suite}` artifacts (7-day retention):

- Includes: pytest logs, container logs, CLI logs, full test output
- Retrieval: `gh run download {run-id} -n test-logs-{suite}`
- Job summaries show pytest summary (capped at 100 lines) with link to full artifact

**Locally:**

```bash
./dev-cli test run --suite shared
./dev-cli test run -t "path/to/test.py"
./dev-cli test run --suite shared --language python

# CI/CD workflow tests (includes act-based validation)
./dev-cli test run -s ci_cd              # All tests (unit + integration)
./dev-cli test run -s ci_cd -m "not slow"  # Skip slow act tests
./dev-cli test run -s ci_cd -m integration  # Only integration tests
```

**With act (local CI):**
See [quick-reference.md](resources/quick-reference.md) and [CI/CD Integration Tests](#cicd-integration-tests) below.

### Troubleshooting CI Failures with GitHub CLI

**GitHub CLI is the primary tool for CI observability and debugging.** When tests fail in CI, use GH CLI to inspect jobs, download artifacts, and analyze logs locally.

**Prerequisites:**

```bash
# Verify GH CLI is installed and authenticated
gh auth status
```

**Common workflows:**

```bash
# List recent workflow runs
gh run list --status=failure --limit 10

# View run details and job summary
gh run view <run-id>

# Watch running job in real-time
gh run watch <run-id>

# Download test artifacts (from job summary)
gh run download <run-id> -n test-logs-{suite}
```

**Job summaries include:**

- Status (pass/fail)
- Artifact download command (copy-paste ready)
- Pytest summary (capped at 100 lines)
- FAILED/SKIPPED/ERROR test listings with file paths

**Artifact contents:**

- `pytest-logs/` - Full pytest output, stack traces, captured stdout/stderr
- `.cache/container-data/` - Docker/adapter logs, DAP traces
- `aidb-logs/` - AIDB core library logs (from ~/.aidb/log/)

**For complete guide** including artifact structure, log locations, and troubleshooting workflows, see [troubleshooting.md](resources/troubleshooting.md).

### Building Adapters

**PR-based workflow:**

```bash
# Create release branch
git checkout -b release/0.1.0
# Add release notes
# Push and open PR to main
git push origin release/0.1.0
```

**Manual dispatch:** Actions UI → Build Debug Adapters

See [Adapter Builds](resources/adapter-builds.md) for architecture.

### Cutting Releases

**PR-based release workflow:**

1. Create release branch: `release/X.Y.Z`
1. Create release notes: `docs/release-notes/X.Y.Z.md`
1. Open PR to `main` → triggers comprehensive release pipeline:
   - Validation, tests, builds (VSIX, wheel)
   - TestPyPI/ProdPyPI uploads
   - Adapter builds (Python/JS/Java × platforms)
   - Draft release creation
1. Merge PR → publishes release

**Testing mode:** Set `CD_SKIP_PYPI=true` to run full workflow without PyPI uploads.

For complete pipeline breakdown, CD_SKIP_PYPI details, and examples, see [Release Workflows](resources/release-workflows.md) or `docs/developer-guide/ci-cd.md` for detailed job list.

### Adding Frameworks

1. Create app in `src/tests/_assets/framework_apps/{lang}/{framework}/`
1. Add deps to `src/tests/_docker/scripts/install-framework-deps.sh`
1. Create test file in `src/tests/frameworks/{lang}/{framework}/e2e/`

Framework dependencies auto-managed via checksum services.

### Modifying Configuration

**Update infrastructure version:**

```json
// versions.json
{
  "infrastructure": {
    "python": {
      "version": "3.12"  // All workflows pick up automatically
    }
  }
}
```

**Add platform:**

```json
{
  "platforms": [
    {
      "os": "linux",
      "arch": "arm64"
    }
  ]
}
```

## Test Orchestration

**For detailed architecture**, see [Test Orchestration](resources/test-orchestration.md) and `docs/developer-guide/ci-cd.md`.

### Parallel Execution

**Main orchestrator:** `test-parallel.yaml`

**Flow:**

1. Load versions from `versions.json`
1. Build adapters and Docker in parallel
1. Run test suites in parallel: cli, shared, mcp, core, frameworks, launch, common/logging, ci_cd

**Performance:**

- 10-15 min wall-clock
- 50% faster than sequential
- Up to 10 parallel runners

### Dynamic Matrix

Frameworks use dynamic matrix from `versions.json`:

- Single source of truth
- Easy enable/disable
- Auto version expansion
- No workflow changes needed

## Troubleshooting

**For comprehensive guide**, see [Troubleshooting](resources/troubleshooting.md).

**For CI test failures and artifact analysis**, see [troubleshooting.md](resources/troubleshooting.md).

### Quick Diagnostics

**Workflow not triggering:**

- Check trigger conditions and branch names
- Review path filters
- Verify workflow enabled

**Test failures:**

- Download artifacts: `gh run download <run-id> -n test-logs-{suite}`
- Check job logs in Actions UI
- Look for adapter build failures
- Review Docker build logs
- Check version conflicts

**Adapter builds:**

- Verify platform compatibility
- Check upstream availability
- Review versions.json adapter configuration

**Version loading:**

- Validate versions.json syntax
- Check Python version consistency

### Investigation Workflow

1. Identify failing job → Download artifacts (`gh run download`)
1. Check logs for errors
1. Review recent changes
1. Reproduce locally with dev-cli or act

**Note:** `actionlint` runs automatically via pre-commit hooks when committing workflow changes.

## Workflow Patterns

See [architecture.md](resources/architecture.md) for job dependencies, conditionals, matrices, and reusable workflows.

## Configuration Files

- **versions.json** - Infrastructure, adapters, platforms (see `docs/developer-guide/ci-cd.md`)

**Secrets:** See `docs/developer-guide/ci-cd.md` for complete list.

## Local CI Testing

Use `act` to run workflows locally. See [quick-reference.md](resources/quick-reference.md) for commands.

## CI/CD Integration Tests

Act-based tests in `src/tests/ci_cd/integration/` validate workflow syntax and matrices.

**Quick commands:**

```bash
./dev-cli test run -s ci_cd              # All tests (unit + integration)
./dev-cli test run -s ci_cd -m "not slow"  # Skip slow tests
```

## Composite Actions

Available in `.github/actions/`: `setup-aidb-env`, `setup-multi-lang`, `download-test-artifacts`, `run-aidb-tests`. See [architecture.md](resources/architecture.md).

## Best Practices

For comprehensive guidelines on workflow development, configuration management, testing strategy, and validation checklists, see [architecture.md](resources/architecture.md).

## Cross-Workflow Dependencies

Workflows run independently by default. See [architecture.md](resources/architecture.md) for cross-workflow coordination patterns.

## Resources

| Resource                                                 | Content                                                                       |
| -------------------------------------------------------- | ----------------------------------------------------------------------------- |
| [architecture.md](resources/architecture.md)             | Workflow patterns, reusable workflows, composite actions, cross-workflow deps |
| [test-orchestration.md](resources/test-orchestration.md) | Parallel execution, test suites, matrix strategy                              |
| [adapter-builds.md](resources/adapter-builds.md)         | Build process, artifacts, platforms, local builds                             |
| [release-workflows.md](resources/release-workflows.md)   | Release process, versioning, CD_SKIP_PYPI                                     |
| [troubleshooting.md](resources/troubleshooting.md)       | Investigation workflow, common issues, debugging                              |
| [quick-reference.md](resources/quick-reference.md)       | Commands, file locations, triggers                                            |

### Internal Documentation

Complete guides: `docs/developer-guide/ci-cd.md`

### Code References

- `.github/workflows/` - Workflow definitions
- `.github/actions/` - Composite actions
- `.github/scripts/` - CI/CD scripts
- `versions.json` - Infrastructure & adapter versions

## Quick Reference

### Workflow Triggers

**Run Tests:** Via `release-pr.yaml` (PR to `main`) or manual dispatch
**Release (draft):** PR to `main` from `release/**` branch
**Release (publish):** PR merge to `main` from `release/**` branch
**Build Adapters:** Part of release workflow (release-pr.yaml)

### Common Commands

```bash
# Local testing
./dev-cli test run --suite {shared|cli|mcp}
./dev-cli test run -t "path/to/test.py"

# Validation
actionlint .github/workflows/**/*.yaml

# Local CI
act -l                    # List
act -j test-shared        # Run job
act push -j build         # With event
```

### Key Files

- `versions.json` - Single source of truth
- `.github/workflows/test-parallel.yaml` - Main orchestrator
- `.github/workflows/load-versions.yaml` - Version loading
- `.actrc` - Local CI config

______________________________________________________________________

**Remember:**

- versions.json is single source of truth
- Use reusable workflows for DRY
- Test locally before pushing
- Validate configuration changes
- Monitor performance metrics
