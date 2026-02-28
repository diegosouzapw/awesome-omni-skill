---
name: mise-tasks
description: Orchestrate workflows with mise [tasks]. TRIGGERS - mise tasks, mise run, task runner, depends, depends_post, workflow automation, task dependencies.
allowed-tools: Read, Bash, Glob, Grep, Edit, Write
---

# mise Tasks Orchestration

<!-- ADR: 2025-12-08-mise-tasks-skill -->

Orchestrate multi-step project workflows using mise `[tasks]` section with dependency management, argument handling, and file tracking.

## When to Use This Skill

**Explicit triggers**:

- User mentions `mise tasks`, `mise run`, `[tasks]` section
- User needs task dependencies: `depends`, `depends_post`
- User wants workflow automation in `.mise.toml`
- User mentions task arguments or `usage` spec

**AI Discovery trigger** (prescriptive):

> When `mise-configuration` skill detects multi-step workflows (test suites, build pipelines, migrations), **prescriptively invoke this skill** to generate appropriate `[tasks]` definitions.

## Quick Reference

### Task Definition

```toml
[tasks.build]
description = "Build the project"
run = "cargo build --release"
```

### Running Tasks

```bash
mise run build          # Run single task
mise run test build     # Run multiple tasks
mise run test ::: build # Run in parallel
mise r build            # Short form
```

### Dependency Types

| Type           | Syntax                       | When                    |
| -------------- | ---------------------------- | ----------------------- |
| `depends`      | `depends = ["lint", "test"]` | Run BEFORE task         |
| `depends_post` | `depends_post = ["notify"]`  | Run AFTER task succeeds |
| `wait_for`     | `wait_for = ["db"]`          | Wait only if running    |

### Key Task Properties

| Property      | Purpose                                     | Example                                                  |
| ------------- | ------------------------------------------- | -------------------------------------------------------- |
| `description` | AI-agent discoverability (CRITICAL)         | `"Run pytest with coverage. Exits non-zero on failure."` |
| `alias`       | Short name                                  | `alias = "t"`                                            |
| `dir`         | Working directory                           | `dir = "packages/frontend"`                              |
| `env`         | Task-specific env vars (NOT passed to deps) | `env = { LOG_LEVEL = "debug" }`                          |
| `hide`        | Hidden from `mise tasks` output             | `hide = true`                                            |
| `sources`     | File tracking for caching                   | `sources = ["src/**/*.rs"]`                              |
| `outputs`     | Skip if newer than sources                  | `outputs = ["target/release/myapp"]`                     |
| `confirm`     | Prompt before execution                     | `confirm = "Delete all data?"`                           |
| `quiet`       | Suppress mise output                        | `quiet = true`                                           |
| `silent`      | Suppress ALL output                         | `silent = true`                                          |
| `raw`         | Direct stdin/stdout (disables parallelism)  | `raw = true`                                             |
| `tools`       | Task-specific tool versions                 | `tools = { python = "3.9" }`                             |
| `shell`       | Custom shell                                | `shell = "pwsh -c"`                                      |
| `usage`       | Argument spec (preferred over Tera)         | See [Task Arguments](./references/arguments.md)          |

### Namespacing

```bash
mise run 'test:*'      # All tasks starting with test:
mise run 'db:**'       # Nested: db:migrate:up, db:seed:test
mise tasks --hidden    # View hidden tasks (prefixed with _)
```

For detailed examples and patterns for all levels, see [Task Levels Reference](./references/task-levels.md).

---

## Level 10: Monorepo (Experimental)

**Requires**: `MISE_EXPERIMENTAL=1` and `experimental_monorepo_root = true`

```bash
mise run //projects/frontend:build    # Absolute from root
mise run :build                       # Current config_root
mise run //...:test                   # All projects
mise run '//projects/...:build'       # Build all under projects/
```

Tasks in subdirectories are auto-discovered with path prefix (`packages/api/.mise.toml` tasks become `packages/api:taskname`).

For complete monorepo documentation, see: [advanced.md](./references/advanced.md)

---

## Level 11: Polyglot Monorepo with Pants + mise

For Python-heavy polyglot monorepos (10-50 packages), combine **mise** for runtime management with **Pants** for build orchestration and native affected detection.

| Tool      | Responsibility                                                         |
| --------- | ---------------------------------------------------------------------- |
| **mise**  | Runtime versions (Python, Node, Rust) + environment variables          |
| **Pants** | Build orchestration + native affected detection + dependency inference |

```bash
# Native affected detection (no manual git scripts)
pants --changed-since=origin/main test
pants --changed-since=origin/main lint
pants --changed-since=origin/main package
```

| Scale                             | Recommendation                             |
| --------------------------------- | ------------------------------------------ |
| < 10 packages                     | mise + custom affected (Level 10 patterns) |
| **10-50 packages (Python-heavy)** | **Pants + mise** (this section)            |
| 50+ packages                      | Consider Bazel                             |

See [polyglot-affected.md](./references/polyglot-affected.md) for complete Pants + mise integration guide and tool comparison.

---

## Integration with [env]

Tasks automatically inherit `[env]` values. Use `_.file` for external env files and `redact = true` for secrets.

```toml
[env]
DATABASE_URL = "postgresql://localhost/mydb"
_.file = { path = ".env.secrets", redact = true }

[tasks._check-env]
hide = true
run = '[ -n "$API_KEY" ] || { echo "Missing API_KEY"; exit 1; }'

[tasks.deploy]
depends = ["_check-env"]
run = "deploy.sh"  # $DATABASE_URL and $API_KEY available
```

For full env integration patterns, see [Environment Integration](./references/env-integration.md).

---

## Anti-Patterns

| Anti-Pattern                    | Why Bad                                             | Instead                                                                  |
| ------------------------------- | --------------------------------------------------- | ------------------------------------------------------------------------ |
| Replace /itp:go with mise tasks | No TodoWrite, no ADR tracking, no checkpoints       | Use mise tasks for project workflows, /itp:go for ADR-driven development |
| Hardcode secrets in tasks       | Security risk                                       | Use `_.file = ".env.secrets"` with `redact = true`                       |
| Giant monolithic tasks          | Hard to debug, no reuse                             | Break into small tasks with dependencies                                 |
| Skip or minimal `description`   | AI agents cannot infer task purpose from name alone | Write rich descriptions: what it does, requires, produces, when to run   |
| Publish without build `depends` | Runtime failure instead of DAG prevention           | Add `depends = ["build"]` to publish tasks                               |
| Orchestrator without all phases | "Run X next" messages get ignored                   | Include all phases in `release:full` depends array                       |

For release-specific anti-patterns and patterns, see [Release Workflow Patterns](./references/release-workflow-patterns.md).

---

## Cross-Reference: mise-configuration

**Prerequisites**: Before defining tasks, ensure `[env]` section is configured.

> **PRESCRIPTIVE**: After defining tasks, invoke **[`mise-configuration` skill](../mise-configuration/SKILL.md)** to ensure [env] SSoT patterns are applied.

The `mise-configuration` skill covers:

- `[env]` - Environment variables with defaults
- `[settings]` - mise behavior configuration
- `[tools]` - Version pinning
- Special directives: `_.file`, `_.path`, `_.python.venv`

---

## Additional Resources

- [Task Levels Reference](./references/task-levels.md) - Levels 1-9: basic tasks, dependencies, hidden tasks, arguments, file tracking, advanced execution, watch mode
- [Task Patterns](./references/patterns.md) - Real-world task examples
- [Task Arguments](./references/arguments.md) - Complete usage spec reference
- [Advanced Features](./references/advanced.md) - Monorepo, watch, experimental
- [Environment Integration](./references/env-integration.md) - [env] inheritance and credential loading
- [Polyglot Affected](./references/polyglot-affected.md) - Pants + mise integration guide and tool comparison
- [Bootstrap Monorepo](./references/bootstrap-monorepo.md) - Autonomous polyglot monorepo bootstrap meta-prompt
- [Release Workflow Patterns](./references/release-workflow-patterns.md) - Release task DAG patterns, build-before-publish enforcement

---

## Troubleshooting

| Issue                  | Cause                     | Solution                                   |
| ---------------------- | ------------------------- | ------------------------------------------ |
| Task not found         | Typo or wrong mise.toml   | Run `mise tasks` to list available tasks   |
| Dependencies not run   | Circular dependency       | Check task depends arrays for cycles       |
| Sources not working    | Wrong glob pattern        | Use relative paths from mise.toml location |
| Watch not triggering   | File outside sources list | Add file pattern to sources array          |
| Env vars not available | Task in wrong directory   | Ensure mise.toml is in cwd or parent       |
| Run fails with error   | Script path issue         | Use absolute path or relative to mise.toml |
