---
name: git-commit
description: Stage and commit git changes with conventional commit messages. Use when user wants to commit changes, or asks to save their work, even when committing your own work after completing a task. Also activates when user says "commit changes", simply "commit", or similar git workflow requests. Never commit without loading this skill.
---

# Git Commit Workflow

Stage all relevant changes, and create a conventional commit following patterns below.

## When to Use

Automatically activate when the user:
- Explicitly asks to push changes ("push this", "commit and push")
- Mentions saving work to remote ("save to github", "push to remote")
- Completes a feature and wants to share it
- Says phrases like "let's push this up" or "commit these changes"

## Commit Message Patterns

Here's a model Git commit message:

```
Capitalized, short (50 chars or less) summary

More detailed explanatory text, if necessary.  Wrap it to about 80
characters or so.  In some contexts, the first line is treated as the
subject of an email and the rest of the text as the body.  The blank
line separating the summary from the body is critical (unless you omit
the body entirely); tools like rebase can get confused if you run the
two together.

Write your commit message in the imperative: "Fix bug" and not "Fixed bug"
or "Fixes bug."  This convention matches up with commit messages generated
by commands like git merge and git revert.

Further paragraphs come after blank lines.

- Bullet points are okay, too
- Typically a hyphen or asterisk is used for the bullet, followed by a
  single space, with blank lines in between, but conventions vary here
- Use a hanging indent
```

The rules:

- The first line is a concise summary of the change, 50 characters or less.
- Wrap all lines in the body to 80 characters maximum.
- The first paragraph must explain what the PROBLEM is before the commit, and
  what impact it had. If there's an obvious counterargument or objection to the
  change ("why not just X?"), acknowledge it briefly and explain why the change
  is still worthwhile.
- The second paragraph must explain what the implemented SOLUTION in the commit
  is. Write it as proper prose, not bullet-point style - start sentences with a
  subject like "This commit...", "This change...", or "This modification...",
  and use "It also..." for follow-up sentences rather than starting with a verb.
  It doesn't need to go into all details that the code itself does, but should
  explain the approach taken, the reasoning. It needs to be as concise as possible.
- The third paragraph should explain what an ideal solution would look like this
  is not what the commit implements. It includes things that could be improved later,
  edge cases. It makes the current solution be intentional.
- Do not push directly after committing. Wait for user confirmation.
- Do not include Co-Authored-By: Claude <noreply@anthropic.com> or equivalent.
  You MUST NOT include any indication that an AI assisted in the commit or that
  the commit message was generated.

## Examples

### Refactoring for clarity

```
Move EmailValidator to Notifications namespace

The EmailValidator class validates email format, delivery status, and bounce
handling, all specific to our notification system. Its generic name and
top-level location made it look like a general-purpose utility when it's
tightly coupled to notification internals. Sure, someone could read the code
to understand this, but the name should communicate intent upfront.

This commit moves the class under the Notifications namespace to clarify its
scope. It also updates the three call sites in the mailer classes accordingly.
```

### Bug fix

```
Fix race condition in payment processing

When two requests hit the payment endpoint simultaneously, both could pass
the idempotency check before either wrote to the database. This caused
duplicate charges in production roughly once per 10k transactions.

This change wraps the check-and-write in a database transaction with row-level
locking. It also adds an index on the idempotency key to keep the lock fast.

A distributed lock (Redis/etc) would handle cross-server races better, but
our current single-database setup makes this sufficient for now.
```

### Simple feature

```
Add retry button to failed export jobs

Users had no way to retry a failed export without re-entering all parameters.
Support tickets about this increased after we added the larger export types
last month.

This commit adds a retry action to the exports controller that clones the
original job's parameters into a new job. It also adds the button to the
job status page, visible only for failed jobs.
```

## Workflow

**ALWAYS use the terminal**: example of passing paragraphs via `-m` flags:

```bash
git commit \
  -m "paragraph 1" \
  -m "paragraph 2" \
  -m "paragraph 3"
```

If you pass all in one line in bash/zsh, you have to use `$'...'` syntax with
`\n\n` for new paragraphs:

```bash
git commit -m $'paragraph 1\n\nparagraph 2\n- bullet point\n- bullet point'
```

Once you commit, let me know what the commit title was.
