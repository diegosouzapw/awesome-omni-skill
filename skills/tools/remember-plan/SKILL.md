---
name: remember-plan
description: Remember a plan with git commit
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git push:*), Bash(git commit:*)
---

# Your task

- Remember a plan by committing it to a file.
- Plans Directory: settings.jsonの `plansDirectory` を参照する。設定がない場合は `{REPOSITORY_ROOT}/docs/plans` をデフォルトとして使う
- File name: `{YYYY-MM-DD}-{plan-name}.md`
- Commit message: `[ci skip] Add plan: {plan-name}`
- IMPORTANT: do not copy original plan file. move file to the plans directory.
