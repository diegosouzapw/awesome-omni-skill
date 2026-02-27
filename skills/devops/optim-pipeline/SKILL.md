---
name: optim-pipeline
description: Complete GPU optimization pipeline. Launches N agents, runs benchmarks, anti-triche verification, and generates report automatically. One command for everything.
argument-hint: [N] [CHUNK_SIZE] [BUILD_JOBS] [TIMEOUT] [LOG_DIR]
disable-model-invocation: true
context: fork
agent: general-purpose
allowed-tools: Bash, Skill, Read, Task
---

# GPU Optimization Pipeline - Full Automation V3

You are the **pipeline orchestrator**. You run the COMPLETE optimization workflow automatically.

## Parameters

Extract parameters from $ARGUMENTS (space-separated):
- **N** = First argument (default: 24)
- **CHUNK_SIZE** = Second argument (default: 4)
- **BUILD_JOBS** = Third argument (default: 4)
- **TIMEOUT** = Fourth argument (default: 1800)
- **LOG_DIR** = Fifth argument (default: `./optim_logs`)

Example: `/optim-pipeline 24 4 4 1800 ./logs`

## Complete Workflow

You will execute ALL phases automatically:

```bash
# Get parameters
PARAMS=($ARGUMENTS)
N_AGENTS=${PARAMS[0]:-24}
CHUNK_SIZE=${PARAMS[1]:-4}
BUILD_JOBS=${PARAMS[2]:-4}
TIMEOUT=${PARAMS[3]:-1800}
LOG_DIR=${PARAMS[4]:-"./optim_logs"}

# Additional pipeline parameters
REPEAT_COUNT=${PARAMS[5]:-10}
COOLDOWN=${PARAMS[6]:-2}
FILTER=${PARAMS[7]:-"3D_Large"}

echo "╔══════════════════════════════════════════════════════════╗"
echo "║     GPU OPTIMIZATION PIPELINE - FULL AUTOMATION          ║"
echo "╚══════════════════════════════════════════════════════════╝"
echo "Agents: $N_AGENTS | Chunk: $CHUNK_SIZE | Jobs: $BUILD_JOBS"
echo "Benchmark: $REPEAT_COUNT reps, ${COOLDOWN}s cooldown"
echo "Filter: $FILTER"
echo "══════════════════════════════════════════════════════════"
echo ""
```

## Phase 1: Orchestrator

Launch optimization agents:

```bash
echo "[1/4] 🚀 Launching $N_AGENTS optimization agents..."

# Launch orchestrator skill
ORCH_OUTPUT=$(optim-orchestrator $N_AGENTS $CHUNK_SIZE $BUILD_JOBS $TIMEOUT "$LOG_DIR")

# Extract session ID from output
SESSION_ID=$(echo "$ORCH_OUTPUT" | grep -oP 'Session ID: \K[0-9_]+' || echo "")

if [ -z "$SESSION_ID" ]; then
  echo "❌ Failed to get session ID from orchestrator"
  echo "$ORCH_OUTPUT"
  exit 1
fi

echo "✓ Session ID: $SESSION_ID"
echo ""
```

## Phase 2: Smart Monitoring

Monitor agents with IMPROVED logic:

```bash
echo "[2/4] ⏳ Monitoring agents (smart mode)..."

SESSION_LOG_DIR="$LOG_DIR/session_${SESSION_ID}"
START_TIME=$(date +%s)
MAX_WAIT=$((TIMEOUT + 300))  # Timeout + 5min grace period

while true; do
  NOW=$(date +%s)
  ELAPSED=$((NOW - START_TIME))

  # Count completed agents
  COMPLETED=0
  STUCK=0
  BUILD_FAILED=0

  for i in $(seq -f "%02g" 1 $N_AGENTS); do
    RESULT_FILE="$SESSION_LOG_DIR/agent_${i}_result.json"

    if [ -f "$RESULT_FILE" ]; then
      STATUS=$(cat "$RESULT_FILE" | jq -r '.status // "unknown"' 2>/dev/null)
      COMPLETED=$((COMPLETED + 1))

      if [ "$STATUS" = "build_failed" ]; then
        BUILD_FAILED=$((BUILD_FAILED + 1))
      fi
    else
      # Check if stuck (no log activity for 3 minutes)
      LOG="$SESSION_LOG_DIR/agent_${i}.log"
      if [ -f "$LOG" ]; then
        LAST_ACTIVITY=$(stat -c %Y "$LOG")
        LOG_AGE=$((NOW - LAST_ACTIVITY))

        # Stuck if no activity for 3min AND at least one build failed nearby
        if [ $LOG_AGE -gt 180 ] && [ $BUILD_FAILED -gt 0 ]; then
          STUCK=$((STUCK + 1))
        fi
      fi
    fi
  done

  # Progress bar (every 60s only)
  if [ $((ELAPSED % 60)) -eq 0 ]; then
    PROGRESS=$((COMPLETED * 100 / N_AGENTS))
    echo "[$(date +%H:%M:%S)] Progress: $COMPLETED/$N_AGENTS ($PROGRESS%) - Stuck: $STUCK - Build failed: $BUILD_FAILED"

    # Early abort if too many failures
    if [ $BUILD_FAILED -gt $((N_AGENTS / 2)) ]; then
      echo "⚠️  More than 50% builds failed, aborting..."
      break
    fi
  fi

  # Check completion
  if [ $COMPLETED -ge $N_AGENTS ]; then
    echo "✓ All agents completed!"
    break
  fi

  # Timeout check
  if [ $ELAPSED -gt $MAX_WAIT ]; then
    echo "⚠️  Timeout after ${ELAPSED}s"
    break
  fi

  sleep 10  # Check every 10s, print every 60s
done

echo ""
```

## Phase 3: Benchmarks

Run sequential GPU benchmarks:

```bash
echo "[3/4] 📊 Running GPU benchmarks..."

# Find worktrees with successful builds
SUCCESSFUL_AGENTS=()
for i in $(seq -f "%02g" 1 $N_AGENTS); do
  WORKTREE="/home/sbstndbs/subsetix_kokkos_optimized_opt${i}"
  if [ -d "$WORKTREE/build-experimental-cuda" ]; then
    RESULT_FILE="$SESSION_LOG_DIR/agent_${i}_result.json"
    if [ -f "$RESULT_FILE" ]; then
      STATUS=$(cat "$RESULT_FILE" | jq -r '.status // "unknown"' 2>/dev/null)
      if [ "$STATUS" = "success" ]; then
        SUCCESSFUL_AGENTS+=("$i")
      fi
    fi
  fi
done

echo "Found ${#SUCCESSFUL_AGENTS[@]} successful builds"

if [ ${#SUCCESSFUL_AGENTS[@]} -eq 0 ]; then
  echo "❌ No successful builds to benchmark"
  exit 1
fi

# Run benchmarks (using the skill)
optim-benchmark $N_AGENTS $SESSION_ID $REPEAT_COUNT $COOLDOWN "$FILTER"

echo "✓ Benchmarks complete"
echo ""
```

## Phase 4: Anti-Triche + Report

```bash
echo "[4/4] 🔍 Running anti-triche and generating report..."

# Anti-triche
optim-antitriche $N_AGENTS $SESSION_ID
echo "✓ Anti-triche complete"

# Generate report
optim-report $N_AGENTS $SESSION_ID

echo ""
echo "╔══════════════════════════════════════════════════════════╗"
echo "║              PIPELINE COMPLETE 🎉                       ║"
echo "╚══════════════════════════════════════════════════════════╝"
echo ""
echo "Results: $SESSION_LOG_DIR"
echo "Report: $SESSION_LOG_DIR/optimization_report_*.md"
echo ""
```

## Return Format

```json
{
  "pipeline": "full_automation",
  "session_id": "$SESSION_ID",
  "n_agents": $N_AGENTS,
  "completed": $COMPLETED,
  "successful_builds": ${#SUCCESSFUL_AGENTS[@]},
  "session_dir": "$SESSION_LOG_DIR",
  "report_path": "$SESSION_LOG_DIR/optimization_report_*.md"
}
```

## Key Improvements

1. **Single command** - All phases run automatically
2. **Smart monitoring** - 10s checks, 60s progress updates, early abort on failures
3. **Successful-only benchmarks** - Only benchmark agents that built successfully
4. **Clean output** - Progress bar instead of spam

Return ONLY the final JSON summary.
