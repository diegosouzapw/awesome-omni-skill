---
name: ML Experiment Tracking
description: Track machine learning experiments with reproducible parameters and metrics
category: ml
version: 1.0.0
triggers:
  - model-training
  - experiment-run
  - hyperparameter-search
globs: "**/models/**,**/experiments/**,**/ml/**"
---

# ML Experiment Tracking Skill

Track machine learning experiments with reproducible parameters and metrics.

## Trigger Conditions
- Model configuration changes or hyperparameter updates
- New experiment run initiated
- User invokes with "track experiment" or "compare models"

## Input Contract
- **Required:** Experiment parameters (model, hyperparameters, data)
- **Required:** Evaluation metrics
- **Optional:** Baseline comparison, hypothesis

## Output Contract
- Experiment log entry with full reproducibility info
- Comparison table against baseline/prior runs
- Recommendation on whether to promote or iterate

## Tool Permissions
- **Read:** Model configs, training data metadata, metric logs
- **Write:** Experiment logs, comparison reports
- **Execute:** Metric collection commands

## Execution Steps
1. Record experiment hypothesis and parameters
2. Capture environment (dependencies, data version, code commit)
3. Execute or observe training run
4. Collect metrics and artifacts
5. Compare against baseline and prior experiments
6. Recommend: promote, iterate, or abandon

## Success Criteria
- Experiment is fully reproducible from logged parameters
- Metrics compared against baseline
- Clear recommendation with rationale

## Escalation Rules
- Escalate if model performance degrades vs. baseline
- Escalate if data drift detected in training set
- Escalate if experiment requires new infrastructure

## Example Invocations

**Input:** "Compare the BERT-base and DistilBERT models for our classification task"

**Output:** Experiment log: BERT-base (F1: 0.92, latency: 45ms, size: 440MB) vs DistilBERT (F1: 0.89, latency: 12ms, size: 260MB). Recommendation: DistilBERT for production (3% F1 trade-off for 73% latency improvement). Promote to staging for A/B test.
