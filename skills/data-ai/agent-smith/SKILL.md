---
name: agent-smith
description: Intelligent financial management skill for Claude Code that provides comprehensive PocketSmith API integration with AI-powered analysis, transaction categorization, rule management, tax intelligence, and scenario planning. Use when working with PocketSmith data for (1) Transaction categorization and rule management, (2) Financial analysis and reporting, (3) Australian tax compliance (ATO) and deduction tracking, (4) Scenario analysis and forecasting, (5) PocketSmith setup health checks, (6) Budget optimization and spending insights.
---

# Agent Smith

An intelligent financial management skill for Claude Code that transforms PocketSmith from a passive tracking tool into an active financial intelligence system.

## Core Capabilities

### 1. Hybrid Rule Engine
- **Platform Rules**: Simple keyword patterns synced to PocketSmith API
- **Local Rules**: Advanced regex, multi-condition logic, confidence scoring
- **Intelligence Modes**: Conservative (manual approval), Smart (auto-apply ≥90%), Aggressive (auto-apply ≥80%)
- **Performance Tracking**: Rule accuracy, matches, user overrides

### 2. 3-Tier Tax Intelligence (Australian ATO)
- **Level 1 (Reference)**: ATO category mappings, basic tax reports, GST tracking
- **Level 2 (Smart)**: Deduction detection, CGT tracking, confidence scoring, expense splitting
- **Level 3 (Full)**: BAS preparation, compliance checks, scenario planning, audit-ready documentation

### 3. Transaction Categorization
- **Batch Processing**: Categorize uncategorized transactions with AI assistance
- **Merchant Intelligence**: Automatic payee normalization and variation detection
- **AI Fallback**: LLM-powered categorization when rules don't match
- **Dry-Run Mode**: Preview categorization before applying

### 4. Financial Analysis & Reporting
- **Spending Analysis**: By category, merchant, time period
- **Trend Detection**: Increasing/decreasing categories, anomaly detection
- **Multi-Format Reports**: Markdown, CSV/JSON, HTML dashboards, Excel
- **Comparative Analysis**: YoY, QoQ, MoM comparisons

### 5. Scenario Analysis
- **Historical**: What-if replays ("What if I cut dining by 30% last year?")
- **Projections**: Spending forecasts, affordability analysis, goal modeling
- **Optimization**: Subscription analysis, expense rationalization, tax optimization
- **Tax Planning**: Purchase timing, income deferral, CGT scenarios

### 6. Health Check System
- **6 Health Dimensions**: Data Quality, Category Structure, Rule Engine, Tax Readiness, Automation, Budget Alignment
- **Automated Monitoring**: Weekly checks, EOFY prep, post-operation validation
- **Prioritized Recommendations**: Top 3 highest-impact improvements

### 7. Advanced Features
- **Smart Alerts**: Budget overruns, tax thresholds, pattern detection
- **Document Tracking**: Receipt requirements ($300 ATO threshold)
- **Multi-User**: Shared expense tracking and settlement
- **Audit Trail**: Complete activity log with undo capability

## Quick Start

### First-Time Setup

1. **Prerequisites**:
   - PocketSmith account with API access
   - Developer API key from PocketSmith (Settings > Security)
   - Python 3.9+ with uv package manager

2. **Configuration**:
   - Copy `.env.sample` to `.env`
   - Add `POCKETSMITH_API_KEY=<your_key>`
   - Set `TAX_INTELLIGENCE_LEVEL=smart` (reference|smart|full)
   - Set `DEFAULT_INTELLIGENCE_MODE=smart` (conservative|smart|aggressive)

3. **Installation**:
   ```bash
   # From the skill directory
   uv sync
   ```

4. **Guided Onboarding**:
   ```bash
   # Launch integrated onboarding (8 stages)
   /agent-smith:install
   ```

The onboarding process will:
- Discover your PocketSmith account structure
- Recommend and apply a rule template
- Help customize rules for your needs
- Configure intelligence modes
- Incrementally categorize transactions
- Show measurable improvement with health scores
- Provide ongoing usage guidance
- Generate intelligent suggestions

**Time required**: 30-60 minutes for first-time setup

### Daily Usage

**Two Usage Modes:**

1. **Guided Journey Mode** - When you start Claude Code, Agent Smith displays a status dashboard with:
   - Your health score
   - Uncategorized transaction count
   - Conflicts awaiting review
   - Suggested next steps with both natural language and command options

2. **Power User Mode** - Use slash commands directly for specific operations.

**Slash Commands** (7 specialized commands):
- `/smith:install` - Installation and onboarding wizard
- `/smith:categorize [--mode] [--period]` - Transaction categorization
- `/smith:review-conflicts` - Review transactions flagged for review
- `/smith:health [--full|--quick]` - Health check and recommendations
- `/smith:insights <spending|trends|scenario|report>` - Financial analysis and reports
- `/smith:tax <deductions|cgt|bas|eofy>` - Tax intelligence (ATO)
- `/smith:schedule [--setup|--status|--remove]` - Automated categorization scheduling

## Python Scripts

All Python scripts are in `scripts/` directory. **Always run with `uv run python -u`** for proper dependency resolution and unbuffered output.

### Core Operations

```bash
# Categorize transactions
uv run python -u scripts/operations/batch_categorize.py --mode=smart --period=2025-11

# Run health check
uv run python -u scripts/health/check.py --full

# Generate spending report
uv run python -u scripts/reporting/generate.py --period=2025 --format=all

# Tax deduction analysis
uv run python -u scripts/tax/deduction_detector.py --period=2024-25
```

### Script Organization

- **`scripts/core/`** - API client, rule engine, unified rules
- **`scripts/operations/`** - Categorization, batch processing, transaction updates
- **`scripts/analysis/`** - Spending analysis, trend detection
- **`scripts/reporting/`** - Multi-format report generation
- **`scripts/tax/`** - ATO mappings, deductions, CGT, BAS preparation
- **`scripts/scenarios/`** - Historical, projections, optimization, tax scenarios
- **`scripts/status/`** - Status dashboard for SessionStart hook
- **`scripts/scheduled/`** - Automated scheduled tasks (cron/launchd)
- **`scripts/health/`** - Health check engine, recommendations, monitoring
- **`scripts/workflows/`** - Interactive categorization workflows
- **`scripts/utils/`** - Backup, validation, logging, merchant normalization

## Unified Rule System

Agent Smith uses a YAML-based unified rule system for transaction categorization and labeling.

### Quick Start with Rules

```bash
# 1. Choose a template for your household type
uv run python scripts/setup/template_selector.py

# 2. Customize rules in data/rules.yaml

# 3. Test with dry run
uv run python scripts/operations/batch_categorize.py --mode=dry_run --period=2025-11

# 4. Apply to transactions
uv run python scripts/operations/batch_categorize.py --mode=apply --period=2025-11
```

### Example Rule

```yaml
rules:
  # Category rule - categorize transactions
  - type: category
    name: WOOLWORTHS → Groceries
    patterns: [WOOLWORTHS, COLES, ALDI]
    category: Food & Dining > Groceries
    confidence: 95

  # Label rule - apply labels based on context
  - type: label
    name: Shared Groceries
    when:
      categories: [Groceries]
      accounts: [Shared Bills]
    labels: [Shared Expense, Essential]
```

### Key Features
- **Two-phase execution**: Categories first, then labels
- **Pattern matching**: Regex patterns with exclusions
- **Confidence scoring**: 0-100% for auto-apply logic
- **LLM fallback**: AI categorization when rules don't match
- **Intelligence modes**: Conservative/Smart/Aggressive
- **Template system**: Pre-built rules for common household types
- **Operational modes**: DRY_RUN/VALIDATE/APPLY for safe testing

See [references/unified-rules-guide.md](references/unified-rules-guide.md) for complete documentation.

## Tax Intelligence

### ATO Compliance (Australian Tax Office)

**Three Levels**:

1. **Reference (Level 1)**:
   - ATO category mappings
   - Basic tax reports
   - GST tracking
   - Resource links

2. **Smart (Level 2)** - Recommended:
   - Deduction detection (14 pattern-based rules)
   - CGT tracking with FIFO matching
   - Confidence scoring
   - Substantiation threshold checking ($300, $75 taxi/Uber)
   - Time-based commuting detection
   - Instant asset write-off tracking

3. **Full (Level 3)** - Power Users:
   - BAS preparation with GST calculations
   - Compliance checks
   - Scenario planning
   - Audit-ready documentation
   - Professional advice disclaimers

**Configure Tax Level**:
```bash
# In .env file
TAX_INTELLIGENCE_LEVEL=smart  # reference|smart|full
TAX_JURISDICTION=AU
FINANCIAL_YEAR_END=06-30
```

### Deduction Detection Example

```python
from scripts.tax.deduction_detector import DeductionDetector

detector = DeductionDetector()
result = detector.detect_deduction(transaction)
# Returns: {"is_deductible": True, "confidence": "high",
#           "reason": "Office supplies", "substantiation_required": True}
```

### CGT Tracking Example

```python
from scripts.tax.cgt_tracker import CGTTracker, AssetType
from decimal import Decimal
from datetime import date

tracker = CGTTracker()
tracker.track_purchase(
    asset_type=AssetType.SHARES,
    name="BHP Group",
    quantity=Decimal("100"),
    purchase_date=date(2023, 1, 1),
    purchase_price=Decimal("45.50"),
    fees=Decimal("19.95")
)

event = tracker.track_sale(
    asset_type=AssetType.SHARES,
    name="BHP Group",
    quantity=Decimal("100"),
    sale_date=date(2024, 6, 1),
    sale_price=Decimal("52.00"),
    fees=Decimal("19.95")
)
# Returns CGTEvent with capital_gain, discount_eligible, holding_period_days
```

## Scenario Analysis

### Historical Analysis
```python
from scripts.scenarios.historical import calculate_what_if_spending

scenario = calculate_what_if_spending(
    transactions=transactions,
    category_name="Dining",
    adjustment_percent=-30.0,  # 30% reduction
    start_date="2025-01-01",
    end_date="2025-12-31"
)
print(f"Savings: ${scenario['savings']:.2f}")
```

### Spending Forecast
```python
from scripts.scenarios.projections import forecast_spending

forecast = forecast_spending(
    transactions=transactions,
    category_name="Groceries",
    months_forward=6,
    inflation_rate=3.0
)
```

### Optimization Suggestions
```python
from scripts.scenarios.optimization import suggest_optimizations

optimizations = suggest_optimizations(transactions=transactions)
print(f"Potential savings: ${optimizations['potential_annual_savings']:.2f}")
```

## Command Architecture Patterns

All Agent Smith commands follow consistent patterns for reliability and maintainability.

### 1. Deterministic Python Scripts

Commands delegate to git-tracked Python scripts in `scripts/` for all operations:

```
User Request → Slash Command → Python Script → PocketSmith API
     (UX)        (Guidance)      (Logic)         (Data)
```

**Why this matters**:
- Same inputs always produce same outputs
- All logic is testable and auditable
- No hidden state or side effects

### 2. Subagent Delegation Pattern

Commands use subagents to preserve main conversation context:

```markdown
**IMPORTANT: Delegate ALL work to a subagent to preserve main context window.**

Use the Task tool with `subagent_type: "general-purpose"` to execute...
```

**When to use subagents**:
- Batch operations (>50 transactions)
- Multi-step workflows with verbose output
- Operations that could pollute context

### 3. Guided Workflow Pattern

Every command follows this structure:

1. **Goal** - What will this accomplish?
2. **Why This Matters** - Business context
3. **Execution Steps** - Numbered steps with scripts
4. **Next Steps** - What to do after completion

### 4. Visual Style Guidelines

| Element | Usage |
|---------|-------|
| Emojis | Status: ✅ success, ⏳ processing, ⚠️ warning, ❌ error |
| Progress | `[23/100] Processing...` |
| Tables | Results summaries |
| ASCII bars | Health dimensions: `████████░░ 80%` |

### 5. Subagent Orchestration

Agent Smith uses intelligent subagent orchestration for complex operations:

**When to delegate to subagents**:
- Transaction count > 100
- Estimated tokens > 5000
- Bulk processing or deep analysis
- Multi-period operations
- Parallelization opportunities

**Subagent types**:
- `general-purpose` - Most operations (categorization, analysis, reporting)
- Purpose-built agents for specialized workflows

**Context preservation**: Main skill maintains user preferences, session state, and high-level decisions while subagents handle heavy processing.

## Health Check System

```bash
# Run comprehensive health check
uv run python scripts/health/check.py --full
```

**6 Health Dimensions** (scored 0-100):
1. **Data Quality (25%)** - Uncategorized rate, duplicates, gaps
2. **Category Structure (20%)** - Depth, unused categories, distribution
3. **Rule Engine (15%)** - Coverage, efficiency, conflicts
4. **Tax Readiness (15%)** - Tax categorization, compliance
5. **Automation (10%)** - Savings automation, bill scheduling
6. **Budget Alignment (15%)** - Variance, overspending

**Output includes**:
- Overall score with status (Poor/Fair/Good/Excellent)
- Individual dimension scores
- Top 3 prioritized recommendations
- Projected score after improvements

**Automated monitoring**:
- Weekly quick health checks
- Monthly full health analysis
- Pre-EOFY comprehensive check
- Post-major-operation validation

## Configuration Files

### `.env` (Required)
```bash
# PocketSmith API
POCKETSMITH_API_KEY=<Your Developer API Key>

# Agent Smith Configuration
TAX_INTELLIGENCE_LEVEL=smart          # reference|smart|full
DEFAULT_INTELLIGENCE_MODE=smart       # conservative|smart|aggressive
AUTO_BACKUP=true
AUTO_ARCHIVE=true
ALERT_NOTIFICATIONS=true

# Tax Configuration (Australia)
TAX_JURISDICTION=AU
FINANCIAL_YEAR_END=06-30             # June 30
GST_REGISTERED=false

# Reporting Preferences
DEFAULT_REPORT_FORMAT=all            # markdown|csv|json|html|excel|all
CURRENCY=AUD

# Advanced
API_RATE_LIMIT_DELAY=100             # ms between calls
CACHE_TTL_DAYS=7
SUBAGENT_MAX_PARALLEL=5
```

### `data/config.json` (User Preferences)
```json
{
  "user_id": 217031,
  "tax_level": "smart",
  "intelligence_mode": "smart",
  "alerts_enabled": true,
  "backup_before_mutations": true,
  "auto_archive": true,
  "default_report_formats": ["markdown", "csv", "html"]
}
```

### `data/rules.yaml` (Unified Rules)
See references/unified-rules-guide.md for complete schema and examples.

## Directory Structure

```
agent-smith/
├── SKILL.md                    # This file
├── .env                        # API configuration (not committed)
├── .env.sample                 # Configuration template
├── data/                       # Working data and state
│   ├── config.json             # User preferences
│   ├── rules.yaml              # Unified categorization rules
│   ├── templates/              # Pre-built rule templates
│   └── [other runtime data]
├── scripts/                    # Python code
│   ├── core/                   # API client, rule engine, utilities
│   ├── operations/             # Categorization, batch processing
│   ├── analysis/               # Spending analysis, trends
│   ├── reporting/              # Multi-format reports
│   ├── tax/                    # Tax intelligence (3-tier)
│   ├── scenarios/              # Scenario analysis
│   ├── orchestration/          # Subagent conductor
│   ├── workflows/              # Interactive workflows
│   ├── features/               # Advanced features
│   ├── health/                 # Health check system
│   └── utils/                  # Utilities
├── references/                 # Documentation
│   ├── pocketsmith-api.md      # PocketSmith API reference
│   ├── unified-rules-guide.md  # Complete rules documentation
│   ├── onboarding-guide.md     # First-time setup guide
│   └── health-check-guide.md   # Health check documentation
├── backups/                    # Timestamped backups (30-day retention)
├── logs/                       # Execution logs (14-day retention)
└── reports/                    # Generated reports (90-day retention)
```

## Critical Operating Rules

**NEVER create custom one-off scripts for core Agent Smith operations.**

Agent Smith has a complete architecture with:
- Slash commands (`/smith:categorize`, `/smith:analyze`, etc.)
- Python workflows in `scripts/workflows/`
- Core operations in `scripts/operations/`
- Proper API client with correct endpoints in `scripts/core/api_client.py`

**If you find yourself writing a new Python script to:**
- Categorize transactions
- Analyze spending
- Generate reports
- Update PocketSmith data
- Run health checks

**STOP. You are reinventing the wheel.**

**Instead:**
1. Check if a slash command exists (`/smith:*`)
2. Check if a workflow exists in `scripts/workflows/`
3. Check if a core operation exists in `scripts/operations/`
4. Use the existing code that has been tested and uses correct API endpoints

**Why this matters:**
- Custom scripts may use wrong API endpoints (e.g., `/users/{id}/transactions/{id}` instead of `/transactions/{id}`)
- Custom scripts bypass rule engine, validation, backup, and audit systems
- Custom scripts duplicate functionality that already exists
- Custom scripts won't benefit from future improvements

**Exception:** Only create new scripts when building NEW features not yet in the design, and always place them in the correct `scripts/` subdirectory.

## Best Practices

### Transaction Categorization
1. **Always use `/smith:categorize`** - Never create custom categorization scripts
2. Start with a rule template matching your household type
3. Use dry-run mode to preview categorization
4. Review and adjust rules based on results
5. Monitor rule performance metrics
6. Let AI handle edge cases (LLM fallback)

### Tax Compliance
1. Configure appropriate tax level for your needs
2. Track all deductible expenses as they occur
3. Maintain receipt documentation (>$300 ATO requirement)
4. Review tax reports before EOFY (June 30)
5. Consult registered tax agent for complex situations

### Financial Health
1. Run health checks monthly
2. Address high-priority recommendations first
3. Maintain 95%+ categorization rate
4. Review and clean up unused categories
5. Set up automated alerts for budget overruns

### Backup and Safety
1. Always backup before bulk operations (automatic)
2. Use dry-run mode for untested operations
3. Review audit trail for unexpected changes
4. Maintain 30 days of recent backups
5. Export tax reports for 7-year ATO retention

## References

For detailed documentation, see the `references/` directory:

- **[PocketSmith API](references/pocketsmith-api.md)** - Complete API reference
- **[Unified Rules Guide](references/unified-rules-guide.md)** - Rule system documentation
- **[Onboarding Guide](references/onboarding-guide.md)** - First-time setup walkthrough
- **[Health Check Guide](references/health-check-guide.md)** - Health system details
- **[Design Document](references/design.md)** - Complete system architecture

## Support

**Version**: 1.6.0

**Documentation**: See references/ directory for comprehensive guides

**Issues**: For questions or issues, refer to the design documentation or create an issue in the repository.

---

**Note**: Agent Smith is designed for Australian tax compliance (ATO). Adapting for other jurisdictions requires updating tax intelligence modules and reference documentation.
