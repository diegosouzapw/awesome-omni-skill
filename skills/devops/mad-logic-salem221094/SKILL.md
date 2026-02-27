---
name: mad-logic
description: "The official MAD-Logic reliability framework for high-stakes multi-step tasks. Uses MAKER principles (Maximal Agentic Decomposition, voting consensus, red-flagging) to achieve zero-error execution. Auto-triggers for complex workflows or critical operations (deployment, security, financial). Compliant with ASP v2.0 Standard."
---

# MAD-Logic Reliability Framework

Implements the MAKER (Massively decomposed AgenT KERnel) framework for zero-error execution of long-horizon tasks.

## Step 0: Modern Technology Research (MANDATORY)

Before applying **MAD-Logic**, verify the following:
1.  **State of the Art**: Check for new research updates to the MAKER paper (arXiv:2511.09030v1) or subsequent meta-agent architectures.
2.  **Model Benchmarks**: Identify the best "cheap-but-reliable" models for voting. Smaller models (e.g., GPT-4o-mini, Claude 3 Haiku) are often superior for high-k voting due to cost and latency.
3.  **MCP Tooling**: Look for existing Model Context Protocol servers that provide specialized sub-actions (e.g., file-editing, browser-control) to use as the base for MAD decomposition.

## Core Principles

### 1. Maximal Agentic Decomposition (MAD)
Break tasks into the smallest possible subtasks (m=1 step per call). Benefits:
- Reduces context length → fewer errors
- Enables per-step error correction
- Allows smaller, focused models to succeed

### 2. First-to-ahead-by-k Voting
For critical steps, sample multiple times and pick the answer that leads by k votes:
- k=3 for standard critical operations
- k=5 for high-stakes (deployment, security, financial)
- k=7+ for mission-critical (rarely needed)

### 3. Red-Flagging
Discard suspicious outputs before they corrupt the pipeline:
- **Too long**: Response exceeds expected length → likely confused
- **Wrong format**: Malformed output → model went off-rails
- **Low confidence**: Hedging language patterns → uncertain

## When This Skill Auto-Triggers

| Condition | Action |
|-----------|--------|
| Task has >5 sequential steps | Apply MAD decomposition |
| Step involves deployment/mutation | Use k=3 voting |
| Step involves security/financial | Use k=5 voting |
| Output seems malformed | Red-flag and resample |

## Usage

### Voting for Critical Decisions

```python
# Import the voting utility
from maker_reliability.voting import vote_until_consensus

# Get consensus on a critical decision
candidates = [generate_response() for _ in range(10)]
winner = vote_until_consensus(candidates, k=3)
```

### Red-Flag Filtering

```python
from maker_reliability.red_flag import is_red_flagged

response = get_llm_response()
if is_red_flagged(response, max_tokens=500, require_format="json"):
    response = resample()  # Discard and try again
```

## Integration with Task Boundaries

This skill works with the existing `task_boundary` tool:
- Each task boundary represents a decomposition level
- Status updates enable progress tracking
- Mode switching (PLANNING/EXECUTION/VERIFICATION) maps to MAKER phases

## Theory Reference

For mathematical foundations (scaling laws, cost formulas, k optimization):
→ See [maker-theory.md](references/maker-theory.md)

## Scripts

| Script | Purpose |
|--------|---------|
| [voting.py](scripts/voting.py) | First-to-ahead-by-k consensus voting |
| [red_flag.py](scripts/red_flag.py) | Output validation and red-flag detection |

## Definition of Done

- [ ] Task is decomposed into single-step subtasks (MAD).
- [ ] Critical decision points have a defined voting margin (k).
- [ ] Consensus logic (vote_until_consensus) is used for all k > 1 steps.
- [ ] Output is validated against red-flag filters prior to being committed.
- [ ] Reliability target (e.g., 99.9%) is calculated and met via appropriate k selection.
