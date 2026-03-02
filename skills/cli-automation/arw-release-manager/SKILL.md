---
name: "ARW Release Manager"
description: "Build and release manager for ARW CLI package. Handles local development, testing, building, documentation, version tagging, and publishing to npm and crates.io. Use when developing locally, running tests, building packages, releasing new versions, or publishing packages."
---

# ARW Release Manager

Build, test, and release orchestration for the ARW (Agent-Ready Web) CLI package supporting multiple build targets and publishing to npm and crates.io.

## Build Targets

The ARW CLI supports **three distinct build targets**:

| Target | Command | Output | Use Case |
|--------|---------|--------|----------|
| **napi-rs** | `npm run build` | `arw-cli.darwin-arm64.node` | Node.js native addon (require in JS) |
| **Standalone CLI** | `cargo build --release --features native` | `target/release/arw` | Executable binary |
| **WASM** | `npm run build:wasm` | `wasm-pkg/` | Browser/bundler usage |

## Prerequisites

- Node.js 18+ and npm 8+
- Rust toolchain (rustc, cargo) for Rust builds
- Git repository
- wasm-pack (for WASM builds): `cargo install wasm-pack`

For publishing only:

- npm account with publish access
- crates.io account with API token

## What This Skill Does

1. **Local development** - Watch mode, hot reload, iterative builds
2. **Testing** - Unit tests, integration tests, WASM tests, linting
3. **Building** - napi-rs native addon, standalone CLI, and WASM builds
4. **Pre-release validation** - Git status, version checks, secrets scanning
5. **Documentation** - CHANGELOG generation, API docs
6. **Version management** - Semantic versioning, git tagging
7. **Publishing** - npm publish, cargo publish with dry-run validation

---

## Quick Start

### Local Development (No Publishing)

```bash
# Build napi-rs native addon (Node.js bindings)
npm install && npm run build

# Build standalone CLI binary
cargo build --release --features native
./target/release/arw --version

# Run tests
npm test && npm run test:wasm && cargo test
```

### Full Release Workflow

```bash
# 1. Verify, test, and build (dry run)
./scripts/quick-publish.sh --dry-run

# 2. If checks pass, publish
./scripts/quick-publish.sh
```

---

## Build Targets Explained

### 1. napi-rs Native Addon (Node.js)

This builds a native Node.js addon using napi-rs. The result is a `.node` file that can be `require()`'d from JavaScript.

```bash
npm run build                    # Release build
npm run build:debug              # Debug build (faster)

# Test the module
node -e "const cli = require('./index.js'); console.log(cli.getVersionInfo());"
```

**Exports:** `validateManifest`, `checkCompatibility`, `generateManifest`, `getVersionInfo`

### 2. Standalone CLI Binary

This builds a standalone executable that can be run directly from the command line.

```bash
cargo build --release --features native

# Run directly
./target/release/arw --version
./target/release/arw --help
./target/release/arw validate path/to/manifest.json
./target/release/arw init
./target/release/arw generate
```

**CLI Commands:** `init`, `generate`, `sitemap`, `validate`, `serve`, `scan`, `policy`, `robots`, `watch`, `actions`, `build`

### 3. WASM Build

For browser and bundler usage.

```bash
npm run build:wasm              # Node.js target
npm run build:wasm:web          # Browser target
npm run build:wasm:bundler      # Bundler target

# Test WASM
npm run test:wasm
```

---

## Local Development Guide

### Node.js Native Addon Development

```bash
# Install dependencies
npm install

# Build native addon (napi-rs)
npm run build              # Release
npm run build:debug        # Debug (faster iteration)

# Test module loading
node -e "const {validateManifest, getVersionInfo} = require('./index.js'); console.log(getVersionInfo());"

# Link for testing in other projects
npm link
```

### Standalone CLI Development

```bash
# Debug build (fast compilation)
cargo build --features native

# Run without installing
cargo run --features native -- --version
cargo run --features native -- --help
cargo run --features native -- validate ./test-manifest.json

# Release build
cargo build --release --features native
./target/release/arw --version

# Install globally
cargo install --path . --features native
arw --version

# Watch mode (requires cargo-watch)
cargo install cargo-watch
cargo watch -x "run --features native -- --help"
```

### WASM Development

```bash
# Build WASM
npm run build:wasm

# Run WASM tests
npm run test:wasm
```

### Development Workflow

1. Make code changes in `src/`
2. Run tests: `cargo test` / `npm run test:wasm`
3. Build target you need:
   - Node.js addon: `npm run build`
   - Standalone CLI: `cargo build --release --features native`
   - WASM: `npm run build:wasm`
4. Test locally
5. Repeat

---

## Testing

### Node.js Tests

```bash
# Run Node.js native tests
npm test

# Run WASM-specific tests
npm run test:wasm
```

### Rust Tests

```bash
# Format check
cargo fmt --check

# Linting with clippy
cargo clippy -- -D warnings

# Run all tests
cargo test

# Run with output visible
cargo test -- --nocapture

# All checks
cargo fmt --check && cargo clippy -- -D warnings && cargo test
```

---

## Building

### napi-rs Native Addon

```bash
# Release build (optimized)
npm run build

# Debug build (faster compilation)
npm run build:debug
```

Output: `arw-cli.<platform>.node` + `index.js` + `index.d.ts`

### Standalone CLI Binary

```bash
# Debug build
cargo build --features native

# Release build (optimized)
cargo build --release --features native
```

Output: `target/release/arw` or `target/debug/arw`

### WASM Builds

```bash
# Node.js WASM
npm run build:wasm

# Browser WASM
npm run build:wasm:web

# Bundler WASM
npm run build:wasm:bundler
```

Output: `wasm-pkg/nodejs/`, `wasm-pkg/web/`, `wasm-pkg/bundler/`

### Cross-Platform Standalone CLI Builds

```bash
cargo build --release --features native --target x86_64-unknown-linux-gnu
cargo build --release --features native --target x86_64-apple-darwin
cargo build --release --features native --target aarch64-apple-darwin
cargo build --release --features native --target x86_64-pc-windows-msvc
```

---

## Pre-Release Validation

Run before any release:

```bash
./scripts/verify-package.sh
```

This checks:

- package.json validity and required fields
- Semantic version format
- Built files exist
- CLI executables have shebangs
- No hardcoded secrets
- Documentation files present
- Package size reasonable

---

## Version Management

### Semantic Versioning Rules

| Change Type | Version Bump | Example |
|-------------|--------------|---------|
| Breaking changes | MAJOR | 1.0.0 → 2.0.0 |
| New features (backwards compatible) | MINOR | 1.0.0 → 1.1.0 |
| Bug fixes | PATCH | 1.0.0 → 1.0.1 |

### Bump Version

```bash
# TypeScript - auto-bump in package.json
npm version patch --no-git-tag-version  # or minor/major

# Rust - manual edit Cargo.toml
# Change: version = "x.y.z"
```

### Update CHANGELOG

Use Keep a Changelog format. See `resources/templates/CHANGELOG.template.md`.

```bash
# Generate commit log since last tag
git log $(git describe --tags --abbrev=0)..HEAD --oneline
```

---

## Publishing

### npm

```bash
# Check login
npm whoami

# Preview package contents
npm pack --dry-run

# Dry run publish
npm publish --dry-run --access public

# Publish for real
npm publish --access public

# Verify
npm view arw-cli
```

### crates.io

```bash
# Login (one-time setup)
cargo login

# Dry run
cargo publish --dry-run

# Publish
cargo publish

# Verify
cargo search arw-cli
```

---

## Available Scripts

| Script | Purpose |
|--------|---------|
| `scripts/build-all.sh` | Build all packages |
| `scripts/build-all.sh --dev` | Development build only |
| `scripts/build-all.sh --release` | Production release build |
| `scripts/verify-package.sh` | Pre-release validation |
| `scripts/quick-publish.sh --dry-run` | Preview release |
| `scripts/quick-publish.sh` | Full release workflow |
| `scripts/quick-publish.sh --skip-tests` | Release without tests |
| `scripts/quick-publish.sh --npm-only` | Publish to npm only |
| `scripts/quick-publish.sh --cargo-only` | Publish to crates.io only |

---

## Troubleshooting

### npm Issues

#### 403 Forbidden

- Check `npm whoami` - must be logged in
- Verify package name isn't taken
- Ensure `--access public` for scoped packages

#### Version Already Exists

- npm doesn't allow republishing same version
- Bump version and try again

#### EPERM / Permission Denied

- Fix npm prefix: `npm config set prefix ~/.npm-global`
- Or use `sudo` (not recommended)

### Cargo Issues

#### Not Logged In

- Run `cargo login` with API token from crates.io
- Token stored in `~/.cargo/credentials.toml`

#### Version Already Published

- crates.io doesn't allow version overwrites
- Must bump version

#### Missing Required Fields

- Cargo.toml needs: name, version, edition, description, license, repository

### Git Issues

#### Dirty Working Tree

- Commit or stash changes before release
- `git stash` or `git add -A && git commit`

#### Tag Already Exists

- Delete and recreate: `git tag -d v1.0.0 && git tag v1.0.0`
- Or use different version

---

## Advanced Configuration

See [docs/ADVANCED.md](docs/ADVANCED.md) for:

- CI/CD integration with GitHub Actions
- Multi-platform release builds
- Automated changelog generation
- GPG signing releases
- Pre-release channels (alpha, beta, rc)

---

## Resources

- Templates: `resources/templates/`
- Advanced docs: `docs/ADVANCED.md`
- [Keep a Changelog](https://keepachangelog.com)
- [Semantic Versioning](https://semver.org)
- [npm publish docs](https://docs.npmjs.com/cli/publish)
- [Cargo publish docs](https://doc.rust-lang.org/cargo/reference/publishing.html)
