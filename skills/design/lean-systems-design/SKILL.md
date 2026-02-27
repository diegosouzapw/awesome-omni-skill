---
name: lean-systems-design
description: "Apply Elon Musk-inspired system design thinking for research, engineering, and business workflows: rigorously challenge requirements, delete steps, simplify/optimize what remains, accelerate iteration, then automate. Use when designing or revising systems, processes, or products that need lean, high-velocity execution."
---

# Lean Systems Design (Musk-inspired)

Use this when shaping any system/process (product design, research workflow, ops runbook). Follow the sequence; do not skip ahead.

## Quickstart
1) State the objective, constraints, success measures, and current system sketch.
2) Run the five-pass loop below in order; capture changes after each pass.
3) Produce a concise plan and, if applicable, experiment/rollout steps.

## Five-Pass Workflow (in order)
- Pass 1 - Make requirements less dumb
  - List every requirement with the requestor's name; reject "dept says."
  - For each, ask: What outcome does this serve? Evidence? What if we drop/relax it?
  - Reframe into testable, minimal success criteria; delete or rewrite fuzzy items.
- Pass 2 - Delete parts/processes
  - Enumerate components/steps; try to remove each. Target at least 10% removal.
  - For every kept step, name the accountable owner. If no owner, delete.
  - Ban "just in case" work; allow re-adding only with a concrete trigger.
- Pass 3 - Simplify before optimizing
  - Merge steps, reduce variants/options, standardize interfaces, name single paths.
  - Collapse handoffs and approvals; prefer defaults over configuration.
  - If it shouldn't exist, don't polish it. Stop optimization of non-critical paths.
- Pass 4 - Accelerate cycle time
  - Shorten feedback loops: smaller batch sizes, faster checkpoints, parallel where safe.
  - Define the fastest safe "learn loop" (build-measure-learn or design-test-review).
  - Add leading indicators to spot drift early.
- Pass 5 - Automate last
  - Automate only stable, high-volume, well-understood steps.
  - Remove redundant in-process checks once end-quality is consistently high.
  - Keep a manual fallback and monitoring for automation drift.

## Heuristics and Checks
- Every requirement has a named owner and measurable outcome.
- Any step without a failure mode it prevents is a deletion candidate.
- Prefer subtraction over addition; default answer to "add a step" is no.
- Bias to single paths over branching; branch only with explicit thresholds.
- Fast loop beats perfect plan; ship thin slices to validate.

## Deliverables to Produce
- Crisp objective and success metrics.
- Simplified system map (pre/post change) highlighting deletions.
- Top risks and the shortest feedback loop to catch them.
- Rollout/experiment plan with owners and timelines.