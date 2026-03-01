---
name: esios
description: Query Spanish electricity market data (ESIOS/REE). Use when the user asks about electricity prices, generation, demand, I90 files, or any ESIOS indicator.
version: 2.0.0
---

# ESIOS Data Assistant

You have access to the `python-esios` CLI and library for querying the Spanish electricity market (ESIOS/REE).

## CLI Reference

### Indicators

```bash
# List all indicators
esios indicators list

# Search by name
esios indicators search "precio"

# Show metadata (unit, granularity, geographies)
esios indicators meta 600

# Historical data
esios indicators history 600 --start 2025-01-01 --end 2025-01-31
esios indicators history 600 -s 2025-01-01 -e 2025-01-31 --format csv --output data.csv
esios indicators history 600 -s 2025-01-01 -e 2025-01-31 --format parquet --output data.parquet

# Filter by geography (ID or name)
esios indicators history 600 -s 2025-01-01 -e 2025-01-31 --geo España
esios indicators history 600 -s 2025-01-01 -e 2025-01-31 --geo 3

# Ad-hoc pandas expressions on fetched data
esios indicators exec 600 -s 2025-01-01 -e 2025-01-31 --expr "df.describe()"
esios indicators exec 600 -s 2025-01-01 -e 2025-01-31 --expr "df.resample('D').mean()"
```

### Archives

```bash
esios archives list
esios archives download 1 --start 2025-01-01 --end 2025-01-31 --output ./data
```

### Cache Management

```bash
esios cache status    # Path, size, geos registry, catalog info
esios cache geos      # Global geo_id → geo_name registry
esios cache path      # Print cache directory
esios cache clear     # Clear indicator cache
esios cache clear --all  # Clear everything (indicators, archives, geos, catalog)
esios cache clear --indicator 600  # Clear one indicator
```

### Configuration

```bash
esios config set token <API_KEY>
esios config show
```

## Common Indicator IDs

| ID | Name | Description | Geos |
|----|------|-------------|------|
| 600 | Precio mercado spot | OMIE spot market price | ES, PT, FR, DE, BE, NL |
| 1001 | Precio mercado diario | Day-ahead market price | ES |
| 10033 | Demanda real | Real-time electricity demand | ES |
| 10034 | Generación eólica | Real-time wind generation | ES |
| 10035 | Generación solar FV | Real-time solar PV generation | ES |
| 1293 | Demanda prevista | Forecasted demand | ES |

Use `esios indicators search "query"` to find more. Use `esios indicators meta <id>` to see full metadata including geographies and units.

## Multi-Geo Indicators

Some indicators (e.g. 600) return data for multiple countries. The output is pivoted so each geography becomes a column:

```
datetime                España  Portugal  Francia  Alemania  Bélgica  Países Bajos
2025-01-01 00:00:00     63.50    63.50     72.10    58.20     58.20     58.20
2025-01-01 01:00:00     55.80    55.80     60.30    48.90     48.90     48.90
```

Filter to specific geos with `--geo`:
```bash
esios indicators history 600 -s 2025-01-01 -e 2025-01-07 --geo España --geo Portugal
```

## Geography Reference

| geo_id | geo_name |
|--------|----------|
| 1 | Portugal |
| 2 | Francia |
| 3 | España |
| 8826 | Alemania |
| 8827 | Bélgica |
| 8828 | Países Bajos |

The `--geo` flag accepts both IDs and names (case-insensitive substring match):
- `--geo 3` or `--geo España` or `--geo españa`
- `--geo "Países Bajos"` or `--geo 8828`

## Python Library

```python
from esios import ESIOSClient

client = ESIOSClient()  # reads config file, then ESIOS_API_KEY env var

# Get indicator handle
handle = client.indicators.get(600)

# Historical data as DataFrame
df = handle.historical("2025-01-01", "2025-01-31")

# Filter by geo
df = handle.historical("2025-01-01", "2025-01-31", geo_ids=[3])  # España only

# Inspect geographies
handle.geos             # List of {"geo_id": int, "geo_name": str}
handle.geos_dataframe() # DataFrame with geo_id and geo_name columns
handle.resolve_geo("España")  # Returns 3

# Search and compare
results = client.indicators.search("precio")
df = client.indicators.compare([600, 10034, 10035], "2025-01-01", "2025-01-07")
```

## I90 Settlement Files

```python
from esios.processing import I90Book

book = I90Book("path/to/I90DIA_20250101.xls")
sheet = book["3.1"]       # Access specific sheet
df = sheet.df             # Preprocessed DataFrame with datetime index
print(sheet.frequency)    # "hourly" or "hourly-quarterly"
```

## Caching Behavior

- Data is cached locally as parquet files (`~/.cache/esios/`)
- Each indicator gets its own directory: `indicators/{id}/data.parquet`
- Indicator metadata cached in `indicators/{id}/meta.json` (7-day TTL)
- Indicator catalog cached in `indicators/catalog.json` (24h TTL)
- Global geo registry at `geos.json` (persisted forever, grows incrementally)
- Data older than 48h is considered final (won't be re-fetched)
- Recent data (last 48h) is re-fetched on each request (electricity market corrections)
- Cache is per-column sparse: fetching `--geo España` only caches that column

## Key Conventions

- All timestamps are in Europe/Madrid timezone
- Date ranges > 3 weeks are auto-chunked into smaller API requests
- Archives support skip-existing (won't re-download cached files)
- I90 sheets detect hourly vs quarter-hourly frequency automatically
- API token resolution: config file (`~/.config/esios/config.toml`) > `ESIOS_API_KEY` env var
- Custom exceptions: `ESIOSError`, `AuthenticationError`, `APIResponseError`, `NetworkError`
