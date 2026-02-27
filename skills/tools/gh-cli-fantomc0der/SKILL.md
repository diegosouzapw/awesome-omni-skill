---
name: gh-cli
description: GitHub CLI (gh) reference for repositories, pull requests, Actions, organizations, and search from the command line.
---

# GitHub CLI (gh)

Comprehensive reference for GitHub CLI (gh) - work seamlessly with GitHub from the command line.

## CLI Structure

```
gh                          # Root command
├── auth                    # Authentication
│   ├── login
│   ├── logout
│   ├── refresh
│   ├── setup-git
│   ├── status
│   ├── switch
│   └── token
├── org                     # Organizations
│   └── list
├── pr                      # Pull Requests
│   ├── create
│   ├── list
│   ├── status
│   ├── checkout
│   ├── checks
│   ├── close
│   ├── comment
│   ├── diff
│   ├── edit
│   ├── lock
│   ├── merge
│   ├── ready
│   ├── reopen
│   ├── revert
│   ├── review
│   ├── unlock
│   ├── update-branch
│   └── view
├── repo                    # Repositories
│   ├── create
│   ├── list
│   ├── archive
│   ├── autolink
│   ├── clone
│   ├── delete
│   ├── deploy-key
│   ├── edit
│   ├── fork
│   ├── gitignore
│   ├── license
│   ├── rename
│   ├── set-default
│   ├── sync
│   ├── unarchive
│   └── view
├── cache                   # Actions caches
│   ├── delete
│   └── list
├── run                     # Workflow runs
│   ├── cancel
│   ├── delete
│   ├── download
│   ├── list
│   ├── rerun
│   ├── view
│   └── watch
├── workflow                # Workflows
│   ├── disable
│   ├── enable
│   ├── list
│   ├── run
│   └── view
├── api                     # API requests
├── ruleset                 # Rulesets
│   ├── check
│   ├── list
│   └── view
├── search                  # Search
│   ├── code
│   ├── commits
│   ├── issues
│   ├── prs
│   └── repos
└── status                  # Status overview
```

## Authentication (gh auth)

### Status

```bash
# Show all authentication status
gh auth status

# Show active account only
gh auth status --active

# Show specific hostname
gh auth status --hostname github.com

# Show token in output
gh auth status --show-token

# JSON output
gh auth status --json hosts

# Filter with jq
gh auth status --json hosts --jq '.hosts | add'
```

## Repositories (gh repo)

### Create Repository

```bash
# Create new repository
gh repo create my-repo

# Create with description
gh repo create my-repo --description "My awesome project"

# Create public repository
gh repo create my-repo --public

# Create private repository
gh repo create my-repo --private

# Create with homepage
gh repo create my-repo --homepage https://example.com

# Create with license
gh repo create my-repo --license mit

# Create with gitignore
gh repo create my-repo --gitignore python

# Initialize as template repository
gh repo create my-repo --template

# Create repository in organization
gh repo create org/my-repo

# Create without cloning locally
gh repo create my-repo --source=.

# Disable issues
gh repo create my-repo --disable-issues

# Disable wiki
gh repo create my-repo --disable-wiki
```

### Clone Repository

```bash
# Clone repository
gh repo clone owner/repo

# Clone to specific directory
gh repo clone owner/repo my-directory

# Clone with different branch
gh repo clone owner/repo --branch develop
```

### List Repositories

```bash
# List all repositories
gh repo list

# List repositories for owner
gh repo list owner

# Limit results
gh repo list --limit 50

# Public repositories only
gh repo list --public

# Source repositories only (not forks)
gh repo list --source

# JSON output
gh repo list --json name,visibility,owner

# Table output
gh repo list --limit 100 | tail -n +2

# Filter with jq
gh repo list --json name --jq '.[].name'
```

### View Repository

```bash
# View repository details
gh repo view

# View specific repository
gh repo view owner/repo

# JSON output
gh repo view --json name,description,defaultBranchRef

# View in browser
gh repo view --web
```

### Edit Repository

```bash
# Edit description
gh repo edit --description "New description"

# Set homepage
gh repo edit --homepage https://example.com

# Change visibility
gh repo edit --visibility private
gh repo edit --visibility public

# Enable/disable features
gh repo edit --enable-issues
gh repo edit --disable-issues
gh repo edit --enable-wiki
gh repo edit --disable-wiki
gh repo edit --enable-projects
gh repo edit --disable-projects

# Set default branch
gh repo edit --default-branch main

# Rename repository
gh repo rename new-name

# Archive repository
gh repo archive
gh repo unarchive
```

### Delete Repository

```bash
# Delete repository
gh repo delete owner/repo

# Confirm without prompt
gh repo delete owner/repo --yes
```

### Fork Repository

```bash
# Fork repository
gh repo fork owner/repo

# Fork to organization
gh repo fork owner/repo --org org-name

# Clone after forking
gh repo fork owner/repo --clone

# Remote name for fork
gh repo fork owner/repo --remote-name upstream
```

### Sync Fork

```bash
# Sync fork with upstream
gh repo sync

# Sync specific branch
gh repo sync --branch feature

# Force sync
gh repo sync --force
```

### Set Default Repository

```bash
# Set default repository for current directory
gh repo set-default

# Set default explicitly
gh repo set-default owner/repo

# Unset default
gh repo set-default --unset
```

### Repository Autolinks

```bash
# List autolinks
gh repo autolink list

# Add autolink
gh repo autolink add \
  --key-prefix JIRA- \
  --url-template https://jira.example.com/browse/<num>

# Delete autolink
gh repo autolink delete 12345
```

### Repository Deploy Keys

```bash
# List deploy keys
gh repo deploy-key list

# Add deploy key
gh repo deploy-key add ~/.ssh/id_rsa.pub \
  --title "Production server" \
  --read-only

# Delete deploy key
gh repo deploy-key delete 12345
```

### Gitignore and License

```bash
# View gitignore template
gh repo gitignore

# View license template
gh repo license mit

# License with full name
gh repo license mit --fullname "John Doe"
```

## Pull Requests (gh pr)

### Create Pull Request

```bash
# Create PR interactively
gh pr create

# Create with title
gh pr create --title "Feature: Add new functionality"

# Create with title and body
gh pr create \
  --title "Feature: Add new functionality" \
  --body "This PR adds..."

# Fill body from template
gh pr create --body-file .github/PULL_REQUEST_TEMPLATE.md

# Set base branch
gh pr create --base main

# Set head branch (default: current branch)
gh pr create --head feature-branch

# Create draft PR
gh pr create --draft

# Add assignees
gh pr create --assignee user1,user2

# Add reviewers
gh pr create --reviewer user1,user2

# Add labels
gh pr create --labels enhancement,feature

# Link to issue
gh pr create --issue 123

# Create in specific repository
gh pr create --repo owner/repo

# Open in browser after creation
gh pr create --web
```

### List Pull Requests

```bash
# List open PRs
gh pr list

# List all PRs
gh pr list --state all

# List merged PRs
gh pr list --state merged

# List closed (not merged) PRs
gh pr list --state closed

# Filter by head branch
gh pr list --head feature-branch

# Filter by base branch
gh pr list --base main

# Filter by author
gh pr list --author username
gh pr list --author @me

# Filter by assignee
gh pr list --assignee username

# Filter by labels
gh pr list --labels bug,enhancement

# Limit results
gh pr list --limit 50

# Search
gh pr list --search "is:open is:pr label:review-required"

# JSON output
gh pr list --json number,title,state,author,headRefName

# Show check status
gh pr list --json number,title,statusCheckRollup --jq '.[] | [.number, .title, .statusCheckRollup[]?.status]'

# Sort by
gh pr list --sort created --order desc
```

### View Pull Request

```bash
# View PR
gh pr view 123

# View with comments
gh pr view 123 --comments

# View in browser
gh pr view 123 --web

# JSON output
gh pr view 123 --json title,body,state,author,commits,files

# View diff
gh pr view 123 --json files --jq '.files[].path'

# View with jq query
gh pr view 123 --json title,state --jq '"\(.title): \(.state)"'
```

### Checkout Pull Request

```bash
# Checkout PR branch
gh pr checkout 123

# Checkout with specific branch name
gh pr checkout 123 --branch name-123

# Force checkout
gh pr checkout 123 --force
```

### Diff Pull Request

```bash
# View PR diff
gh pr diff 123

# View diff with color
gh pr diff 123 --color always

# Output to file
gh pr diff 123 > pr-123.patch

# View diff of specific files
gh pr diff 123 --name-only
```

### Merge Pull Request

```bash
# Merge PR
gh pr merge 123

# Merge with specific method
gh pr merge 123 --merge
gh pr merge 123 --squash
gh pr merge 123 --rebase

# Delete branch after merge
gh pr merge 123 --delete-branch

# Merge with comment
gh pr merge 123 --subject "Merge PR #123" --body "Merging feature"

# Merge draft PR
gh pr merge 123 --admin

# Force merge (skip checks)
gh pr merge 123 --admin
```

### Close Pull Request

```bash
# Close PR (as draft, not merge)
gh pr close 123

# Close with comment
gh pr close 123 --comment "Closing due to..."
```

### Reopen Pull Request

```bash
# Reopen closed PR
gh pr reopen 123
```

### Edit Pull Request

```bash
# Edit interactively
gh pr edit 123

# Edit title
gh pr edit 123 --title "New title"

# Edit body
gh pr edit 123 --body "New description"

# Add labels
gh pr edit 123 --add-label bug,enhancement

# Remove labels
gh pr edit 123 --remove-label stale

# Add assignees
gh pr edit 123 --add-assignee user1,user2

# Remove assignees
gh pr edit 123 --remove-assignee user1

# Add reviewers
gh pr edit 123 --add-reviewer user1,user2

# Remove reviewers
gh pr edit 123 --remove-reviewer user1

# Mark as ready for review
gh pr edit 123 --ready
```

### Ready for Review

```bash
# Mark draft PR as ready
gh pr ready 123
```

### Pull Request Checks

```bash
# View PR checks
gh pr checks 123

# Watch checks in real-time
gh pr checks 123 --watch

# Watch interval (seconds)
gh pr checks 123 --watch --interval 5
```

### Comment on Pull Request

```bash
# Add comment
gh pr comment 123 --body "Looks good!"

# Comment on specific line
gh pr comment 123 --body "Fix this" \
  --repo owner/repo \
  --head-owner owner --head-branch feature

# Edit comment
gh pr comment 123 --edit 456789 --body "Updated"

# Delete comment
gh pr comment 123 --delete 456789
```

### Review Pull Request

```bash
# Review PR (opens editor)
gh pr review 123

# Approve PR
gh pr review 123 --approve

--approve-body "LGTM!"

# Request changes
gh pr review 123 --request-changes \
  --body "Please fix these issues"

# Comment on PR
gh pr review 123 --comment --body "Some thoughts..."

# Dismiss review
gh pr review 123 --dismiss
```

### Update Branch

```bash
# Update PR branch with latest base branch
gh pr update-branch 123

# Force update
gh pr update-branch 123 --force

# Use merge strategy
gh pr update-branch 123 --merge
```

### Lock/Unlock Pull Request

```bash
# Lock PR conversation
gh pr lock 123

# Lock with reason
gh pr lock 123 --reason off-topic

# Unlock
gh pr unlock 123
```

### Revert Pull Request

```bash
# Revert merged PR
gh pr revert 123

# Revert with specific branch name
gh pr revert 123 --branch revert-pr-123
```

### Pull Request Status

```bash
# Show PR status summary
gh pr status

# Status for specific repository
gh pr status --repo owner/repo
```

## GitHub Actions

### Workflow Runs (gh run)

```bash
# List workflow runs
gh run list

# List for specific workflow
gh run list --workflow "ci.yml"

# List for specific branch
gh run list --branch main

# Limit results
gh run list --limit 20

# JSON output
gh run list --json databaseId,status,conclusion,headBranch

# View run details
gh run view 123456789

# View run with verbose logs
gh run view 123456789 --log

# View specific job
gh run view 123456789 --job 987654321

# View in browser
gh run view 123456789 --web

# Watch run in real-time
gh run watch 123456789

# Watch with interval
gh run watch 123456789 --interval 5

# Rerun failed run
gh run rerun 123456789

# Rerun specific job
gh run rerun 123456789 --job 987654321

# Cancel run
gh run cancel 123456789

# Delete run
gh run delete 123456789

# Download run artifacts
gh run download 123456789

# Download specific artifact
gh run download 123456789 --name build

# Download to directory
gh run download 123456789 --dir ./artifacts
```

### Workflows (gh workflow)

```bash
# List workflows
gh workflow list

# View workflow details
gh workflow view ci.yml

# View workflow YAML
gh workflow view ci.yml --yaml

# View in browser
gh workflow view ci.yml --web

# Enable workflow
gh workflow enable ci.yml

# Disable workflow
gh workflow disable ci.yml

# Run workflow manually
gh workflow run ci.yml

# Run with inputs
gh workflow run ci.yml \
  --raw-field \
  version="1.0.0" \
  environment="production"

# Run from specific branch
gh workflow run ci.yml --ref develop
```

### Action Caches (gh cache)

```bash
# List caches
gh cache list

# List for specific branch
gh cache list --branch main

# List with limit
gh cache list --limit 50

# Delete cache
gh cache delete 123456789

# Delete all caches
gh cache delete --all
```

## Organizations (gh org)

```bash
# List organizations
gh org list

# List for user
gh org list --user username

# JSON output
gh org list --json login,name,description

# View organization
gh org view orgname

# View organization members
gh org view orgname --json members --jq '.members[] | .login'
```

## Search (gh search)

```bash
# Search code
gh search code "TODO"

# Search in specific repository
gh search code "TODO" --repo owner/repo

# Search commits
gh search commits "fix bug"

# Search issues
gh search issues "label:bug state:open"

# Search PRs
gh search prs "is:open is:pr review:required"

# Search repositories
gh search repos "stars:>1000 language:python"

# Limit results
gh search repos "topic:api" --limit 50

# JSON output
gh search repos "stars:>100" --json name,description,stargazers

# Order results
gh search repos "language:rust" --order desc --sort stars

# Search with extensions
gh search code "import" --extension py

# Web search (open in browser)
gh search prs "is:open" --web
```

## Status (gh status)

```bash
# Show status overview
gh status

# Status for specific repositories
gh status --repo owner/repo

# JSON output
gh status --json
```

## Rulesets (gh ruleset)

```bash
# List rulesets
gh ruleset list

# View ruleset
gh ruleset view 123

# Check ruleset
gh ruleset check --branch feature

# Check specific repository
gh ruleset check --repo owner/repo --branch main
```

## Global Flags

| Flag                       | Description                            |
| -------------------------- | -------------------------------------- |
| `--help` / `-h`            | Show help for command                  |
| `--version`                | Show gh version                        |
| `--repo [HOST/]OWNER/REPO` | Select another repository              |
| `--hostname HOST`          | GitHub hostname                        |
| `--jq EXPRESSION`          | Filter JSON output                     |
| `--json FIELDS`            | Output JSON with specified fields      |
| `--template STRING`        | Format JSON using Go template          |
| `--web`                    | Open in browser                        |
| `--paginate`               | Make additional API calls              |
| `--verbose`                | Show verbose output                    |
| `--debug`                  | Show debug output                      |
| `--timeout SECONDS`        | Maximum API request duration           |
| `--cache CACHE`            | Cache control (default, force, bypass) |

## Output Formatting

### JSON Output

```bash
# Basic JSON
gh repo view --json name,description

# Nested fields
gh repo view --json owner,name --jq '.owner.login + "/" + .name'

# Array operations
gh pr list --json number,title --jq '.[] | select(.number > 100)'

# Complex queries
gh issue list --json number,title,labels \
  --jq '.[] | {number, title: .title, tags: [.labels[].name]}'
```

### Template Output

```bash
# Custom template
gh repo view \
  --template '{{.name}}: {{.description}}'

# Multiline template
gh pr view 123 \
  --template 'Title: {{.title}}
Author: {{.author.login}}
State: {{.state}}
'
```

## Common Workflows

### Create PR from Issue

```bash
# Create branch from issue
gh issue develop 123 --branch feature/issue-123

# Make changes, commit, push
git add .
git commit -m "Fix issue #123"
git push

# Create PR linking to issue
gh pr create --title "Fix #123" --body "Closes #123"
```

### Bulk Operations

```bash
# Close multiple issues
gh issue list --search "label:stale" \
  --json number \
  --jq '.[].number' | \
  xargs -I {} gh issue close {} --comment "Closing as stale"

# Add label to multiple PRs
gh pr list --search "review:required" \
  --json number \
  --jq '.[].number' | \
  xargs -I {} gh pr edit {} --add-label needs-review
```

### Repository Setup Workflow

```bash
# Create repository with initial setup
gh repo create my-project --public \
  --description "My awesome project" \
  --clone \
  --gitignore python \
  --license mit

cd my-project

# Set up branches
git checkout -b develop
git push -u origin develop

# Create labels
gh label create bug --color "d73a4a" --description "Bug report"
gh label create enhancement --color "a2eeef" --description "Feature request"
gh label create documentation --color "0075ca" --description "Documentation"
```

### CI/CD Workflow

```bash
# Run workflow and wait
RUN_ID=$(gh workflow run ci.yml --ref main --jq '.databaseId')

# Watch the run
gh run watch "$RUN_ID"

# Download artifacts on completion
gh run download "$RUN_ID" --dir ./artifacts
```

### Fork Sync Workflow

```bash
# Fork repository
gh repo fork original/repo --clone

cd repo

# Add upstream remote
git remote add upstream https://github.com/original/repo.git

# Sync fork
gh repo sync

# Or manual sync
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

## Environment Setup

### Shell Integration

```bash
# Add to ~/.bashrc or ~/.zshrc
eval "$(gh completion -s bash)"  # or zsh/fish

# Create useful aliases
alias gs='gh status'
alias gpr='gh pr view --web'
alias gir='gh issue view --web'
alias gco='gh pr checkout'
```

### Git Configuration

```bash
# Use gh as credential helper
gh auth setup-git

# Set gh as default for repo operations
git config --global credential.helper 'gh !gh auth setup-git'

# Or manually
git config --global credential.helper github
```

## Best Practices

1. **Authentication**: Use environment variables for automation

   ```bash
   export GH_TOKEN=$(gh auth token)
   ```

2. **Default Repository**: Set default to avoid repetition

   ```bash
   gh repo set-default owner/repo
   ```

3. **JSON Parsing**: Use jq for complex data extraction

   ```bash
   gh pr list --json number,title --jq '.[] | select(.title | contains("fix"))'
   ```

4. **Pagination**: Use --paginate for large result sets

   ```bash
   gh issue list --state all --paginate
   ```

5. **Caching**: Use cache control for frequently accessed data
   ```bash
   gh api /user --cache force
   ```

## Getting Help

```bash
# General help
gh --help

# Command help
gh pr --help
gh issue create --help

# Help topics
gh help formatting
gh help environment
gh help exit-codes
gh help accessibility
```

## References

- Official Manual: https://cli.github.com/manual/
- GitHub Docs: https://docs.github.com/en/github-cli
- REST API: https://docs.github.com/en/rest
- GraphQL API: https://docs.github.com/en/graphql