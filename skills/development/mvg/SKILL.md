---
name: mvg
description: Munich public transport (MVG) - departures, schedules, and alerts. Use when user shares location in Munich, asks about transport schedules, nearby stops, or transit disruptions.
---

# MVG Skill 🚇

Munich public transport departures, schedules, and alerts.

## 1. Departures by Location

When user shares location or asks about nearby transport:

```bash
python3 mvg-departures.py <lat> <lng>

# With options
python3 mvg-departures.py <lat> <lng> --limit 5 --offset 3 --compact
```

### Options

| Flag | Description |
|------|-------------|
| `--limit N` | Max departures (default: 8) |
| `--offset MIN` | Walking time to stop (filters out departures leaving too soon) |
| `--types U,S,BUS,TRAM` | Filter by transport type |
| `--compact` | One-line format |
| `--json` | JSON output |

### Examples

```bash
# Central Munich
python3 mvg-departures.py 48.137 11.575

# Only U-Bahn and S-Bahn, 5 min walk to station
python3 mvg-departures.py 48.137 11.575 --types U,S --offset 5

# Compact output
python3 mvg-departures.py 48.137 11.575 --compact
```

### Output Format

```
📍 **Marienplatz** (München)

🚇 U3 → Fürstenried West (2 min)
🚇 U6 → Klinikum Großhadern (3 min)
🚆 S1 → Freising (5 min +2)
```

Delay shown as `(+N)` minutes.

## 2. On Location Share

When user sends a location via Telegram/Signal, automatically show nearby departures:

```bash
python3 mvg-departures.py <lat> <lng> --limit 6 --compact
```

## 3. Alerts & Disruptions

Check for strikes, disruptions, construction:

```bash
web_search "München MVG Streik Störung" --freshness pd
```

### Keywords

- `Streik` — strike
- `Störung` — disruption  
- `Sperrung` — closure
- `Ausfall` — cancellation
- `Verspätung` — delay
- `Bauarbeiten` — construction

### MVG vs S-Bahn

| Service | Operator | During MVG Strike |
|---------|----------|-------------------|
| U-Bahn | MVG | ❌ Affected |
| Tram | MVG | ❌ Affected |
| Bus (1-199) | MVG | ❌ Affected |
| **S-Bahn** | **DB** | ✅ Usually runs |
| Regional trains | DB | ✅ Usually runs |

⚠️ S-Bahn is operated by Deutsche Bahn, not MVG!

## 4. Heartbeat Integration

Add to HEARTBEAT.md for automated checks:

```markdown
### 🚨 MVG Alerts (every heartbeat)
- web_search: "München MVG Streik Störung" freshness=pd
- Report NEW alerts only (check reportedAlerts in state)

### 📍 On Location
- python3 mvg-departures.py <lat> <lng> --limit 6
```

Track reported alerts in state file:

```json
{
  "reportedAlerts": ["mvg-streik-2026-02-02"],
  "acknowledgedAlerts": []
}
```

## 5. API Reference

Uses unofficial MVG API:

```
Base: https://www.mvg.de/api/bgw-pt/v3

GET /stations/nearby?latitude=X&longitude=Y
GET /departures?globalId=STATION_ID&limit=N&offsetInMinutes=M
```

Python package: `pip install mvg`

## Requirements

```bash
pip install mvg
```

## Munich Tips

- **Rush hours**: 7:00-9:00, 16:30-19:00
- **Oktoberfest**: U4/U5 crowded (late Sept)
- **Allianz Arena**: U6 packed on match days
- **Night service**: Limited after 00:30 (weekdays)
