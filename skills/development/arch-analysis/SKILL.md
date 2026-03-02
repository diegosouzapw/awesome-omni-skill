---
name: arch-analysis
description: Analyze LangGraph application architecture, identify bottlenecks, and propose multiple improvement strategies
---

# LangGraph Architecture Analysis Skill

A skill for analyzing LangGraph application architecture, identifying bottlenecks, and proposing multiple improvement strategies.

## 📋 Overview

This skill analyzes existing LangGraph applications and proposes graph structure improvements:

1. **Current State Analysis**: Performance measurement and graph structure understanding
2. **Problem Identification**: Organizing bottlenecks and architectural issues
3. **Improvement Proposals**: Generate 3-5 diverse improvement proposals (**all candidates for parallel exploration**)

**Important**:
- This skill only performs analysis and proposals. It does not implement changes.
- **Output all improvement proposals**. The arch-tune command will implement and evaluate them in parallel.

## 🎯 When to Use

Use this skill in the following situations:

1. **When performance improvement of existing applications is needed**
   - Latency exceeds targets
   - Cost is too high
   - Accuracy is insufficient

2. **When considering architecture-level improvements**
   - Prompt optimization (fine-tune) has limitations
   - Graph structure changes are needed
   - Considering introduction of new patterns

3. **When you want to compare multiple improvement options**
   - Unclear which architecture is optimal
   - Want to understand trade-offs

## 📖 Analysis and Proposal Workflow

### Step 1: Verify Evaluation Environment

**Purpose**: Prepare for performance measurement

**Actions**:
1. Verify existence of evaluation program (`.langgraph-master/evaluation/` or specified directory)
2. If not present, confirm evaluation criteria with user and create
3. Verify test cases

**Output**: Evaluation program ready

### Step 2: Measure Current Performance

**Purpose**: Establish baseline

**Actions**:
1. Run test cases 3-5 times
2. Record each metric (accuracy, latency, cost, etc.)
3. Calculate statistics (mean, standard deviation, min, max)
4. Save as baseline

**Output**: `baseline_performance.json`

### Step 3: Analyze Graph Structure

**Purpose**: Understand current architecture

**Actions**:
1. **Identify graph definitions with Serena MCP**
   - Search for StateGraph, MessageGraph with `find_symbol`
   - Identify graph definition files (typically `graph.py`, `main.py`, etc.)

2. **Analyze node and edge structure**
   - List node functions with `get_symbols_overview`
   - Verify edge types (sequential, parallel, conditional)
   - Check for subgraphs

3. **Understand each node's role**
   - Read node functions
   - Verify presence of LLM calls
   - Summarize processing content

**Output**: Graph structure documentation

### Step 4: Identify Bottlenecks

**Purpose**: Identify performance problem areas

**Actions**:
1. **Latency Bottlenecks**
   - Identify nodes with longest execution time
   - Verify delays from sequential processing
   - Discover unnecessary processing

2. **Cost Issues**
   - Identify high-cost nodes
   - Verify unnecessary LLM calls
   - Evaluate model selection optimality

3. **Accuracy Issues**
   - Identify nodes with frequent errors
   - Verify errors due to insufficient information
   - Discover architecture constraints

**Output**: List of issues

### Step 5: Consider Architecture Patterns

**Purpose**: Identify applicable LangGraph patterns

**Actions**:
1. **Consider patterns based on problems**
   - Latency issues → Parallelization
   - Diverse use cases → Routing
   - Complex processing → Subgraph
   - Staged processing → Prompt Chaining, Map-Reduce

2. **Reference langgraph-master skill**
   - Verify characteristics of each pattern
   - Evaluate application conditions
   - Reference implementation examples

**Output**: List of applicable patterns

### Step 6: Generate Improvement Proposals

**Purpose**: Create 3-5 diverse improvement proposals (all candidates for parallel exploration)

**Actions**:
1. **Create improvement proposals based on each pattern**
   - Change details (which nodes/edges to modify)
   - Expected effects (impact on accuracy, latency, cost)
   - Implementation complexity (low/medium/high)
   - Estimated implementation time

2. **Evaluate improvement proposals**
   - Feasibility
   - Risk assessment
   - Expected ROI

**Important**: Output all improvement proposals. The arch-tune command will **implement and evaluate all proposals in parallel**.

**Output**: Improvement proposal document (including all proposals)

### Step 7: Create Report

**Purpose**: Organize analysis results and proposals

**Actions**:
1. Current state analysis summary
2. Organize issues
3. **Document all improvement proposals in `improvement_proposals.md`** (with priorities)
4. Present recommendations for reference (first recommendation, second recommendation, reference)

**Important**: Output all proposals to `improvement_proposals.md`. The arch-tune command will read these and implement/evaluate them in parallel.

**Output**:
- `analysis_report.md` - Current state analysis and issues
- `improvement_proposals.md` - **All improvement proposals** (Proposal 1, 2, 3, ...)

## 📊 Output Formats

### baseline_performance.json

```json
{
  "iterations": 5,
  "test_cases": 20,
  "metrics": {
    "accuracy": {
      "mean": 75.0,
      "std": 3.2,
      "min": 70.0,
      "max": 80.0
    },
    "latency": {
      "mean": 3.5,
      "std": 0.4,
      "min": 3.1,
      "max": 4.2
    },
    "cost": {
      "mean": 0.020,
      "std": 0.002,
      "min": 0.018,
      "max": 0.023
    }
  }
}
```

### analysis_report.md

```markdown
# Architecture Analysis Report

Execution Date: 2024-11-24 10:00:00

## Current Performance

| Metric | Mean | Std Dev | Target | Gap |
|--------|------|---------|--------|-----|
| Accuracy | 75.0% | 3.2% | 90.0% | -15.0% |
| Latency | 3.5s | 0.4s | 2.0s | +1.5s |
| Cost | $0.020 | $0.002 | $0.010 | +$0.010 |

## Graph Structure

### Current Configuration

\```
analyze_intent → retrieve_docs → generate_response
\```

- **Node Count**: 3
- **Edge Type**: Sequential only
- **Parallel Processing**: None
- **Conditional Branching**: None

### Node Details

#### analyze_intent
- **Role**: Classify user input intent
- **LLM**: Claude 3.5 Sonnet
- **Average Execution Time**: 0.5s

#### retrieve_docs
- **Role**: Search related documents
- **Processing**: Vector DB query + reranking
- **Average Execution Time**: 1.5s

#### generate_response
- **Role**: Generate final response
- **LLM**: Claude 3.5 Sonnet
- **Average Execution Time**: 1.5s

## Issues

### 1. Latency Bottleneck from Sequential Processing

- **Issue**: analyze_intent and retrieve_docs are sequential
- **Impact**: Total 2.0s delay (57% of total)
- **Improvement Potential**: -0.8s or more reduction possible through parallelization

### 2. All Requests Follow Same Flow

- **Issue**: Simple and complex questions go through same processing
- **Impact**: Unnecessary retrieve_docs execution (wasted Cost and Latency)
- **Improvement Potential**: -50% reduction possible for simple cases through routing

### 3. Use of Low-Relevance Documents

- **Issue**: retrieve_docs returns only top-k (no reranking)
- **Impact**: Low Accuracy (75%)
- **Improvement Potential**: +10-15% improvement possible through multi-stage RAG

## Applicable Architecture Patterns

1. **Parallelization** - Parallelize analyze_intent and retrieve_docs
2. **Routing** - Branch processing flow based on intent
3. **Subgraph** - Dedicated subgraph for RAG processing (retrieve → rerank → select)
4. **Orchestrator-Worker** - Execute multiple retrievers in parallel and integrate results
```

### improvement_proposals.md

```markdown
# Architecture Improvement Proposals

Proposal Date: 2024-11-24 10:30:00

## Proposal 1: Parallel Document Retrieval + Intent Analysis

### Changes

**Current**:
\```
analyze_intent → retrieve_docs → generate_response
\```

**After Change**:
\```
START → [analyze_intent, retrieve_docs] → generate_response
      ↓ parallel execution ↓
\```

### Implementation Details

1. Add parallel edges to StateGraph
2. Add join node to wait for both results
3. generate_response receives both results

### Expected Effects

| Metric | Current | Expected | Change | Change Rate |
|--------|---------|----------|--------|-------------|
| Accuracy | 75.0% | 75.0% | ±0 | - |
| Latency | 3.5s | 2.7s | -0.8s | -23% |
| Cost | $0.020 | $0.020 | ±0 | - |

### Implementation Complexity

- **Level**: Low
- **Estimated Time**: 1-2 hours
- **Risk**: Low (no changes to existing nodes required)

### Recommendation Level

⭐⭐⭐⭐ (High) - Effective for Latency improvement with low risk

---

## Proposal 2: Intent-Based Routing

### Changes

**Current**:
\```
analyze_intent → retrieve_docs → generate_response
\```

**After Change**:
\```
analyze_intent
    ├─ simple_intent → simple_response (lightweight)
    └─ complex_intent → retrieve_docs → generate_response
\```

### Implementation Details

1. Conditional branching based on analyze_intent output
2. Create new simple_response node (using Haiku)
3. Routing with conditional_edges

### Expected Effects

| Metric | Current | Expected | Change | Change Rate |
|--------|---------|----------|--------|-------------|
| Accuracy | 75.0% | 82.0% | +7.0% | +9% |
| Latency | 3.5s | 2.8s | -0.7s | -20% |
| Cost | $0.020 | $0.014 | -$0.006 | -30% |

**Assumption**: 40% simple cases, 60% complex cases

### Implementation Complexity

- **Level**: Medium
- **Estimated Time**: 2-3 hours
- **Risk**: Medium (adding routing logic)

### Recommendation Level

⭐⭐⭐⭐⭐ (Highest) - Balanced improvement across all metrics

---

## Proposal 3: Multi-Stage RAG with Reranking Subgraph

### Changes

**Current**:
\```
analyze_intent → retrieve_docs → generate_response
\```

**After Change**:
\```
analyze_intent → [RAG Subgraph] → generate_response
                  ↓
            retrieve (k=20)
                  ↓
            rerank (top-5)
                  ↓
            select (best context)
\```

### Implementation Details

1. Convert RAG processing to dedicated subgraph
2. Retrieve more candidates in retrieve node (k=20)
3. Evaluate relevance in rerank node (Cross-Encoder)
4. Select optimal context in select node

### Expected Effects

| Metric | Current | Expected | Change | Change Rate |
|--------|---------|----------|--------|-------------|
| Accuracy | 75.0% | 88.0% | +13.0% | +17% |
| Latency | 3.5s | 3.8s | +0.3s | +9% |
| Cost | $0.020 | $0.022 | +$0.002 | +10% |

### Implementation Complexity

- **Level**: Medium-High
- **Estimated Time**: 3-4 hours
- **Risk**: Medium (introducing new model, subgraph management)

### Recommendation Level

⭐⭐⭐ (Medium) - Effective when Accuracy is priority, Latency will degrade

---

## Recommendations

**Note**: The following recommendations are for reference. The arch-tune command will **implement and evaluate all Proposals above in parallel** and select the best option based on actual results.

### 🥇 First Recommendation: Proposal 2 (Intent-Based Routing)

**Reasons**:
- Balanced improvement across all metrics
- Implementation complexity is manageable at medium level
- High ROI (effect vs cost)

**Next Steps**:
1. Run parallel exploration with arch-tune command
2. Implement and evaluate Proposals 1, 2, 3 simultaneously
3. Select best option based on actual results

### 🥈 Second Recommendation: Proposal 1 (Parallel Retrieval)

**Reasons**:
- Simple implementation with low risk
- Reliable Latency improvement
- Can be combined with Proposal 2

### 📝 Reference: Proposal 3 (Multi-Stage RAG)

**Reasons**:
- Effective when Accuracy is most important
- Only when Latency trade-off is acceptable
```

## 🔧 Tools and Technologies Used

### MCP Server Usage

- **Serena MCP**: Codebase analysis
  - `find_symbol`: Search graph definitions
  - `get_symbols_overview`: Understand node structure
  - `search_for_pattern`: Search specific patterns

### Reference Skills

- **langgraph-master skill**: Architecture pattern reference

### Evaluation Program

- User-provided or auto-generated
- Metrics: accuracy, latency, cost, etc.

## ⚠️ Important Notes

1. **Analysis Only**
   - This skill does not implement changes
   - Only outputs analysis and proposals

2. **Evaluation Environment**
   - Evaluation program is required
   - Will be created if not present

3. **Serena MCP**
   - If Serena is unavailable, manual code analysis
   - Use ls, read tools

## 🔗 Related Resources

- [langgraph-master skill](../langgraph-master/SKILL.md) - Architecture patterns
- [arch-tune command](../../commands/arch-tune.md) - Command that uses this skill
- [fine-tune skill](../fine-tune/SKILL.md) - Prompt optimization
