---
name: atft-pipeline
description: Manage J-Quants ingestion, feature graph generation, and cache hygiene for the ATFT-GAT-FAN dataset pipeline.
proactive: true
---

# ATFT Pipeline Skill

## Mission
- Provision fresh or historical parquet datasets for ATFT-GAT-FAN with GPU-accelerated ETL.
- Maintain deterministic feature graphs (approx. 395 engineered factors, 307 active).
- Guard J-Quants API quota, credential sanity, and cache health to prevent training stalls.

## When To Engage
- Any request mentioning dataset builds, ETL, J-Quants, cache, RAPIDS/cuDF, or feature graph refresh.
- Pre-training sanity checks (“ensure latest dataset”, “verify cache integrity”).
- Recovery tasks (“resume interrupted dataset job”, “clean corrupted cache shards”).

## Preflight Checklist
- Confirm `nvidia-smi` reports at least one free A100 80GB GPU; fallback to CPU only if GPU unavailable.
- Validate credentials: `.env` contains `JQUANTS_AUTH_EMAIL/PASSWORD` and `JQUANTS_PLAN_TIER`.
- Ensure `python -m pip install -e .` already executed (dependencies + entry points).
- Check latest health snapshot: `tools/project-health-check.sh --section dataset`.
- Inspect existing dataset for reuse: `ls -lh output/ml_dataset_latest_full.parquet`.

## Core Playbooks

### 1. Background Five-Year Refresh (default)
1. `make dataset-check-strict` — GPU + secrets verification.
2. `make dataset-bg START=<optional> END=<optional>` — SSH-safe background run with logging in `_logs/dataset`.
3. `tail -f _logs/dataset/*.log` — monitor progress (auto prints PID + PGID).
4. `make cache-stats` — ensure cache hit-rate & size in expected bounds (<2.5 TB).
5. `python scripts/pipelines/run_full_dataset.py --dry-run` — confirm metadata integrity without rebuild.

### 2. Hotfix / Forced Refresh
1. `make dataset-gpu-refresh START=YYYY-MM-DD END=YYYY-MM-DD` — bypasses cached parquet + API throttle aware.
2. `make datasets-prune` — keep latest dataset generation only.
3. `make cache-prune CACHE_TTL_DAYS=90` — evict stale graph shards to recover disk.

### 3. Resource-Constrained Fallback
1. `make dataset-check` — relaxed diagnostics (CPU acceptable).
2. `make dataset-cpu START=YYYY-MM-DD END=YYYY-MM-DD` — chunked Pandas path.
3. `make dataset-safe-resume` — resume from last safe checkpoint if memory pressure triggered fallback.

### 4. Graph Feature Investigation
1. `python scripts/pipelines/run_full_dataset.py --inspect-graph --start YYYY-MM-DD --end YYYY-MM-DD`.
2. `python -c "import polars as pl; df = pl.read_parquet('output/ml_dataset_latest_full.parquet'); print(df.select(pl.all().is_null().sum()))"` — null audit.
3. `make cache-monitor` — per-window edge density + overlap stats.

## Observability Hooks
- `_logs/dataset/` for job logs, `cache/*.json` metadata for cache.
- `ml_dataset_latest_full_metadata.json` for column coverage & horizon alignment.
- `benchmark_output/dataset_timestamps.json` to confirm pipeline duration vs baseline (target: <42m GPU path).

## Failure Triage
- **Credential errors** → run `python scripts/pipelines/run_full_dataset.py --auth-test`.
- **CUDA OOM** → rerun with `make dataset-safe` (40GB RMM pool pre-configured).
- **API rate limits** → throttle via `make dataset-gpu REFRESH_THROTTLE=1`.
- **Corrupted parquet** → `make dataset-rebuild` then `python tools/parquet_validator.py output/ml_dataset_latest_full.parquet`.

## Codex Collaboration
- Escalate complex ETL debugging or architectural refactors via `./tools/codex.sh "Diagnose dataset pipeline bottleneck"` (leverages OpenAI Codex deep reasoning).
- For long-running autonomous maintenance, schedule `./tools/codex.sh --max --exec "Perform full dataset pipeline audit"` off-hours (uses `.mcp.json` from the Codex repo for filesystem/git context).
- When Codex proposes changes, sync learnings back here and refresh dataset runbooks if any commands or defaults shift.

## Handoff Notes
- Always update `dataset_features_detail.json` if schema changes.
- Announce new dataset snapshot in `EXPERIMENT_STATUS.md` with generation timestamp and settings.
- Surface anomalies (missing tickers, new features) via `docs/data_quality/` reports.
