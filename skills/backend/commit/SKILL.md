---
name: commit
description: Stage changes, run pre-commit hooks, and create a well-formatted git commit
allowed-tools: Bash, Read, Grep, Glob
---

# Git Commit Workflow

Create a clean, well-documented git commit following this project's conventions.

## Step 1: Review changes

```bash
git status
git diff --stat
```

Show the user a summary of what will be committed.

## Step 2: Stage files

Stage files **selectively** (never `git add .` or `git add -A` blindly):

```bash
# Stage specific files
git add <file1> <file2> ...
```

### Files to NEVER commit

| Pattern | Reason |
|---------|--------|
| `.env` | Contains `HUGGINGFACE_ACCESS_TOKEN` |
| `*.pyc`, `__pycache__/` | Compiled Python |
| `.DS_Store` | macOS junk |
| `model_output/` | Downloaded models |
| `data/eval_results/` | Generated evaluation data |
| `output/` | Pipeline outputs |
| `.claude/settings.local.json` | Personal Claude settings |
| `.claude/CLAUDE.local.md` | Personal Claude memory |

If `$ARGUMENTS` says "all" or "todo", stage all modified/untracked files except those above.

## Step 3: Pre-commit hooks

This project has pre-commit hooks configured in `.pre-commit-config.yaml`:
- **ruff** (linter) - auto-fixes code style
- **ruff-format** (formatter) - auto-formats code
- **gitleaks** - scans for leaked secrets

If a hook fails:
1. The commit is **aborted** (it did NOT happen)
2. Fix the issues (ruff usually auto-fixes, just re-stage)
3. Create a **NEW** commit (never `--amend` after a hook failure - that would modify the wrong commit)

## Step 4: Commit message format

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
```

### Types

| Type | When to use |
|------|-------------|
| `feat` | New feature or functionality |
| `fix` | Bug fix |
| `docs` | Documentation changes (README, CLAUDE.md, comments) |
| `refactor` | Code restructuring without behavior change |
| `chore` | Maintenance (deps, configs, cleanup) |
| `style` | Formatting, linting (no logic change) |
| `test` | Adding or modifying tests |
| `perf` | Performance improvements |
| `ci` | CI/CD changes |

### Scopes (optional, use the most relevant)

| Scope | Files |
|-------|-------|
| `inference` | model/inference/, infrastructure/inference_pipeline_api.py |
| `rag` | application/rag/ |
| `eval` | model/evaluation/ |
| `training` | model/finetuning/ |
| `etl` | application/crawlers/, configs/digital_data_etl_* |
| `dataset` | application/dataset/ |
| `settings` | settings.py, .env, configs/ |
| `deps` | pyproject.toml, poetry.lock |
| `infra` | docker-compose.yml, ZenML configs |

### Examples

```
feat(rag): Add author filtering to Qdrant search
fix(inference): Add task parameter to HuggingFaceEndpoint
docs: Rewrite README for local-only setup
chore(deps): Upgrade langchain-huggingface to 1.2.0
refactor(eval): Replace OpenAI judge with HuggingFace
```

## Step 5: Create the commit

Use HEREDOC for proper message formatting:

```bash
git commit -m "$(cat <<'EOF'
<type>(<scope>): <description>

<body if needed>

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

## Step 6: Verify

```bash
git log --oneline -3
```

Show the user the new commit hash and message.

## After the commit

Do NOT push automatically. Tell the user they can push with `/push` or `git push`.

## If pre-commit hook fails

1. Check what failed: usually ruff auto-fixed files
2. Re-stage the auto-fixed files: `git add <fixed-files>`
3. Create a **NEW** commit (do NOT use `--amend`)
4. If gitleaks fails: a secret was detected. Remove it from the staged files before retrying.
