---
name: validate-historical
description: Validate historical data completeness and quality over date ranges
---

# Historical Data Validation

You are performing comprehensive historical data validation for the NBA stats scraper. This skill identifies data gaps, assesses cascade impacts, and provides actionable remediation plans.

## Your Mission

Audit historical data integrity across a date range, identify gaps that may have cascading effects on predictions, and provide specific remediation steps in the correct dependency order.

## Critical Concept: The Data Cascade Problem

**Missing historical data corrupts downstream predictions:**

```
2026-01-18: Player scores 30 points → Missing from DB (pipeline failure)
2026-01-19: points_avg_last_5 = 20 (WRONG - should be 22 with the 30)
2026-01-20: ML features use wrong average
2026-01-21: Prediction degraded (wrong features)
2026-01-22→01-28: Error propagates for 5-10 more days
```

**Cascade Window**: Missing data on date X affects rolling averages for:
- **L5 averages**: 5 days forward
- **L10 averages**: 10 days forward
- **ML features using longer windows**: up to 21 days forward

---

## Interactive Mode (User Preference Gathering)

**If the user invoked the skill without specific parameters or flags**, ask them what they want to validate:

Use the AskUserQuestion tool to gather preferences:

**Question 1: "What date range would you like to validate?"**
```
Options:
  - "Last 7 days (Recommended)" - Quick weekly health check
  - "Last 14 days" - Bi-weekly validation
  - "Last 30 days" - Monthly audit
  - "Full season (Oct 22 - today)" - Comprehensive season audit (slower)
  - "Custom date range" - Specify exact dates
```

**Question 2: "What type of validation do you need?"**
```
Options:
  - "Standard validation (Recommended)" - Find gaps, assess quality, get remediation plan
  - "Deep check" - Recalculate rolling averages from source to verify accuracy
  - "Player-specific" - Deep dive into a single player's data history
  - "Game-specific" - Validate all data for a single game
  - "Verify backfill" - Confirm a recent backfill succeeded and cascade is resolved
  - "Quick coverage scan" - Fast completeness check without deep analysis
  - "Find anomalies" - Detect statistical outliers and suspicious data
```

**Question 3 (conditional): Follow-up questions based on mode**

If they chose:
- **Player-specific**: Ask "Which player?" (free text)
- **Verify backfill**: Ask "What date was backfilled?" (free text, format: YYYY-MM-DD)
- **Custom date range**: Ask "What date range?" (free text, format: YYYY-MM-DD YYYY-MM-DD)

**Based on their answers**, construct the effective command and proceed:

Examples:
- Q1: "Last 7 days", Q2: "Standard" → Run standard validation for last 7 days
- Q1: "Last 14 days", Q2: "Deep check" → Run deep check for last 14 days
- Q1: "Last 7 days", Q2: "Player-specific", Q3: "LeBron James" → Player validation for LeBron
- Q1: Custom "2026-01-18", Q2: "Verify backfill" → Verify backfill for 2026-01-18

**If the user already provided parameters** (e.g., `--deep-check 2026-01-18` or `--player "LeBron James"`), skip the questions and parse their intent directly.

---

## Understanding User Intent

Parse the command to determine mode and date range:

### Date Range Parsing

```
/validate-historical              → Last 7 days
/validate-historical 14           → Last 14 days
/validate-historical 30           → Last 30 days
/validate-historical season       → Full season (Oct 22 - today)
/validate-historical 2026-01-01 2026-01-15  → Specific range
```

### Mode Detection (Flags)

```
--deep-check          → Verify calculations from source (recalculate rolling avgs)
--player <name>       → Single player deep dive
--game <id>           → Single game validation
--verify-backfill     → Confirm backfill succeeded
--coverage-only       → Quick completeness scan (no deep analysis)
--anomalies           → Statistical outlier detection
--compare-sources     → Cross-source reconciliation
--export <path>       → Save results to JSON
```

**Examples**:
- `/validate-historical --deep-check 2026-01-18` → Deep check single date
- `/validate-historical --player "LeBron James"` → Player-specific validation (last 7 days)
- `/validate-historical 14 --coverage-only` → Quick scan last 14 days

---

## Mode 1: Standard Validation (Default)

**When**: No special flags provided
**Purpose**: Comprehensive health check with gap detection and quality trends

### Workflow

#### Step 1: Data Completeness Check

For each date in range, verify all pipeline stages:

```bash
# Query for completeness
bq query --use_legacy_sql=false "
WITH dates AS (
  SELECT date
  FROM UNNEST(GENERATE_DATE_ARRAY('2026-01-18', '2026-01-25')) AS date
),
raw_data AS (
  SELECT
    game_date,
    COUNT(DISTINCT game_id) as games,
    COUNT(*) as raw_records
  FROM \`nba-props-platform.nba_raw.nbac_gamebook_player_stats\`
  WHERE game_date BETWEEN '2026-01-18' AND '2026-01-25'
  GROUP BY game_date
),
analytics_data AS (
  SELECT
    game_date,
    COUNT(*) as analytics_records,
    COUNTIF(minutes_played IS NOT NULL) as minutes_coverage,
    COUNTIF(usage_rate IS NOT NULL) as usage_coverage
  FROM \`nba-props-platform.nba_analytics.player_game_summary\`
  WHERE game_date BETWEEN '2026-01-18' AND '2026-01-25'
  GROUP BY game_date
),
predictions AS (
  SELECT
    game_date,
    COUNT(*) as prediction_count
  FROM \`nba-props-platform.nba_predictions.player_prop_predictions\`
  WHERE game_date BETWEEN '2026-01-18' AND '2026-01-25'
    AND is_active = TRUE
  GROUP BY game_date
)
SELECT
  d.date,
  COALESCE(r.games, 0) as games,
  COALESCE(r.raw_records, 0) as raw_records,
  COALESCE(a.analytics_records, 0) as analytics_records,
  COALESCE(a.minutes_coverage, 0) as minutes_coverage,
  COALESCE(a.usage_coverage, 0) as usage_coverage,
  COALESCE(p.prediction_count, 0) as prediction_count,
  CASE
    WHEN r.raw_records > 0 AND a.analytics_records = 0 THEN 'RAW_ONLY'
    WHEN a.analytics_records > 0 AND a.analytics_records < r.raw_records * 0.9 THEN 'INCOMPLETE'
    WHEN a.analytics_records > 0 THEN 'COMPLETE'
    ELSE 'NO_DATA'
  END as status
FROM dates d
LEFT JOIN raw_data r ON d.date = r.game_date
LEFT JOIN analytics_data a ON d.date = a.game_date
LEFT JOIN predictions p ON d.date = p.game_date
ORDER BY d.date DESC"
```

#### Step 2: Identify Gaps

**Gap Types**:
1. **NO_DATA**: No games scheduled (off-day) → Expected, not an error
2. **RAW_ONLY**: Raw data exists but analytics missing → Phase 3 failure, backfill needed
3. **INCOMPLETE**: Analytics < 90% of raw → Partial failure, investigate cause

#### Step 3: Assess Cascade Impact

For each gap found, calculate affected dates:

**Cascade Window**: Gap date + 5-21 days forward (depending on rolling avg window: L5=5 days, L10=10 days, longer features=up to 21 days)

```bash
# Find features affected by gap
bq query --use_legacy_sql=false "
SELECT
  game_date,
  COUNT(DISTINCT player_lookup) as affected_players,
  COUNT(*) as affected_features
FROM \`nba-props-platform.nba_predictions.ml_feature_store_v2\`
WHERE DATE('2026-01-18') IN UNNEST(historical_completeness.contributing_game_dates)
  AND game_date > DATE('2026-01-18')
  AND game_date <= DATE_ADD(DATE('2026-01-18'), INTERVAL 21 DAY)
GROUP BY game_date
ORDER BY game_date"
```

#### Step 4: Quality Trend Analysis

**Run spot checks for the date range**:

```bash
python scripts/spot_check_data_accuracy.py \
  --start-date 2026-01-18 \
  --end-date 2026-01-25 \
  --samples 10 \
  --checks rolling_avg,usage_rate
```

**Check ML feature quality trends (Session 139)**:

```bash
# Trend is_quality_ready percentage over time
bq query --use_legacy_sql=false "
SELECT
  game_date,
  COUNT(*) as total_rows,
  COUNTIF(is_quality_ready = TRUE) as quality_ready,
  ROUND(100.0 * COUNTIF(is_quality_ready = TRUE) / COUNT(*), 1) as pct_quality_ready,
  ROUND(AVG(matchup_quality_pct), 1) as avg_matchup_quality,
  COUNTIF(quality_alert_level = 'red') as red_alerts
FROM \`nba-props-platform.nba_predictions.ml_feature_store_v2\`
WHERE game_date BETWEEN '2026-01-18' AND '2026-01-25'
GROUP BY game_date
ORDER BY game_date DESC"
```

**Quality trend checks**:
- `pct_quality_ready` dropping below 60% indicates Phase 4 processor failures
- `avg_matchup_quality` dropping below 50% catches Session 132-style degradation (all matchup features defaulted)
- Rising `red_alerts` count signals systemic quality regression -- investigate per-feature scores

**Analyze trends**:
- Is accuracy stable or declining?
- Are failures clustered around specific dates?
- Do failures correlate with identified gaps?

#### Step 5: Provide Remediation Plan

For each gap, provide commands in **correct dependency order**:

1. **Phase 3 (Analytics)**: Regenerate player_game_summary
2. **Phase 4 (Precompute)**: Regenerate player_daily_cache
3. **Verification**: Run spot checks to confirm fix

---

## Mode 2: Deep Check (`--deep-check`)

**When**: `--deep-check` flag present
**Purpose**: Verify cached rolling averages match recalculation from source

### Workflow

This mode **recalculates** rolling averages from raw data and compares to what's stored in cache.

```bash
# For each sample, run manual recalculation
bq query --use_legacy_sql=false "
WITH player_games AS (
  SELECT
    player_lookup,
    game_date,
    points,
    ROW_NUMBER() OVER (PARTITION BY player_lookup ORDER BY game_date DESC) as game_num
  FROM \`nba-props-platform.nba_analytics.player_game_summary\`
  WHERE player_lookup = 'lebronjames'
    AND game_date < DATE('2026-01-26')
  ORDER BY game_date DESC
  LIMIT 5
),
cached_avg AS (
  SELECT
    points_avg_last_5
  FROM \`nba-props-platform.nba_precompute.player_daily_cache\`
  WHERE player_lookup = 'lebronjames'
    AND cache_date = DATE('2026-01-25')  -- cache_date = game_date - 1
)
SELECT
  (SELECT AVG(points) FROM player_games) as recalculated_avg,
  (SELECT points_avg_last_5 FROM cached_avg) as cached_avg,
  ABS((SELECT AVG(points) FROM player_games) - (SELECT points_avg_last_5 FROM cached_avg)) as difference"
```

**Sample random player-dates** (10-20 samples) and compare:
- If difference > 2% → MISMATCH (investigate)
- If difference ≤ 2% → MATCH (floating point tolerance)

**Mismatch Investigation**:
1. Check which games were used in cache calculation
2. Check which games SHOULD have been used
3. Identify missing game (the gap date)
4. Estimate cascade impact (players affected, dates affected)

### Sample Size Recommendations

| Date Range | Recommended Samples | Rationale |
|------------|---------------------|-----------|
| 1-3 days | 10-15 samples | Focused check, quick |
| 7 days | 20 samples | Weekly health check |
| 14 days | 30 samples | Bi-weekly audit |
| 30+ days | 50 samples | Monthly/season audit |

**Trade-off**: More samples = higher confidence but longer runtime (~2-3 seconds per sample for BigQuery queries).

**Minimum**: Always check at least 10 samples for statistical relevance.

---

## Mode 3: Player-Specific (`--player <name>`)

**When**: `--player` flag with player name
**Purpose**: Deep dive into single player's data history

### Workflow

```bash
# Get player's game history
bq query --use_legacy_sql=false "
SELECT
  game_date,
  opponent_team_abbr,
  points,
  minutes_played,
  usage_rate,
  -- Check data in each stage
  CASE WHEN game_id IN (SELECT DISTINCT game_id FROM \`nba-props-platform.nba_raw.nbac_gamebook_player_stats\` WHERE player_lookup = 'lebronjames') THEN '✅' ELSE '❌' END as has_raw,
  CASE WHEN minutes_played IS NOT NULL THEN '✅' ELSE '❌' END as has_analytics,
  -- Note: This is simplified - real check would join to cache and predictions tables
FROM \`nba-props-platform.nba_analytics.player_game_summary\`
WHERE player_lookup = 'lebronjames'
  AND game_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 14 DAY)
ORDER BY game_date DESC"
```

Show game-by-game breakdown with data stage completion for each game.

Run deep check on player's rolling averages to verify integrity.

---

## Mode 4: Game-Specific (`--game <id>`)

**When**: `--game` flag with game ID or description
**Purpose**: Validate all data for a single game - all players, all stats, all pipeline stages

### Workflow

#### Step 1: Identify the Game

```bash
# If user provided game_id directly
GAME_ID="0022500123"

# If user provided description, find the game
bq query --use_legacy_sql=false "
SELECT game_id, game_date, home_team_abbr, away_team_abbr
FROM \`nba-props-platform.nba_raw.nbac_schedule\`
WHERE game_date = DATE('2026-01-25')
  AND (home_team_abbr = 'LAL' OR away_team_abbr = 'LAL')
LIMIT 1"
```

#### Step 2: Check All Players for Game

```bash
bq query --use_legacy_sql=false "
WITH raw AS (
  SELECT player_lookup, points as raw_points, minutes as raw_minutes
  FROM \`nba-props-platform.nba_raw.nbac_gamebook_player_stats\`
  WHERE game_id = '0022500123'
),
analytics AS (
  SELECT player_lookup, points as analytics_points, minutes_played, usage_rate
  FROM \`nba-props-platform.nba_analytics.player_game_summary\`
  WHERE game_id = '0022500123'
),
predictions AS (
  SELECT player_lookup, COUNT(*) as prediction_count
  FROM \`nba-props-platform.nba_predictions.player_prop_predictions\`
  WHERE game_id = '0022500123' AND is_active = TRUE
  GROUP BY player_lookup
)
SELECT
  COALESCE(r.player_lookup, a.player_lookup) as player,
  r.raw_points,
  a.analytics_points,
  a.usage_rate,
  COALESCE(p.prediction_count, 0) as predictions,
  CASE
    WHEN r.player_lookup IS NULL THEN 'MISSING_RAW'
    WHEN a.player_lookup IS NULL THEN 'MISSING_ANALYTICS'
    WHEN a.usage_rate IS NULL THEN 'MISSING_USAGE'
    ELSE 'COMPLETE'
  END as status
FROM raw r
FULL OUTER JOIN analytics a ON r.player_lookup = a.player_lookup
LEFT JOIN predictions p ON a.player_lookup = p.player_lookup
ORDER BY status DESC, player"
```

#### Step 3: Verify Team Totals

```bash
# Check team stats exist and sum correctly
bq query --use_legacy_sql=false "
SELECT
  team_abbr,
  SUM(points) as team_points,
  (SELECT points FROM \`nba-props-platform.nba_analytics.team_offense_game_summary\` t
   WHERE t.game_id = '0022500123' AND t.team_abbr = p.team_abbr) as recorded_team_points
FROM \`nba-props-platform.nba_analytics.player_game_summary\` p
WHERE game_id = '0022500123'
GROUP BY team_abbr"
```

### Output Format

```markdown
## Game Validation: LAL vs GSW (2026-01-25)

**Game ID**: 0022500123

### Player Data Completeness

| Player | Raw | Analytics | Usage | Predictions | Status |
|--------|-----|-----------|-------|-------------|--------|
| lebronjames | 28 | 28 | 32.1% | 4 | ✅ COMPLETE |
| anthonydavis | 24 | 24 | 28.5% | 4 | ✅ COMPLETE |
| austinreaves | 18 | 18 | NULL | 3 | ⚠️ MISSING_USAGE |

### Team Totals

| Team | Player Sum | Recorded | Match |
|------|------------|----------|-------|
| LAL | 118 | 118 | ✅ |
| GSW | 112 | 112 | ✅ |

### Issues Found
- 2 players missing usage_rate (team stats may be incomplete)
```

---

## Mode 5: Verify Backfill (`--verify-backfill`)

**When**: `--verify-backfill <date>` provided
**Purpose**: Confirm backfill succeeded and downstream data is now correct

### Workflow

#### Step 1: Confirm Backfilled Date Complete

```bash
# Check the backfilled date has data
bq query --use_legacy_sql=false "
SELECT
  COUNT(*) as player_records,
  COUNT(DISTINCT game_id) as games,
  COUNTIF(minutes_played IS NOT NULL) as minutes_coverage,
  COUNTIF(usage_rate IS NOT NULL) as usage_coverage
FROM \`nba-props-platform.nba_analytics.player_game_summary\`
WHERE game_date = DATE('2026-01-18')"
```

**Expected**: Record count matches number of players who played that day (~180-300)

#### Step 2: Verify Downstream Dates Fixed

Run deep check on dates AFTER the backfill (next 5-7 days):

```bash
python scripts/spot_check_data_accuracy.py \
  --start-date 2026-01-19 \
  --end-date 2026-01-25 \
  --samples 20 \
  --checks rolling_avg
```

**Expected**: Spot check accuracy returns to ≥95% for downstream dates

#### Step 3: Check Cascade Resolution

```bash
# Verify features recalculated
bq query --use_legacy_sql=false "
SELECT
  game_date,
  COUNT(*) as features_with_backfilled_date
FROM \`nba-props-platform.nba_predictions.ml_feature_store_v2\`
WHERE DATE('2026-01-18') IN UNNEST(historical_completeness.contributing_game_dates)
  AND game_date > DATE('2026-01-18')
GROUP BY game_date
ORDER BY game_date"
```

**Expected**: Features using backfilled date now exist with is_complete = true

---

## Mode 6: Coverage Only (`--coverage-only`)

**When**: `--coverage-only` flag present
**Purpose**: Fast completeness scan without deep analysis

```bash
# Quick completeness table
bq query --use_legacy_sql=false --format=pretty "
SELECT
  game_date,
  COUNT(DISTINCT game_id) as games,
  COUNT(*) as players,
  COUNTIF(minutes_played IS NOT NULL) as has_minutes,
  COUNTIF(usage_rate IS NOT NULL) as has_usage
FROM \`nba-props-platform.nba_analytics.player_game_summary\`
WHERE game_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
GROUP BY game_date
ORDER BY game_date DESC"
```

Output: Simple table, no investigation, fast execution.

---

## Mode 7: Anomalies (`--anomalies`)

**When**: `--anomalies` flag present
**Purpose**: Find suspicious data that might indicate corruption

### Anomalies to Flag

```bash
# Statistical outliers (Enhanced 2026-01-27 with new data quality fields)
bq query --use_legacy_sql=false "
SELECT
  game_date,
  player_lookup,
  points,
  minutes_played,
  usage_rate,
  usage_rate_valid,
  usage_rate_anomaly_reason,
  is_partial_game_data,
  game_completeness_pct,
  CASE
    WHEN points > 60 THEN 'Suspiciously high points (>60)'
    WHEN points < 0 THEN 'Negative points (corruption)'
    WHEN minutes_played = 0 AND points > 0 THEN 'Points without minutes (join issue)'
    WHEN usage_rate > 100 THEN 'INVALID usage rate (>100%) - partial team data'
    WHEN usage_rate > 50 THEN 'Excessive usage rate (>50%)'
    WHEN usage_rate < 0 THEN 'Negative usage rate (corruption)'
    WHEN is_partial_game_data = TRUE THEN 'Partial game data flag set'
  END as anomaly_type
FROM \`nba-props-platform.nba_analytics.player_game_summary\`
WHERE game_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 14 DAY)
  AND (
    points > 60 OR
    points < 0 OR
    (minutes_played = 0 AND points > 0) OR
    usage_rate > 50 OR
    usage_rate < 0 OR
    is_partial_game_data = TRUE
  )
ORDER BY game_date DESC"
```

### Usage Rate Anomaly Deep Dive (Added 2026-01-27)

For investigating usage_rate > 100% issues specifically:

```bash
# Find all invalid usage_rate records
bq query --use_legacy_sql=false "
SELECT
  game_date,
  game_id,
  player_lookup,
  team_abbr,
  usage_rate,
  usage_rate_raw,
  usage_rate_anomaly_reason,
  -- Team data context
  source_team_completeness_pct,
  game_status_at_processing,
  is_partial_game_data
FROM \`nba-props-platform.nba_analytics.player_game_summary\`
WHERE game_date BETWEEN '{start_date}' AND '{end_date}'
  AND usage_rate > 100
ORDER BY usage_rate DESC"
```

**Root Cause Analysis**:
- usage_rate > 100% typically indicates team stats had incomplete data
- Check `source_team_completeness_pct` - should be 100% for valid usage_rate
- Check `game_status_at_processing` - should be 'Final'
- If `is_partial_game_data = TRUE`, the record was correctly flagged

### DNP (Did Not Play) Visibility (Added 2026-01-27)

Check for DNP players in historical data:

```bash
bq query --use_legacy_sql=false "
SELECT
  game_date,
  team_abbr,
  COUNT(*) as total_roster,
  COUNTIF(is_dnp = TRUE) as dnp_count,
  COUNTIF(is_active = TRUE) as active_count,
  ARRAY_AGG(CASE WHEN is_dnp = TRUE THEN CONCAT(player_lookup, ' (', dnp_reason_category, ')') END IGNORE NULLS) as dnp_players
FROM \`nba-props-platform.nba_analytics.player_game_summary\`
WHERE game_date BETWEEN '{start_date}' AND '{end_date}'
GROUP BY game_date, team_abbr
HAVING COUNTIF(is_dnp = TRUE) > 0
ORDER BY game_date DESC, dnp_count DESC"
```

**Note**: DNPs are now visible in the data (previously filtered out). Use `is_dnp = TRUE` to identify them.

---

## Mode 8: Compare Sources (`--compare-sources`)

**When**: `--compare-sources` flag present
**Purpose**: Find discrepancies between data sources

```bash
# Compare NBA.com vs BallDontLie
bq query --use_legacy_sql=false "
WITH nbac AS (
  SELECT
    game_id,
    player_lookup,
    points as nbac_points,
    minutes as nbac_minutes
  FROM \`nba-props-platform.nba_raw.nbac_gamebook_player_stats\`
  WHERE game_date = DATE('2026-01-25')
),
bdl AS (
  SELECT
    game_id,
    player_lookup,
    points as bdl_points,
    minutes as bdl_minutes
  FROM \`nba-props-platform.nba_raw.bdl_player_boxscores\`
  WHERE game_date = DATE('2026-01-25')
)
SELECT
  COALESCE(n.game_id, b.game_id) as game_id,
  COALESCE(n.player_lookup, b.player_lookup) as player_lookup,
  n.nbac_points,
  b.bdl_points,
  n.nbac_points - b.bdl_points as points_diff,
  CASE
    WHEN ABS(n.nbac_points - b.bdl_points) > 2 THEN 'MISMATCH'
    ELSE 'MATCH'
  END as status
FROM nbac n
FULL OUTER JOIN bdl b
  ON n.game_id = b.game_id AND n.player_lookup = b.player_lookup
WHERE ABS(COALESCE(n.nbac_points, 0) - COALESCE(b.bdl_points, 0)) > 2"
```

### Source Distribution Check (Session 128)

Check which source was used for player_game_summary records. Useful for validating same-night analytics vs morning gamebook processing.

```bash
# Check source distribution for a date range
bq query --use_legacy_sql=false "
SELECT
  game_date,
  primary_source_used,
  COUNT(*) as records,
  ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER(PARTITION BY game_date), 1) as pct
FROM \`nba-props-platform.nba_analytics.player_game_summary\`
WHERE game_date BETWEEN DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY) AND CURRENT_DATE()
GROUP BY game_date, primary_source_used
ORDER BY game_date DESC, records DESC"
```

**Expected Distribution**:
- `nbac_gamebook` - Morning processing (after 6 AM ET)
- `nbac_boxscores` - Same-night processing (before 6 AM ET)
- Mixed sources indicate re-processing occurred

**Session 128 Context**: Same-night analytics uses `nbac_boxscores` fallback when gamebook isn't available yet. This is expected behavior during evening hours.

---

## Mode 9: Export Results (`--export <path>`)

**When**: `--export` flag with file path
**Purpose**: Save validation results to JSON for tracking, alerting, or dashboards

### Usage

Can be combined with any other mode:

```bash
/validate-historical 7 --export validation-2026-01-26.json
/validate-historical --deep-check 2026-01-18 --export deep-check-results.json
```

### Output Format

```json
{
  "validation_type": "standard",
  "date_range": {
    "start": "2026-01-19",
    "end": "2026-01-26"
  },
  "run_timestamp": "2026-01-26T17:45:00Z",
  "summary": {
    "status": "ISSUES_FOUND",
    "total_dates": 8,
    "complete_dates": 7,
    "gap_dates": 1,
    "overall_integrity": 87.5
  },
  "gaps": [
    {
      "date": "2026-01-23",
      "type": "INCOMPLETE",
      "raw_records": 240,
      "analytics_records": 45,
      "completion_pct": 18.75,
      "cascade_impact": {
        "affected_dates": ["2026-01-24", "2026-01-25", "..."],
        "affected_predictions": 150
      },
      "severity": "P1",
      "remediation": "python scripts/backfill_player_game_summary.py --date 2026-01-23"
    }
  ],
  "quality_metrics": {
    "spot_check_accuracy": 72.0,
    "usage_coverage": 35.0,
    "minutes_coverage": 100.0
  }
}
```

### Use Cases

1. **Automated Monitoring**: CI/CD pipeline runs validation, exports results, alerts on P1/P2 issues
2. **Historical Tracking**: Store daily validation results, build trend dashboards
3. **Integration**: Feed results into Slack alerts, PagerDuty, or custom dashboards

---

## Investigation Guidance

### When Gaps Found

1. **Determine gap type**:
   - Raw data missing? → Re-scrape if source still available
   - Analytics missing? → Regenerate Phase 3
   - Cache missing? → Regenerate Phase 4

2. **Check cascade impact**:
   - How many downstream dates affected?
   - How many predictions potentially degraded?

3. **Provide remediation in correct order**:
   ```bash
   # Always: Phase 3 → Phase 4 → Verify

   # Step 1: Regenerate analytics
   python scripts/backfill_player_game_summary.py --start-date 2026-01-18 --end-date 2026-01-18

   # Step 2: Regenerate cache (for cascade window)
   python scripts/regenerate_player_daily_cache.py --start-date 2026-01-18 --end-date 2026-02-08

   # Step 3: Verify fix
   python scripts/spot_check_data_accuracy.py --start-date 2026-01-19 --end-date 2026-01-25 --samples 20
   ```

### When Deep Check Fails

1. **Identify which games are missing** from the rolling average calculation
2. **Check if those games exist** in player_game_summary
3. **Check cache_date semantics**: cache_date = game_date - 1
4. **Check date filter**: Should be `< cache_date` NOT `<= cache_date`

### When Anomalies Found

1. **High points (>60)**: Check if actual performance or double-counting
2. **Zero minutes with stats**: Data join issue, check game_id format
3. **Negative stats**: Data corruption, needs manual investigation
4. **Excessive usage rate**: Team stats may be missing or wrong

---

## BigQuery Schema Reference

**Key Tables** (use these for validation queries):

### `nba_analytics.player_game_summary`
```
Fields:
- player_lookup (STRING) - normalized name (e.g., 'lebronjames')
- game_id (STRING)
- game_date (DATE)
- points, assists, rebounds (INT64)
- minutes_played (INT64) - decimal format (32, not "32:00")
- usage_rate (FLOAT64) - can be NULL if team stats missing
- source_team_last_updated (TIMESTAMP) - join to team stats
```

### `nba_precompute.player_daily_cache`
```
Fields:
- player_lookup (STRING)
- cache_date (DATE) - CRITICAL: cache_date = game_date - 1
- game_date (DATE) - upcoming game date
- points_avg_last_5, points_avg_last_10 (FLOAT64)
- minutes_avg_last_10 (FLOAT64)
- games_played_season (INT64)
```

### `nba_predictions.ml_feature_store_v2`
```
Fields:
- player_lookup (STRING)
- game_id (STRING)
- game_date (DATE)
- features (ARRAY<FLOAT64>)
- historical_completeness (STRUCT):
    - games_found (INT64)
    - games_expected (INT64)
    - is_complete (BOOL)
    - contributing_game_dates (ARRAY<DATE>) - games used in features
```

### `nba_raw.nbac_gamebook_player_stats`
```
Fields:
- player_lookup (STRING)
- game_id (STRING)
- game_date (DATE)
- points, assists, total_rebounds (INT64)
- minutes (STRING) - "MM:SS" format
```

**Schema Gotchas**:
- cache_date = game_date - 1 (the day BEFORE the game)
- contributing_game_dates contains dates used in rolling averages
- minutes format differs: raw is "MM:SS", analytics is INT64 decimal

---

## Output Format

### Standard Mode Output

```markdown
## Historical Data Validation - [DATE RANGE]

### Summary
[Overall assessment in one sentence]

### Data Completeness

| Date | Games | Players | Analytics | Predictions | Status |
|------|-------|---------|-----------|-------------|--------|
| 01-25 | 7 | 210 | 210 (100%) | 180 | ✅ COMPLETE |
| 01-24 | 6 | 180 | 180 (100%) | 165 | ✅ COMPLETE |
| 01-23 | 8 | 240 | 45 (19%) | 0 | ❌ INCOMPLETE |
| 01-22 | 0 | 0 | 0 | 0 | ℹ️ OFF-DAY |

### Gaps Detected

🔴 **P1 CRITICAL**: 2026-01-23 - Analytics 19% complete (45/240 records)
  - **Raw data**: ✅ Available (240 records)
  - **Cascade impact**: Affects 2026-01-24 through 2026-02-13 (~21 days)
  - **Affected predictions**: ~150 predictions potentially degraded
  - **Root cause**: Phase 3 processor failed (check logs)

### Cascade Impact Assessment

| Date Range | Affected Players | Affected Predictions | Integrity |
|------------|------------------|----------------------|-----------|
| 01-24 → 02-13 | ~180 | ~1,500 | ❌ Compromised |

**Explanation**: Missing 01-23 data means rolling averages (L5, L10) for subsequent days will have wrong game counts or skip that date entirely (L5 affected for 5 days, L10 for 10 days, longer features for up to 21 days).

### Quality Trends

| Week | Spot Check Accuracy | Usage Coverage | Notes |
|------|---------------------|----------------|-------|
| Jan 20-26 | 72% ❌ | 35% ❌ | Degraded (01-23 gap) |
| Jan 13-19 | 95% ✅ | 88% ✅ | Healthy |

### Remediation Plan

#### Step 1: Regenerate Phase 3 (Analytics)
```bash
python scripts/backfill_player_game_summary.py \
  --start-date 2026-01-23 \
  --end-date 2026-01-23
```

#### Step 2: Regenerate Phase 4 (Cache) for Cascade Window
```bash
python scripts/regenerate_player_daily_cache.py \
  --start-date 2026-01-23 \
  --end-date 2026-02-13
```

#### Step 3: Verify Fix
```bash
# Check backfilled date
python scripts/validate_tonight_data.py --date 2026-01-23

# Spot check downstream dates
python scripts/spot_check_data_accuracy.py \
  --start-date 2026-01-24 \
  --end-date 2026-02-05 \
  --samples 20 \
  --checks rolling_avg

# Verify cascade resolution
/validate-historical --verify-backfill 2026-01-23
```
```

### Deep Check Output (Example)

```markdown
## Deep Check Results - 2026-01-20 to 2026-01-25

Samples checked: 15 random player-game records

### Results: 3 MISMATCHES FOUND (80% integrity)

❌ **LeBron James** (2026-01-25)
   - points_avg_last_5 in cache: 28.4
   - Recalculated from raw: 26.8
   - Difference: 1.6 points (6% error)
   - **Root cause**: 2026-01-23 game (32 pts) missing from calculation
   - Games used in cache: [Jan 24, 22, 21, 20, 19]
   - Games that SHOULD be used: [Jan 24, 23, 22, 21, 20]

✅ **Stephen Curry** (2026-01-24) - MATCH
✅ **Kevin Durant** (2026-01-23) - MATCH
... (12 more samples)

### Cascade Root Cause

**Gap date**: 2026-01-23 (missing from analytics)
**Players affected**: ~180
**Downstream dates with wrong averages**: 2026-01-24 through 2026-02-13
**Predictions degraded**: ~150
```

---

## Key Commands Reference

```bash
# Completeness check
bq query --use_legacy_sql=false "SELECT game_date, COUNT(*) FROM nba_analytics.player_game_summary WHERE game_date >= '2026-01-01' GROUP BY game_date"

# Gap detection
python scripts/backfill_player_game_summary.py --check-only --start-date 2026-01-01 --end-date 2026-01-26

# Spot checks (historical)
python scripts/spot_check_data_accuracy.py --start-date 2026-01-18 --end-date 2026-01-25 --samples 20

# Regenerate analytics (Phase 3)
python scripts/backfill_player_game_summary.py --start-date 2026-01-23 --end-date 2026-01-23

# Regenerate cache (Phase 4)
python scripts/regenerate_player_daily_cache.py --start-date 2026-01-23 --end-date 2026-02-13

# Verify backfill
python scripts/validate_tonight_data.py --date 2026-01-23

# Cascade detection
bq query --use_legacy_sql=false "SELECT COUNT(*) FROM nba_predictions.ml_feature_store_v2 WHERE DATE('2026-01-23') IN UNNEST(historical_completeness.contributing_game_dates)"
```

---

## Important Guidelines

1. **Always check cascade impact**: Missing data doesn't just affect that day
2. **Remediate in dependency order**: Phase 3 → Phase 4 → Verify
3. **Use cache_date semantics correctly**: cache_date = game_date - 1
4. **Distinguish gaps from off-days**: No data on Sunday? Check if games scheduled
5. **Provide specific commands**: Not "regenerate data" but exact date ranges
6. **Classify severity appropriately**: Use P1-P5 from /validate-daily
7. **Verify fixes**: Always run spot checks after remediation

---

## Severity Classification (P1-P5)

**🔴 P1 CRITICAL**: Data gap affecting >100 predictions
- Analytics missing for game day
- Cache corrupted for multiple days
- Cascade window includes important games

**🟡 P2 HIGH**: Data gap affecting 50-100 predictions
- Partial analytics (50-90% complete)
- Single player's data corrupt
- Cascade window is short (1-5 days)

**🟠 P3 MEDIUM**: Data quality issue, <50 predictions affected
- Spot check accuracy 90-94%
- Single game missing predictions
- Anomaly detected but limited scope

**🟢 P4 LOW**: Minor issue, no prediction impact
- Off-day with no data (expected)
- Very old data gap (outside prediction window)
- Cosmetic data issues

**ℹ️ P5 INFO**: Informational, no action needed
- Data complete and healthy
- Quality trends stable

---

**Remember**: This skill finds historical problems that `/validate-daily` may have missed. The goal is comprehensive data integrity audit with actionable remediation plans.
