---
name: project-context-discovery
description: Discover project structure, package managers, test frameworks, and automation without hardcoded assumptions
---

# Project Context Discovery Skill

Provides expertise in discovering project characteristics through exploration
rather than assumptions.

## Purpose

This skill equips agents with strategies for:

- Identifying package managers and build systems
- Discovering test frameworks and commands
- Finding automation scripts and tooling
- Learning project conventions from configuration

## When to Use

Use this skill when:

- Need to run tests but don't know the framework
- Setting up a new project environment
- Installing dependencies without hardcoded assumptions
- Discovering how CI/CD runs commands

## Discovery Strategy

### Phase 1: Project Structure Exploration

1. **List root directory contents**

   ```bash
   ls -la
   ```

2. **Identify language(s) from file extensions**

   - `.rb` → Ruby
   - `.js`/`.ts` → JavaScript/TypeScript
   - `.py` → Python
   - `.go` → Go
   - `.rs` → Rust
   - Multiple languages? Note all

3. **Find configuration files**

   Look for package manager indicators:

   - `package.json` + `package-lock.json` → npm
   - `package.json` + `yarn.lock` → yarn
   - `package.json` + `pnpm-lock.yaml` → pnpm
   - `Gemfile` + `Gemfile.lock` → Bundler (Ruby)
   - `Cargo.toml` + `Cargo.lock` → Cargo (Rust)
   - `go.mod` + `go.sum` → Go modules
   - `pyproject.toml` + `poetry.lock` → Poetry
   - `requirements.txt` → pip

### Phase 2: Test Framework Discovery

1. **Check CI configuration first**

   CI config is the **source of truth** for what actually runs:

   - `.github/workflows/*.yml` → GitHub Actions
   - `.gitlab-ci.yml` → GitLab CI
   - `.circleci/config.yml` → CircleCI
   - `.travis.yml` → Travis CI
   - `Jenkinsfile` → Jenkins
   - `azure-pipelines.yml` → Azure Pipelines

2. **Parse CI test commands**

   Look for `run:` or `script:` sections with test commands
   Identify job names like: test, tests, unit-test, integration-test, ci

3. **Check package manager configuration**

   - `package.json` → check `scripts.test` field
   - `Gemfile` → check for test-related gems
   - `Cargo.toml` → check for test dependencies
   - `pyproject.toml` → check test tool configuration

4. **Look for test directories**

   ```bash
   ls -d test/ tests/ spec/ __tests__/ 2>/dev/null
   ```

5. **Check for framework config files**

   Common test framework indicators:

   - `jest.config.js`, `vitest.config.js` → JavaScript test frameworks
   - `.rspec`, `spec/spec_helper.rb` → RSpec
   - `pytest.ini`, `test/test_helper.rb` → pytest or minitest
   - Config in package.json, pyproject.toml, or setup.cfg

### Phase 3: Automation Script Discovery

1. **Check documented commands** (prefer explicit over implicit)

   - `README.md` → look for "Testing", "Development", "Getting Started"
   - `CONTRIBUTING.md` → look for contribution workflow
   - `Makefile` → look for test targets

2. **Check automation directories**

   ```bash
   ls bin/ scripts/ script/ .local/bin/ 2>/dev/null
   ```

3. **Check package manager scripts**

   - npm: `npm run` (lists all scripts)
   - Bundler: `bundle exec rake -T`
   - Make: `make help` or `make -n`

### Phase 4: Dependency Installation

Based on discovered package manager, use appropriate install command:

**General strategy:**

- Prefer lockfile-based installs (reproducible, CI-friendly)
- Use CI-optimized commands when available (`npm ci` vs `npm install`)
- Check for frozen-lockfile options to prevent unexpected updates

**Example decision tree for Node.js:**

```bash
if [ -f package-lock.json ]; then npm ci
elif [ -f yarn.lock ]; then yarn install --frozen-lockfile
elif [ -f pnpm-lock.yaml ]; then pnpm install --frozen-lockfile
else npm install; fi
```

**Other languages follow similar patterns:**

- Ruby: Check for `Gemfile.lock` → use `bundle install`
- Python: Check for `poetry.lock` or `pyproject.toml` →
  use `poetry install` or `pip install`
- Go: Check for `go.sum` → use `go mod download`
- Rust: Check for `Cargo.lock` → use `cargo build`

### Phase 5: Test Command Construction

**Preference order (highest to lowest priority):**

1. **Explicit documentation** (README/CONTRIBUTING) - what humans are told
   to run
2. **CI configuration** (source of truth) - what automation actually runs
3. **Package manager scripts** (npm test, rake test) - defined shortcuts
4. **Framework defaults** (only as fallback) - conventions when nothing
   explicit exists

**Decision process:**

If CI config defines test command → use that (it's what's actually running
in production)

If package.json/Gemfile/etc has test script → use the defined script

If neither, infer from framework detection:

- JavaScript with Jest config → likely `npx jest`
- Ruby with RSpec directories → likely `bundle exec rspec`
- Python with pytest.ini → likely `pytest`
- Go with test files → likely `go test ./...`
- Rust with tests/ → likely `cargo test`

## Graceful Degradation

If discovery fails at any phase:

1. Note what couldn't be determined
2. Make best-effort guess with caveats
3. Provide fallback options
4. Report incomplete discovery to user

**Never fail completely** - always provide a path forward.

## Example Usage

```markdown
Agent using this skill:

1. Read project root directory
2. Identify languages: JavaScript (package.json found)
3. Check CI: .github/workflows/test.yml exists
4. Parse CI: runs "npm run test:unit && npm run test:integration"
5. Check package.json scripts: confirms test:unit and test:integration exist
6. Determine package manager: package-lock.json exists → npm
7. Discovery complete:
   - Language: JavaScript
   - Package manager: npm
   - Install command: npm ci (uses lockfile)
   - Test command: npm run test:unit && npm run test:integration (from CI)
8. Execute: npm ci && npm run test:unit && npm run test:integration
```

## Key Principles

- **Discover, don't assume**: Never hardcode project-specific commands
- **Prefer explicit over implicit**: Use documented/configured commands over
  framework defaults
- **CI is truth**: What runs in CI is what the project actually uses
- **Multiple indicators**: Combine signals from configs, directories, and
  dependencies
- **Priority order matters**: Check sources in order of reliability
  (CI → docs → conventions)
- **Graceful degradation**: Provide best-effort fallbacks when discovery is
  incomplete
