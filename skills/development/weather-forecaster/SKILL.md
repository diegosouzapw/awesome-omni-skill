---
name: weather-forecaster
description: "Use this skill whenever the user asks about weather forecasts, weather conditions, or weekly weather outlooks for any location. Triggers include: any mention of '天气', 'weather', '预报', 'forecast', '下雨', '温度', '穿什么', or planning outdoor activities. Also use when the user asks 'should I bring an umbrella', 'what to wear', or any travel weather planning. This skill uses the free Open-Meteo API to provide 7-day weather forecasts with temperature, precipitation, wind, and UV index data, and generates a beautiful visual weather report."
---

# Weather Forecaster Skill 🌤️

## Overview

This skill provides 7-day weather forecasts for any location worldwide using **free APIs** (no API key required):

- **Open-Meteo Geocoding API** — Convert location names to coordinates
- **Open-Meteo Forecast API** — Get detailed 7-day weather forecasts

## Workflow

### Step 1: Identify the Location

Parse the user's request to extract the location name. Examples:
- "北京未来一周天气" → location = "北京" (Beijing)
- "What's the weather in Tokyo?" → location = "Tokyo"
- "墨尔本下周会下雨吗" → location = "墨尔本" (Melbourne)

### Step 2: Geocode the Location

Use the Open-Meteo Geocoding API to convert the location name to latitude/longitude:

```
https://geocoding-api.open-meteo.com/v1/search?name={location}&count=1&language=en
```

For Chinese location names, also try with `language=zh`:
```
https://geocoding-api.open-meteo.com/v1/search?name={location}&count=1&language=zh
```

### Step 3: Fetch 7-Day Forecast

Use the Open-Meteo Forecast API:

```
https://api.open-meteo.com/v1/forecast?latitude={lat}&longitude={lon}&daily=weather_code,temperature_2m_max,temperature_2m_min,apparent_temperature_max,apparent_temperature_min,precipitation_sum,precipitation_probability_max,wind_speed_10m_max,wind_direction_10m_dominant,uv_index_max,sunrise,sunset&timezone=auto&forecast_days=7
```

### Step 4: Run the Python Script

Execute the script at `scripts/weather_report.py` to fetch and format the data:

```bash
python3 scripts/weather_report.py "location_name"
```

The script will:
1. Geocode the location
2. Fetch 7-day forecast data
3. Output a formatted JSON report

### Step 5: Generate Visual Report

After getting the data, create a **React (.jsx) artifact** that displays a beautiful weather dashboard with:

- Location name and coordinates
- 7-day forecast cards showing:
  - Date and day of week
  - Weather icon/emoji based on weather code
  - High/Low temperatures
  - Precipitation probability and amount
  - Wind speed and direction
  - UV index
- Color coding for temperature (blue=cold, green=comfortable, orange=warm, red=hot)
- Precipitation warnings highlighted
- A summary paragraph with clothing/umbrella recommendations

### Weather Code Reference

| Code | Description | Emoji |
|------|------------|-------|
| 0 | Clear sky | ☀️ |
| 1 | Mainly clear | 🌤️ |
| 2 | Partly cloudy | ⛅ |
| 3 | Overcast | ☁️ |
| 45, 48 | Fog | 🌫️ |
| 51, 53, 55 | Drizzle | 🌦️ |
| 61, 63, 65 | Rain | 🌧️ |
| 66, 67 | Freezing rain | 🌧️❄️ |
| 71, 73, 75 | Snowfall | 🌨️ |
| 77 | Snow grains | 🌨️ |
| 80, 81, 82 | Rain showers | 🌧️ |
| 85, 86 | Snow showers | 🌨️ |
| 95 | Thunderstorm | ⛈️ |
| 96, 99 | Thunderstorm with hail | ⛈️🧊 |

### Response Style

- Use Chinese if the user writes in Chinese, English if the user writes in English
- Provide practical advice: 穿衣建议、是否带伞、出行建议
- Highlight extreme weather warnings
- Be friendly and conversational, like a TV weather presenter

## Error Handling

- If location not found: Ask user to provide a more specific location name or try English/Chinese alternative
- If API fails: Inform user that the weather service is temporarily unavailable
- If network is disabled: Let user know this skill requires network access to fetch weather data
