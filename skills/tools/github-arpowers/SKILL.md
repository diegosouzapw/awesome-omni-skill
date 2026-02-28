---
name: github
description: Clone, modify, and push to GitHub repos
version: 1.0.0
triggers:
  - github
  - git
  - repo
  - ci
  - deploy failed
---

# GitHub Skill

Manage GitHub repositories using git CLI and GITHUB_TOKEN.

## Authentication

```bash
git clone https://${GITHUB_TOKEN}@github.com/arpowers/bot.git /tmp/bot
```

## CI Failure Workflow

When notified of a CI failure:

1. **Clone the repo**
```bash
rm -rf /tmp/bot
git clone https://${GITHUB_TOKEN}@github.com/arpowers/bot.git /tmp/bot
cd /tmp/bot
```

2. **Check the error** - Visit the GitHub Actions log URL provided

3. **Fix the issue** - Edit files as needed

4. **Commit and push**
```bash
cd /tmp/bot
git config user.name "Ari Bot"
git config user.email "bot@andrewpowers.com"
git add -A
git commit -m "fix: <description>"
git push
```

## Repos

| Repo | Purpose |
|------|---------|
| arpowers/bot | This bot's code |
