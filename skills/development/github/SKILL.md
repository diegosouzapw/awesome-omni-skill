---
name: github
description: "Interact with GitHub repositories using the GitHub API"
metadata:
  emoji: "🐙"
  requires:
    optional: true
  tags: ["github", "git", "repository", "code"]
---

# GitHub Skill

Interact with GitHub repositories using the GitHub API. No authentication required for public repositories (60 requests/hour), but you can add a token in config for higher limits (5000 requests/hour).

## Available Tools

You have access to the following GitHub tools:

### Search Repositories
```javascript
github_search_repos({
  query: "language:javascript stars:>1000",
  sort: "stars",  // optional: "stars", "forks", "updated"
  limit: 10       // optional: max 30
})
```

### Get Repository Information
```javascript
github_get_repo({
  owner: "HKUDS",
  repo: "nanobot"
})
```

### List Issues
```javascript
github_list_issues({
  owner: "HKUDS",
  repo: "nanobot",
  state: "open",  // optional: "open", "closed", "all"
  limit: 10       // optional: max 30
})
```

### Get File Contents
```javascript
github_get_file({
  owner: "HKUDS",
  repo: "nanobot",
  path: "README.md",
  branch: "main"  // optional
})
```

## Examples

**Find popular JavaScript projects:**
```javascript
github_search_repos({
  query: "language:javascript stars:>5000",
  sort: "stars",
  limit: 5
})
```

**Check repository details:**
```javascript
github_get_repo({
  owner: "facebook",
  repo: "react"
})
```

**Read a specific file:**
```javascript
github_get_file({
  owner: "HKUDS",
  repo: "nanobot",
  path: "nanobot/agent/loop.py"
})
```

## Configuration

To increase rate limits, add a GitHub token to `~/.chatdock/settings.json`:

```json
{
  "github": {
    "token": "ghp_your_token_here"
  }
}
```

Get a token at: https://github.com/settings/tokens
