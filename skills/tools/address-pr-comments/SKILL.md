---
name: address-pr-comments
description: Addresses PR review comments by making code changes and posting replies. Takes a PR number or auto-detects from current branch. Use when the user wants to address review feedback, respond to PR comments, fix PR feedback, or says "address comments on PR X".
user-invocable: true
allowed-tools: Bash, Read, Write, Edit, Grep, Glob, AskUserQuestion
---

# Address PR Comments

## Purpose

Read PR review comments and address them by making code changes, answering questions, or discussing feedback with the user. Post replies to acknowledge addressed comments.

## Prerequisites

- Must be on the PR's branch (not main/master)
- PR must exist and be open

## Workflow

### Step 1: Verify Environment

Check we have a GitHub remote and are on a feature branch:

```bash
git remote -v
git status
git rev-parse --abbrev-ref HEAD
```

**STOP if:**
- No remote exists → "This skill requires a GitHub remote. Please add one with `git remote add origin <url>` first."
- On main/master → "Please switch to the PR's branch first. You can find it with `gh pr view <number> --json headRefName`"
- Uncommitted changes → "Please commit or stash your changes first."

### Step 2: Identify the PR

User will provide a PR number (e.g., `#12` or `12`), or auto-detect from current branch:

```bash
gh pr view --json number,state,title
```

If no PR exists for the current branch, ask the user for a PR number.

Validate the PR is open. If closed/merged, inform the user and stop.

### Step 3: Fetch PR Comments

Retrieve general PR comments:

```bash
gh pr view <number> --comments
```

Present the comments to the user, showing:
- Comment author
- Comment body
- When it was posted

If no comments exist, inform the user and stop.

### Step 4: Process Comments

For each comment, work with the user to determine the appropriate action:

1. **Present the comment** clearly to the user

2. **Ask what action to take** using AskUserQuestion:
   - "Make code changes" → implement the requested changes
   - "Post a reply" → draft and post a response
   - "Discuss first" → talk through the feedback before acting
   - "Skip this comment" → move to the next comment

3. **Execute the action:**
   - **Code changes**: Make the changes, then optionally post a reply confirming what was done
   - **Reply**: Draft a reply and post it using `gh pr comment`
   - **Discussion**: Explore the codebase if needed, discuss with user, then decide on action

4. **Track progress** using TodoWrite to show which comments have been addressed

5. **Loop** until all comments are processed or user wants to stop

### Step 5: Post Replies

When posting replies to acknowledge addressed comments:

```bash
gh pr comment <number> --body "<reply>"
```

Keep replies concise and professional. Reference specific changes made if applicable.

### Step 6: Commit and Push

After making code changes:

```bash
git add .
git status
git commit -m "<summary of changes addressing review feedback>"
git push
```

Report what was changed and pushed.

### Step 7: Report Completion

Summarise what was done:
- Number of comments addressed
- Code changes made (files modified)
- Replies posted
- Any comments skipped or left unresolved

## Guidelines

**DO**:
- Present each comment clearly before asking for action
- Make requested code changes accurately
- Post concise, professional replies
- Commit and push after making changes
- Track progress through comments

**DON'T**:
- Make changes without user approval
- Post replies the user hasn't approved
- Skip comments without asking
- Leave uncommitted changes

## Handling Common Scenarios

**Comment requests a code change:**
Implement the change, verify it works, then offer to post a reply confirming it's done.

**Comment asks a question:**
Discuss with the user, explore the codebase if needed, then draft a reply.

**Comment is unclear:**
Ask the user for clarification before taking action.

**Multiple comments on same topic:**
Group them together and address as a single unit.

**Disagreement with feedback:**
Discuss with user, then post a reply explaining the reasoning if they want to push back.

## Checklist

- [ ] Verify on feature branch (not main/master)
- [ ] Identify PR (from argument or auto-detect)
- [ ] Verify PR is open
- [ ] Fetch and display comments
- [ ] Process each comment with user
- [ ] Make code changes as needed
- [ ] Post replies as needed
- [ ] Commit and push changes
- [ ] Report what was done
