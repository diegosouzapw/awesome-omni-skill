---
name: git
description: Enforces conventional commit messages and good git hygiene. Use this skill automatically whenever creating commits, preparing PRs, or performing any git operations in this repository.
globs: "**/*"
---

# Git Conventional Commit Skill

You enforce [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) on every commit in this repository. This is critical for consistent changelogs and semantic version bumps.

## Commit message format

Every commit message MUST follow this format:

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types (required)

| Type       | When to use                                      | Version bump |
|------------|--------------------------------------------------|-------------|
| `feat`     | A new feature or capability                      | minor       |
| `fix`      | A bug fix                                        | patch       |
| `docs`     | Documentation only                               | none        |
| `style`    | Formatting, semicolons, whitespace — no logic    | none        |
| `refactor` | Code change that neither fixes a bug nor adds a feature | none  |
| `perf`     | Performance improvement                          | patch       |
| `test`     | Adding or correcting tests                       | none        |
| `build`    | Build system or external dependency changes      | none        |
| `ci`       | CI configuration changes                         | none        |
| `chore`    | Maintenance tasks (deps, tooling, releases)      | none        |

### Scopes (recommended)

Use a scope to identify the affected area:

| Scope    | Meaning                                     |
|----------|---------------------------------------------|
| `core`   | Elixir core (`lib/opal/` directory)             |
| `cli`    | TypeScript CLI (`cli/` directory)           |
| `rpc`    | JSON-RPC protocol layer                     |
| `agent`  | Agent loop / GenServer                      |
| `tools`  | Built-in tool implementations               |
| `mcp`    | MCP bridge                                  |
| `ci`     | GitHub Actions workflows                    |
| `release`| Release and versioning infrastructure       |
| `docs`   | Documentation                               |
| `sdk`    | TypeScript SDK client                       |

Omit the scope only when the change truly spans the entire project.

### Breaking changes

Append `!` after the type/scope for breaking changes:

```
feat(rpc)!: change message envelope format
```

Or add a `BREAKING CHANGE:` footer in the body.

## Rules

1. **Always** use conventional commit format. Never commit with a bare message like "fix stuff" or "updates".
2. **Description** must be lowercase, imperative mood, no period at the end.
3. **Scope** should match the table above when the change is localized.
4. **Body** is optional but encouraged for non-trivial changes — explain *why*, not *what*.
5. **One logical change per commit.** Don't bundle unrelated changes.
6. Before committing, verify the message parses as a valid conventional commit.

## Examples

```
feat(cli): add --json flag for machine-readable output
fix(core): prevent crash when tool returns empty response
docs(tools): document edit tool hashline format
refactor(agent): extract token counting into dedicated module
ci: add Elixir dialyzer step to CI pipeline
chore(release): 0.1.10
feat(rpc)!: switch from JSON-RPC 2.0 to msgpack framing

BREAKING CHANGE: RPC wire format changed from JSON to msgpack.
Clients must update to sdk >= 0.2.0.
```
