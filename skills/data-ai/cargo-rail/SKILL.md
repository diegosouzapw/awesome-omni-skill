---
name: cargo-rail
description: Rust workspace monorepo orchestration. Use when working with cargo-rail for dependency unification, unused dep detection, dead feature pruning, MSRV computation, affected crate testing, crate extraction, or release orchestration.
---

# cargo-rail

Graph-aware monorepo orchestration for Rust workspaces.

## What It Replaces

| Problem | Before | After |
|---------|--------|-------|
| **Build graph drift** | `cargo-hakari`, workspace-hack crates | `cargo rail unify` |
| **Unused deps** | `cargo-udeps`, `cargo-machete`, `cargo-shear` | `cargo rail unify` |
| **Dead features** | `cargo-features-manager`, manual audit | `cargo rail unify` |
| **MSRV computation** | `cargo-msrv`, compile-and-fail loops | `cargo rail unify` |
| **CI waste** | `paths-filter` + shell scripts | `cargo rail affected` |
| **CI costs** | Test everything, bill for everything | Test what changed |
| **Crate extraction** | `git subtree`, `git-filter-repo`, Google's Copybara | `cargo rail split` |
| **Release orchestration** | `release-plz`, `cargo-release`, `git-cliff` | `cargo rail release` |

**11 dependencies. One config file.**

## MSRV policy

- **MSRV source of truth**: `Cargo.toml` (`rust-version`, written as `major.minor.patch`)
- **Cargo requirement**: Cargo shipped with that Rust release (newer Cargo is fine)
- **CI**: builds on MSRV to prevent accidental bumps
- **Workspaces**: `cargo rail unify` writes `[workspace.package].rust-version`; enable `[unify].enforce_msrv_inheritance = true` to set `[package].rust-version = { workspace = true }` in member crates

## Quick Start

```bash
cargo rail init              # generate .config/rail.toml
cargo rail unify --check     # preview what would change (read-only)
cargo rail unify             # apply changes
```

## Commands

### `affected` / `test`

Graph-aware change detection. Only test what's affected:

```bash
cargo rail affected                    # list affected crates
cargo rail affected --merge-base       # compare against merge-base (CI)
cargo rail affected -f cargo-args      # output: -p crate1 -p crate2
cargo rail affected -f github-matrix   # output: JSON matrix for Actions
cargo rail test                        # run tests for affected crates
cargo rail test --explain              # show why each crate is affected
```

**CI Integration:**

```yaml
- uses: loadingalias/cargo-rail-action@v1
  id: rail

- run: cargo nextest run ${{ steps.rail.outputs.cargo-args }}
  if: steps.rail.outputs.should-test == 'true'
```

### `unify`

Dependency unification based on Cargo's resolved output:

```bash
cargo rail unify --check    # preview changes (exits 1 if drift detected)
cargo rail unify            # apply to workspace
cargo rail unify --explain  # show reasoning for each change
cargo rail unify undo       # restore from backup
```

What it does:

- **Unifies versions** — writes to `[workspace.dependencies]`, converts members to `workspace = true`
- **Prunes dead features** — removes features never enabled in the resolved graph
- **Fixes undeclared features** — adds missing feature declarations to member manifests
- **Detects unused deps** — flags dependencies not used anywhere (auto-removes on apply)
- **Computes MSRV** — derives minimum Rust version from dependency graph
- **Pins transitives** — replaces `cargo-hakari` without a workspace-hack crate

Multi-target aware: runs `cargo metadata` per target triple in parallel, computes feature *intersections* not unions.

### `split` / `sync`

Extract crates with full git history. Bidirectional sync with 3-way conflict resolution:

```bash
cargo rail split init crate/s         # configure extraction
cargo rail split run crate/s          # extract with history
cargo rail split run crate/s --check  # preview (dry-run)

cargo rail sync crate/s               # bidirectional sync
cargo rail sync crate/s --to-remote   # push changes to split repo
cargo rail sync crate/s --from-remote # pull changes (creates PR branch)
```

**Three modes:**

- `single` — one crate → one repo (most common)
- `combined` — multiple crates → one repo (shared utilities)
- `workspace` — multiple crates → workspace structure (mirrors monorepo)

Safety: refuses dirty worktree by default. `--allow-dirty` to override, `--yes` for CI.

### `release`

Dependency-order publishing with changelog generation:

```bash
cargo rail release check crate/s              # validate release readiness
cargo rail release run crate/s --bump minor   # bump, tag, publish
cargo rail release run crate/s --check        # preview release plan
```

Safety: detects default branch, refuses detached HEAD, warns on non-default branch.

### `config`

Manage configuration:

```bash
cargo rail init              # generate .config/rail.toml
cargo rail config locate     # print active config path
cargo rail config print      # print effective config with defaults
cargo rail config validate   # check for errors and unknown keys
cargo rail config sync       # update config with detected targets (incredibly useful on update)
```

---

## Configuration

Generated by `cargo rail init` at `.config/rail.toml`:

```toml
targets = ["x86_64-unknown-linux-gnu", "aarch64-apple-darwin"]

[unify]
pin_transitives = false      # enable for hakari replacement
detect_unused = true
prune_dead_features = true

msrv = true
msrv_source = "max"          # deps | workspace | max
enforce_msrv_inheritance = false

[release]
tag_format = "{crate}-{prefix}{version}"
publish_delay = 5            # seconds between publishes

[change-detection]
infrastructure = [".github/**", "scripts/**", "*.sh"]
```

See [full reference](references/config.md).

## Notes

- **vs cargo-hakari**: No workspace-hack crate. Writes directly to `[workspace.dependencies]`. Use `pin_transitives = true` for equivalent behavior.
- **Workspace inheritance**: Yes. Converts members to `{ workspace = true }`.
- **Virtual workspaces**: Supported. Auto-selects transitive host or configure explicitly.
- **Private registries**: Works via `cargo metadata`, respects `.cargo/config.toml`.
- **Undeclared features**: Detects when crates "borrow" features from siblings (works in workspace, breaks on publish). Auto-fixes by adding missing declarations.

## References

- [Config Reference](references/config.md)
- [Migration Guide](references/migrate-hakari.md)