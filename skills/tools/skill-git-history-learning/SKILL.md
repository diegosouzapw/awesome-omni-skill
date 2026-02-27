---
name: skill-git-history-learning
description: Use git history as a learning asset for onboarding and decision tracking.
metadata:
  author: RyoMurakami1983
  tags: [git, history, learning, documentation, workflow]
  invocable: false
---

# Git History Learning

Use commit history, tags, and notes to learn why changes were made and to onboard faster.

**Architecture Decision Record (ADR)**: A document that records key design choices.
**Changelog**: A release summary derived from commits and PRs.
**Git Notes**: Metadata attached to commits without changing history.

Progression: Simple → Intermediate → Advanced examples improve clarity because each step adds context.
Reason: Added context makes learning and audits faster.

## When to Use This Skill

Use this skill when:
- Onboarding new teammates by walking them through curated commit history
- Investigating past decisions when a bug or refactor needs context
- Building changelogs or release notes from labeled commits and PRs
- Training reviewers with real examples from prior PR and commit flows
- Capturing lessons from incidents so fixes are easy to trace later
- Turning tribal knowledge into shared assets for cross-team reuse

## Related Skills

- **`skill-git-commit-practices`** - Commit messages with context
- **`skill-git-review-standards`** - Review patterns and checklists
- **`skill-github-pr-workflow`** - PR workflow and merge policies

---

## Dependencies

- Git 2.30+
- Access to repository history

---

## Core Principles

1. **History is a Library** - Use it for learning, not blame
2. **Why Matters** - Capture decision context
3. **Traceability** - Link commits to issues and PRs
4. **Repeatable Learning** - Make examples reusable
5. **Shared Growth** - Teach using real changes

---

## Pattern 1: Read History with Context

### Overview

Use graph and decoration to understand flow.

### Basic Example

```bash
# ✅ CORRECT
git log --oneline --graph --decorate -20

# ❌ WRONG
git log
```

| Goal | Command | Use When |
|------|---------|----------|
| Quick scan | `git log --oneline` | Daily review |
| Flow view | `git log --graph` | Branch analysis |

### Intermediate Example

```bash
# Include merges and authors
git log --graph --decorate --pretty="%h %an %s" -20
```

### Advanced Example

- Use `git log --since="30 days"` for focused reviews

### When to Use

- When learning a feature's origin
- When reviewing release scope

---

## Pattern 2: Learn with git blame

### Overview

Find the reason behind a line of code.

### Basic Example

```bash
# ✅ CORRECT
git blame path/to/file.cs

# ❌ WRONG
git blame -w
```

### Intermediate Example

- Pair blame output with the linked PR

### Advanced Example

- Use blame to find risk areas before refactor

### When to Use

- When a bug appeared recently
- When onboarding into a subsystem

---

## Pattern 3: Write Learning-Friendly Commits

### Overview

Commit messages should explain "why" not only "what".

### Basic Example

```text
feat: cache user profile for 30s to reduce API load

# ❌ WRONG
update
```

### Intermediate Example

- Include a short reason and impact

### Advanced Example

- Link to issue numbers or decision logs

### When to Use

- When changes are hard to infer from diff
- When future debugging is expected

---

## Pattern 4: Generate Changelogs from History

### Overview

Turn commit history into release notes.

### Basic Example

Release configuration file (config) example:

```yaml
# .github/release.yml
categories:
  - title: "Features"
    labels: ["feature"]
```

### Intermediate Example

```python
# ✅ CORRECT - fail fast if git log fails
import subprocess

try:
    output = subprocess.check_output(["git", "log", "--oneline", "-5"], text=True)
    print(output)
except subprocess.CalledProcessError as exc:
    raise SystemExit(exc.returncode)
```

### Advanced Example

- Automate release notes in CI

### When to Use

- When release notes are required
- When marketing or support needs summaries

---

## Pattern 5: Onboarding Walkthroughs

### Overview

Use a curated set of commits to teach features.

### Basic Example

```text
Learning path:
1) Initial API setup
2) Authentication flow
3) Payment integration
```

### Intermediate Example

- Include PR links and discussion threads

### Advanced Example

- Record a short walkthrough session

### When to Use

- When new teammates join
- When transferring ownership

---

## Pattern 6: Metrics for Retrospectives

### Overview

Use history to learn from delivery trends.

### Basic Example

```bash
# ✅ CORRECT
git shortlog -s -n --since="90 days"
```

### Intermediate Example

- Track changes per subsystem

### Advanced Example

```csharp
// ✅ CORRECT - register history analyzer
using Microsoft.Extensions.DependencyInjection;

services.AddSingleton<HistoryAnalyzer>();
```

### When to Use

- When analyzing team velocity
- When finding bottlenecks

---

## Pattern 7: Tags and Notes for Memory

### Overview

Use tags and notes to preserve decisions.

### Basic Example

```bash
# ✅ CORRECT
git tag v1.2.0
```

### Intermediate Example

```bash
# Add decision note
git notes add -m "Hotfix for issue #123"
```

### Advanced Example

- Store notes in a shared namespace

### When to Use

- When auditing releases
- When documenting incident fixes

---

## Best Practices

- Use history for learning, not blame
- Define a consistent changelog format
- Create curated onboarding paths
- Apply tags and notes for decisions
- Avoid one-word commit messages

---

## Common Pitfalls

1. **Only reading recent commits**  
Fix: Use time filters to expand scope.

2. **Missing why in commits**  
Fix: Add short rationale in messages.

3. **Ignoring history in onboarding**  
Fix: Use curated commit walkthroughs.

---

## Anti-Patterns

- Blaming individuals based on git blame
- Deleting tags without recording reasons
- Writing one-word commit messages

---

## Quick Reference

### History Learning Checklist

- [ ] Read log with graph
- [ ] Use blame for intent
- [ ] Capture why in commits
- [ ] Build release notes
- [ ] Teach from real history

### Decision Table

| Situation | Action | Decision |
|-----------|--------|----------|
| New member | Curated path | Decision: faster onboarding |
| Incident | Notes + tag | Decision: preserve evidence |
| Large release | Changelog | Decision: summarize impact |

---

## FAQ

**Q: Is git blame used for blame?**  
A: No. Use it to find context and learn why.

**Q: Should we delete old tags?**  
A: Only with written reasons and a replacement tag.

**Q: How do we teach history?**  
A: Use curated commit paths and PR links.

---

## Resources

- https://git-scm.com/docs/git-log
- https://git-scm.com/docs/git-blame
- https://github.com/release-drafter/release-drafter
