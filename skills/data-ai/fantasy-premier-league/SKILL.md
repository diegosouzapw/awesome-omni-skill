---
name: Fantasy Premier League
description: This skill should be used when the user asks about "FPL", "Fantasy Premier League", "my FPL team", "captain pick", "who should I captain", "transfer suggestions", "best transfers", "FPL player stats", "fixture difficulty", "gameweek", "FPL points", "wildcard", "free hit", "bench boost", "triple captain", or any fantasy football related queries for the English Premier League.
version: 0.1.0
---

# Fantasy Premier League Assistant

Comprehensive Fantasy Premier League assistant providing team analysis, transfer suggestions, captain picks, fixture planning, and player statistics using the official FPL API.

## FPL API Overview

The official FPL API is publicly accessible at `https://fantasy.premierleague.com/api/`. No authentication required for public endpoints.

### Key Endpoints

| Endpoint | Description |
|----------|-------------|
| `/bootstrap-static/` | All players, teams, gameweeks, game settings |
| `/entry/{team_id}/` | Manager info and overall stats |
| `/entry/{team_id}/event/{gw}/picks/` | Team picks for a gameweek |
| `/entry/{team_id}/transfers/` | Transfer history |
| `/entry/{team_id}/history/` | Season history and past seasons |
| `/element-summary/{player_id}/` | Individual player detailed stats |
| `/fixtures/` | All fixtures (add `?event={gw}` for specific gameweek) |
| `/event/{gw}/live/` | Live gameweek data with bonus points |

### Finding Team ID

The user's team ID is found in their FPL URL: `https://fantasy.premierleague.com/entry/{TEAM_ID}/event/{GW}`

## Core Workflows

### Fetching Bootstrap Data

Always start by fetching bootstrap data for current game state:

```bash
curl -s "https://fantasy.premierleague.com/api/bootstrap-static/" | jq .
```

Key data in bootstrap response:
- `elements` - All players with stats, prices, ownership
- `teams` - All 20 Premier League teams
- `events` - All 38 gameweeks with deadlines
- `element_types` - Position types (GKP, DEF, MID, FWD)

### Analyzing a User's Team

1. Get team ID from user
2. Fetch current gameweek from bootstrap data
3. Fetch team picks: `/entry/{team_id}/event/{gw}/picks/`
4. Cross-reference with bootstrap `elements` for player details
5. Analyze formation, captain choice, bench order

### Transfer Suggestions

1. Fetch user's current team
2. Analyze players by form, fixtures, expected points
3. Identify underperforming assets
4. Suggest replacements considering:
   - Budget (selling price vs purchase price)
   - Fixture difficulty rating (FDR)
   - Form (recent points per game)
   - Expected goals/assists (if discussing)

### Captain Recommendations

Evaluate captain options based on:
- Fixture difficulty (home vs away, opponent strength)
- Recent form (points in last 5 gameweeks)
- Historical performance against opponent
- Set piece involvement
- Penalty taker status

### Fixture Difficulty Analysis

Use the `team_h_difficulty` and `team_a_difficulty` from fixtures endpoint. Scale is 1-5:
- 1-2: Easy fixtures (green)
- 3: Medium fixtures (yellow)
- 4-5: Hard fixtures (red)

## Chip Strategy

### Wildcard
- Best used during international breaks or injury crises
- Two per season (one before GW20, one after)
- Unlimited free transfers that gameweek

### Free Hit
- One per season
- Temporary unlimited transfers for single gameweek
- Best for blank or double gameweeks

### Bench Boost
- All bench players score points
- Best during double gameweeks with 15 playing assets

### Triple Captain
- Captain scores 3x instead of 2x
- Best for premium asset in double gameweek

## Data Interpretation

### Player Status Codes
- `a` - Available
- `d` - Doubtful (yellow flag)
- `i` - Injured (red flag)
- `s` - Suspended
- `u` - Unavailable

### Key Player Metrics
- `form` - Average points over last 30 days
- `points_per_game` - Season average
- `selected_by_percent` - Ownership percentage
- `now_cost` - Current price (divide by 10 for actual price)
- `cost_change_event` - Price change this gameweek
- `ict_index` - Influence + Creativity + Threat combined

## Usage Patterns

### Quick Team Check
When user asks "how's my team doing" or "analyze my team":
1. Request team ID if not known
2. Fetch current picks
3. Provide formation, total value, captain choice
4. Flag injury concerns

### Transfer Planning
When user asks "who should I transfer" or "best transfers":
1. Analyze current squad
2. Identify problem positions
3. Suggest 2-3 options per position with reasoning

### Gameweek Preview
When user asks about upcoming gameweek:
1. Show deadline time
2. Highlight fixture standouts
3. Suggest captain options
4. Note any blank/double gameweek implications

## Additional Resources

### Reference Files
- **`references/api-guide.md`** - Detailed API endpoint documentation
- **`references/metrics.md`** - Player metrics and what they mean

### Scripts
- **`scripts/fetch-team.sh`** - Fetch and format team data
- **`scripts/fetch-fixtures.sh`** - Get upcoming fixtures with FDR
