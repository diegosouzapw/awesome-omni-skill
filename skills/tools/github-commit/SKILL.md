---
model: sonnet
name: github-commit
description: Create a well-formatted git commit following best practices
---

# Git Commit Skill

When this skill is invoked, create a git commit following these steps:

## Instructions

1. **Check git status** to see what files are staged/unstaged
2. **Review the changes** using `git diff` (or `git diff --staged` if files are already staged)
3. **Verify no sensitive data** is being committed (secrets, private keys, API keys, tokens, credentials)
4. **Stage files if needed** with `git add`
5. **Create the commit** using `git commit -S` to sign the commit

## Commit Message Guidelines

Follow these seven rules for great commit messages:

1. **Separate subject from body with a blank line**
2. **Limit the subject line to 50 characters**
3. **Capitalize the subject line**
4. **Do not end the subject line with a period**
5. **Use the imperative mood in the subject line** (e.g., "Add feature" not "Added feature")
6. **Wrap the body at 72 characters**
7. **Use the body to explain what and why vs. how**

Additional guidelines:

- **Style**: Casual and human, but professional
- **Language**: Always in English
- **Format**:
  - Short title describing the change
  - If more context is needed, add a blank line followed by a commit body with further clarifications

## Important Notes

- Always use `-S` flag to sign commits with GPG
- Be specific but concise
- No need for issue numbers unless critical
- Avoid generic messages like "fix bug" or "update code"
- **Never commit sensitive data**: secrets, private keys, API keys, tokens, .env files, wallet private keys, seed phrases, mnemonics, keystore files, or any credentials

## References

Based on: <https://chris.beams.io/git-commit>
