---
name: workflows-expert
description: Activate when requests involve workflow execution, CI/CD pipelines, git automation, or multi-step task orchestration. This skill provides workflows-mcp MCP server integration with tag-based workflow discovery, DAG-based execution, and variable syntax expertise. Trigger on phrases like "run workflow", "execute workflow", "orchestrate tasks", "automate CI/CD", or "workflow information".
---

# Workflows MCP Expert Skill

Activate when requests involve workflow execution, multi-step task orchestration, or CI/CD automation using the workflows-mcp MCP server.

## Activation Triggers

Activate when user requests mention:

- "run workflow", "execute workflow", "list workflows"
- "orchestrate tasks", "multi-step process", "DAG execution"
- "CI/CD pipeline", "git automation", "automated testing"
- "workflow information", "workflow details"
- Task coordination with dependencies
- "Chain commands", "automate workflow"

Common user phrases: "run tests before deploying", "create build pipeline", "execute only if previous succeeds", "run tasks in parallel"

## Core Workflow Pattern

### Standard execution: Discover → Inspect → Execute

Always use tag-based discovery instead of guessing workflow names. Inspect workflows before execution to understand required inputs.

### Step 1: Discover by Tags

Use `list_workflows` with tags (AND logic - workflows must have ALL tags):

```text
Tool: list_workflows
Parameters:
  tags: ['python', 'ci']
  format: 'markdown'
```

**Common tag combinations:**

- Python: `['python']`, `['python', 'testing']`, `['python', 'ci']`
- Git: `['git']`, `['git', 'commit']`, `['git', 'checkout']`
- CI/CD: `['ci']`, `['ci', 'deployment']`
- TDD: `['tdd']`, `['tdd', 'phase1']`

### Step 2: Inspect Workflow

Call `get_workflow_info` before executing to understand inputs, outputs, and structure:

```text
Tool: get_workflow_info
Parameters:
  workflow: 'python-ci-pipeline'
  format: 'markdown'
```

### Step 3: Execute Workflow

```text
Tool: execute_workflow
Parameters:
  workflow: 'python-ci-pipeline'
  inputs: {project_path: '/path/to/project'}
  response_format: 'minimal'
```

**Response status values:**

- `success` - Workflow completed successfully
- `failure` - Workflow failed (check `error` field)
- `paused` - Workflow paused for user input (use `resume_workflow`)

## Available MCP Tools

### Discovery & Information

**list_workflows(tags, format)** - Discover workflows by tags

- Filter with AND logic: `tags=['python', 'testing']`
- Returns workflow names only

**get_workflow_info(workflow, format)** - Get detailed workflow metadata

- Shows inputs, outputs, blocks, dependencies
- Call before executing

### Execution

**execute_workflow(workflow, inputs, response_format)** - Execute registered workflow

- Provide required inputs from `get_workflow_info`
- Use `response_format: 'minimal'` (default) or `'detailed'` (debugging only)

**execute_inline_workflow(workflow_yaml, inputs, response_format)** - Execute YAML directly

- For one-off or custom workflows
- Validate first with `validate_workflow_yaml`

**validate_workflow_yaml(yaml_content)** - Validate workflow before execution

- Catches syntax errors early
- Always validate inline workflows

### Checkpoint Management

**resume_workflow(checkpoint_id, response, response_format)** - Resume paused workflow

- For interactive workflows with Prompt blocks
- Provide user response to continue

**list_checkpoints(workflow_name, format)** - View saved checkpoints

**get_checkpoint_info(checkpoint_id, format)** - Inspect checkpoint details

**delete_checkpoint(checkpoint_id)** - Clean up old checkpoints

## Variable Syntax Quick Reference

All variables use four-namespace architecture:

**Inputs:** `{{inputs.project_name}}`, `{{inputs.workspace}}`

**Metadata:** `{{metadata.workflow_name}}`, `{{metadata.start_time}}`

**Blocks:** `{{blocks.run_tests.outputs.exit_code}}`, `{{blocks.test.succeeded}}`

**Status shortcuts (use for 90% of conditionals):**

- `{{blocks.test.succeeded}}` - True if completed successfully
- `{{blocks.build.failed}}` - True if failed
- `{{blocks.optional.skipped}}` - True if skipped

For detailed variable syntax, load `./references/variable-syntax.md`.

## Essential Patterns

### Conditional Execution

```yaml
blocks:
  - id: run_tests
    type: Shell
    inputs:
      command: pytest tests/

  - id: deploy
    type: Shell
    inputs:
      command: ./deploy.sh
    condition: "{{blocks.run_tests.succeeded}}"
    depends_on: [run_tests]
```

### Workflow Composition

```yaml
blocks:
  - id: ci_pipeline
    type: Workflow
    inputs:
      workflow: "python-ci-pipeline"
      inputs:
        project_path: "{{inputs.workspace}}"
```

### Parallel Execution

Blocks without dependencies run in parallel automatically.

## Best Practices

1. **Use tag-based discovery** - Call `list_workflows(tags=[...])` instead of guessing names
2. **Inspect before executing** - Call `get_workflow_info()` to understand requirements
3. **Use status shortcuts** - Prefer `.succeeded` over `.outputs.exit_code == 0`
4. **Minimal response format** - Use `response_format='minimal'` unless debugging
5. **Validate inline workflows** - Use `validate_workflow_yaml()` before `execute_inline_workflow()`
6. **Explicit namespaces** - Always use `inputs.*`, `blocks.*`, `metadata.*`

## Reference Documentation

Load these files as needed using the Read tool:

### Variable Syntax Reference

[`./references/variable-syntax.md`](./references/variable-syntax.md)

**Load when:** Resolving variable syntax errors, understanding namespace architecture, debugging variable resolution, or learning advanced patterns.

**Contains:** Complete variable resolution rules, recursive resolution, cross-block references, common mistakes, debugging techniques.

### Block Executors Reference

[`./references/block-executors.md`](./references/block-executors.md)

**Load when:** Understanding available block types (Shell, Workflow, CreateFile, ReadFile, etc.), checking input/output specs, learning execution patterns, troubleshooting block issues.

**Contains:** Complete block type reference, input/output specifications, execution states, status vs outcome distinction, patterns and troubleshooting.

### Complete Workflow Examples

[`./references/examples.md`](./references/examples.md)

**Load when:** Implementing complex multi-stage workflows, building parallel execution pipelines, creating file processing workflows, designing interactive approval workflows, learning advanced patterns.

**Contains:** Full workflow examples with documentation, multi-stage deployments, parallel testing with aggregation, file processing pipelines, interactive deployments.

## Example Workflow Templates

**Directory:** `./examples/`

**Available templates:**

- `./examples/simple-ci-pipeline.yaml` - Basic CI pipeline with sequential execution
- `./examples/conditional-deploy.yaml` - Environment-based deployment with conditions
- `./examples/parallel-testing.yaml` - Parallel test execution with result aggregation

**Usage:** Copy and modify YAML templates for custom workflows. Execute using `execute_inline_workflow` tool.

## Troubleshooting Quick Guide

**Variable errors:** Verify namespace (`inputs.*`, `blocks.*`, `metadata.*`), check block ID case-sensitivity, ensure `depends_on` for output references

**Execution failures:** Check `error` field, use `response_format: "detailed"` for debugging, verify required inputs, validate condition syntax

**Workflow not found:** Use `list_workflows()` to see available workflows, check name spelling (case-sensitive), verify MCP connection

## Summary

The workflows-mcp server enables workflow orchestration through:

1. **Tag-based discovery** - Find workflows by purpose
2. **Workflow inspection** - Understand requirements before execution
3. **DAG-based execution** - Automatic dependency resolution and parallel execution
4. **Conditional logic** - Boolean shortcuts for clean conditions
5. **Workflow composition** - Build complex pipelines from reusable workflows

### Key Pattern

Always follow: Discover → Inspect → Execute
