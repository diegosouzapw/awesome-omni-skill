---
name: safety
description: Git, command, Kubernetes, data, workspace, and temporary files safety rules. Use when committing, pushing, using kubectl, handling multi-repo workspaces, or performing destructive operations.
---
# Safety Rules

## Git Safety

### Absolute Rules (Cannot be broken)

- Do not commit unless explicitly requested
- Do not push unless explicitly requested
- Never commit to main/master branch unless explicitly requested
- Never run `git reset --hard` without explicit user approval

### Core Safety Guidelines

- Do not change git stage without being asked
- Do not make commits without reviews unless explicitly requested
- Always create feature branches for changes
- Use `/git-branch` command for safe branch creation
- `/commit` command automatically creates feature branch when on main/master
- Verify branch before committing
- Never commit unstaged changes without explicit request
- Always validate conventional commit format
- Create backups before destructive operations
- Show what will be committed before execution
- Never push directly to main/master
- Always create pull requests for main branch changes
- Use `/pr-create` command for safe PR creation
- Verify remote branch exists before pushing

## Command Safety

### Shell Safety

- Prefer terminal commands over GUI operations when possible

### Kubernetes Safety

- Never execute `kubectl delete` or `kubectl apply`
- Use `/k8s-check` for safe inspection
- Use `/k8s-validate` for manifest validation
- Use `/k8s-diff` for change preview

### Git Destructive Operations

- Never run `git reset --hard` without explicit approval
- Use `/git-reset` for safe reset with backup
- Always create stash before destructive operations
- Provide recovery instructions

## Data Safety

- Always create backups before destructive operations
- Use git stash for uncommitted changes
- Document recovery procedures
- Test backup restoration

## Workspace Safety

### Multi-Repository Handling

- Always check current working directory and understand repository boundaries
- Never assume single git repository when working in multi-repo workspace
- Always verify which repository operations are targeting before execution
- When working with staged changes, identify which specific repository they belong to
- Navigate to correct repository directory before running git operations
- Treat each repository as separate entity with its own git state

### Error Prevention

- Always ask for clarification when workspace structure is unclear
- Confirm target repository before running git commands
- Use non-destructive commands first (git stash, git log) to understand situation

## Temporary Files Safety

- When creating temporary files, use temporary directories (`./tmp` or system tmp)
- Automatically clean up temporary files after use
- Never commit temporary files to version control
