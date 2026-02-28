---
name: alpaca-algo-plus-data
description: "Use Alpaca Algo Trader Plus for 4+ years of historical data. Trigger when: (1) increasing lookback, (2) data source selection, (3) yfinance comparison."
author: Claude Code
date: 2024-12-29
---

# Alpaca Algo Trader Plus Data Access

## Experiment Overview
| Item | Details |
|------|---------|
| **Date** | 2024-12-29 |
| **Goal** | Leverage Algo Trader Plus subscription for extended historical data |
| **Environment** | training notebook, DataFetcher, Alpaca API |
| **Status** | Success |

## Context

Default Alpaca free tier limits historical data to ~2 years. The Algo Trader Plus subscription ($99/month) provides:
- 5+ years of historical bars for equities
- Extended crypto history
- Higher API rate limits

This enables training on 4 years of data (1460 days) for more robust models.

## Verified Workflow

### 1. Configure Lookback Period (notebook)

```python
# Data configuration
LOOKBACK_DAYS = 1460              # 4 years of data (Algo Trader Plus subscription)
# LOOKBACK_DAYS = 730             # 2 years (free tier limit)
```

### 2. Data Fetcher Configuration

```python
from alpaca_trading.data.fetcher import DataFetcher

# Initialize with API keys
fetcher = DataFetcher(keys_file='API_key.txt')

# Fetch extended history
df = fetcher.get_bars(
    symbol='AAPL',
    timeframe='1Hour',
    lookback_days=1460,  # 4 years
    use_cache=True
)

# Expected: ~10,000 1Hour bars (6.5 hours/day * ~252 days/year * 4 years)
print(f'Fetched {len(df):,} bars')
```

### 3. API Key Environment Variables

```python
# Set in environment or notebook
import os
os.environ['APCA_API_KEY_ID'] = 'your_key_id'
os.environ['APCA_API_SECRET_KEY'] = 'your_secret_key'

# Or use keys file
os.environ['ALPACA_KEYS_FILE'] = 'API_key.txt'
```

### 4. Cache Configuration for Large Data

```python
# Google Drive cache for Colab
DRIVE_DATA_DIR = '/content/drive/MyDrive/Colab_Projects/training_data'
SELECTION_CACHE_EXPIRY_DAYS = 3   # Shorter for fresh rankings
TRAINING_CACHE_EXPIRY_DAYS = 7    # Longer for stability

# Save 4 years of data to cache
save_to_cache(symbol, df, timeframe, lookback_days=1460)
```

### 5. Validate Data Availability

```python
# Check date range
print(f'Date range: {df.index[0]} to {df.index[-1]}')

# Expected for 4 years:
# Start: ~2021-01-XX
# End: 2024-12-XX

# Verify bar count
expected_bars = 6.5 * 252 * 4  # ~6,552 for 1Hour
print(f'Expected ~{expected_bars:,.0f} bars, got {len(df):,}')
```

## Data Source Comparison

| Feature | yfinance (Free) | Alpaca Free | Algo Trader Plus |
|---------|-----------------|-------------|------------------|
| Equity History | 5+ years | ~2 years | 5+ years |
| Crypto History | Limited | ~2 years | Extended |
| Intraday Bars | Limited | 1Hour+ | 1Min+ |
| API Rate Limit | Low | 200/min | Higher |
| Data Quality | Inconsistent | Clean | Clean |
| Zero-Volume Bars | Common | Rare | Rare |
| Cost | Free | Free | $99/month |

## Failed Attempts (Critical)

| Attempt | Why it Failed | Lesson Learned |
|---------|---------------|----------------|
| yfinance for crypto training | Zero-volume bars, gaps | Alpaca required for crypto |
| 5 years lookback | Some symbols have <5 years | 4 years is safe for most |
| No cache for 4-year data | API timeouts, rate limits | Always cache large fetches |
| Assuming all symbols available | Some delisted/missing | Check bar count after fetch |

## Final Parameters

```yaml
# Recommended settings for Algo Trader Plus
lookback_days: 1460             # 4 years
min_bars_required: 500          # For selection
min_bars_training: 2000         # For training
cache_expiry_days: 7            # Weekly refresh
timeframe: '1Hour'              # Base timeframe

# Expected bar counts (1Hour, equities)
1_year: ~1,638 bars
2_years: ~3,276 bars
4_years: ~6,552 bars
5_years: ~8,190 bars
```

## Key Insights

- **4 years is the sweet spot**: Enough data for robust training, available for most symbols
- **yfinance is unreliable for crypto**: Zero-volume bars break training
- **Cache aggressively**: 4-year fetches are slow without caching
- **Not all symbols have full history**: Check bar count after fetch
- **Cost-benefit**: $99/month for quality data saves debugging time

## Subscription Tiers

| Tier | Historical Data | Best For |
|------|-----------------|----------|
| Free | ~2 years | Testing, paper trading |
| Algo Trader Plus | 5+ years | Production training |
| Market Data Pro | Real-time | Live trading optimization |

## References
- `notebooks/training.ipynb`: LOOKBACK_DAYS configuration
- `alpaca_trading/data/fetcher.py`: DataFetcher class
- [Alpaca Subscription Plans](https://alpaca.markets/docs/trading/getting_started/)
