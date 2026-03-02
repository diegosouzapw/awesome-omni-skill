---
name: tempest-weather
description: Fetches live weather data from a WeatherFlow Tempest weather station and returns structured JSON with current conditions, wind, precipitation, and lightning. Use when the user asks about current weather, outdoor conditions, their Tempest station, wind speed, rain, lightning nearby, or any live sensor readings — even if they don't mention Tempest or API explicitly.
version: 1.0.0
metadata:
  openclaw:
    emoji: "🌤️"
    homepage: https://weatherflow.github.io/Tempest/api/
    primaryEnv: TEMPEST_TOKEN
    requires:
      env:
        - TEMPEST_TOKEN
        - TEMPEST_STATION_ID
      bins:
        - python3
      anyBins:
        - curl
        - python3
---

# Tempest Weather Skill

Fetches on-demand weather data from a WeatherFlow Tempest station via the REST API and returns clean, structured JSON.

## Configuration

Credentials are read from environment variables — never hardcoded:

| Env Var | Description |
|---|---|
| `TEMPEST_TOKEN` | Personal Access Token from tempestwx.com → Settings → Data Authorizations |
| `TEMPEST_STATION_ID` | Numeric station ID (find it by calling the `/stations` endpoint with your token) |

If either env var is missing, inform the user and show them how to set them:
```bash
export TEMPEST_TOKEN="your_token_here"
export TEMPEST_STATION_ID="your_station_id_here"
```

---

## Primary Endpoint

```
GET https://swd.weatherflow.com/swd/rest/observations/station/{STATION_ID}?token={TEMPEST_TOKEN}
```

This returns the latest observation for the station, including all sensor data.

Fallback (by device): `GET https://swd.weatherflow.com/swd/rest/observations/?device_id={DEVICE_ID}&token={TEMPEST_TOKEN}`

To list available stations/devices: `GET https://swd.weatherflow.com/swd/rest/stations?token={TEMPEST_TOKEN}`

---

## Workflow

1. **Check for credentials** — If the user hasn't provided a token and station ID, ask for them.
2. **Fetch the observation** — Use bash (curl) or Python to call the REST endpoint.
3. **Parse the response** — Extract the relevant fields from `obs` (see field reference below).
4. **Return structured JSON** — Output a clean, normalized JSON object (see Output Schema).
5. **Handle errors gracefully** — 401 = bad token, 404 = station not found, empty obs = no data yet.

### Fetching with curl

```bash
curl -s "https://swd.weatherflow.com/swd/rest/observations/station/${STATION_ID}?token=${TEMPEST_TOKEN}"
```

### Fetching with Python

```python
import requests, json

url = f"https://swd.weatherflow.com/swd/rest/observations/station/{STATION_ID}"
resp = requests.get(url, params={"token": TEMPEST_TOKEN})
resp.raise_for_status()
data = resp.json()
```

---

## Output Schema

Always return data in this normalized JSON structure:

```json
{
  "station_id": 12345,
  "station_name": "My Backyard",
  "timestamp": "2024-01-15T14:32:00Z",
  "timestamp_epoch": 1705329120,
  "conditions": {
    "temperature_c": 18.5,
    "temperature_f": 65.3,
    "humidity_pct": 62,
    "pressure_mb": 1013.4,
    "pressure_trend": "steady"
  },
  "wind": {
    "speed_avg_ms": 3.2,
    "speed_avg_mph": 7.2,
    "speed_lull_ms": 1.1,
    "speed_gust_ms": 5.8,
    "speed_gust_mph": 13.0,
    "direction_deg": 247,
    "direction_cardinal": "WSW"
  },
  "precipitation": {
    "rain_accumulated_mm": 0.0,
    "rain_daily_mm": 2.4,
    "precip_type": "none",
    "precip_analysis": "rain_check_on"
  },
  "lightning": {
    "strike_count": 0,
    "avg_distance_km": null
  },
  "solar": {
    "uv_index": 3,
    "solar_radiation_wm2": 412,
    "illuminance_lux": 28500
  },
  "battery_volts": 2.42,
  "data_source": "tempest_rest_api"
}
```

---

## Field Reference

See `references/obs_fields.md` for the complete field mapping from Tempest API array indices to human-readable names, units, and conversion formulas.

---

## Conversion Helpers

- **°C → °F**: `(C * 9/5) + 32`
- **m/s → mph**: `ms * 2.237`
- **Degrees → Cardinal**: Read the lookup table in `references/obs_fields.md`
- **Precip type codes**: `0 = none`, `1 = rain`, `2 = hail`
- **Precip analysis codes**: `0 = none`, `1 = rain_check_on`, `2 = rain_check_off`

---

## Error Handling

| HTTP Status | Meaning | Action |
|---|---|---|
| 200 | Success | Parse and return data |
| 401 | Invalid token | Ask user to re-check their token |
| 403 | Forbidden | Token doesn't have access to that station |
| 404 | Station not found | Ask user to confirm their station ID |
| 200 + empty obs | No data | Station may be offline; inform user |

If `obs` is an empty array or `null`, report: "Station found but no recent observations available — the device may be offline."

---

## Example User Interactions

**"What's the weather at my Tempest station?"**
→ Fetch latest obs, return full JSON output.

**"Is it raining?"**
→ Fetch obs, check `precipitation.precip_type` and `precipitation.rain_accumulated_mm`, return focused JSON.

**"Any lightning nearby?"**
→ Fetch obs, check `lightning.strike_count` and `lightning.avg_distance_km`, return lightning sub-object.

**"How windy is it?"**
→ Return `wind` sub-object including gust, lull, avg, direction.
