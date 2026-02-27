---
name: task-next
description: Show dependency-resolved next tasks from ROADMAP.md active sprint
---

# Task Next - Dependency-Aware Task Selection

Shows which tasks are ready to work on based on dependency resolution.

## When to Use

- Tasks related to task-next
- User explicitly invokes `/task-next`

## When NOT to Use

- When the task doesn't match this skill's scope -- check related skills
- When a more specialized skill exists for the specific task

## Usage

```
/task-next              # Show ready (unblocked) tasks
/task-next --all        # Show ready, blocked, and completed
/task-next --blocked    # Show only blocked tasks
```

## How It Works

1. Reads ROADMAP.md and finds all task items (`- [ ] **ID:** Description`)
2. Parses `[depends: X1, X2]` annotations on each task
3. Builds a directed acyclic graph (DAG) of dependencies
4. Runs Kahn's topological sort to determine execution order
5. Reports which tasks are ready (all dependencies completed) vs blocked

## Steps

### 1. Run the dependency resolver

```bash
node scripts/tasks/resolve-dependencies.js
```

### 2. Present results to the user

Format the output clearly:

**Ready tasks** — these can be started now:

- List each with its ID, description, and satisfied dependencies
- Suggest the highest-priority one based on track ordering

**Blocked tasks** — these are waiting on other tasks:

- List each with what it's waiting for
- Highlight if a blocker is close to completion

### 3. Help the user pick

If the user wants to work on a task:

1. Confirm the task ID
2. Mark it as the active task in TodoWrite
3. Begin implementation

### 4. After completing a task

When a task is done:

1. Check off the item in ROADMAP.md (`- [ ]` -> `- [x]`)
2. Re-run the resolver to see what's newly unblocked
3. Tell the user what tasks are now available

## Dependency Annotation Format

Add `[depends: X1, X2]` to any task item in ROADMAP.md:

```markdown
- [ ] **B3:** Lighthouse CI Integration [depends: B1, B2]
- [ ] **B5:** Lighthouse Dashboard Tab [depends: B3, B4]
```

Rules:

- Dependencies reference task IDs (B1, B3, CANON-0011, DEBT-0944, etc.)
- Multiple dependencies separated by commas
- Annotation must be in square brackets at the end of the line
- Tasks without `[depends:]` are considered independent (always ready)
- Circular dependencies are detected and reported as errors

---

## Version History

| Version | Date       | Description            |
| ------- | ---------- | ---------------------- |
| 1.0     | 2026-02-25 | Initial implementation |
