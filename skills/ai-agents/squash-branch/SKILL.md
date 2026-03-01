---
name: squash-branch
description: Squash all commits on a feature branch into a single commit. Use when the user wants to clean up commit history, squash commits, or consolidate branch changes into one commit.
user-invocable: true
---

# Squash Branch Skill

Squash all commits on a feature branch into a single commit by generating a patch, recreating the branch from the base, and applying the patch.

## Parameters

The user should provide:
- **feature_branch**: The branch to squash (e.g., `jose/vectors`, `feature/my-feature`)
- **base_branch**: The branch to squash against (e.g., `mainline`, `main`, `master`)
- **commit_message** (optional): If not provided, prompt the user for a commit message

## Instructions

1. **Verify current state**: Ensure the working directory is clean and you're on the feature branch
   ```bash
   git status
   git branch --show-current
   ```

2. **Generate the patch**: Create a diff between the base branch and feature branch
   ```bash
   git diff <base_branch>...<feature_branch> > /tmp/squash-branch.patch
   ```

3. **Switch to base branch and delete feature branch**:
   ```bash
   git checkout <base_branch>
   git branch -D <feature_branch>
   ```

4. **Recreate feature branch from base and apply patch**:
   ```bash
   git checkout -b <feature_branch>
   git apply /tmp/squash-branch.patch
   ```

5. **Prompt for commit message**: If no commit message was provided, use AskUserQuestion to ask the user what commit message they want

6. **Stage and commit**:
   ```bash
   git add -A
   git commit -m "<commit_message>

   Co-Authored-By: Claude <noreply@anthropic.com>"
   ```

7. **Show final result**: Display the new clean commit history
   ```bash
   git log --oneline <base_branch>..<feature_branch>
   git diff --stat <base_branch>...<feature_branch>
   ```

## Example Usage

User: "Squash my feature/auth branch against main with message 'Add user authentication'"

User: "Clean up the commit history of jose/vectors against mainline"

User: "/squash-branch feature_branch=dev/new-feature base_branch=main"

## Important Notes

- This operation is destructive to the local branch history - make sure changes are not pushed and nothing other than the feature branch is modified
- Always verify the patch was created successfully before deleting the branch
- If the patch application fails, the original branch is already deleted - warn the user about this risk
- The patch file is stored in /tmp and will be cleaned up by the system
