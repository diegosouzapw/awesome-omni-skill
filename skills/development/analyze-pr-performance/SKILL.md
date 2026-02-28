---
name: analyze-pr-performance
description: Analyze code review pipeline performance for a specific PR. Use when investigating slow PRs, identifying bottlenecks, or debugging performance issues in code reviews.
argument-hint: "<prNumber> <orgId> [--days=N] [--legacy] [--env=PATH]"
---

# Analyze PR Performance

Analyze code review pipeline performance for a specific PR.

## Usage

Run the analyze-pr-performance CLI script with the provided arguments:

```bash
npx ts-node scripts/analyze-pr-performance.cli.ts $ARGUMENTS
```

## Arguments

- `prNumber` (required): The PR number to analyze
- `orgId` (required): The organization ID

## Options

- `--days=N`: Number of days to search back (default: 7)
- `--legacy`: Also search in legacy collection (observability_logs)
- `--env=PATH`: Path to .env file (e.g., `--env=.env.prod`)

## Examples

```bash
# Analyze performance for PR #558 in production
/analyze-pr-performance 558 04bd288b-595a-4ee1-87cd-8bbbdc312b3c --env=.env.prod

# Analyze with extended date range
/analyze-pr-performance 723 97442318-9d2a-496b-a0d2-b45fb --days=14 --env=.env.prod

# Analyze with legacy logs included
/analyze-pr-performance 701 97442318-9d2a-496b-a0d2-b45fb --legacy --env=.env.prod
```

## What it analyzes

1. **Pipeline identification**: Finds the pipelineId and correlationId for the PR

2. **Stage times**: Shows duration of each pipeline stage:
   - ValidateNewCommitsStage
   - ResolveConfigStage
   - FetchChangedFilesStage
   - PRLevelReviewStage
   - FileAnalysisStage
   - CreateFileCommentsStage
   - UpdateCommentsAndGenerateSummaryStage
   - And all other stages...

3. **LLM calls**: Details of each LLM operation:
   - Operation name (analyzeCodeWithAI, selectReviewMode, kodyRulesAnalyzeCodeWithAI, etc.)
   - Duration
   - Model used
   - Token counts (input/output)

4. **Summary metrics**:
   - Total pipeline duration
   - Total LLM calls count
   - Total tokens (input/output)
   - Slow calls count (> 60s)
   - Models used

5. **Bottlenecks**: Highlights stages and LLM calls taking > 60 seconds

6. **Pipeline status**: Whether the pipeline completed, failed, or is unknown

## Output

The script outputs:
- Pipeline and correlation IDs
- Organization and repository info
- Stage times table with duration and percentage of total
- LLM calls table with model, tokens, and duration
- Summary metrics
- Bottleneck list (stages and LLM calls > 60s)
- Final pipeline status

## How to Respond

- Identify the slowest stages and explain why they might be slow
- Look for LLM calls that are taking too long (> 2 minutes is concerning)
- Check if multiple slow LLM calls are running sequentially vs in parallel
- Note any patterns (e.g., all selectReviewMode calls are slow = possible model issue)
- Suggest potential optimizations if bottlenecks are clear
- If FileAnalysisStage is slow, it's usually due to many files or large files being analyzed
- If PRLevelReviewStage is slow, check the KodyRules and PR-level analysis calls
- Compare token counts to durations - high tokens with proportional time is expected, low tokens with high time indicates API latency issues
- Report the pipeline status and check if it completed successfully
