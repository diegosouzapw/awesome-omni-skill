---
name: sports-oracle
description: Sports data for prediction market trading. Get live scores, team stats, schedules, and injury reports for NFL, NBA, MLB, NHL.
metadata: {"moltbot":{"emoji":"🏈","requires":{"bins":["curl","jq"]}}}
---

# Sports Oracle 🏈

Real-time sports data to support prediction market trading decisions.

## Quick Start

```bash
# Get today's NFL scores
./scripts/scores.sh nfl

# Get team stats
./scripts/team.sh nfl patriots

# Get upcoming schedule
./scripts/schedule.sh nfl

# Get injuries for a team
./scripts/injuries.sh nfl chiefs
```

## Supported Sports

| Sport | Code | Coverage |
|-------|------|----------|
| NFL | `nfl` | Scores, standings, schedule, injuries |
| NBA | `nba` | Scores, standings, schedule, injuries |
| MLB | `mlb` | Scores, standings, schedule |
| NHL | `nhl` | Scores, standings, schedule |

## Commands

### Live Scores
```bash
./scripts/scores.sh <sport>
# Example: ./scripts/scores.sh nfl
```

### Team Info & Stats
```bash
./scripts/team.sh <sport> <team>
# Example: ./scripts/team.sh nfl seahawks
```

### Schedule
```bash
./scripts/schedule.sh <sport> [weeks]
# Example: ./scripts/schedule.sh nfl 2
```

### Injuries
```bash
./scripts/injuries.sh <sport> <team>
# Example: ./scripts/injuries.sh nfl patriots
```

### Standings
```bash
./scripts/standings.sh <sport>
# Example: ./scripts/standings.sh nba
```

## Data Sources

- ESPN API (unofficial, no key required)
- Free tier, rate-limited

## Use Cases

1. **Pre-game research** - Check injuries, recent form before betting
2. **Live monitoring** - Track scores during games
3. **Historical analysis** - Review past matchups
4. **Schedule awareness** - Know when games are happening

## For Prediction Markets

This skill helps with:
- Polymarket sports bets
- Limitless prediction markets
- Any sports-related forecasting

---

Built by [@pipaitrader](https://github.com/pipaitrader) 🎯
