---
name: commit
description: "Create atomic commits with clear, descriptive messages following WordPress core style. Use when committing code, making git commits, or when asked to commit changes."
---

# WordPress-Style Atomic Commit Skill

## When to Use

Invoke this skill when:
- User asks to "commit" changes
- User says "make a commit" or "commit my changes"
- After completing a task that should be committed
- User explicitly invokes `/commit`

## Atomic Commit Principles

Every commit should be:

1. **Smallest meaningful unit** - The minimum change that makes sense on its own
2. **Single responsibility** - One task, one fix, or one feature per commit
3. **No mixed concerns** - Don't combine refactors with features, or style fixes with bug fixes
4. **Functional** - Each commit should leave the codebase in a working state

### When to Split Commits

Split into multiple commits when changes include:
- A bug fix AND a new feature
- Refactoring AND behavioral changes
- Multiple unrelated fixes
- Code changes AND dependency updates

## Commit Message Format

### Subject Line (Required)
- **60 characters maximum**
- Imperative mood ("Fix bug" not "Fixed bug" or "Fixes bug")
- No trailing period
- Capitalize first word
- Be specific about what changed

### Body (When Needed)
- Separate from subject with a blank line
- Explain **why** the change was made, not what changed
- Wrap at 72 characters
- Include context that isn't obvious from the code

## Procedure

1. **Review changes**
   ```bash
   git status
   git diff
   ```

2. **Check for mixed concerns**
   - If changes serve multiple purposes, stage and commit separately
   - Use `git add -p` for partial staging if needed

3. **Write the subject line**
   - Start with a verb: Add, Fix, Update, Remove, Refactor, Improve
   - Be specific: "Fix null check in user validation" not "Fix bug"
   - Count characters - must be 60 or fewer

4. **Write the body (if needed)**
   - Explain the reasoning behind the change
   - Mention any non-obvious side effects
   - Reference related context

5. **Create the commit using HEREDOC**
   ```bash
   git commit -m "$(cat <<'EOF'
   Subject line here (60 chars max)

   Body explaining why this change was made. Focus on the
   reasoning and context, not describing what the code does.
   EOF
   )"
   ```

6. You should not add a 'co-authored by' byline.

## Examples

### Good Commits

```
Fix responsive breakpoint in navigation component

The mobile menu was collapsing at 768px instead of 782px,
causing layout issues on iPad portrait mode. This aligns
the breakpoint with WordPress admin responsive standards.
```

```
Add input validation for email field
```

```
Remove deprecated getUserData function

This function was replaced by fetchUserProfile in v2.1 and
all callers have been migrated.
```

### Bad Commits (Avoid These)

```
# Too vague
Fixed stuff

# Past tense instead of imperative
Added new feature

# Multiple concerns in one commit
Fix bug and add tests and update styles

# Subject too long (over 60 chars)
This commit fixes the bug where the navigation menu was not displaying correctly on mobile devices

# Trailing period
Fix navigation bug.

# Describes what, not why
Change maxLength from 50 to 100

# Co-authored by line included (should be avoided in commit messages)

Examples of lines that should NOT be included in the commit message:
```
Generated with [Claude Code](https://claude.ai/code)
via [Happy](https://happy.engineering)

Co-Authored-By: Claude <noreply@anthropic.com>
Co-Authored-By: Happy <yesreply@happy.engineering>
```

## Verification Checklist

Before finalizing the commit:

- [ ] Subject line is 60 characters or fewer
- [ ] Subject uses imperative mood (Add/Fix/Update, not Added/Fixed/Updated)
- [ ] Subject has no trailing period
- [ ] Commit addresses only ONE concern
- [ ] Body explains why (if change isn't self-evident)
- [ ] Co-Authored-By line is NOT included

## Quick Reference

| Element | Requirement |
|---------|-------------|
| Subject length | 60 chars max |
| Subject mood | Imperative (Add, Fix, Update) |
| Subject punctuation | No trailing period |
| Body separation | Blank line after subject |
| Body purpose | Explain why, not what |
