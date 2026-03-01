---
name: github
description: Access GitHub repositories via the GitHub REST API. Use this skill when the user wants to interact with GitHub including reading files, creating/updating files, listing repos, managing branches, viewing commits, working with issues, or managing pull requests. All scripts use PEP 723 inline metadata for dependencies and run via `uv run`. Requires GITHUB_TOKEN environment variable (a Personal Access Token with appropriate scopes).
---

# Skill Overview

This skill provides access to GitHub repositories via a set of Python scripts that wrap the GitHub REST API.

## Prerequisites

**Tool Dependency**:
- `uv` - The scripts in this skill require the [uv](https://docs.astral.sh/uv/) package manager/runner. Most cloud-based AI agents have `uv` pre-installed (or they can install it). Local agents should install it via `curl -LsSf https://astral.sh/uv/install.sh | sh` or see the [uv installation docs](https://docs.astral.sh/uv/getting-started/installation/).

**Environment Variables** (must be set before running any script):
- `GITHUB_TOKEN` - A GitHub Personal Access Token (classic or fine-grained) with appropriate scopes

**Recommended Token Scopes** (for classic PAT):
- `repo` - Full control of private repositories (or `public_repo` for public only)
- `read:org` - Read organization membership (optional, for org repos)

**Important**: Use a [fine-grained personal access token](https://github.com/settings/personal-access-tokens/new) when possible for better security. Configure only the repositories and permissions you need.

## Network Access

**Important**: The scripts in this skill require network access to the following domain:

- `api.github.com`

If you (the AI agent) have network restrictions, the user may need to whitelist this domain in the agent's settings for this skill to function.

## Common Code Used by All Scripts

This skill uses a shared common module (`github_common.py`) to centralize authentication, token management, HTTP header construction, repository string parsing, error handling, and retry logic with exponential backoff.

All scripts import from `github_common.py`, which makes maintenance easier and ensures consistent behavior across all operations.

## API Versioning

This skill uses explicit GitHub API versioning for long-term stability:
- API Version: `2022-11-28`
- Header: `X-GitHub-Api-Version: 2022-11-28`

This ensures consistent behavior even if GitHub releases new API versions with breaking changes.

## Available Scripts

All scripts include PEP 723 inline metadata declaring their dependencies. Just run with `uv run` — no manual dependency installation needed.

---

## Repository Operations

### List Repositories (`scripts/repo_list.py`)

```bash
uv run scripts/repo_list.py                          # List your repos
uv run scripts/repo_list.py --user octocat           # List user's repos
uv run scripts/repo_list.py --org github             # List org's repos
uv run scripts/repo_list.py --type public --sort updated --json
```

| Argument | Description |
|----------|-------------|
| `--user` | List repos for this user |
| `--org` | List repos for this organization |
| `--type` | Filter: all, public, private, forks, sources, member |
| `--sort` | Sort by: created, updated, pushed, full_name |
| `--per-page` | Results per page (max 100, default: 30) |
| `--page` | Page number (default: 1) |
| `--json`, `-j` | Output as JSON |

---

### Get Repository Contents (`scripts/repo_contents.py`)

```bash
uv run scripts/repo_contents.py owner/repo                    # Root listing
uv run scripts/repo_contents.py owner/repo --path README.md   # Get file
uv run scripts/repo_contents.py owner/repo --path src/        # List directory
uv run scripts/repo_contents.py owner/repo --path config.json --ref develop --json
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `--path`, `-p` | Path to file or directory (default: root) |
| `--ref`, `-r` | Git ref (branch, tag, or commit SHA) |
| `--json`, `-j` | Output as JSON with full metadata |

---

### Get Repository Tree (`scripts/repo_tree.py`)

```bash
uv run scripts/repo_tree.py owner/repo               # Full tree
uv run scripts/repo_tree.py owner/repo --ref develop # Specific branch
uv run scripts/repo_tree.py owner/repo --path src/   # Filter by path
uv run scripts/repo_tree.py owner/repo --json
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `--ref`, `-r` | Git ref (branch, tag, or commit SHA) |
| `--path`, `-p` | Filter to paths starting with this prefix |
| `--json`, `-j` | Output as JSON |

---

## File Operations

### Create or Update File (`scripts/file_write.py`)

```bash
# Create new file
uv run scripts/file_write.py owner/repo \
    --path docs/README.md \
    --content "# Documentation" \
    --message "Add docs"

# Update existing file (SHA required)
uv run scripts/file_write.py owner/repo \
    --path README.md \
    --content "# Updated" \
    --message "Update README" \
    --sha abc123...

# From local file
uv run scripts/file_write.py owner/repo \
    --path remote/script.py \
    --from-file local/script.py \
    --message "Upload script"

# Create an executable script (with --mode)
uv run scripts/file_write.py owner/repo \
    --path scripts/build.sh \
    --from-file build.sh \
    --message "Add build script" \
    --mode 755
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `--path`, `-p` | Path for the file in the repo (required) |
| `--content`, `-c` | File content as string |
| `--from-file`, `-f` | Read content from this local file |
| `--message`, `-m` | Commit message (required) |
| `--sha` | SHA of file being replaced (required for updates) |
| `--branch`, `-b` | Branch to commit to |
| `--mode` | File mode (e.g., 755 for executable). Creates a second commit to set mode. |
| `--json`, `-j` | Output as JSON |

**Note on `--mode`:** The GitHub Contents API doesn't support setting file modes directly. When `--mode` is specified, the script first creates/updates the file, then makes a second commit to set the mode using the Git Data API. Common modes: `755` (executable), `644` (regular file).

---

### Change File Mode (`scripts/file_chmod.py`)

Change file permissions (mode) for existing files. Useful for making scripts executable or reverting to regular file mode.

```bash
# Make a single file executable
uv run scripts/file_chmod.py owner/repo --path script.py --mode 755

# Make multiple files executable in one commit
uv run scripts/file_chmod.py owner/repo \
    --path scripts/build.sh \
    --path scripts/deploy.sh \
    --path scripts/test.py \
    --mode 755

# Remove executable bit
uv run scripts/file_chmod.py owner/repo --path script.py --mode 644

# On a specific branch with custom message
uv run scripts/file_chmod.py owner/repo \
    --path scripts/run.py \
    --mode 755 \
    --branch develop \
    --message "Make run.py executable"
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `--path`, `-p` | Path to file (required, can be repeated for multiple files) |
| `--mode`, `-m` | Target mode (required, e.g., 755 or 644) |
| `--branch`, `-b` | Branch to commit to (default: main) |
| `--message` | Commit message (auto-generated if not provided) |
| `--json`, `-j` | Output as JSON |

**Common modes:**
- `755` - Executable file (rwxr-xr-x)
- `644` - Regular file (rw-r--r--)

**How it works:** This script uses the Git Data API to modify file modes, which requires creating a new tree object and commit. Multiple files can be changed in a single commit for efficiency. Files that already have the target mode are skipped with a warning.

---

### Delete File (`scripts/file_delete.py`)

```bash
uv run scripts/file_delete.py owner/repo \
    --path docs/old.md \
    --sha abc123... \
    --message "Remove old doc"
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `--path`, `-p` | Path to the file to delete (required) |
| `--sha` | SHA of the file to delete (required) |
| `--message`, `-m` | Commit message (required) |
| `--branch`, `-b` | Branch to delete from |
| `--json`, `-j` | Output as JSON |

---

## Branch Operations

### List Branches (`scripts/branch_list.py`)

```bash
uv run scripts/branch_list.py owner/repo
uv run scripts/branch_list.py owner/repo --json
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `--per-page` | Results per page (max 100, default: 30) |
| `--page` | Page number (default: 1) |
| `--json`, `-j` | Output as JSON |

---

### Create Branch (`scripts/branch_create.py`)

```bash
uv run scripts/branch_create.py owner/repo --name feature/new-feature
uv run scripts/branch_create.py owner/repo --name hotfix/123 --from develop
uv run scripts/branch_create.py owner/repo --name release/v1.0 --from abc123...
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `--name`, `-n` | Name for the new branch (required) |
| `--from`, `-f` | Source branch or commit SHA |
| `--json`, `-j` | Output as JSON |

---

### Delete Branch (`scripts/branch_delete.py`)

```bash
uv run scripts/branch_delete.py owner/repo --name feature/old-branch
uv run scripts/branch_delete.py owner/repo --name merged-branch --force
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `--name`, `-n` | Branch name to delete (required) |
| `--force`, `-f` | Skip confirmation prompt |
| `--json`, `-j` | Output as JSON |

---

## Commit Operations

### List Commits (`scripts/commit_list.py`)

```bash
uv run scripts/commit_list.py owner/repo
uv run scripts/commit_list.py owner/repo --branch develop
uv run scripts/commit_list.py owner/repo --path src/main.py
uv run scripts/commit_list.py owner/repo --author octocat
uv run scripts/commit_list.py owner/repo --since 2024-01-01 --json
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `--branch`, `-b` | Branch name |
| `--path`, `-p` | Filter to commits affecting this path |
| `--author`, `-a` | Filter by author username or email |
| `--since` | Only commits after this date (ISO 8601) |
| `--until` | Only commits before this date (ISO 8601) |
| `--per-page` | Results per page (max 100, default: 30) |
| `--page` | Page number (default: 1) |
| `--json`, `-j` | Output as JSON |

---

### Get Commit Details (`scripts/commit_get.py`)

```bash
uv run scripts/commit_get.py owner/repo abc123def456
uv run scripts/commit_get.py owner/repo abc123def456 --json
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `sha` | Commit SHA (required) |
| `--json`, `-j` | Output as JSON |

---

## Issue Operations

### List Issues (`scripts/issue_list.py`)

```bash
uv run scripts/issue_list.py owner/repo
uv run scripts/issue_list.py owner/repo --state all
uv run scripts/issue_list.py owner/repo --labels "bug,urgent"
uv run scripts/issue_list.py owner/repo --assignee octocat --json
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `--state` | Filter: open, closed, all (default: open) |
| `--labels` | Comma-separated list of label names |
| `--assignee` | Filter by assignee username |
| `--sort` | Sort by: created, updated, comments |
| `--direction` | Sort direction: asc, desc |
| `--per-page` | Results per page (max 100) |
| `--page` | Page number |
| `--json`, `-j` | Output as JSON |

---

### Create Issue (`scripts/issue_create.py`)

```bash
uv run scripts/issue_create.py owner/repo --title "Bug report" --body "Description..."
uv run scripts/issue_create.py owner/repo --title "Feature" --labels "enhancement"
uv run scripts/issue_create.py owner/repo --title "Task" --assignees "user1,user2"
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `--title`, `-t` | Issue title (required) |
| `--body`, `-b` | Issue body/description |
| `--labels` | Comma-separated list of label names |
| `--assignees` | Comma-separated list of usernames |
| `--milestone` | Milestone number |
| `--json`, `-j` | Output as JSON |

---

### Update Issue (`scripts/issue_update.py`)

```bash
uv run scripts/issue_update.py owner/repo 123 --title "New title"
uv run scripts/issue_update.py owner/repo 123 --state closed
uv run scripts/issue_update.py owner/repo 123 --state closed --reason not_planned
uv run scripts/issue_update.py owner/repo 123 --labels "bug,urgent"
uv run scripts/issue_update.py owner/repo 123 --assignees "user1,user2"
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `issue_number` | Issue number to update (required) |
| `--title`, `-t` | New title |
| `--body`, `-b` | New body/description |
| `--state`, `-s` | New state: open, closed |
| `--reason` | Close reason: completed, not_planned |
| `--labels` | Comma-separated labels (replaces existing) |
| `--assignees` | Comma-separated usernames (replaces existing) |
| `--milestone` | Milestone number (0 to clear) |
| `--json`, `-j` | Output as JSON |

---

## Pull Request Operations

### List Pull Requests (`scripts/pr_list.py`)

```bash
uv run scripts/pr_list.py owner/repo
uv run scripts/pr_list.py owner/repo --state all
uv run scripts/pr_list.py owner/repo --base main
uv run scripts/pr_list.py owner/repo --sort updated --json
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `--state` | Filter: open, closed, all (default: open) |
| `--base` | Filter by base branch |
| `--head` | Filter by head branch |
| `--sort` | Sort by: created, updated, popularity, long-running |
| `--direction` | Sort direction: asc, desc |
| `--per-page` | Results per page (max 100) |
| `--page` | Page number |
| `--json`, `-j` | Output as JSON |

---

### Create Pull Request (`scripts/pr_create.py`)

```bash
uv run scripts/pr_create.py owner/repo --title "Add feature" --head feature-branch
uv run scripts/pr_create.py owner/repo --title "Fix bug" --head fix-123 --base develop
uv run scripts/pr_create.py owner/repo --title "WIP" --head wip-branch --draft
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `--title`, `-t` | PR title (required) |
| `--head`, `-h` | Branch containing changes (required) |
| `--base`, `-b` | Branch to merge into (default: default branch) |
| `--body` | PR description |
| `--draft`, `-d` | Create as draft PR |
| `--json`, `-j` | Output as JSON |

---

### Get Pull Request Details (`scripts/pr_get.py`)

```bash
uv run scripts/pr_get.py owner/repo 123
uv run scripts/pr_get.py owner/repo 123 --json
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `pr_number` | Pull request number (required) |
| `--json`, `-j` | Output as JSON |

---

### Merge Pull Request (`scripts/pr_merge.py`)

```bash
uv run scripts/pr_merge.py owner/repo 123
uv run scripts/pr_merge.py owner/repo 123 --method squash
uv run scripts/pr_merge.py owner/repo 123 --method rebase
uv run scripts/pr_merge.py owner/repo 123 --method squash --title "Feature (#123)"
```

| Argument | Description |
|----------|-------------|
| `repo` | Repository in owner/repo format (required) |
| `pr_number` | Pull request number (required) |
| `--method`, `-m` | Merge method: merge, squash, rebase |
| `--title`, `-t` | Custom commit title |
| `--message` | Custom commit message body |
| `--sha` | Expected head SHA (for safety) |
| `--json`, `-j` | Output as JSON |

---

## Common Patterns

### Setting Credentials

```bash
# Set for current session
export GITHUB_TOKEN="ghp_xxxxxxxxxxxx"

# Or inline with command
GITHUB_TOKEN="ghp_xxx" uv run scripts/repo_list.py
```

### Updating a File (Full Workflow)

To update a file, you need its current SHA:

```bash
# 1. Get the current file with its SHA
uv run scripts/repo_contents.py owner/repo --path README.md --json > file_info.json

# 2. Extract the SHA
SHA=$(jq -r '.sha' file_info.json)

# 3. Update the file with the SHA
uv run scripts/file_write.py owner/repo \
    --path README.md \
    --content "New content here" \
    --message "Update README" \
    --sha "$SHA"
```

### Creating and Merging a PR (Full Workflow)

```bash
# 1. Create a branch
uv run scripts/branch_create.py owner/repo --name feature/my-feature

# 2. Make changes (push commits via git or file_write.py)
uv run scripts/file_write.py owner/repo \
    --path feature.py \
    --content "# New feature" \
    --message "Add feature" \
    --branch feature/my-feature

# 3. Create a PR
uv run scripts/pr_create.py owner/repo \
    --title "Add my feature" \
    --head feature/my-feature \
    --body "This PR adds..."

# 4. Merge the PR
uv run scripts/pr_merge.py owner/repo 123 --method squash

# 5. Delete the branch
uv run scripts/branch_delete.py owner/repo --name feature/my-feature --force
```

### JSON Output for Processing

All scripts support `--json` for machine-readable output:

```bash
# List repos and filter with jq
uv run scripts/repo_list.py --json | jq '.[] | select(.language == "Python")'

# Get commit count
uv run scripts/commit_list.py owner/repo --json | jq 'length'

# Get all open PR numbers
uv run scripts/pr_list.py owner/repo --json | jq '.[].number'
```

## Error Handling

Scripts exit with non-zero status on errors. Common issues:

- **401 Unauthorized**: Check that `GITHUB_TOKEN` is set and valid
- **403 Forbidden**: Token lacks required scopes, or rate limit exceeded
- **404 Not Found**: Repository, file, or branch doesn't exist (or token lacks access)
- **409 Conflict**: SHA mismatch when updating (file was modified since you read it)
- **422 Validation Failed**: Invalid input (check branch name format, file path, etc.)

## Rate Limits

The GitHub API has rate limits:
- Authenticated requests: 5,000 per hour
- Search API: 30 per minute

The skill includes automatic retry logic with exponential backoff for rate limit errors.

Check your current limits as follows:

```bash
curl -H "Authorization: token $GITHUB_TOKEN" https://api.github.com/rate_limit
```
