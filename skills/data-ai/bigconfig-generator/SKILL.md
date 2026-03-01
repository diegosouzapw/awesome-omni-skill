---
name: bigconfig-generator
description: Use this skill when creating or updating Bigeye monitoring configurations (bigconfig.yml files) for BigQuery tables. Works with metadata-manager skill.
---

# Bigconfig Generator

**Composable:** Works with metadata-manager (for schema/metadata generation) and bigquery-etl-core (for conventions)
**When to use:** Creating/updating Bigeye configurations, data quality monitoring

## Overview

Generate and manage Bigeye monitoring configurations for BigQuery tables in the Mozilla bigquery-etl repository. Bigeye is Mozilla's data quality monitoring platform that checks for freshness, volume anomalies, null values, uniqueness violations, and custom business logic validation.

This skill helps configure monitoring through:
1. **metadata.yaml** - High-level monitoring settings (freshness, volume, collections)
2. **bigconfig.yml** - Detailed metric definitions (auto-generated via bqetl CLI)
3. **bigeye_custom_rules.sql** - Custom SQL validation rules (optional, for complex business logic)

**Official Documentation:**
- **bigConfig Reference:** https://mozilla.github.io/bigquery-etl/reference/bigconfig/ (docs/reference/bigconfig.md)
- **Bigeye Intro:** https://mozilla.github.io/data-docs/cookbooks/data_monitoring/intro.html
- **Bigeye Official Docs:** https://docs.bigeye.com/docs/bigconfig

## 🚨 REQUIRED READING - Start Here

**BEFORE creating monitoring configurations, READ these resources:**

1. **Existing Collections:** READ `references/existing_collections.md`
   - Collections already in use across the repository
   - Notification channels by dataset/team
   - Helps maintain consistency and avoid creating duplicate collections

2. **Monitoring Patterns:** READ `references/monitoring_patterns.md`
   - Common monitoring scenarios
   - Freshness vs volume monitoring
   - When to use custom rules
   - Configuration workflow

## 📋 Templates - Copy These Structures

**When adding monitoring to metadata.yaml, READ and COPY from these templates:**

- **Basic monitoring (most tables)?** → READ `assets/metadata_monitoring_basic.yaml`
  - Standard freshness and volume checks
  - Collection assignment

- **Critical table (high priority)?** → READ `assets/metadata_monitoring_critical.yaml`
  - More aggressive monitoring settings
  - Faster alerting

- **View (non-partitioned)?** → READ `assets/metadata_monitoring_view.yaml`
  - Monitoring for views without partitions

**For custom validation rules:**
- **Custom SQL checks?** → READ `assets/custom_rules_template.sql`
  - Template for bigeye_custom_rules.sql
  - Shows how to write validation queries

## When to Use This Skill

Use this skill when:
- Creating new tables and user wants to enable monitoring
- User explicitly requests "create a bigeye config for..."
- User asks about adding data quality monitoring
- Setting up freshness or volume checks
- Creating custom validation rules
- Troubleshooting monitoring configurations

**Integration with metadata-manager:**
When metadata-manager creates new tables, it should ask the user: "Would you like to enable Bigeye monitoring for this table?" If yes, invoke this skill.

## 🚨 IMPORTANT: Deployment Safety

**Manual deployment is BLOCKED for safety reasons.**

If a user asks to run `./bqetl monitoring deploy`, **warn them:**

> ⚠️ **Manual deployment can accidentally delete existing metrics.** The recommended workflow is to commit your changes and let the `bqetl_artifact_deployment` DAG deploy automatically. Manual deployment is disabled in this environment.
>
> If you need to manually deploy for testing purposes, you'll need to:
> 1. Ensure you have `BIGEYE_API_KEY` set
> 2. Understand that deploying only specific tables can remove metrics from other tables
> 3. Use `--dry-run` first to review changes
> 4. Contact Data Engineering if you're unsure
>
> **Proceed with caution - this can affect production monitoring.**

The standard workflow (update → validate → commit → push) is safe and recommended.

## Prerequisites

- Table must have metadata.yaml file
- Table must be deployed to BigQuery
- Understanding of table's update schedule (daily, hourly, etc.)
- For manual deployment (discouraged): `BIGEYE_API_KEY` environment variable must be set

## Staying Current with Documentation

**Always prefer official documentation over this skill's references:**

1. **For bigConfig syntax and structure:** Read docs/reference/bigconfig.md or use WebFetch on https://mozilla.github.io/bigquery-etl/reference/bigconfig/
2. **For available saved metrics:** Check sql/bigconfig.yml in the repository (source of truth)
3. **For Bigeye concepts:** Use WebFetch on https://mozilla.github.io/data-docs/cookbooks/data_monitoring/intro.html
4. **For bqetl CLI commands:** Check `./bqetl monitoring --help` or the monitoring.py source code

**When to use WebFetch:**
- User asks about specific bigConfig features not covered in this skill
- Need to verify current syntax or available options
- References in this skill seem outdated or incomplete
- Troubleshooting issues not covered in common patterns

This skill focuses on **workflow and decision-making** rather than being a comprehensive bigConfig reference.

## Workflow

### Step 1: Determine Monitoring Requirements

Ask the user what type of monitoring they need:

**For new tables created by metadata-manager:**
"Would you like to enable Bigeye monitoring for this table? This can check for:
- Freshness (when data was last updated)
- Volume (row count anomalies)
- Column-level validation (nulls, uniqueness, formats)
- Custom business logic validation"

**For existing tables:**
"What type of monitoring would you like to configure?
1. Basic (freshness + volume)
2. Critical (freshness + volume with blocking)
3. Column-level validation
4. Custom SQL rules
5. All of the above"

**After determining monitoring type, check existing collections:**

Before configuring metadata.yaml, READ `references/existing_collections.md` to:
- Find the dataset in "Collections by Dataset" section
- Check if there's an existing collection for this dataset/team
- Note the notification channels used by similar tables

Ask the user: "Based on existing configurations, would you like to use the [Collection Name] collection with [notification channels]? Or create a new collection?"

### Step 2: Configure metadata.yaml

Add a `monitoring` section to metadata.yaml based on table type:

- **Basic (most tables):** `assets/metadata_monitoring_basic.yaml` - Freshness + volume, non-blocking
- **Critical (production):** `assets/metadata_monitoring_critical.yaml` - Blocking failures, collection assignment
- **Views:** `assets/metadata_monitoring_view.yaml` - Requires explicit partition_column

**Key settings:**
- `blocking: true` - Failures block deployments (use for critical tables)
- `collection` - Groups related tables, configures alerts
- `partition_column` - Required for views (or null if non-partitioned)

### Step 3: Generate bigconfig.yml

Use the bqetl CLI to auto-generate bigconfig.yml from metadata.yaml:

```bash
./bqetl monitoring update <dataset>.<table>
```

This command:
- Reads monitoring settings from metadata.yaml
- Generates appropriate metric definitions in bigconfig.yml
- Adds freshness/volume checks based on configuration
- Uses saved metrics from sql/bigconfig.yml

**What gets generated:**
- If `freshness.enabled: true` → Adds freshness metric
- If `volume.enabled: true` → Adds volume metric
- If `blocking: true` → Uses `freshness_fail`/`volume_fail` variants
- If `collection` specified → Groups under that collection

### Step 4: Customize bigconfig.yml (Optional)

Manually edit the generated bigconfig.yml for advanced use cases:

**Column-level validation:** Add `tag_deployments` section with `column_selectors` and metrics (is_not_null, is_unique, is_valid_client_id, etc.). See `sql/bigconfig.yml` for all available saved metrics.

**Lookback windows:** Adjust how far back Bigeye scans data (0=latest partition, 7=last 7 days, 28=last 28 days). Use longer lookback for tables with sporadic updates.

**When to customize:** Column-specific validation, custom thresholds, infrequent updates, different notification channels per metric.

See `references/monitoring_patterns.md` for examples.

### Step 5: Add Custom SQL Rules (Optional)

For complex business logic validation (cross-column checks, format validation, business rules), create `bigeye_custom_rules.sql` in the table directory.

**Use template:** `assets/custom_rules_template.sql` contains structure, JSON configuration block, and examples.

**Key points:**
- Query returns percentage (0-100) or count
- JSON comment block configures name, range, collections, owner, schedule
- Supports Jinja variables: `{{ project_id }}`, `{{ dataset_id }}`, `{{ table_name }}`

### Step 6: Validate Configuration

Validate bigconfig.yml syntax and configuration:

```bash
./bqetl monitoring validate <dataset>.<table>
```

**What it checks:**
- Valid YAML syntax
- No duplicate metric deployments
- Saved metric IDs exist
- For views: partition_column is explicitly set in metadata.yaml

**Common validation errors:**
- "Duplicate deployments" → Consolidate metrics under single deployment
- "Invalid metric" → Check saved_metric_id exists in sql/bigconfig.yml
- "Partition column needs to be configured" → Set `partition_column` and `partition_column_set: true` for views

### Step 7: Deploy to Bigeye

**Recommended approach: Automatic deployment via Airflow DAG**

After validation passes, commit and push your changes to the main branch:

```bash
git add sql/<project>/<dataset>/<table>/
git commit -m "Add Bigeye monitoring for <dataset>.<table>"
git push origin main
```

**What happens automatically:**
1. The `bqetl_artifact_deployment` DAG detects bigconfig.yml changes
2. The `publish_bigeye_monitors` task deploys all bigConfig files
3. Bigeye metrics are created/updated based on your configuration
4. Custom SQL rules are deployed (if bigeye_custom_rules.sql exists)

**This approach is recommended because:**
- Ensures all bigconfig.yml files are deployed together (prevents accidental deletions)
- No need to manage `BIGEYE_API_KEY` locally
- Consistent with Mozilla's deployment practices
- Deployment history tracked in git

**Alternative: Manual deployment (discouraged)**

> **⚠️ CAUTION:** Avoid running `./bqetl monitoring deploy` locally unless absolutely necessary. Local deployment can accidentally delete metrics if config files are not included. See [docs/reference/bigconfig.md](../../../docs/reference/bigconfig.md) for details.

If you must deploy manually (e.g., for testing in non-production):
```bash
./bqetl monitoring deploy <dataset>.<table> --dry-run  # Review changes first
./bqetl monitoring deploy <dataset>.<table>            # Requires BIGEYE_API_KEY
```

### Step 8: Test Monitoring (Optional)

After deployment, you can manually trigger monitoring checks to verify configuration:

```bash
./bqetl monitoring run <dataset>.<table>  # Requires BIGEYE_API_KEY
```

**What it does:**
- Triggers all metric checks for the table
- Runs custom SQL rules
- Returns success/failure status
- Provides links to Bigeye UI for details

**When to test:**
- After automatic deployment via DAG completes
- After modifying monitoring configuration
- Debugging false positives/negatives

**Alternative:** Wait for Bigeye's scheduled runs or check results in the Bigeye UI

## Common Monitoring Patterns

**Standard workflow for all patterns:**
1. Add/update `monitoring` section in metadata.yaml
2. Run: `./bqetl monitoring update <dataset>.<table>`
3. Run: `./bqetl monitoring validate <dataset>.<table>`
4. Commit and push to main branch (automatic deployment)

### Pattern 1: Basic Daily Table
Use `assets/metadata_monitoring_basic.yaml` template. Enables freshness and volume checks, non-blocking.

### Pattern 2: Critical Production Table
Use `assets/metadata_monitoring_critical.yaml` template. Sets `blocking: true` and assigns to "Operational Checks" collection.

### Pattern 3: View with Monitoring
Use `assets/metadata_monitoring_view.yaml` template. Must set `partition_column` and `partition_column_set: true`.

### Pattern 4: Column-Level Validation
After generating basic bigconfig.yml, manually edit to add column-specific metrics. See `sql/bigconfig.yml` for available saved metrics (is_not_null, is_unique, is_valid_client_id, etc.).

### Pattern 5: Custom Business Logic
Create `bigeye_custom_rules.sql` using `assets/custom_rules_template.sql`. Query must return percentage (0-100) or count. Configure via JSON comment block.

## Integration with Other Skills

### Works with metadata-manager

**When metadata-manager creates new tables:**
- metadata-manager should ask: "Would you like to enable Bigeye monitoring?"
- If yes, metadata-manager invokes bigconfig-generator skill
- bigconfig-generator adds monitoring configuration to metadata.yaml
- Generates bigconfig.yml via bqetl CLI

**Workflow:**
1. metadata-manager creates schema.yaml, metadata.yaml
2. metadata-manager asks about monitoring
3. If yes → invoke bigconfig-generator
4. bigconfig-generator adds monitoring section to metadata.yaml
5. bigconfig-generator runs `./bqetl monitoring update`
6. User validates, commits, and pushes to main (automatic deployment via DAG)

### Works with bigquery-etl-core

- Uses project structure conventions
- Follows naming patterns (dataset.table)
- References common partitioning strategies (submission_date)

## Troubleshooting

### Deployment Errors

**Deployment delays:**
- Deployment happens automatically after merge to main via `bqetl_artifact_deployment` DAG
- Check DAG status in Airflow UI if deployment seems delayed
- Typical deployment time: within 1 hour of merge

**"Table does not exist in Bigeye"**
- Table not yet ingested by Bigeye
- Wait for next schema sync or manually sync in Bigeye UI
- Check with Data Engineering if table is not appearing

**"Partition column does not exist"**
- Verify `partition_column` matches actual column in schema.yaml
- Check for typos in column name

**Manual deployment errors (if using ./bqetl monitoring deploy):**
**"Bigeye API token needs to be set"**
- Set `BIGEYE_API_KEY` environment variable
- Note: Manual deployment is discouraged; prefer automatic DAG deployment

### Validation Errors

**"Duplicate deployments"**
- Same column selector appears multiple times
- Consolidate metrics under single deployment

**"Invalid metric"**
- Referencing non-existent saved_metric_id
- Check sql/bigconfig.yml for available metrics

**"Partition column needs to be configured"**
- For views with monitoring enabled
- Add `partition_column` and `partition_column_set: true` to metadata.yaml

### False Positives

**Freshness checks failing:**
- Verify table actually updated (query BigQuery)
- Check partition_column is correct
- Verify Bigeye's schedule aligns with table update schedule
- Consider longer lookback window

**Volume checks failing:**
- Normal for tables with varying row counts
- Consider disabling volume checks
- Use longer lookback window
- Adjust thresholds in bigconfig.yml

## Best Practices

### When to Enable Monitoring

**Always enable:**
- Production tables in dashboards/reports
- Tables with SLAs or freshness requirements
- Critical pipeline outputs

**Consider enabling:**
- Development/staging tables (for testing configs)
- Tables with known data quality issues

**Skip monitoring:**
- Temporary/scratch tables
- One-time analysis tables
- Tables with no consumers

### Blocking vs Non-Blocking

**Use `blocking: true` when:**
- Failures must halt deployments
- Table is production-critical
- False positives are rare and quickly resolved

**Use `blocking: false` when:**
- Failures should alert but not block
- Table is still stabilizing
- False positives are expected

### Collections

**Use consistent naming:**
- Group related tables by team/product
- Configure notification channels once per collection
- Makes alert management easier

**Common collections:**
- Team: "Subscription Platform", "Ads Team", "Growth Team"
- Function: "Operational Checks", "Data Quality"
- Environment: "Test", "Staging"

### Custom Rules

**Best practices:**
- Return percentage (0-100) for "value" alert_conditions
- Return count for "count" alert_conditions
- Use descriptive rule names
- Set appropriate min/max ranges
- Document rule purpose in comments
- Test rules manually before deploying

## Reference Documentation

**Official Documentation (Always Preferred):**
- **docs/reference/bigconfig.md** - Canonical reference for bigConfig in this repository
- **sql/bigconfig.yml** - Source of truth for available saved metrics
- **https://mozilla.github.io/bigquery-etl/reference/bigconfig/** - Published docs
- **https://mozilla.github.io/data-docs/cookbooks/data_monitoring/intro.html** - Bigeye intro
- **https://docs.bigeye.com/docs/bigconfig** - Bigeye official documentation

**Quick Reference (This Skill):**
- `references/monitoring_patterns.md` - Workflow guidance and common patterns (may be outdated)
- `assets/metadata_monitoring_basic.yaml` - Basic monitoring config template
- `assets/metadata_monitoring_critical.yaml` - Critical table config template
- `assets/metadata_monitoring_view.yaml` - View monitoring config template
- `assets/custom_rules_template.sql` - Custom SQL rule template

**Priority:** When in doubt, read docs/reference/bigconfig.md or use WebFetch on the online docs.

## Quick Reference: bqetl Monitoring Commands

```bash
# Refresh the collections reference file (run periodically to stay current)
python3 .claude/skills/bigconfig-generator/scripts/extract_collections.py

# Generate/update bigconfig.yml from metadata.yaml
./bqetl monitoring update <dataset>.<table>

# Validate bigconfig.yml syntax and configuration
./bqetl monitoring validate <dataset>.<table>

# ⚠️ DISCOURAGED: Manual deployment (prefer automatic DAG deployment)
./bqetl monitoring deploy <dataset>.<table> --dry-run  # Requires BIGEYE_API_KEY
./bqetl monitoring deploy <dataset>.<table>            # Requires BIGEYE_API_KEY

# Manually trigger monitoring checks (requires BIGEYE_API_KEY)
./bqetl monitoring run <dataset>.<table>

# Delete deployed monitoring (requires BIGEYE_API_KEY)
./bqetl monitoring delete <dataset>.<table> --metrics --custom-sql
```

**Recommended workflow:**
1. Check `references/existing_collections.md` for appropriate collection/channels
2. Update/create bigconfig.yml using `monitoring update`
3. Validate using `monitoring validate`
4. Commit and push to main branch
5. `bqetl_artifact_deployment` DAG automatically deploys changes

