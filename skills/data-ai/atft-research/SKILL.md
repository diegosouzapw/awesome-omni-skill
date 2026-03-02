---
name: atft-research
description: Drive quantitative analysis, factor diagnostics, and reporting for ATFT-GAT-FAN outputs.
proactive: true
---

# ATFT Research Skill

## Mission
- Quantify performance (Sharpe, RankIC, hit ratio) across horizons and cohorts.
- Inspect feature contributions, leakage risks, and stability of graph-based factors.
- Produce stakeholder-ready artifacts (reports, dashboards, notebooks).

## Engagement Signals
- Requests to “analyze results”, “generate research report”, “compare to baseline”, “explain factor drift”.
- Need to validate new model output or dataset revisions before release.
- Desire for exploratory notebooks, plots, or KPI dashboards.

## Baseline Workflow
1. Confirm availability of latest run: `ls -lt runs | head`.
2. Load metrics: `python scripts/research/summarize_run.py --run runs/<timestamp>`.
3. Compute comparison vs baseline:
   - `make research-baseline RUN=runs/<timestamp>` — compares to curated benchmark.
   - `make research-plus RUN=runs/<timestamp>` — full bundle (feature importance, turnover, drawdowns).
4. Plot diagnostics:
   - `python scripts/research/plot_metrics.py --run runs/<timestamp> --horizons 1 5 10 20`.
   - `python scripts/research/graph_analytics.py --dataset output/ml_dataset_latest_full.parquet`.
5. Publish:
   - Output stored in `reports/<timestamp>/`.
   - Update `docs/research/weekly_digest.md`.

## Specialized Analyses

### Factor Stability / Drift
- `python scripts/research/factor_drift.py --window 60 --features top50`.
- `python scripts/research/check_leakage.py --dataset output/ml_dataset_latest_full.parquet`.
- Alert when drift Z-score > 2.3 or leakage detection fails; escalate to pipeline skill to rebuild dataset.

### Regime Segmentation
- `python scripts/research/regime_detector.py --regimes 4 --method gaussian_hmm`.
- `python scripts/research/evaluate_by_regime.py --run runs/<timestamp> --regime-file output/regimes/latest.parquet`.

### Risk & Compliance
- `python scripts/research/limit_checker.py --run runs/<timestamp>` — verifies VAR, exposure, and shorting constraints.
- `pytest tests/research/test_safety_constraints.py -k exposure` if guard fails.

## Visualization Arsenal
- `make research-report FACTORS=returns_5d,ret_1d_vs_sec HORIZONS=1,5,10,20`.
- `python scripts/research/notebooks/render.py docs/notebooks/performance_atlas.ipynb`.
- `python tools/chart_creator.py --input reports/<timestamp>/summary.json --output outputs/figures/`.

## Data Sources
- Primary dataset: `output/ml_dataset_latest_full.parquet`
- Model outputs: `runs/<timestamp>/predictions.parquet`
- Feature metadata: `dataset_features_detail.json`
- Market benchmarks: `data/benchmarks/nikkei225.parquet`

## Reporting Standards
- Include KPIs: Sharpe, RankIC, Top/Bottom decile returns, MaxDD, Turnover.
- Break out metrics by sector (33 TSE industry codes) and market cap terciles.
- Document experiment context: dataset version hash, training config file, git SHA.
- Archive final report under `docs/research/archive/<YYYY-MM-DD>_run_<timestamp>.md`.

## Codex Collaboration
- Engage `./tools/codex.sh "Generate new factor hypothesis from latest run"` to synthesize research leads using Codex search + reasoning stack.
- Run `codex exec --model gpt-5-codex "Summarize regime analysis findings in docs/research/weekly_digest.md"` for automated reporting drafts.
- Feed Codex-generated notebooks or scripts back through this skill for validation before sharing with stakeholders.
