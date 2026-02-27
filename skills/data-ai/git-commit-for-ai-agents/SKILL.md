---
name: git-commit-for-ai-agents
description: Commit changes to a git repository. Use whenever a git commit is to be executed.
license: GPL-3.0-or-later
compatibility: Requires git.
metadata:
  author: Rafael Guterres Jeffman <rjeffman@redhat.com>
  version: "1.0"
---

**CRITICAL**: A commit to git must **always** be the result of explicit requests, and **never** can be done automatically.

### Files

- **Always** follow the rules found in `.gitignore` files.
- If agent is Claude, ensure that the setting `respectGitignore` is set to `true`.

### Commit message

- **Always** limit line text to 72 characters.
- **CRITICAL**: Always add "Assisted-by: AGENT IDENTIFICATION <AGENT E-MAIL>"
- **CRITICAL**: Always commit with '-s/--signoff'
- Each commit should represent a single, logical change
- Use bullet points for multiple changes.
- **CRITICAL**: Always add a bullet list of the prompts used to generate the change being commited.
- **NEVER** add promps like "commit changes" to the list of prompts.
- The message summary should contain the component that is affected by the changes and then a brief summary of the changes.

This is an example of a good commit message:

```
CLAUDE.md: Sync with AGENTS.md and add sync rule

Updated CLAUDE.md to include all rules from AGENTS.md:
- Added AGENTS.md synchronization check requirement
- Added rule to ignore docs/prompts.md when analyzing
- Expanded commit message rules to include prompt tracking
- Added requirement for single logical change per commit
- Added .gitignore compliance requirement
- Reorganized Git section into Files and Commit Message
  subsections

Prompts:
- Update CLAUDE.md with current AGENTS.md, and add a rule
  to always update it again, if AGENTS.md is newer than
  CLAUDE.md

Assisted-by: Claude Sonnet 4.5 <noreply@anthropic.com>
Signed-off-by: Rafael Guterres Jeffman <rjeffman@redhat.com>
```
