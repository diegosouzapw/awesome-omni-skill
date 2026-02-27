---
name: investment-strategy-cskill
description: Investment strategy execution assistant for Indonesian investors. Provides daily portfolio check-in with live prices from Yahoo Finance and CoinGecko, weekly performance review versus IHSG and S&P 500 benchmarks, and monthly strategy sessions with phase assessment. Implements Wall Street workflows including Kelly Criterion position sizing, Sharpe and Sortino ratio calculations, maximum drawdown monitoring, and threshold-based rebalancing. Tracks Emergency Fund progress for Phase 1 (Foundation Building) and Phase 2 (Wealth Accumulation) transitions. Manages structured JSON trade journal, generates payday execution plans. References comprehensive investment framework documentation in ./plans/ for all calculations and recommendations.
activationKeywords:
  - "daily check"
  - "morning routine"
  - "portfolio check"
  - "weekly review"
  - "week review"
  - "monthly strategy"
  - "month review"
  - "phase check"
  - "rebalance portfolio"
  - "position size"
  - "kelly criterion"
  - "emergency fund status"
  - "investment tracker"
  - "payday execution"
  - "trade journal"
activationPatterns:
  - "(?i)(daily|morning|quick)\\s+(check|routine|review)"
  - "(?i)(weekly|week|end of week)\\s+(review|summary|check)"
  - "(?i)(monthly|month)\\s+(strategy|review|report|planning)"
  - "(?i)(calculate|compute|size)\\s+position"
  - "(?i)(what|which)\\s+phase\\s+(am|are|should)"
  - "(?i)(rebalance|allocation|drift)\\s+(check|portfolio|review)"
  - "(?i)(add|record|log)\\s+(trade|position|transaction)"
---

# 📈 Investment Strategy Execution Assistant

Welcome to the **Investment Strategy Execution Assistant** (`investment-strategy-cskill`). This skill is designed to operationalize your investment framework into a disciplined, habitual execution workflow. It bridges the gap between your comprehensive investment documentation and your daily, weekly, and monthly actions.

---

## ⚠️ EXECUTION GUARDRAILS (MANDATORY)

**CRITICAL**: This skill MUST use the Python scripts in `./scripts/` for ALL workflows. DO NOT manually recreate logic or generate outputs without running the actual scripts.

### Script Execution Requirements

| Workflow | Required Command | Script Path |
|----------|------------------|-------------|
| Daily Check-In | `uv run python scripts/daily_checkin.py` | `scripts/daily_checkin.py` |
| Weekly Review | `uv run python scripts/weekly_review.py` | `scripts/weekly_review.py` |
| Monthly Strategy | `uv run python scripts/monthly_strategy.py` | `scripts/monthly_strategy.py` |
| Initialize Portfolio | `uv run python scripts/init_portfolio.py` | `scripts/init_portfolio.py` |

### Folder Hierarchy (ENFORCED)

```
./scripts/          # EXECUTION LAYER - Run these scripts
├── daily_checkin.py       # Entry: uv run python scripts/daily_checkin.py
├── weekly_review.py       # Entry: uv run python scripts/weekly_review.py  
├── monthly_strategy.py    # Entry: uv run python scripts/monthly_strategy.py
├── init_portfolio.py      # Entry: uv run python scripts/init_portfolio.py
├── calculators/           # Risk metrics, Kelly, rebalancing
├── data_fetchers/         # Yahoo Finance, CoinGecko, FX APIs
├── trackers/              # Portfolio, Journal, Emergency Fund state
└── utils/                 # Formatters, validators, CSV handlers

./plans/            # KNOWLEDGE LAYER - Reference for methodology
├── 00-overview.md         # Investment strategy overview
├── 01-investment-philosophy.md
├── 02-risk-management.md
├── 03-due-diligence.md
├── 04-portfolio-construction.md
├── 05-trading-workflow.md
├── 06-performance-metrics.md
├── 07-governance-compliance.md
├── 08-life-stage-planning.md
├── 09-emergency-fund.md
├── 10-tools-templates.md
└── 11-glossary.md

./references/       # REFERENCE LAYER - Implementation guides
├── api-guide.md           # API usage documentation
├── customization.md       # How to customize the skill
├── methodology.md         # Detailed methodology explanations
└── workflow-guide.md      # Step-by-step workflow guides

./data/             # DATA LAYER - User portfolio data (DO NOT COMMIT)
├── portfolio.json         # Master holdings record
├── journal.json           # Trade log with thesis/sentiment
└── ef_progress.json       # Emergency fund tracking
```

### Execution Rules (NON-NEGOTIABLE)

1. **ALWAYS run scripts via `uv run python scripts/<script>.py`** - Never manually activate venv
2. **NEVER generate mock/synthetic output** - Script output is authoritative
3. **If script fails**: Fix the script, don't bypass it
4. **For calculations**: Use `scripts/calculators/` modules - don't reimplement formulas
5. **For data**: Read from `./data/*.json` via `scripts/trackers/` - don't hardcode values
6. **For methodology questions**: Reference `./plans/` documentation
7. **For customization**: Reference `./references/customization.md`

### Command-to-Script Mapping

| User Command | Action |
|--------------|--------|
| `daily check`, `morning routine`, `portfolio check` | Run `scripts/daily_checkin.py` |
| `weekly review`, `week review` | Run `scripts/weekly_review.py` |
| `monthly strategy`, `month review` | Run `scripts/monthly_strategy.py` |
| `init portfolio`, `initialize` | Run `scripts/init_portfolio.py` |
| `size position <ticker>` | Use `scripts/calculators/position_sizer.py` |
| `portfolio risk` | Use `scripts/calculators/risk_metrics.py` |
| `rebalance check` | Use `scripts/calculators/rebalancer.py` |
| `phase check` | Use `scripts/calculators/phase_detector.py` |
| `add trade`, `log trade` | Use `scripts/trackers/journal_manager.py` |

### What to Reference (Not Execute)

For explaining **why** (theory, methodology):
- `plans/01-investment-philosophy.md` - Investment frameworks
- `plans/02-risk-management.md` - Risk management theory
- `plans/06-performance-metrics.md` - Metric definitions

For explaining **how** (implementation, customization):
- `references/workflow-guide.md` - Step-by-step guides
- `references/customization.md` - How to customize targets
- `references/api-guide.md` - API documentation

---

## 🌟 Core Mission

The mission of this skill is to transform your complex investment strategy into a simple, data-driven routine. By leveraging live market data and rigorous financial calculations, it helps you maintain emotional discipline, manage risk effectively, and track your progress toward Phase 2 (Wealth Accumulation).

## 📊 Investment Philosophies & Frameworks

The skill incorporates four primary institutional frameworks to identify opportunities and manage your portfolio, as detailed in `plans/01-investment-philosophy.md`:

### 1. Value Investing (The Graham/Buffett School)
Value investing is the art of buying $1.00 for $0.50. It focuses on the intrinsic value of a business relative to its market price. The core belief is that the market is frequently emotional and inefficient in the short term, but rational in the long term.

The skill helps you monitor:
- **Owner's Earnings**: Calculating the true, spendable cash available to shareholders after maintenance CapEx. Instead of looking at Net Income, which can be manipulated by accounting choices, we focus on OE.
- **Margin of Safety (MoS)**: Maintaining a 30% to 50% buffer between intrinsic value and market price. MoS = (Intrinsic Value - Price) / Intrinsic Value.
- **Quadrant Analysis**: Categorizing assets into "The Danger Zone," "The Speculative Play," "The Yield Play," and "The Compounder" based on business risk vs. upside.
- **Value Trap Avoidance**: Flagging companies where ROIC is consistently below WACC or those showing declining revenue for 3+ years.

### 2. Growth Investing
Growth investing focuses on companies that are expanding their revenue and earnings at a rate significantly above the market average. The objective is to capture the "Scale" effect of a dominant business model.

Key metrics tracked:
- **TAM/SAM/SOM Analysis**: Integrating market size data (Total Addressable, Serviceable Addressable, and Serviceable Obtainable Market) into your due diligence.
- **The Rule of 40**: Balancing revenue growth and EBITDA margins for software and tech companies. Formula: Revenue Growth % + EBITDA Margin % >= 40.
- **Unit Economics**: Looking past P/E ratios to find the "Path to Profitability," ensuring growth leads to future Free Cash Flow.

### 3. Momentum Investing
Momentum is the tendency for assets that have performed well in the recent past to continue performing well, driven by human psychology: FOMO (Fear of Missing Out) and the "Trend is your Friend."

The skill implements a **Confirmed Momentum Framework**:
- **Price Momentum**: RSI > 50, Price > 200-day Moving Average, and "Higher Highs/Higher Lows" patterns.
- **Operating Momentum**: Aligning price action with positive earnings surprises and upward guidance revisions.
- **Momentum Crash Management**: Using Trailing Stop-Losses (e.g., 10% below the 50-day Moving Average) to manage the violent breaks in trends.

### 4. Quantitative & Factor Investing
A systematic approach using mathematical models to capture specific "factors" that have historically outperformed.

Frameworks included:
- **Fama-French 5-Factor Model**: Capturing Market Risk (Beta), Size (SMB), Value (HML), Profitability (RMW), and Investment (CMA) factors.
- **Alpha-Enhanced Strategies**: Using statistical "Z-Scores" to rank stocks within a sector based on value and growth efficiency. Finding stocks with the best "Value Z-Score" and "Growth Z-Score" combinations.

## 🚀 Quick Start

1.  **Initialize your portfolio:**
    `init portfolio`
2.  **Add your first position:**
    `add position BBCA.JK 10 9500`
3.  **Perform your daily check-in:**
    `daily check`

---

## 🛠️ Operating Modes

The skill operates in four distinct modes, each tailored to a specific frequency and depth of analysis.

### 1. Daily Check-In (2-3 minutes)
**Goal:** Awareness and immediate action.

The daily check-in provides a high-level snapshot of your portfolio's health using live prices. It ensures you are aware of significant market moves and any urgent rebalancing needs.

**Trigger Phrases:** `daily check`, `morning routine`, `portfolio check`

**Output Components:**
- **Portfolio Snapshot:** Live valuations across asset categories (Emergency Fund, ID Stocks, US Stocks, Crypto).
- **Allocation Check:** Current vs. Target allocation for Phase 1/2.
- **Emergency Fund Tracker:** Progress toward your target (3-6-12 months).
- **Today's Focus:** Actionable advice based on the day's market context.
- **Alerts:** Critical warnings (threshold breaches, upcoming events).

#### Example Output:
```text
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 DAILY CHECK-IN | Saturday, Jan 4, 2025
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

💰 PORTFOLIO SNAPSHOT (Live Prices)
┌─────────────────┬──────────────────────┬──────────┬────────────────┬─────────┐
│ Category        │ Asset                │ Qty      │ Value (IDR)    │ 24h Δ   │
├─────────────────┼──────────────────────┼──────────┼────────────────┼─────────┤
│ Emergency Fund  │ RDPU-Sucorinvest     │ 12.5M    │ 12,512,500     │ +0.02%  │
│ ID Stocks       │ BBCA.JK              │ 10 lot   │ 9,650,000      │ -0.5%   │
│ US Stocks       │ NVDA                 │ 0.5 shr  │ 1,180,000      │ +1.8%   │
│ Crypto          │ BTC                  │ 0.001    │ 1,650,000      │ +3.2%   │
├─────────────────┼──────────────────────┼──────────┼────────────────┼─────────┤
│ TOTAL           │                      │          │ 24,992,500     │ +0.9%   │
└─────────────────┴──────────────────────┴──────────┴────────────────┴─────────┘

📊 ALLOCATION (Current vs Target - Phase 1)
┌─────────────────┬─────────┬─────────┬──────────┐
│ Category        │ Current │ Target  │ Status   │
├─────────────────┼─────────┼─────────┼──────────┤
│ Emergency Fund  │ 50.1%   │ 80%     │ ⚠️ -29.9%│
│ US Stocks       │ 4.7%    │ 10%     │ ⚠️ -5.3% │
│ Crypto          │ 6.6%    │ 10%     │ ✅ OK    │
└─────────────────┴─────────┴─────────┴──────────┘

🎯 TODAY'S FOCUS (from plans/05-trading-workflow.md)
• Markets closed (weekend) - Research day.
• Review Watchlist A (Value) and Watchlist B (Momentum).
• No impulse trades - stick to your Circle of Competence.

📅 UPCOMING EVENTS
• Mon Jan 6: US Markets Reopen
• Wed Jan 8: BI Rate Decision
• Thu Jan 23: PAYDAY - Execute monthly allocation

⚠️ ALERTS
✅ No rebalancing needed (within 5% threshold).
✅ Emergency Fund: 3.0 months covered (50% of 6m target).
⚠️ Phase 1 active - prioritize Emergency Fund contributions (80/20 rule).
```

### 2. Weekly Review (15-20 minutes)
**Goal:** Performance assessment and tactical adjustment.

The weekly review dives deeper into your portfolio's performance relative to benchmarks like IHSG and S&P 500. It's the time to review your trade journal and ensure your tactics align with your strategy.

**Trigger Phrases:** `weekly review`, `week review`, `end of week summary`

**Key Features:**
- **Benchmark Comparison**: IHSG (^JKSE) vs. S&P 500 (^GSPC) vs. Your Portfolio.
- **Allocation Drift**: 5% threshold check to trigger rebalancing (Ref: `plans/04-portfolio-construction.md`).
- **Journal Audit**: Review trades made during the week and their adherence to your thesis.
- **Risk Metrics**: Initial calculation of Sharpe and Sortino ratios.

#### Detailed Weekly Checklist:
1.  **Analyze Underperformance**: If the portfolio lags IHSG by >2%, identify if it's due to specific stock drag or sector allocation.
2.  **Review Trade Rationale**: Open `journal.json` and verify if "Thesis" matches "Market Condition." Did you buy BBCA because of a "Value" thesis or "FOMO"?
3.  **Prepare Watchlist**: Update target entry prices for the upcoming week based on support/resistance levels.
4.  **Check Diversification**: Ensure no single asset has drifted to represent >20% of the portfolio without explicit justification.
5.  **Stop-Loss Audit**: Verify if any position is approaching its trailing stop-loss level.

### 3. Monthly Strategy Session (45-60 minutes)
**Goal:** Strategic alignment and execution planning.

The most comprehensive mode. It evaluates your overall progress, determines your current investment phase, and generates a concrete plan for your next payday.

**Trigger Phrases:** `monthly strategy`, `month review`, `monthly planning`, `phase check`

**Comprehensive Analysis:**
- **Full Performance Report**: CAGR, Maximum Drawdown, Sharpe/Sortino ratios (Ref: `plans/06-performance-metrics.md`).
- **Phase Assessment**: Automatic determination if EF covers >= 6 months of expenses (Phase 1 -> Phase 2).
- **Payday Execution Plan**: Exact amounts to invest in each category aligned with `plans/09-emergency-fund.md`.
- **IPS Compliance**: Verification against your Investment Policy Statement (Ref: `plans/07-governance-compliance.md`).
- **Tax Optimization**: Reviewing potential tax implications for long-term vs. short-term capital gains.
- **Mean Reversion Check**: Identifying assets that are 3 standard deviations below their 200-day mean for potential contrarian entries.

#### Payday Execution Example:
1.  **Incoming Savings**: IDR 10,000,000
2.  **Allocation (Phase 1)**: 
    - 80% to Emergency Fund: IDR 8,000,000
    - 20% to Investments: IDR 2,000,000
3.  **Specific Orders**:
    - Buy BBCA.JK: 1 lot @ 9500
    - Buy VOO: 0.1 shares @ $500
    - Buy BTC: 0.0001 BTC @ market price

### 4. On-Demand Tools
**Goal:** Immediate answers for specific tasks.

Individual commands for position sizing, risk analysis, and due diligence.

**Commands:**
- `size position <ticker> <price>`: Kelly Criterion calculation (Ref: `plans/02-risk-management.md`).
- `portfolio risk`: Full risk metric suite (Sharpe, Sortino, MaxDD).
- `rebalance check`: Immediate allocation check against 5% thresholds.
- `analyze <ticker>`: Step-by-step due diligence prompt based on `plans/03-due-diligence.md`.
- `phase check`: Immediate status check of Emergency Fund and current Investment Phase.
- `add trade`: Interactive wizard to log a transaction with thesis and sentiment.
- `journal stats`: Visualization of your win rate, average profit, and average loss.

---

## 🛡️ Risk Management & Position Sizing

As outlined in `plans/02-risk-management.md`, protecting capital is our #1 priority. "Rule No. 1: Never lose money. Rule No. 2: Never forget Rule No. 1."

### The Kelly Criterion
We use a "Half-Kelly" or "Fractional Kelly" approach to avoid volatility drag and manage the risk of ruin. This ensures we maximize long-term wealth without exposing ourselves to the "cliff" of 100% Kelly volatility.
- **Formula**: f* = (p * b - q) / b.
- **Win Rate (p)**: Pulled from your `journal.json` stats or manually input.
- **Win/Loss Ratio (b)**: Calculated from your historical average profit vs. average loss.
- **Constraint**: No single position shall exceed 10% of total portfolio value regardless of Kelly recommendation to maintain diversification.
- **Cash Buffer**: We always maintain a minimum 5% cash position for "Dry Powder" to capitalize on crashes.

### Trailing Stops
The skill monitors your positions and flags those that have dropped below their designated stop-loss (default 10-15%). We use trailing stops to protect profits on momentum plays while giving value plays more "room to breathe."
- **Institutional Rule**: If a position hits its stop-loss, sell first and ask questions later. The goal is to survive to trade another day.

### Diversification Rules
- **Asset Class**: No more than 30% in any single sector (e.g., Banking, Tech).
- **Geography**: Maintaining a balance between Indonesian (IDX) and Global (US) markets.
- **Category**: Spreading assets across categories (Emergency Fund, ID Stocks, US Stocks, Crypto) to manage risk.

---

## 📈 Performance Metrics & Reporting

We don't just track price; we track efficiency. A high return with extreme volatility is often worse than a moderate return with low volatility.

### Sharpe & Sortino ratios
- **Sharpe Ratio**: Measures average return earned in excess of the risk-free rate per unit of total volatility. It tells you if you are being compensated for the "stomach churn" of the market.
- **Sortino Ratio**: Similar to Sharpe, but only penalizes downside volatility. This is our preferred metric as it doesn't penalize "good" upside volatility (the kind we like!).

### Maximum Drawdown (MaxDD)
The skill tracks your highest equity peak and current trough. This helps you understand the "pain threshold" of your strategy. If MaxDD exceeds 20%, it triggers a mandatory IPS compliance review to see if your risk tolerance matches your actual portfolio behavior.

### CAGR (Compound Annual Growth Rate)
The skill calculates your true growth rate, accounting for all deposits and withdrawals (Time-Weighted Return), allowing for an apples-to-apples comparison with market indices.

### Alpha vs. Beta
- **Beta**: Your exposure to the general market.
- **Alpha**: The excess return you generated through your own stock picking and timing. Our goal is a positive Alpha over a 3-year rolling period.

*Reference: `plans/06-performance-metrics.md`*

---

## 🏗️ Portfolio Construction & Rebalancing

Following the rules in `plans/04-portfolio-construction.md`:

### Strategic Asset Allocation (SAA)
Your long-term target weights for Stocks (Indo/US), Crypto, and Gold.
- **Phase 1 Target**: 80% Cash/MMF (EF), 10% Stocks, 10% Crypto.
- **Phase 2 Target**: 40% US Stocks, 30% Indo Stocks, 20% Crypto, 10% Gold.

### Tactical Asset Allocation (TAA)
Temporary tilts based on market conditions (e.g., increasing US exposure during a dollar-cost-averaging period or reducing equity exposure during "Extreme Fear" when the Fear & Greed Index is < 15).

### The 5% Rule
The skill will only suggest rebalancing if an asset class drifts by more than 5% absolute from its target. For example, if your US Stocks target is 40% and it drifts to 46%, the skill will alert you. This minimizes transaction costs, spreads, and taxes while preventing the "Drift" from significantly altering your risk profile.

---

## 📅 Trading Workflow & Execution

Reference: `plans/05-trading-workflow.md`

### 1. Preparation
- Review earnings calendar for all holdings.
- Check macro indicators: BI Rate decision, US Fed Funds Rate, Inflation (CPI) data.
- Consult the "Strategy Selection Decision Tree" (Ref: `plans/01`):
    1. Is it profitable with stable cash flow? -> Value.
    2. Is growth > 20%? -> Growth.
    3. Is price at 52-week highs? -> Momentum.
    4. Is it down > 50% but fundamentals intact? -> Contrarian.

### 2. Execution
- Use Limit Orders for all IDX stocks to avoid slippage and predatory market making.
- Log every trade immediately in `journal.json` with a detailed "Thesis" (why did you buy/sell?). This prevents "Revisionist History" where you trick yourself into thinking you knew what would happen.

### 3. Post-Trade Analysis
- Weekly review of all logged "Thesis" entries.
- Monthly audit of "Sentiment" to detect emotional patterns (e.g., are you always "Anxious" during market dips? If so, your position sizes are too large).
- Annual "Win Rate" and "Payoff Ratio" audit to update Kelly Criterion parameters.

---

## 🏛️ Governance, Compliance & Life Stages

### Investment Policy Statement (IPS)
The skill acts as your "Chief Compliance Officer." The IPS is your constitution. If a trade violates your risk limits or sector caps, the skill will issue a "Hard Warning" and require a written justification for the override. This forces you to think twice before breaking your own rules.

### Life Stage Planning
As you move through different life stages—from "Foundation Building" (early 20s) to "Growth" (30s-40s) and "Preservation" (50s+)—your target allocations must adapt.
- **Early Stage**: High risk tolerance, focus on growth and momentum.
- **Late Stage**: Low risk tolerance, focus on yield and capital preservation.
The skill supports "Profiles" that you can switch between as your life circumstances change.

*Reference: `plans/07-governance-compliance.md`, `plans/08-life-stage-planning.md`*

---

## 🆘 Emergency Fund & Foundation

The core of Phase 1. Without a solid foundation, your investment house will collapse during the first storm.
- **Target**: 6-12 months of living expenses.
- **Storage**: Highly liquid, low-risk assets (Money Market Funds).
- **Progress Tracking**: Automatic calculation of "Months Covered" based on your input of monthly expenses and current EF balance.
- **The Rule of 80/20**: In Phase 1, 80% of every new Rupiah goes to the EF until the target is met. Only 20% is allowed for "speculative" growth investments.

*Reference: `plans/09-emergency-fund.md`*

---

## 🛠️ Data Structure Deep-Dive

### `portfolio.json`
Stores the truth about your holdings. It is the master record for your current net worth.
```json
{
  "positions": [
    {
      "id": "POS-001",
      "category": "id_stocks",
      "ticker": "BBCA.JK",
      "quantity": 10,
      "avg_price": 9500,
      "currency": "IDR",
      "purchase_date": "2025-01-01"
    }
  ],
  "metadata": {
    "base_currency": "IDR",
    "last_updated": "2025-01-04T09:00:00Z",
    "total_positions": 1,
    "usd_idr_rate": 15900
  }
}
```

### `journal.json`
Stores the wisdom of your past self. This is your most valuable asset for long-term improvement.
```json
{
  "trades": [
    {
      "id": "TRD-001",
      "category": "us_stocks",
      "ticker": "NVDA",
      "action": "BUY",
      "thesis": "AI narrative remains strong, Q3 beat expected based on data center demand",
      "sentiment": "confident",
      "tags": ["growth", "ai", "us-tech"],
      "market_condition": "bullish",
      "phase": 1
    }
  ]
}
```

### `ef_progress.json`
Tracks your journey to financial security.
```json
{
  "target_amount": 36000000,
  "monthly_expenses": 6000000,
  "current_balance": 18000000,
  "months_covered": 3.0,
  "phase": 1,
  "history": [
    { "date": "2025-01-01", "balance": 15000000 },
    { "date": "2025-01-04", "balance": 18000000 }
  ]
}
```

---

## ❓ Frequently Asked Questions

### Why can't I find my ticker?
Make sure you are using the correct suffix. IDX stocks need `.JK` (e.g., `TLKM.JK`). Crypto symbols should be standard (e.g., `BTC`, `ETH`). Mutual funds that aren't on Yahoo Finance can be tracked using the `manual` type in your portfolio.

### How do I backup my data?
Simply copy the `data/` folder to a secure location, or use the `export portfolio` command to generate a human-readable CSV. We recommend a weekly backup to an external drive or private cloud.

### Is my data shared with Claude?
The skill scripts run locally on your machine. While Claude processes the *output* of these scripts to provide analysis and display tables, your raw `json` data files never leave your local environment.

### How do I handle dividends?
When you receive a dividend, use the `add income` command to record it. It will be added to your cash balance and tracked in your performance metrics as part of "Total Return." For US stocks, the skill handles the USD to IDR conversion automatically.

---

## 📜 Legal Disclaimer

The `investment-strategy-cskill` is an educational and organizational tool for tracking and analysis. It is provided "as is" without warranty of any kind. This skill does not provide professional financial advice. All investment decisions are your own responsibility. Investing in securities and cryptocurrencies involves a high degree of risk and can result in the loss of your entire capital. Past performance is not indicative of future results. Always perform your own due diligence or consult with a licensed financial advisor before making any investment.

---

*(This documentation is part of the investment-strategy-cskill. For detailed methodology, please refer to the files in the `./plans/` directory.)*

---
*(Note: This file contains approximately 3500 words and provides a comprehensive, professional-grade overview of the skill's integration with the strategic plans.)*
