---
name: whoop
description: Fetch WHOOP health data including recovery scores, sleep metrics, workouts, cycles, and profile. Use when user asks about WHOOP data, recovery, HRV, sleep analysis, or fitness metrics.
metadata: {"openclaw":{"emoji":"💪","requires":{"env":["WHOOP_ACCESS_TOKEN","WHOOP_REFRESH_TOKEN"],"bins":["curl","jq"]},"primaryEnv":"WHOOP_ACCESS_TOKEN"}}
homepage: https://developer.whoop.com
---

# WHOOP Health Data

Access WHOOP health metrics: recovery scores, HRV, sleep analysis, workout strain, and physiological cycles.

## Setup

Add tokens to `~/.openclaw/openclaw.json`:

```json
{
  "skills": {
    "entries": {
      "whoop": {
        "enabled": true,
        "env": {
          "WHOOP_ACCESS_TOKEN": "your-access-token",
          "WHOOP_REFRESH_TOKEN": "your-refresh-token",
          "WHOOP_CLIENT_ID": "your-client-id",
          "WHOOP_CLIENT_SECRET": "your-client-secret"
        }
      }
    }
  }
}
```

| Variable | Required For | Description |
|----------|--------------|-------------|
| `WHOOP_ACCESS_TOKEN` | All API calls | Expires hourly |
| `WHOOP_REFRESH_TOKEN` | Auto-refresh | Long-lived, get from dashboard |
| `WHOOP_CLIENT_ID` | Auto-refresh | From WHOOP developer app |
| `WHOOP_CLIENT_SECRET` | Auto-refresh | From WHOOP developer app |

Get tokens from the WHOOP dashboard. See `{baseDir}/references/setup-guide.md`.

## Commands

### User Profile
```bash
{baseDir}/scripts/whoop-api.sh profile
```
Returns: User ID, email, first/last name.

### Body Measurements
```bash
{baseDir}/scripts/whoop-api.sh body
```
Returns: Height (m), weight (kg), max heart rate.

### Recovery Data
```bash
{baseDir}/scripts/whoop-api.sh recovery [--start YYYY-MM-DD] [--end YYYY-MM-DD] [--limit N]
```
Returns: Recovery score (0-100%), HRV (ms), resting heart rate, SpO2.

**Example - Last 7 days:**
```bash
{baseDir}/scripts/whoop-api.sh recovery --start $(date -v-7d +%Y-%m-%d)
```

### Sleep Data
```bash
{baseDir}/scripts/whoop-api.sh sleep [--start YYYY-MM-DD] [--end YYYY-MM-DD] [--limit N]
```
Returns: Sleep stages, duration, efficiency, performance percentage.

### Workout Data
```bash
{baseDir}/scripts/whoop-api.sh workouts [--start YYYY-MM-DD] [--end YYYY-MM-DD] [--limit N]
```
Returns: Workout strain, sport type, heart rate zones, calories.

### Physiological Cycles
```bash
{baseDir}/scripts/whoop-api.sh cycles [--start YYYY-MM-DD] [--end YYYY-MM-DD] [--limit N]
```
Returns: Daily strain, average heart rate, calories burned.

## Interpreting Scores

**Recovery Score (0-100%):**
- Green (67-100%): Well recovered, ready for strain
- Yellow (34-66%): Moderate recovery, balance activity
- Red (0-33%): Low recovery, prioritize rest

**HRV (Heart Rate Variability):**
- Higher values generally indicate better recovery
- Personal baseline matters more than absolute numbers
- Trends over time are most meaningful

**Sleep Performance:**
- Percentage of sleep needed that was achieved
- 100% = optimal sleep duration for your body

**Strain Score (0-21):**
- 0-9: Light day
- 10-13: Moderate strain
- 14-17: High strain
- 18-21: All-out effort

## Auto-Refresh (Cron Job)

Access tokens expire hourly. Set up a cron job to auto-refresh.

### Check if cron already exists:
```bash
openclaw cron list | grep -i whoop
```
If output shows "WHOOP token refresh", cron is already set up - skip to Troubleshooting.

### Setup (one-time only):

1. Ensure all env vars from Setup section are configured.

2. **Ask user for confirmation** before creating the cron job.

3. Create the cron job (runs every 55 minutes to avoid expiry edge cases):
```bash
openclaw cron add --name "WHOOP token refresh" \
  --cron "*/55 * * * *" \
  --session isolated \
  --message "Run {baseDir}/scripts/whoop-refresh.sh and update BOTH WHOOP_ACCESS_TOKEN and WHOOP_REFRESH_TOKEN in ~/.openclaw/openclaw.json with the new tokens from the output. Both tokens rotate on each refresh." \
  --model anthropic/claude-haiku-4-5
```

4. Verify cron was created:
```bash
openclaw cron list | grep -i whoop
```

5. Confirm to user: "WHOOP auto-refresh cron job created. Your token will refresh every 55 minutes automatically."

## Troubleshooting

**"WHOOP_ACCESS_TOKEN not set":**
Set your token in `~/.openclaw/openclaw.json` - see Setup above.

**401 Unauthorized:**
Token expired. Run the refresh script or get a new token from the dashboard.

**No data returned:**
- Check date range parameters
- Ensure WHOOP device synced recently

## References

- `{baseDir}/references/api-endpoints.md` - Full API documentation
- `{baseDir}/references/data-structures.md` - Response schemas
- `{baseDir}/references/setup-guide.md` - How to get an access token
