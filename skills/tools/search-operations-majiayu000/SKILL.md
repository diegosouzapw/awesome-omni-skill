---
name: search-operations
description: Search GitHub - find code, issues, users, and repositories across GitHub using gh CLI
---
# GitHub Search Operations Skill

This skill provides comprehensive search capabilities across GitHub including searching for code, issues, pull requests, repositories, and users.

## Available Operations

### 1. Search Code
Search for code across GitHub repositories.

### 2. Search Issues and Pull Requests
Search for issues and PRs with advanced filters.

### 3. Search Repositories
Find repositories matching specific criteria.

### 4. Search Users
Find GitHub users and organizations.

## Usage Examples

### Search Code

**Basic code search:**
```bash
gh search code "function authenticate" --limit 20
```

**Search in specific language:**
```bash
gh search code "async function" --language javascript --limit 30
```

**Search in specific repository:**
```bash
gh search code "TODO" --repo owner/repo-name
```

**Search in organization:**
```bash
gh search code "API_KEY" --owner myorg
```

**Search with extension filter:**
```bash
gh search code "class.*Component" --extension tsx
```

**Search by file path:**
```bash
gh search code "config" --path "src/config"
```

**Complex code search:**
```bash
gh search code "useState" --language typescript --extension tsx --owner facebook
```

**Search with size filter:**
```bash
gh search code "import React" --size ">1000"
```

**JSON output for processing:**
```bash
gh search code "security vulnerability" --json path,repository,url --jq '.[] | {file: .path, repo: .repository.fullName}'
```

### Search Issues and Pull Requests

**Search all issues:**
```bash
gh search issues "memory leak" --limit 30
```

**Search in specific repository:**
```bash
gh search issues "bug" --repo owner/repo-name
```

**Search open issues only:**
```bash
gh search issues "crash" --state open
```

**Search closed issues:**
```bash
gh search issues "fixed" --state closed
```

**Search by label:**
```bash
gh search issues "bug" --label "critical"
```

**Multiple labels:**
```bash
gh search issues "" --label "bug" --label "security"
```

**Search by author:**
```bash
gh search issues "feature" --author username
```

**Search by assignee:**
```bash
gh search issues "" --assignee username
```

**Search in organization:**
```bash
gh search issues "todo" --owner myorg
```

**Search by date:**
```bash
gh search issues "bug" --created ">2025-01-01"
gh search issues "feature" --updated "<2025-01-01"
```

**Search by comments count:**
```bash
gh search issues "help wanted" --comments ">10"
```

**Search PRs only:**
```bash
gh search prs "feat:" --state open --limit 20
```

**Complex issue search:**
```bash
gh search issues "is:open is:issue label:bug assignee:@me"
```

**Search with reactions:**
```bash
gh search issues "good first issue" --reactions ">5"
```

**JSON output:**
```bash
gh search issues "security" --json number,title,state,url --jq '.[] | "\(.number): \(.title)"'
```

### Search Repositories

**Basic repository search:**
```bash
gh search repos "machine learning" --limit 20
```

**Search by language:**
```bash
gh search repos "web framework" --language python
```

**Search by stars:**
```bash
gh search repos "react" --stars ">10000"
```

**Search by forks:**
```bash
gh search repos "kubernetes" --forks ">1000"
```

**Search by size (KB):**
```bash
gh search repos "starter template" --size "<1000"
```

**Search in organization:**
```bash
gh search repos "" --owner microsoft
```

**Search by topic:**
```bash
gh search repos "topic:docker topic:kubernetes"
```

**Search archived repositories:**
```bash
gh search repos "old-project" --archived
```

**Search by license:**
```bash
gh search repos "utility library" --license mit
```

**Search by creation date:**
```bash
gh search repos "created:>2025-01-01"
```

**Search by last update:**
```bash
gh search repos "updated:>2025-01-01" --stars ">100"
```

**Search by number of followers:**
```bash
gh search repos "followers:>1000"
```

**Complex repository search:**
```bash
gh search repos "stars:>5000 language:python topic:machine-learning"
```

**JSON output:**
```bash
gh search repos "awesome" --json name,owner,stars,url --jq '.[] | "\(.owner.login)/\(.name) - \(.stars) stars"'
```

### Search Users

**Basic user search:**
```bash
gh search users "john doe" --limit 10
```

**Search by location:**
```bash
gh search users "location:seattle"
```

**Search by language:**
```bash
gh search users "language:python"
```

**Search by followers:**
```bash
gh search users "followers:>1000"
```

**Search by repositories:**
```bash
gh search users "repos:>50"
```

**Search organizations:**
```bash
gh search users "type:org"
```

**Search individual users:**
```bash
gh search users "type:user"
```

**Search by creation date:**
```bash
gh search users "created:>2020-01-01"
```

**Complex user search:**
```bash
gh search users "location:california language:javascript followers:>100"
```

**JSON output:**
```bash
gh search users "location:london" --json login,name,url --jq '.[] | "\(.login): \(.name)"'
```

## Advanced Search Queries

### GitHub Search Syntax

**Boolean operators:**
```bash
# AND (default)
gh search code "class AND function"

# OR
gh search code "bug OR error"

# NOT
gh search code "NOT deprecated"
```

**Exact phrase matching:**
```bash
gh search code '"exact phrase here"'
```

**Wildcards:**
```bash
gh search repos "test*"
```

**Range queries:**
```bash
gh search repos "stars:10..50"
gh search issues "created:2025-01-01..2025-12-31"
```

**Qualifiers:**
```bash
# Search code
gh search code "repo:owner/name path:src/ language:js"

# Search issues
gh search issues "is:open is:issue label:bug assignee:@me"

# Search PRs
gh search prs "is:pr is:open review:approved"

# Search repos
gh search repos "stars:>1000 forks:>500 language:python"
```

## Common Patterns

### Find Vulnerable Code

```bash
# Search for exposed API keys
gh search code "API_KEY" --language javascript

# Search for TODO security items
gh search code "TODO.*security" --owner myorg

# Find hardcoded passwords
gh search code "password.*=.*['\"]" --language python

# Search for SQL injection vulnerabilities
gh search code "query.*=.*+.*req\." --language javascript
```

### Find Examples and Documentation

```bash
# Find usage examples
gh search code "import MyLibrary" --language typescript

# Find configuration examples
gh search code "filename:config.yml" --owner popular-org

# Find test examples
gh search code "describe.*test" --path "test/" --language javascript
```

### Project Discovery

```bash
# Find active projects
gh search repos "stars:>100 pushed:>2025-01-01" --language rust

# Find good first issues
gh search issues "label:good-first-issue state:open"

# Find projects needing help
gh search issues "label:help-wanted state:open" --sort comments

# Find trending repositories
gh search repos "created:>2025-01-01" --sort stars
```

### Competitive Analysis

```bash
# Find similar projects
gh search repos "web framework language:python stars:>1000"

# Find what competitors are building
gh search repos "owner:competitor-org"

# Track competitor issues
gh search issues "org:competitor-org is:open"

# See popular forks
gh search repos "fork:true" --sort stars
```

### Team Collaboration

```bash
# Find your open issues
gh search issues "assignee:@me state:open"

# Find PRs needing review
gh search prs "review-requested:@me state:open"

# Find team's open PRs
gh search prs "team:myorg/myteam state:open"

# Find stale issues
gh search issues "updated:<2024-01-01 state:open"
```

## Sorting and Filtering

### Sort Options

**For repositories:**
- `--sort stars`: Sort by star count
- `--sort forks`: Sort by fork count
- `--sort updated`: Sort by last update
- `--sort help-wanted-issues`: Sort by help wanted issues

```bash
gh search repos "javascript framework" --sort stars --order desc
```

**For issues:**
- `--sort comments`: Sort by comment count
- `--sort created`: Sort by creation date
- `--sort updated`: Sort by update date
- `--sort reactions`: Sort by reaction count

```bash
gh search issues "bug" --sort comments --order desc
```

**For code:**
- `--sort indexed`: Sort by when indexed (default)

### Pagination

**Limit results:**
```bash
gh search repos "python" --limit 100
```

**Get more results (max 100):**
```bash
gh search code "function" --limit 100
```

## Output Formats

### JSON Output

**Basic JSON:**
```bash
gh search repos "docker" --json name,description,stars
```

**Pipe to jq for processing:**
```bash
gh search repos "kubernetes" --json name,stars,url \
  | jq '.[] | select(.stars > 1000) | {name, stars}'
```

**Extract specific fields:**
```bash
gh search issues "bug" --json number,title,state --jq '.[] | "\(.number): \(.title) [\(.state)]"'
```

## Error Handling

### Rate Limiting
```bash
# Check rate limit status
gh api rate_limit

# If rate limited, wait or authenticate
gh auth login
```

### No Results Found
```bash
# Verify query syntax
gh search repos "test" --limit 1

# Try broader query
gh search repos "test*" --limit 10
```

### Search Timeout
```bash
# Try more specific query
gh search code "function" --repo owner/specific-repo

# Use fewer qualifiers
```

## Best Practices

1. **Be specific**: Use multiple qualifiers to narrow results
2. **Use quotes**: For exact phrase matching
3. **Filter early**: Apply language, repo, or org filters
4. **Limit results**: Use `--limit` to get manageable result sets
5. **Save queries**: Document frequently used search queries
6. **Use JSON output**: For programmatic processing
7. **Check rate limits**: Monitor API usage
8. **Sort appropriately**: Use `--sort` to find most relevant results
9. **Combine with other tools**: Pipe to jq, grep, etc.
10. **Test queries**: Start broad, then refine

## Integration with Other Skills

- Use `repository-management` after finding repos to clone or fork
- Use `issue-management` to work with found issues
- Use `pull-request-management` for found PRs
- Use `code-review` to review found code

## References

- [GitHub CLI Search Documentation](https://cli.github.com/manual/gh_search)
- [GitHub Search Syntax](https://docs.github.com/en/search-github/getting-started-with-searching-on-github/understanding-the-search-syntax)
- [Searching Code](https://docs.github.com/en/search-github/searching-on-github/searching-code)
- [Searching Issues and PRs](https://docs.github.com/en/search-github/searching-on-github/searching-issues-and-pull-requests)
- [Searching Repositories](https://docs.github.com/en/search-github/searching-on-github/searching-for-repositories)
- [Searching Users](https://docs.github.com/en/search-github/searching-on-github/searching-users)
