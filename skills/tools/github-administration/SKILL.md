---
name: github-administration
description: Technical guidance for GitHub issue and PR administration via gh, including command safety, quoting, verification, and recovery.
---

# GitHub Administration Skill

## Purpose

Use this skill whenever you interact with GitHub issues or pull requests through the `gh` CLI.

This skill is technical-only and focuses on execution mechanics:

- selecting and running the correct `gh` commands
- handling quoting safely
- applying metadata (assignee, labels)
- verifying outcomes and fixing command/result issues

Issue content design is out of scope here and belongs to `github-issue-designer`.

## Mandatory Usage

Always use `github-administration` whenever you want to interact with a GitHub issue.

## Pairing Rule

When an issue should be created or updated, use:

- `github-issue-designer` for issue content design
- `github-administration` for command execution via `gh`

## Preflight

Before mutating issues/PRs:

```sh
gh api user --jq .login
gh repo view --json nameWithOwner
gh label list --json name --limit 100
```

## Safe Command Rules

1. Prefer one mutation per command.
2. Always use `--body-file` for multi-line issue bodies.
3. Use double quotes for arguments with spaces.
4. Verify immediately after create/edit/close operations.
5. In persistent terminals, avoid heredoc for long multi-line bodies; write a temporary file first and pass it via `--body-file`.
6. Do not auto-assign newly created issues unless the navigator explicitly requests assignment at creation time.

## Command Recipes

### Create issue

```sh
gh issue create --title "<title>" --body-file /tmp/issue.md --label <label> --assignee <login>
```

### View/verify issue

```sh
gh issue view <number> --json number,title,assignees,labels,url
```

### Edit issue

```sh
gh issue edit <number> --title "<new-title>" --body-file /tmp/issue-updated.md
```

### Comment on issue

```sh
gh issue comment <number> --body "<progress update>"
```

### Close issue

```sh
gh issue close <number> --comment "<closure note>"
```

### Create PR linked to issue

```sh
git push origin <branch>
gh pr create --title "Fix #<issue>: ..." --body "...\n\nFixes #<issue>" --head <branch>
```

## Troubleshooting

### Label not found

Error: `could not add label: '<label>' not found`

Fix:

- choose an existing label from `gh label list`, or
- create the missing label in GitHub UI and retry.

### PR push prompt appears

Cause: branch is not pushed.

Fix:

```sh
git push origin <branch>
gh pr create --head <branch> ...
```

### Shell quoting breaks (`dquote>`)

Fix:

- reset shell or open a new terminal

Prevention:

- use `--body-file` for multiline input

### Heredoc gets stuck (`heredoc>`)

Cause: unterminated or corrupted heredoc input in a persistent shell session.

Fix:

- abort and recover shell state
- avoid reusing heredoc for long content
- create the body in a temporary/workspace file and retry with `--body-file`

## Post-action Checklist

After each mutation:

- verify issue/PR metadata and links
- correct mistakes immediately

## Retrospective Requirement

After creating an issue and after closing a merged issue, run `self-learning-skills` retrospective automatically.
