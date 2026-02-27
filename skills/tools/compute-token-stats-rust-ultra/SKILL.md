---
name: compute-token-stats-rust-ultra
description: Ultra-fast token+cost computation for Codex + Claude SSE logs (Rust) plus Python anchor/fast comparisons; includes build+run+benchmark guide.
---

# Compute Token Stats (Rust Ultra)
*[Created by Codex: 019beb87-b904-7a81-a7c9-5b5d5575d2f7 2026-01-23]*
*[Edited by Claude: 9d1228e9-aabe-4ab3-9b9c-c0c1d1f64ebc 2026-01-27]*

## ⚠️ CRITICAL: Agent Instructions

**If the user asked you to load this skill, you MUST use it for token cost computation.**

1. **ALWAYS use the Rust binaries** - they are ultra-fast and purpose-built for this task
2. **NEVER write custom Python** to parse tokens - the Rust tool already does this
3. If you think you need Python, **STOP and ask the user first** - 99% of the time the answer is NO
4. **These binaries are PIPE-ONLY** - they read from stdin, not files directly

### ⚠️ NEVER RUN STANDALONE - WILL HANG!

```bash
# WRONG - will hang waiting for stdin forever!
/tmp/.claude/skills/compute-token-stats-rust-ultra/rust/target/release/token_costs

# CORRECT - always pipe data to it
cat file.jsonl | /tmp/.claude/skills/compute-token-stats-rust-ultra/rust/target/release/token_costs
```

### Filtering by SID

To compute tokens for specific sessions, filter first then pipe:

```bash
# Filter by single SID
grep '"sid":"abc123"' ~/centralized-logs/claude/sse_lines.jsonl | \
  /tmp/.claude/skills/compute-token-stats-rust-ultra/rust/target/release/claude_token_costs

# Filter by multiple SIDs (use grep -F with file)
grep -F -f /tmp/sids.txt ~/centralized-logs/codex/sse_lines.jsonl | \
  /tmp/.claude/skills/compute-token-stats-rust-ultra/rust/target/release/token_costs
```

---

Compute token totals + estimated costs from massive JSONL SSE logs:
- Codex: `~/centralized-logs/codex/sse_lines.jsonl`
- Claude: `~/centralized-logs/claude/sse_lines.jsonl`

This skill includes:
- **Rust Ultra** (streaming, byte-scan, no JSON DOM): `rust/` → `token_costs` + `claude_token_costs`
- **Python anchor** (ground truth): `python/run_compute_token_costs_anchor.py` → runs `~/swe/token-computation/compute_token_costs.py`
- **Python fast** (byte-scan): `python/compute_{codex,claude}_token_costs_fast_pipe.py`

## Build (Rust Ultra)

```bash
cd /tmp/.claude/skills/compute-token-stats-rust-ultra/rust && \
  cargo build --release --bins
```

## Run (Rust Ultra)

Codex:
```bash
cat ~/centralized-logs/codex/sse_lines.jsonl | \
  /tmp/.claude/skills/compute-token-stats-rust-ultra/rust/target/release/token_costs
```

Claude:
```bash
cat ~/centralized-logs/claude/sse_lines.jsonl | \
  /tmp/.claude/skills/compute-token-stats-rust-ultra/rust/target/release/claude_token_costs
```

## Run (Python anchor / ground truth)

Generate a pricing snapshot (recommended; makes runs reproducible):
```bash
python /Users/sotola/swe/token-computation/pull_pricing.py \
  --providers openai,anthropic --timeout 20 --snapshot-dir ~/centralized-logs/pull-pricing/snapshots
```

Then run:
```bash
python /tmp/.claude/skills/compute-token-stats-rust-ultra/python/run_compute_token_costs_anchor.py \
  --codex ~/centralized-logs/codex/sse_lines.jsonl \
  --claude ~/centralized-logs/claude/sse_lines.jsonl \
  --pricing-snapshot ~/centralized-logs/pull-pricing/snapshots/<pick-one>.json
```

## Quick benchmark + cost diff (%)

This compares **Rust Ultra** (streaming stdin) vs **Python anchor** on the same datasets and prints:
- seconds
- lines/sec (Python uses file line count / wall time)
- cost diff (%)

```bash
python /tmp/.claude/skills/compute-token-stats-rust-ultra/python/bench_rust_vs_anchor.py
```

## Notes on “unknown” Codex sessions

When `turn.session_configured` is missing for some `sid` (log truncation / invalid state machine), per-`sid` model attribution is unknowable.

- The **anchor** (`compute_token_costs.py`) prices Codex totals with a single detected `codex.model`.
- Rust Ultra keeps per-model attribution when possible; for `(unknown)` it prices using the detected pricing model to stay consistent with the anchor.
