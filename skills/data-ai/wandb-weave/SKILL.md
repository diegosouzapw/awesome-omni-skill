---
name: wandb-weave
description: Query and analyze W&B experiment data and Weave LLM traces using Python scripts. Use when working with Weights & Biases data, including (1) querying ML experiment runs, metrics, and hyperparameters, (2) analyzing LLM traces and evaluations, (3) creating W&B reports, (4) listing projects and entities.
---

# W&B & Weave Data Tools

Python scripts to query W&B experiment data and Weave LLM traces.

## Prerequisites

```bash
pip install wandb weave
export WANDB_API_KEY="your-api-key"
```

## Workflow Decision Tree

```
What do you want to do?
│
├─ Query ML experiments (runs, metrics, sweeps)
│  └─ Run: scripts/query_runs.py
│
├─ Analyze LLM traces
│  ├─ Need trace data? → scripts/query_traces.py
│  └─ Just need count? → scripts/query_traces.py --count-only
│
├─ Create a report
│  └─ Run: scripts/create_report.py
│
└─ List projects
   └─ Run: scripts/list_projects.py
```

## Scripts

### query_runs.py

Query W&B experiment runs with filtering and sorting.

```bash
# List recent runs
python scripts/query_runs.py <entity> <project> --limit 10

# Filter by state
python scripts/query_runs.py my-team my-project --state finished

# Sort by metric (best first)
python scripts/query_runs.py my-team my-project --sort "-summary_metrics.accuracy"

# Custom filter
python scripts/query_runs.py my-team my-project --filter '{"config.model": "gpt-4"}'
```

| Option | Description |
|--------|-------------|
| `--limit N` | Max results (default: 20) |
| `--state` | Filter: running, finished, crashed, failed |
| `--sort` | Sort field (prefix `-` for desc) |
| `--filter` | JSON filter dict |
| `--output` | json or table |

### query_traces.py

Query Weave LLM traces with filtering.

```bash
# List recent traces
python scripts/query_traces.py <entity> <project> --limit 50

# Filter by status
python scripts/query_traces.py my-team my-project --status success

# Filter by model
python scripts/query_traces.py my-team my-project --model gpt-4o

# Find slow traces
python scripts/query_traces.py my-team my-project --min-latency 5000

# Count only
python scripts/query_traces.py my-team my-project --count-only
```

| Option | Description |
|--------|-------------|
| `--limit N` | Max results (default: 50) |
| `--status` | Filter: success, error, running |
| `--model` | Filter by model name |
| `--min-latency` | Min latency in ms |
| `--roots-only` | Only root traces |
| `--count-only` | Return count, not data |
| `--filter` | Custom JSON filter (advanced) |

For advanced filter syntax (when `--status`, `--model`, `--min-latency` are not enough), see [references/weave_filters.md](references/weave_filters.md).

### list_projects.py

List entities and projects.

```bash
# List all entities and projects
python scripts/list_projects.py

# List projects for specific entity
python scripts/list_projects.py my-team

# List entities only
python scripts/list_projects.py --entities-only
```

### create_report.py

Create W&B reports programmatically.

```bash
# Create with inline content
python scripts/create_report.py my-team my-project "Weekly Summary" \
    --content "## Results\n\n- Accuracy: 95%\n- Loss: 0.05"

# Create from markdown file
python scripts/create_report.py my-team my-project "Analysis" --file report.md

# With description
python scripts/create_report.py my-team my-project "Q4 Report" \
    --content "..." --description "Quarterly analysis"
```

## Common Workflows

### Analyze Experiment Performance

```bash
# 1. Find your project
python scripts/list_projects.py my-team

# 2. Query best runs
python scripts/query_runs.py my-team my-project \
    --state finished \
    --sort "-summary_metrics.accuracy" \
    --limit 10

# 3. Create summary report
python scripts/create_report.py my-team my-project "Best Runs" \
    --content "## Top 10 Runs by Accuracy\n\n..."
```

### Debug LLM Application

```bash
# 1. Count errors
python scripts/query_traces.py my-team my-project --status error --count-only

# 2. Get error details
python scripts/query_traces.py my-team my-project --status error --limit 20

# 3. Find slow traces
python scripts/query_traces.py my-team my-project --min-latency 5000
```

## Resources

- **Advanced trace filters**: Load [references/weave_filters.md](references/weave_filters.md) when `--filter` option is needed for complex queries not covered by built-in options
