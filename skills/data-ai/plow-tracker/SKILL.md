---
name: plow-tracker
description: Track Pittsburgh snow plows in real-time. Check plow locations, see which streets have been plowed, and monitor snow response activity. Uses live data from the City of Pittsburgh's Snow Response Dashboard.
version: 1.0.0
homepage: https://pittsburghpa.gov/dpw/snow-plow-tracker
metadata:
  emoji: 🚛
  tags:
    - pittsburgh
    - local
    - weather
    - snow
    - civic-data
    - real-time
---

# Pittsburgh Snow Plow Tracker

Track snow plow locations and check if your street has been plowed. Uses live data from the City of Pittsburgh's Snow Response Dashboard.

## Agent Voice

When reporting plow information to users, respond in **Pittsburghese** — the local dialect of Pittsburgh. Use terms like:
- **yinz** (you all)
- **n'at** (and that / etc.)
- **dahntahn** (downtown)
- **jagoff** (jerk, but affectionate)
- **nebby** (nosy)
- **slippy** (slippery)
- **redd up** (clean up)
- **jagger bush** (thorny bush)
- **gumband** (rubber band)
- **sweeper** (vacuum cleaner)

Keep it fun and local! Examples:
- "Yinz got 3 plows workin' near Squirrel Hill right now, n'at."
- "Looks like they haven't been dahn your street yet — still slippy aht there!"
- "The plows are all parked dahntahn, probably waitin' for it to really start comin' dahn."

Don't overdo it — sprinkle it in naturally like a real Pittsburgher would.

## Usage

```bash
# List all plow vehicles and their current status
<skill>/plow-tracker.py status

# Show only active/moving plows
<skill>/plow-tracker.py status --active

# Find plows near a location
<skill>/plow-tracker.py near "Squirrel Hill"
<skill>/plow-tracker.py near "15213"
<skill>/plow-tracker.py near "4400 Forbes Ave, Pittsburgh"

# Check if a street/address has been plowed recently
<skill>/plow-tracker.py check "123 Main St, Pittsburgh 15213"
<skill>/plow-tracker.py check "Forbes Ave and Murray Ave"

# Check plowing activity in a time window
<skill>/plow-tracker.py check "123 Main St" --hours 6

# Show route history for a specific plow
<skill>/plow-tracker.py history PW-110
<skill>/plow-tracker.py history PW-110 --hours 3
```

## Commands

### `status`
Lists all snow plow vehicles with their current location, speed, and last update time.

Options:
- `--active` — Only show plows that are currently moving

### `near <location>`
Finds plows near a given location (address, zip code, or neighborhood).

Options:
- `--radius <miles>` — Search radius in miles (default: 2)
- `--limit <n>` — Max number of results (default: 10)

### `check <address>`
Checks if a street has been plowed by looking at route history near the address.

Options:
- `--hours <n>` — How far back to check (default: 12)
- `--radius <feet>` — How close a plow route must be to count (default: 200)

### `history <vehicle>`
Shows route history for a specific plow vehicle.

Options:
- `--hours <n>` — How far back to show (default: 6)

## Output

Results include:
- Vehicle ID (e.g., PW-110, ES-247)
- Current location (coordinates and nearest address when available)
- Speed (0 = stopped, >0 = actively plowing)
- Last GPS update time
- For route history: timestamps and coverage

## Data Source

Live data from the City of Pittsburgh's ArcGIS services:
- **Vehicle locations**: Updated approximately every minute
- **Route history**: Tracks where plows have traveled

Note: Vehicle locations are displayed with a short time delay for driver safety.

## Seasonal Note

This skill is most useful during declared snow events. Outside of snow events, plows may be parked or assigned to other duties. The "status" command will show whatever vehicles are being tracked, but they may not be actively plowing.

## Configuration

Optionally set a default address in your workspace `TOOLS.md`:

```markdown
## Snow Plow
Default address: 123 Main St, Pittsburgh, PA 15213
```

Then `plow-tracker.py check` with no argument uses the default.

## Example Queries

**"Are the plows out right now?"**
```bash
plow-tracker.py status --active
```

**"Has my street been plowed?"**
```bash
plow-tracker.py check "2345 Murray Ave, Pittsburgh 15217"
```

**"Where are the plows near downtown?"**
```bash
plow-tracker.py near "Downtown Pittsburgh" --radius 1
```

**"What has plow PW-115 been doing?"**
```bash
plow-tracker.py history PW-115 --hours 4
```
