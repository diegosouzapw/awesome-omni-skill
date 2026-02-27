---
name: workflow-orchestration
description: Design and implement DAG-based workflows with parallel execution, retries, and error handling. Use when building complex multi-step agent workflows.
license: Apache-2.0
compatibility: Python 3.10+, AsyncIO
metadata:
  author: paracle-core-team
  version: "1.0.0"
  category: automation
  level: advanced
  display_name: "Workflow Orchestration"
  tags:
    - workflows
    - dag
    - orchestration
    - async
  capabilities:
    - workflow_design
    - dag_execution
    - parallel_processing
    - error_handling
allowed-tools: Read Write Bash(python:*)
---

# Workflow Orchestration Skill

## When to use this skill

Use when:

- Building multi-step agent workflows
- Designing task dependencies (DAG)
- Implementing parallel execution
- Handling workflow errors and retries
- Orchestrating multiple agents

## Workflow Structure

```yaml
# .parac/workflows/data-pipeline.yaml
name: data-pipeline
description: Extract, transform, and load data

steps:
  - id: extract
    agent: data-extractor
    input:
      source: database

  - id: transform
    agent: data-transformer
    depends_on: [extract]
    input:
      rules: transformation_rules.json

  - id: load
    agent: data-loader
    depends_on: [transform]
    input:
      destination: warehouse

  - id: validate
    agent: data-validator
    depends_on: [load]

  - id: notify
    agent: notifier
    depends_on: [validate]
    on_error: continue  # Don't fail workflow if notification fails

retry:
  max_attempts: 3
  backoff: exponential

timeout: 3600  # 1 hour
```

## DAG Construction

```python
from paracle_orchestration import DAG, Step

# Create DAG
dag = DAG(name="data-pipeline")

# Add steps
extract = Step(
    id="extract",
    agent_id="data-extractor",
    input={"source": "database"},
)

transform = Step(
    id="transform",
    agent_id="data-transformer",
    depends_on=["extract"],
    input={"rules": "transformation_rules.json"},
)

load = Step(
    id="load",
    agent_id="data-loader",
    depends_on=["transform"],
    input={"destination": "warehouse"},
)

# Add to DAG
dag.add_steps([extract, transform, load])

# Validate DAG (check for cycles)
dag.validate()
```

## Parallel Execution

```python
from paracle_orchestration import DAG, Step

dag = DAG(name="parallel-processing")

# Steps that can run in parallel
fetch_data = Step(id="fetch-data", agent_id="fetcher")

# These three steps have no dependencies, so they run in parallel
process_a = Step(id="process-a", agent_id="processor-a", depends_on=["fetch-data"])
process_b = Step(id="process-b", agent_id="processor-b", depends_on=["fetch-data"])
process_c = Step(id="process-c", agent_id="processor-c", depends_on=["fetch-data"])

# Merge results after all processing is done
merge = Step(
    id="merge",
    agent_id="merger",
    depends_on=["process-a", "process-b", "process-c"],
)

dag.add_steps([fetch_data, process_a, process_b, process_c, merge])

# Execution order:
# 1. fetch-data
# 2. process-a, process-b, process-c (parallel)
# 3. merge
```

## Error Handling & Retries

```python
from paracle_orchestration import Step, RetryPolicy

# Retry configuration
retry_policy = RetryPolicy(
    max_attempts=3,
    backoff="exponential",  # 1s, 2s, 4s
    retry_on=[TimeoutError, ConnectionError],  # Only retry these
)

step = Step(
    id="api-call",
    agent_id="api-agent",
    retry_policy=retry_policy,
    timeout=30,  # 30 seconds per attempt
)

# Custom error handling
step = Step(
    id="critical-step",
    agent_id="critical-agent",
    on_error="fail",  # fail (default), continue, retry
)

step = Step(
    id="optional-step",
    agent_id="optional-agent",
    on_error="continue",  # Don't fail workflow if this fails
)
```

## Best Practices

1. **Design clear DAGs** - Visualize dependencies
2. **Use parallel execution** - For independent steps
3. **Handle errors gracefully** - Retries and fallbacks
4. **Monitor workflows** - Track progress and failures
5. **Test workflows** - Unit and integration tests
6. **Version workflows** - Track changes over time
7. **Implement rollbacks** - For critical operations

## Common Patterns

### Fan-out / Fan-in

```python
# One step spawns multiple parallel steps
fetch = Step(id="fetch", ...)
process_1 = Step(id="p1", depends_on=["fetch"])
process_2 = Step(id="p2", depends_on=["fetch"])
process_3 = Step(id="p3", depends_on=["fetch"])
merge = Step(id="merge", depends_on=["p1", "p2", "p3"])
```

### Sequential Pipeline

```python
# Steps execute one after another
step1 = Step(id="s1", ...)
step2 = Step(id="s2", depends_on=["s1"])
step3 = Step(id="s3", depends_on=["s2"])
```

## Resources

- Orchestration Engine: `packages/paracle_orchestration/`
- DAG Implementation: `packages/paracle_orchestration/dag.py`
- Workflow Examples: `content/examples/workflows/`
- Engine Documentation: `content/docs/technical/workflow-orchestration.md`
