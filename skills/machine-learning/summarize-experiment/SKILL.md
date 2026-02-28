---
name: summarize-experiment
description: Create a lightweight summary of experiment results from a completed (fine-tuned and evaluated) experiment. Use after run-experiment to capture key metrics from the experiment in textual form.   
---

# Summarize Experiment

Generate a `summary.md` file capturing key metrics from a completed experiment. Think R's `summary()` for experiment results.

## Your Task

Create a lightweight summary of experiment results:

1. Parse run status from experiment_summary.yaml
2. Extract final training loss from SLURM stdout
3. Extract accuracy from inspect-ai .eval files
4. Generate summary.md in experiment directory
5. Log the process in logs/summarize-experiment.log

## Prerequisites

- experiment_summary.yaml exists
- At least some runs have completed (partial results acceptable)
- run-experiment has been executed (or manual SLURM jobs run)
- **Conda environment activated** - The `parse_eval_log.py` script requires inspect-ai. Activate the conda environment from `claude.local.md` before running extraction commands.

## Workflow

### 1. Locate Experiment

Find the experiment directory:
- If in an experiment directory (contains experiment_summary.yaml): use current directory
- Otherwise: ask user for path

### 2. Parse Run Status

Read experiment_summary.yaml to identify runs:

**From `runs:` section:**
- `name`: Run identifier
- `type`: "fine-tuned" or "control"
- `model`: Model name
- `parameters`: Dict of hyperparameters (empty for control runs)

**From `evaluation.matrix:` section:**
- `run`: Run name
- `tasks`: List of evaluation task names
- `epochs`: List of epochs to evaluate (null for control runs)

**Determine status by checking filesystem:**
- Fine-tuning: Check for `{output_base}/ck-out-{run_name}/` and SLURM outputs
- Evaluation: Check for `{run_dir}/eval/logs/*.eval` files

### 3. Extract Training Loss

For each COMPLETED fine-tuning run:

1. Find SLURM stdout in the **output directory**:
   - Parse experiment_summary.yaml "Output" section for `output_dir_base`
   - Look in: `{output_dir_base}/ck-out-{run_name}/slurm-*.out`
   - If multiple files, use most recent by modification time
2. Extract final loss using regex: `(\d+)\|(\d+)\|Loss: ([0-9.]+)`
   - Pattern matches: `{epoch}|{step}|Loss: {value}`
   - Take the LAST match to get final loss
   - The step number (group 2) from the last match is the total training steps
3. Record: run_name, final_loss, total_steps, epoch, step

**Note:** Training SLURM outputs are in the output directory, NOT the run directory.

**If SLURM stdout missing:**
- Log warning
- Record "N/A" for loss
- Continue with other runs

### 4. Extract Evaluation Accuracy

For each COMPLETED evaluation:

1. Find .eval files: `{run_dir}/eval/logs/*.eval`
2. For each .eval file, run:
   ```bash
   python tools/inspect/parse_eval_log.py {path}
   ```
3. Parse JSON output for accuracy
4. **Map to epoch using SLURM job names** (see below)
5. For binary tasks, also run `summary_binary.py` to get balanced accuracy and F1
6. Record: run_name, task, epoch, accuracy, balanced_accuracy, f1, samples

**Script output format:**
```json
{
  "status": "success",
  "task": "capitalization",
  "accuracy": 0.85,
  "samples": 100,
  "scorer": "exact_match",
  "model": "..."
}
```

#### Mapping Epochs via SLURM Job Names

The `.eval` files don't currently store epoch information directly. To reliably map each evaluation to its epoch:

1. **Find SLURM output files** in the eval directory: `{run_dir}/eval/slurm-*.out`
2. **Extract job IDs** from filenames (e.g., `slurm-2773062.out` → job ID 2773062)
3. **Query job names via sacct:**
   ```bash
   sacct -j {job_ids} --format=JobID,JobName%50
   ```
4. **Parse epoch from job name** - scaffold-inspect names jobs like `eval-{task}-{run}-ep{N}`:
   - `eval-general_eval-lowlr-ep0` → epoch 0
   - `eval-general_eval-lowlr-ep9` → epoch 9
5. **Extract accuracy from SLURM output:**
   ```bash
   grep -oP 'match/accuracy: \K[0-9.]+' slurm-{jobid}.out
   ```

**Example workflow:**
```bash
# Get job names for all eval jobs
sacct -j 2773062,2773063,2773065 --format=JobID,JobName%50

# Output shows epoch in job name:
# 2773062  eval-general_eval-lowlr-ep0
# 2773063  eval-general_eval-lowlr-ep1
# 2773065  eval-general_eval-lowlr-ep2
```

This approach is reliable because:
- Job names are set by scaffold-inspect and include epoch info
- Works regardless of submission order or timing
- Survives job failures and resubmissions

**If extraction fails:**
- Script returns `{"status": "error", "message": "..."}`
- Log the error
- Record "ERROR" for accuracy
- Continue with other evaluations

#### Computing Balanced Accuracy and F1 (Binary Classification)

For binary classification tasks (0/1 targets), use `summary_binary.py` to compute additional metrics:

```bash
python tools/inspect/summary_binary.py {path_to_eval_file} --json
```

**JSON output format:**
```json
{
  "status": "success",
  "path": "/path/to/file.eval",
  "samples": 100,
  "accuracy": 0.85,
  "balanced_accuracy": 0.83,
  "f1": 0.82,
  "precision_1": 0.80,
  "recall_1": 0.84,
  "recall_0": 0.82,
  "confusion_matrix": {"tp": 42, "tn": 43, "fp": 7, "fn": 8, "other": 0}
}
```

**Why these metrics matter for imbalanced data:**
- **Balanced Accuracy** = (Recall_0 + Recall_1) / 2 — not inflated by majority class
- **F1 Score** = harmonic mean of precision and recall — penalizes class imbalance

**Note:** For non-binary tasks, only accuracy is reported (Bal. Acc and F1 shown as "-").

### 5. Generate summary.md

Create `{experiment_dir}/summary.md` with the following structure:

```markdown
# Experiment Summary

**Experiment:** `{experiment_name}` | **Generated:** {timestamp} | **Status:** {X}/{Y} complete

## Run Status

| Run | Type | Fine-tuning | Evaluation |
|-----|------|-------------|------------|
| rank4_lr1e-5 | Fine-tuned | COMPLETED | COMPLETED |
| rank8_lr1e-5 | Fine-tuned | COMPLETED | COMPLETED |
| base_model | Control | N/A | COMPLETED |

## Training Results

| Run | Final Loss | Total Steps | Epochs | Duration |
|-----|------------|-------------|--------|----------|
| rank4_lr1e-5 | 0.234 | 250 | 2 | 8m 15s |
| rank8_lr1e-5 | 0.198 | 250 | 2 | 9m 02s |

**Notes:**
- Base model runs have no training loss (control)
- Duration from SLURM elapsed time (if available)

## Evaluation Results

| Run | Task | Epoch | Accuracy | Bal. Acc | F1 | Samples |
|-----|------|-------|----------|----------|------|---------|
| rank4_lr1e-5 | capitalization | 0 | 0.85 | 0.83 | 0.82 | 100 |
| rank4_lr1e-5 | capitalization | 1 | 0.88 | 0.86 | 0.85 | 100 |
| rank8_lr1e-5 | capitalization | 0 | 0.82 | 0.80 | 0.78 | 100 |
| rank8_lr1e-5 | capitalization | 1 | 0.91 | 0.89 | 0.88 | 100 |
| base_model | capitalization | - | 0.45 | 0.50 | 0.31 | 100 |

**Best performing:** rank8_lr1e-5 (epoch 1) with 89% balanced accuracy

## Incomplete Runs

| Run | Stage | Status | Notes |
|-----|-------|--------|-------|
| rank16_lr1e-5 | Fine-tuning | FAILED | Check slurm-12345.out |

## Next Steps

1. View detailed evaluation results: `inspect view --port=$(get_free_port)`
2. Export raw data: `inspect log export {run_dir}/eval/logs/*.eval --format csv`
3. Full analysis: `analyze-experiment` (when available)

---
*Generated by summarize-experiment skill*
```

### 6. Create Log

Document the process in `{experiment_dir}/logs/summarize-experiment.log`.

See [logging.md](logging.md) for action types and format.

## Error Handling

### If SLURM stdout missing
- Log warning with action type `EXTRACT_LOSS`
- Record "N/A" for loss in summary
- Continue with other runs

### If .eval file cannot be parsed
- Log error with file path
- Record "ERROR" for accuracy in summary
- Continue with other evaluations

### If all runs failed
- Generate summary noting all failures
- Include failure states in "Incomplete Runs" section
- Suggest troubleshooting steps

### If partial results
- Generate summary with available data
- Clearly indicate which runs are missing in "Incomplete Runs" section
- Still identify best performing run from available data

## Idempotency

Running summarize-experiment multiple times overwrites summary.md. This is intentional:
- Allows re-running after fixing failed runs
- Summary always reflects current state

## Output Files

```
{experiment_dir}/
├── summary.md                    # Human-readable summary (new)
└── logs/
    └── summarize-experiment.log  # Process log (new)
```

## Relationship to Other Skills

- **After:** run-experiment (or manual execution)
- **Before:** analyze-experiment (when available)
- **Optional hook:** run-experiment can invoke this at completion

## Future Compatibility

When analyze-experiment is built, summarize-experiment can either:
- Remain as a quick summary option (text only, no plots)
- Be deprecated in favor of richer output
- Become a first stage that analyze-experiment builds upon
