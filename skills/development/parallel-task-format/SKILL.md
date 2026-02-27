---
name: parallel-task-format
description: Compact YAML format for defining parallel task specifications with scope, boundaries, and agent assignments. Use when creating task files for parallel development.
user-invocable: false
---

# Task Specification Format

Compact YAML format for parallel task files in `parallel/TS-XXXX-slug/tasks/`.

## Complete Task Example

```yaml
---
id: task-001
component: users
wave: 1
deps: []
blocks: [task-004, task-005]
agent: python-experts:django-expert
skills: [python-experts:python-style, python-experts:django-dev, python-experts:django-api]
tech_spec: TS-0042
contracts: [contracts/types.py, contracts/api-schema.yaml]
---
# task-001: User Management

## Scope
CREATE: apps/users/{models,views,serializers,urls}.py, apps/users/tests/*.py
MODIFY: config/urls.py
BOUNDARY: apps/orders/*, apps/products/*, apps/*/migrations/*

## Requirements
- User model with email authentication
- UserSerializer with explicit fields
- UserViewSet (list, retrieve, create, update)
- Email uniqueness validation

## Checklist
- [ ] Model matches UserDTO in contracts/types.py
- [ ] API matches /api/users/* in contracts/api-schema.yaml
- [ ] pytest apps/users/ passes
- [ ] mypy apps/users/ passes
- [ ] ruff check apps/users/ passes
- [ ] No files modified outside scope
```

## YAML Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `id` | Yes | Task identifier (task-NNN or task-NNN-component) |
| `component` | Yes | System component name |
| `wave` | Yes | Dependency wave number (1, 2, 3...) |
| `deps` | Yes | Task IDs this depends on (empty list `[]` if none) |
| `blocks` | No | Task IDs this blocks (optional) |
| `agent` | Yes | Recommended agent type |
| `skills` | Yes | Skills the agent should invoke (list from agent-skills-mapping.yaml) |
| `tech_spec` | No | Tech Spec ID (if applicable) |
| `contracts` | Yes | Contract files to reference (relative paths) |

**Note:** `skills` are stored in task files so prompt generation can include them in `=== REQUIRED SKILLS ===` sections. They are NOT stored in manifest.json.

## Scope Section Format

Use compact notation with three directives:

### CREATE
Files to create (use glob patterns). **A file can only be in CREATE for ONE task.**
```
CREATE: apps/users/{models,views,serializers,urls}.py, apps/users/tests/*.py
```

### MODIFY
Existing files to modify. Use **scoped syntax** for parallel modifications:

**Unscoped** (whole file - only ONE task per wave can use this):
```
MODIFY: config/urls.py, config/settings.py
```

**Scoped** (specific section - multiple tasks in same wave can modify different scopes):
```
MODIFY: apps/users/models.py::User.save          # Owns User.save method
MODIFY: apps/users/models.py::User.clean         # Different task owns User.clean
MODIFY: apps/users/views.py::UserViewSet         # Owns entire class
MODIFY: config/urls.py::urlpatterns              # Owns urlpatterns list
```

**Scoped syntax rules:**
- `file.py::ClassName` - owns entire class
- `file.py::function_name` - owns entire function
- `file.py::ClassName.method` - owns specific method
- Scopes must NOT overlap (no nesting like `::Class` and `::Class.method` in same wave)

### BOUNDARY
Files NOT to touch (owned by other tasks):
```
BOUNDARY: apps/orders/*, apps/products/*, apps/*/migrations/*
```

## Task Naming Convention

```
task-{number}-{component}.md

Examples:
- task-001-users.md
- task-002-products.md
- task-003-orders.md
- task-004-api.md
- task-005-integration.md
```

## Agent Type Selection

| Task Files | Agent | Description |
|------------|-------|-------------|
| `apps/*/models.py`, `apps/*/views.py` | `python-experts:django-expert` | Django models, views, serializers |
| `api/*.py`, `routers/*.py` | `python-experts:fastapi-expert` | FastAPI endpoints |
| `src/components/*.tsx` | `frontend-experts:react-typescript-expert` | React components |
| `**/test_*.py`, `**/tests/*.py` | `python-experts:python-testing-expert` | Python tests |
| `*.spec.ts`, `*.test.tsx` | `frontend-experts:playwright-testing-expert` | TypeScript/E2E tests |
| `terraform/`, `docker-compose.yml` | `devops-data:devops-expert` | Infrastructure |
| Integration, architecture | `devops-data:cto-architect` | Cross-cutting concerns |

## Contract References

Contracts are in the same parallel directory:
```
parallel/TS-0042-slug/
  contracts/
    types.py        # Reference as: contracts/types.py
    api-schema.yaml # Reference as: contracts/api-schema.yaml
  tasks/
    task-001-users.md
```

## Wave Dependencies

Tasks in Wave N can only depend on tasks in Waves 1 to N-1:

```
Wave 1: task-001, task-002 (no dependencies, run in parallel)
Wave 2: task-003 (depends on task-001, task-002)
Wave 3: task-004 (depends on task-003)
```

### deps vs blocks

- `deps`: Tasks that MUST complete before this task starts
- `blocks`: Tasks that CANNOT start until this task completes

Both express the same relationship from different perspectives:
```yaml
# task-001
blocks: [task-003]

# task-003
deps: [task-001]
```

## Requirements Section

Clear, actionable requirements:

```markdown
## Requirements
- Implement `User` model with fields: `email`, `username`, `password`, `is_active`
- Create `UserSerializer` with all User fields (hide password)
- Implement `UserViewSet` with: list, retrieve, create, update
- Add email validation and uniqueness constraint
- Test coverage: minimum 85%
```

## Checklist Section

Verification criteria:

```markdown
## Checklist
- [ ] Model matches DTO in contracts/types.py
- [ ] API matches schema in contracts/api-schema.yaml
- [ ] pytest apps/users/ passes
- [ ] mypy apps/users/ --strict passes
- [ ] Coverage >= 85%
- [ ] No files modified outside scope
```

## Why Compact Format?

1. **Token efficiency**: Less tokens for agent context
2. **Faster parsing**: YAML frontmatter is standard
3. **Clear boundaries**: Scope section is scannable
4. **Actionable checklist**: Verification is explicit

## Validation Rules

Before using tasks:

- [ ] Every task has unique `id`
- [ ] Every task has `agent` assigned
- [ ] Every task has `skills` list (from agent-skills-mapping.yaml)
- [ ] Every task has `contracts` referenced
- [ ] Every task has BOUNDARY section
- [ ] No circular dependencies in `deps`
- [ ] Wave numbers are sequential (1, 2, 3...)
- [ ] Wave 1 tasks have `deps: []`

## Output Format

The Output Format JSON block is **not included in task files or generated prompts**. It's managed via system prompt by the external execution tool (`cpo` orchestrator).

This ensures consistent JSON output format across all agents without duplicating the schema in every prompt.
