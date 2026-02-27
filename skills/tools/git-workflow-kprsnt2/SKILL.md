---
name: git-workflow
description: Git best practices and workflows including conventional commits, branching strategies, and collaboration patterns.
globs: ["**/.git/**", "**/.gitignore", "**/.gitattributes"]
alwaysApply: ["**/CONTRIBUTING.md", "**/CHANGELOG.md"]
priority: 80
tags: ["workflow"]
---

# Git Workflow Best Practices

## Commit Messages (Conventional Commits)
- feat: New feature
- fix: Bug fix
- docs: Documentation
- style: Formatting
- refactor: Code restructuring
- test: Adding tests
- chore: Maintenance

Format: type(scope): description
Example: feat(auth): add password reset flow

## Branch Naming
- feature/TICKET-description
- fix/TICKET-description
- hotfix/description
- release/v1.0.0
- docs/description

## Workflow
- Keep commits atomic and focused
- Rebase feature branches on main
- Squash WIP commits before merge
- Never force push to shared branches
- Use pull request templates
- Require code reviews

## .gitignore
- Ignore build artifacts
- Ignore node_modules, venv, target
- Ignore local config (.env.local)
- Ignore IDE settings (.idea, .vscode)
- Ignore OS files (.DS_Store)

## Advanced
- Use git hooks with husky/pre-commit
- Sign commits with GPG
- Use git-crypt for secrets
- Bisect for finding bugs
