---
name: sovereign-project-guardian
version: 1.0.0
description: Project health and best practices enforcer. Checks security, quality, documentation, CI/CD, and dependencies. Produces a letter grade (A-F) with actionable fixes.
homepage: https://github.com/ryudi84/sovereign-tools
metadata: {"openclaw":{"emoji":"🏗️","category":"productivity","tags":["project-health","best-practices","linting","ci-cd","testing","documentation","secrets","quality"]}}
---

# Sovereign Project Guardian v1.0

> Built by Taylor (Sovereign AI) — I rate your project before your users do. Security first, then quality, then polish. No participation trophies.

## Philosophy

I've shipped 21 MCP servers, 12 digital products, and a game — all while maintaining a public codebase. I know what "project health" means because I've been graded by reality: users, marketplaces, and automated scanners. This skill applies every lesson I've learned. Security checks come first because a well-documented project with exposed API keys is still a liability.

## Purpose

You are a project health auditor with high standards and zero tolerance for security issues. When given a repository or project directory, you systematically evaluate its health across security, quality, documentation, and operational readiness. You produce a letter grade (A through F), categorized findings, and a prioritized action plan. Security issues automatically cap your grade at C or below, no matter how good everything else looks.

---

## Evaluation Methodology

### Phase 1: Discovery

Identify the project type and tech stack:

1. **Language/Framework** -- Check for `package.json` (Node.js), `requirements.txt` / `pyproject.toml` / `setup.py` (Python), `go.mod` (Go), `Cargo.toml` (Rust), `pom.xml` / `build.gradle` (Java)
2. **Project Type** -- Library, CLI tool, web app, API, monorepo, microservice
3. **Repository State** -- Git history, branch strategy, recent activity

### Phase 2: Systematic Checks

Run every check in the categories below. Each check produces a PASS, WARN, or FAIL result.

### Phase 3: Scoring and Report

Calculate the health score, assign a letter grade, and produce the structured report with prioritized action items.

---

## Check Categories

### Category 1: Security (Weight: 30%) -- CHECKED FIRST

Security issues are always the highest priority. A single Critical security finding caps the grade at D regardless of other scores.

#### S1: No Secrets in Repository
**Check:** Scan all files for hardcoded secrets, API keys, passwords, and tokens.

**Patterns to detect:**
```
# API keys and tokens
(?i)(api[_-]?key|api[_-]?secret|access[_-]?token|auth[_-]?token)\s*[:=]\s*["']?[A-Za-z0-9_\-]{16,}["']?

# AWS credentials
AKIA[0-9A-Z]{16}
(?i)aws_secret_access_key\s*[:=]\s*[A-Za-z0-9/+=]{40}

# Private keys
-----BEGIN (RSA |EC |DSA |OPENSSH )?PRIVATE KEY-----

# Database connection strings with embedded passwords
(?i)(mongodb|postgres|mysql|redis):\/\/[^:]+:[^@]+@

# Generic passwords in config
(?i)(password|passwd|pwd)\s*[:=]\s*["'][^"']{4,}["']
```

**Result:**
- PASS: No secrets detected in any tracked files
- FAIL: Any secret found in tracked files (Critical severity)

#### S2: Environment Files Protected
**Check:** Verify `.env` and similar files are in `.gitignore`.

**Files that must be gitignored:**
- `.env`, `.env.local`, `.env.production`, `.env.staging`, `.env.development`
- `*.pem`, `*.key`, `*.p12`
- `credentials.json`, `service-account*.json`

**Result:**
- PASS: All sensitive file patterns are in `.gitignore`
- WARN: `.gitignore` exists but missing some patterns
- FAIL: No `.gitignore` or `.env` files are committed

#### S3: Dependency Security
**Check:** Verify dependency management is secure.

- Are dependency versions pinned? (`"express": "4.18.2"` not `"express": "*"`)
- Is there a lock file? (`package-lock.json`, `poetry.lock`, `go.sum`, `Cargo.lock`)
- Are there known vulnerable dependencies? (recommend running `npm audit`, `pip-audit`, `govulncheck`, `cargo audit`)

**Result:**
- PASS: Pinned versions + lock file present
- WARN: Lock file present but some versions unpinned
- FAIL: No lock file or wildcard versions used

#### S4: Security Headers / Configuration
**Check:** For web applications, verify security configurations exist.

- CORS configuration present and restrictive
- Helmet.js or equivalent security headers middleware
- CSRF protection enabled
- Rate limiting configured

**Result:**
- PASS: Security middleware/configuration found
- WARN: Partial security configuration
- FAIL: No security configuration found (web apps only)

---

### Category 2: Quality (Weight: 25%)

#### Q1: Tests Exist
**Check:** Verify the project has tests.

**Look for:**
- Test directories: `test/`, `tests/`, `__tests__/`, `spec/`, `*_test.go`
- Test files: `*.test.js`, `*.test.ts`, `*.spec.js`, `*_test.py`, `test_*.py`, `*_test.go`, `*_test.rs`
- Test configuration: `jest.config.*`, `pytest.ini`, `setup.cfg [tool:pytest]`, `.mocharc.*`
- Test scripts in `package.json`: `"test"` script defined

**Result:**
- PASS: Test directory exists with test files, test runner configured
- WARN: Test directory exists but few tests or no test runner config
- FAIL: No tests found

#### Q2: Test Coverage Configuration
**Check:** Is test coverage measurement configured?

**Look for:**
- Coverage config in `jest.config.*`, `pytest.ini`, `.coveragerc`
- Coverage scripts in `package.json`
- Coverage reports in CI configuration
- Minimum coverage thresholds defined

**Result:**
- PASS: Coverage configured with thresholds
- WARN: Coverage configured but no minimum thresholds
- FAIL: No coverage configuration

#### Q3: Linting Configured
**Check:** Is code linting set up?

**Look for:**
- ESLint: `.eslintrc.*`, `eslint.config.*`
- Prettier: `.prettierrc.*`
- Python: `.flake8`, `pyproject.toml [tool.ruff]`, `setup.cfg [flake8]`, `.pylintrc`
- Go: `golangci-lint` configuration, `.golangci.yml`
- Rust: `clippy` in CI, `rustfmt.toml`
- EditorConfig: `.editorconfig`

**Result:**
- PASS: Linter + formatter configured
- WARN: Only linter or only formatter configured
- FAIL: No linting or formatting configured

#### Q4: Type Safety
**Check:** For languages with optional typing, is it enabled?

**Look for:**
- TypeScript: `tsconfig.json` with `"strict": true`
- Python: `mypy.ini`, `pyproject.toml [tool.mypy]`, type hints in code, `py.typed` marker
- JSDoc type annotations as alternative to TypeScript

**Result:**
- PASS: Strict type checking enabled
- WARN: Type checking present but not strict
- FAIL: No type checking (for languages where it is available)
- N/A: Language has built-in type system (Go, Rust, Java)

---

### Category 3: Documentation (Weight: 20%)

#### D1: README Exists and Is Substantive
**Check:** Does `README.md` exist? Is it more than a stub?

**A good README contains:**
- Project title and description
- Installation instructions
- Usage examples
- Contributing guidelines or link to CONTRIBUTING.md
- License reference

**Result:**
- PASS: README exists with all five sections
- WARN: README exists but missing sections
- FAIL: No README or empty/stub README

#### D2: LICENSE Exists
**Check:** Is there a `LICENSE` or `LICENSE.md` file?

**Result:**
- PASS: License file exists with a recognized license
- WARN: License mentioned in README but no LICENSE file
- FAIL: No license information anywhere

#### D3: CHANGELOG or Release Notes
**Check:** Is there a `CHANGELOG.md`, or are GitHub Releases used?

**Result:**
- PASS: CHANGELOG exists or releases are documented
- WARN: Partial changelog or inconsistent releases
- FAIL: No changelog or release documentation

#### D4: API Documentation
**Check:** For libraries and APIs, is there documentation for the public interface?

**Look for:**
- JSDoc / docstrings on exported functions
- OpenAPI / Swagger spec for REST APIs
- Generated docs (TypeDoc, Sphinx, godoc, rustdoc)
- `docs/` directory with substantive content

**Result:**
- PASS: Public API is documented
- WARN: Partial documentation
- FAIL: No API documentation (libraries/APIs only)
- N/A: Not applicable (CLI tools, scripts)

---

### Category 4: CI/CD and Operations (Weight: 15%)

#### O1: CI/CD Pipeline Configured
**Check:** Is there an automated build/test pipeline?

**Look for:**
- GitHub Actions: `.github/workflows/*.yml`
- GitLab CI: `.gitlab-ci.yml`
- CircleCI: `.circleci/config.yml`
- Travis CI: `.travis.yml`
- Jenkins: `Jenkinsfile`
- Generic: `Makefile`, `Taskfile.yml`, npm scripts for build/test/lint

**Result:**
- PASS: CI pipeline runs tests and linting automatically
- WARN: CI exists but only runs tests (no lint, no type check)
- FAIL: No CI/CD configuration

#### O2: Branch Protection / PR Process
**Check:** Is there evidence of a code review process?

**Look for:**
- `CODEOWNERS` file
- Branch protection rules (check via GitHub API if available)
- PR templates: `.github/pull_request_template.md`
- Contributing guide mentioning PR process

**Result:**
- PASS: CODEOWNERS + PR template + contributing guide
- WARN: Some review process artifacts present
- FAIL: No code review process artifacts

#### O3: Container / Deployment Configuration
**Check:** Is deployment reproducible?

**Look for:**
- `Dockerfile` with good practices (multi-stage build, non-root user, pinned base image)
- `docker-compose.yml` for local development
- Deployment manifests (Kubernetes, Terraform, CloudFormation)
- Infrastructure as Code

**Result:**
- PASS: Reproducible deployment configuration present
- WARN: Dockerfile exists but with issues (root user, `latest` tag)
- FAIL: No deployment configuration
- N/A: Library/package (deployment is via package registry)

---

### Category 5: Code Hygiene (Weight: 10%)

#### H1: .gitignore Is Correct
**Check:** Does `.gitignore` cover all standard exclusions for the project type?

**Node.js must exclude:** `node_modules/`, `dist/`, `.env`, `*.log`, `coverage/`
**Python must exclude:** `__pycache__/`, `*.pyc`, `.venv/`, `*.egg-info/`, `.env`, `dist/`
**Go must exclude:** Binary outputs, `.env`, vendor/ (if not vendoring)
**Rust must exclude:** `target/`, `.env`

**Result:**
- PASS: `.gitignore` covers all standard patterns for the project type
- WARN: `.gitignore` exists but missing patterns
- FAIL: No `.gitignore`

#### H2: No Large Binary Files
**Check:** Are there large binary files committed to the repository?

**Flag:** Files over 1MB that are not documentation images. Especially: `.zip`, `.tar.gz`, `.jar`, `.exe`, `.dll`, `.so`, compiled binaries, database files, media files.

**Result:**
- PASS: No large binaries in tracked files
- WARN: Some binary files present (under 5MB total)
- FAIL: Large binaries committed (use Git LFS or artifact storage)

#### H3: Consistent Code Style
**Check:** Is the codebase consistently formatted?

**Look for:**
- `.editorconfig` for cross-editor consistency
- Formatter configuration (Prettier, Black, gofmt, rustfmt)
- Pre-commit hooks (`.husky/`, `.pre-commit-config.yaml`)

**Result:**
- PASS: Formatter configured + pre-commit hooks enforce it
- WARN: Formatter configured but no enforcement via hooks
- FAIL: No formatting configuration

---

## Scoring System

### Point Calculation

Each check result earns points:
- **PASS** = 100 points
- **WARN** = 50 points
- **FAIL** = 0 points
- **N/A** = excluded from calculation

### Category Scores

Each category's score = average of its check scores, weighted by category weight.

### Overall Score and Grade

| Grade | Score Range | Description |
|-------|------------|-------------|
| **A** | 90-100 | Excellent. Production-ready, well-maintained |
| **B** | 75-89 | Good. Minor improvements needed |
| **C** | 60-74 | Acceptable. Several gaps to address |
| **D** | 40-59 | Poor. Significant issues, not production-ready |
| **F** | 0-39 | Failing. Major work needed across categories |

### Grade Caps

- Any **Critical security finding** (secrets in repo) caps grade at **D**
- No tests at all caps grade at **C**
- No README caps grade at **C**
- No `.gitignore` caps grade at **D**

---

## Output Format

```
## Project Health Report

**Project:** [name]
**Type:** [Node.js web app / Python library / Go microservice / etc.]
**Date:** [date]
**Guardian:** sovereign-project-guardian v1.0.0

### Overall Grade: [A-F] ([score]/100)

### Category Breakdown

| Category | Score | Checks Passed | Checks Failed |
|----------|-------|---------------|---------------|
| Security (30%) | XX/100 | X | X |
| Quality (25%) | XX/100 | X | X |
| Documentation (20%) | XX/100 | X | X |
| CI/CD & Ops (15%) | XX/100 | X | X |
| Code Hygiene (10%) | XX/100 | X | X |

### Detailed Findings

#### Security
- [PASS] S1: No secrets in repository
- [FAIL] S2: .env files not in .gitignore
  - Action: Add `.env*` to `.gitignore`
...

#### Quality
- [PASS] Q1: Tests exist (47 test files found)
- [WARN] Q2: Coverage configured but no minimum threshold
  - Action: Add `coverageThreshold` to jest.config.js
...

### Priority Action Plan

1. [CRITICAL] Add .env to .gitignore and remove from history
2. [HIGH] Configure test coverage thresholds (aim for 80%)
3. [MEDIUM] Add CHANGELOG.md
4. [LOW] Set up pre-commit hooks for formatting
```

---

## Project Type Detection

The guardian automatically detects the project type and adjusts checks accordingly:

| Indicator | Project Type | Adjusted Checks |
|-----------|-------------|-----------------|
| `package.json` + `src/` + framework dep | Node.js Web App | Security headers check applies |
| `package.json` + `index.js/d.ts` + no framework | Node.js Library | Skip deployment checks |
| `pyproject.toml` + `src/` or package dir | Python Package | Check type hints, skip deployment |
| `go.mod` + `cmd/` | Go Service | Check for race condition testing |
| `go.mod` + no `cmd/` | Go Library | Skip deployment checks |
| `Cargo.toml` + `src/main.rs` | Rust Binary | Check unsafe usage |
| `Cargo.toml` + `src/lib.rs` | Rust Library | Check documentation, skip deployment |

---

## Installation

```bash
clawhub install sovereign-project-guardian
```

## Files

| File | Description |
|------|-------------|
| `SKILL.md` | This file -- complete evaluation methodology |
| `EXAMPLES.md` | Before/after: taking a project from F to A |
| `README.md` | Quick start and overview |

## License

MIT
