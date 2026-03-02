---
name: atft-training
description: Run and monitor ATFT-GAT-FAN training loops, hyper-parameter sweeps, and safety modes on A100 GPUs.
proactive: true
---

# ATFT Training Skill

## Mission
- Launch production-grade training for the Graph Attention Network forecaster with correct dataset/version parity.
- Tune hyper-parameters (LR, batch size, horizons, latent dims) exploiting 80GB GPU headroom.
- Safely resume, stop, or monitor long-running jobs and record experiment metadata.

## Engagement Triggers
- Requests to “train”, “fine-tune”, “HP optimize”, “resume training”, or “monitor training logs”.
- Need to validate new dataset compatibility with model code.
- Investigations into training stalls, divergence, or GPU under-utilization.

## Preflight Safety Checks
1. Dataset freshness: `ls -lh output/ml_dataset_latest_full.parquet` then `python scripts/utils/dataset_guard.py --assert-recency 72`.
2. Environment health: `tools/project-health-check.sh --section training`.
3. GPU allocation: `nvidia-smi --query-gpu=utilization.gpu,memory.used --format=csv` (target >60% util, <76GB used baseline).
4. Git hygiene: `git status --short` ensure working tree state is understood (avoid accidental overrides during long runs).

## Training Playbooks

### 1. Production Optimized Training (default 120 epochs)
1. `make train-optimized DATASET=output/ml_dataset_latest_full.parquet` — compiles TorchInductor + FlashAttention2.
2. `make train-monitor` — tails `_logs/training/train-optimized.log`.
3. `make train-status` — polls background process; ensure ETA < 7h.
4. Post-run validation:
   - `python scripts/eval/aggregate_metrics.py runs/latest` — compute Sharpe, RankIC, hit ratios.
   - Update `results/latest_training_summary.md`.

### 2. Quick Validation / Smoke
1. `make train-quick EPOCHS=3` — run in foreground.
2. `python scripts/smoke_test.py --max-epochs 1 --subset 512` for additional regression guard.
3. `pytest tests/integration/test_training_loop.py::test_forward_backward` if suspicious gradients.

### 3. Safe Mode / Debug
1. `make train-safe` — disables compile, single-worker dataloading.
2. `make train-stop` if hung jobs detected (consult `_logs/training/pids/`).
3. `python scripts/integrated_ml_training_pipeline.py --profile --epochs 2 --no-compile` — capture flamegraph to `benchmark_output/`.

### 4. Hyper-Parameter Exploration
1. Ensure `mlflow` backend running if required (`make mlflow-up`).
2. `make hpo-run HPO_TRIALS=24 HPO_STUDY=atft_prod_lr_sched` — uses Optuna integration.
3. `make hpo-status` — track trial completions.
4. Promote winning config → `configs/training/atft_prod.yaml` and document in `EXPERIMENT_STATUS.md`.

## Monitoring & Telemetry
- Training logs: `_logs/training/*.log` (includes gradient norms, learning rate schedule, GPU temp).
- Metrics JSONL: `runs/<timestamp>/metrics.jsonl`.
- Checkpoint artifacts: `models/checkpoints/<timestamp>/epoch_###.pt`.
- GPU telemetry: `watch -n 30 nvidia-smi` or `python tools/gpu_monitor.py --pid $(cat _logs/training/pids/train.pid)`.

## Failure Handling
- **NaN loss** → run `make train-safe` with `FP32=1`, inspect `runs/<ts>/nan_batches.json`.
- **Slow dataloading** → regenerate dataset with `make dataset-gpu GRAPH_WINDOW=90` or enable PyTorch compile caching.
- **OOM** → set `GRADIENT_ACCUMULATION_STEPS=2` or reduce `BATCH_SIZE`; confirm memory fragments via `python tools/gpu_memory_report.py`.
- **Divergent metrics** → verify `configs/training/schedule.yaml`; run `pytest tests/unit/test_loss_functions.py`.

## Codex Collaboration
- Invoke `./tools/codex.sh --max "Design a new learning rate policy for ATFT-GAT-FAN"` when novel optimizer or architecture strategy is required.
- Use `codex exec --model gpt-5-codex "Analyze runs/<timestamp>/metrics.jsonl and suggest fixes"` for automated postmortems.
- Share Codex-discovered tuning insights in `results/training_runs/` and update config files/documents accordingly.

## Post-Training Handoff
- Persist summary in `results/training_runs/<timestamp>.md` noting dataset hash and commit SHA.
- Push model weights to `models/artifacts/` with naming `gatfan_<date>_Sharpe<score>.pt`.
- Notify research team via `docs/research/changelog.md`.
