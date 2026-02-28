---
name: Load Test Designer
description: Design load tests with realistic workload models and performance criteria
category: testing
version: 1.0.0
triggers:
  - release-preparation
  - performance-baseline
  - capacity-validation
globs: "**/tests/load/**,**/k6/**,**/benchmarks/**"
---

# Load Test Designer Skill

Design load tests with realistic workload models and performance criteria.

## Trigger Conditions
- Pre-release performance gate
- Performance baseline request
- User invokes with "design load test" or "performance test"

## Input Contract
- **Required:** System under test and target endpoints
- **Required:** Performance targets (latency SLOs, throughput)
- **Optional:** Production traffic profile, expected growth

## Output Contract
- Load test configuration with workload model
- Performance criteria (pass/fail thresholds)
- Results analysis with regression detection
- Capacity recommendations

## Tool Permissions
- **Read:** Production access logs, performance baselines, infra configs
- **Write:** Load test configs, results reports
- **Execute:** Load test tools (k6, locust, artillery)

## Execution Steps
1. Build workload model from production traffic patterns
2. Define performance criteria (p50, p95, p99 latency targets)
3. Configure load test scenarios (ramp-up, sustained, spike)
4. Execute tests in staging environment
5. Collect and analyze results
6. Compare against baseline; flag regressions >5% p99
7. Generate report with capacity recommendations

## Success Criteria
- Workload model reflects production patterns
- All SLO targets met during sustained load
- No regression >5% on p99 latency
- Saturation point identified

## Escalation Rules
- Escalate if p99 regression >20% on critical path
- Escalate if saturation point is below expected peak traffic
- Escalate if soak test reveals memory leak or connection exhaustion

## Example Invocations

**Input:** "Design a load test for the /api/v1/search endpoint targeting 1000 RPS"

**Output:** Load test config (k6): ramp 0→1000 RPS over 5min, sustain 1000 RPS for 15min, spike to 2000 RPS for 2min. Pass criteria: p50 <100ms, p95 <300ms, p99 <800ms, error rate <0.1%. Saturation test: ramp until p99 >1s. Soak: 500 RPS for 8 hours monitoring heap and FD count.
