---
name: github-repo-analysis
description: Analyze GitHub repositories to extract insights about commit frequency, outstanding contributors, release timeline, and project health metrics. Use when users request repository analysis, commit history investigation, contributor identification, release tracking, or development activity assessment for any GitHub project.
---

# GitHub Repository Analysis

This skill guides analysis of GitHub repositories to extract meaningful insights about development activity, contributor patterns, and release cycles.

## Core Analysis Capabilities

### Commit Frequency Analysis
- Extract commit history via GitHub API or git commands
- Calculate commit frequency by time period (daily, weekly, monthly)
- Identify patterns in development activity
- Detect periods of high/low activity
- Generate time-series visualizations of commit trends

### Outstanding Contributors Analysis
- Identify top contributors by commit count
- Calculate contribution distribution (percentage per contributor)
- Analyze commit patterns by contributor
- Track first-time vs. recurring contributors
- Generate contributor leaderboards

### Release Timeline Analysis
- Extract release/tag history
- Calculate time between releases
- Identify release patterns and cycles
- Track version numbering schemes
- Map releases to major commit periods

## Implementation Approaches

### Approach 1: GitHub API (Recommended for Public Repos)

Use GitHub's REST or GraphQL API for efficient data retrieval:

```python
import requests
from datetime import datetime

def analyze_commits(owner, repo, token=None):
    headers = {'Authorization': f'token {token}'} if token else {}
    url = f'https://api.github.com/repos/{owner}/{repo}/commits'
    
    all_commits = []
    page = 1
    
    while True:
        response = requests.get(url, headers=headers, params={'page': page, 'per_page': 100})
        commits = response.json()
        if not commits:
            break
        all_commits.extend(commits)
        page += 1
    
    return all_commits

def analyze_contributors(commits):
    contributor_stats = {}
    for commit in commits:
        author = commit['commit']['author']['name']
        contributor_stats[author] = contributor_stats.get(author, 0) + 1
    
    return sorted(contributor_stats.items(), key=lambda x: x[1], reverse=True)

def analyze_releases(owner, repo, token=None):
    headers = {'Authorization': f'token {token}'} if token else {}
    url = f'https://api.github.com/repos/{owner}/{repo}/releases'
    response = requests.get(url, headers=headers)
    return response.json()
```

**Benefits:**
- No repository cloning needed
- Efficient pagination
- Access to additional metadata
- Rate limiting: 60 requests/hour without auth, 5000 with token

### Approach 2: Local Git Repository

Use git commands for detailed analysis when repository is already cloned:

```bash
# Get commit history with timestamps
git log --pretty=format:"%h|%an|%ae|%ad|%s" --date=iso > commits.txt

# Count commits by author
git shortlog -sn --all

# Get all tags/releases
git tag -l --sort=-version:refname

# Commit frequency by week
git log --pretty=format:"%ad" --date=short | awk '{print $1}' | uniq -c

# Commits per month
git log --pretty=format:"%ad" --date=format:"%Y-%m" | sort | uniq -c
```

### Approach 3: Hybrid Approach

Combine both methods for comprehensive analysis:
1. Use API for releases and high-level stats
2. Clone repository for detailed commit analysis
3. Use git commands for advanced filtering

## Analysis Workflow

1. **Repository Identification**
   - Parse GitHub URL or accept owner/repo parameters
   - Validate repository exists and is accessible

2. **Data Collection**
   - Fetch commit history (API or git log)
   - Retrieve release/tag information
   - Collect contributor metadata

3. **Data Processing**
   - Parse timestamps and author information
   - Group commits by time periods
   - Calculate statistics and metrics

4. **Insight Generation**
   - Identify top contributors with percentages
   - Calculate commit frequency trends
   - Map release timeline with intervals
   - Detect anomalies or interesting patterns

5. **Visualization & Reporting**
   - Create charts/graphs for trends
   - Generate summary statistics
   - Present findings in structured format

## Key Metrics to Calculate

### Commit Metrics
- Total commits
- Commits per day/week/month
- Average commits per active period
- Longest streak of daily commits
- Periods of inactivity

### Contributor Metrics
- Total unique contributors
- Top N contributors (typically top 5-10)
- Contribution percentage per contributor
- One-time vs. recurring contributors
- New contributors over time

### Release Metrics
- Total releases
- Time between releases (min, max, average)
- Release frequency trend
- Semantic versioning patterns
- Pre-release vs. stable releases

## Output Formats

### Summary Report
```
Repository: owner/repo
Analysis Period: YYYY-MM-DD to YYYY-MM-DD

Commit Activity:
- Total Commits: N
- Active Contributors: N
- Average Commits/Week: N
- Most Active Period: YYYY-MM

Top Contributors:
1. Name (N commits, X%)
2. Name (N commits, X%)
...

Recent Releases:
- v1.2.3 (YYYY-MM-DD) - N days since previous
- v1.2.2 (YYYY-MM-DD) - N days since previous
...
```

### Detailed Analytics
- Time-series data in CSV/JSON format
- Visualization-ready datasets
- Contributor breakdown by time period
- Release calendar with annotations

## Common Patterns & Tips

**Handle rate limiting:** Always check API rate limit headers and implement exponential backoff

**Large repositories:** For repos with 10k+ commits, consider:
- Analyzing recent history only (e.g., last 12 months)
- Sampling commits rather than processing all
- Using shallow clones for git-based analysis

**Privacy considerations:** GitHub API exposes public data only; private repos require authentication

**Timezone handling:** Normalize all timestamps to UTC for consistent analysis

**Bot commits:** Filter out automated commits (dependabot, renovate) for human contributor analysis

**Email normalization:** Same contributor may use different email addresses; consider consolidation

## Error Handling

- Repository not found: Verify owner/repo spelling
- Rate limit exceeded: Implement retry logic or use authentication
- Empty history: Check if repository has been initialized
- API changes: GitHub API versioning may affect endpoints
