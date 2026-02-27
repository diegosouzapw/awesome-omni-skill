---
name: weather
description: "Get weather information and forecasts using wttr.in"
metadata:
  emoji: "🌤️"
  requires:
    optional: true
  tags: ["weather", "forecast", "climate"]
  attribution: "Uses wttr.in API"
---

# Weather Skill

Get current weather and forecasts for any location using the wttr.in API. No API key required!

## Available Tools

You have access to the following weather tools:

### Get Current Weather
```javascript
get_weather({
  location: "New York",  // city name, airport code, or coordinates
  units: "metric"        // optional: "metric" (Celsius) or "imperial" (Fahrenheit)
})
```

### Get Weather Forecast
```javascript
get_forecast({
  location: "London",
  units: "metric"  // optional
})
```

## Examples

**Current weather in a city:**
```javascript
get_weather({
  location: "Tokyo",
  units: "metric"
})
```

**Weather forecast:**
```javascript
get_forecast({
  location: "San Francisco",
  units: "imperial"
})
```

**Using coordinates:**
```javascript
get_weather({
  location: "40.7,-74.0"  // latitude,longitude
})
```

**Using airport code:**
```javascript
get_weather({
  location: "JFK"
})
```

## Response Format

### Current Weather
Returns:
- Location name and country
- Current temperature (both Celsius and Fahrenheit)
- Feels like temperature
- Weather condition description
- Humidity percentage
- Wind speed and direction
- Precipitation
- Visibility
- UV index
- Observation time

### Forecast
Returns 3-day forecast with:
- Date
- Max/min/average temperatures
- Weather condition
- Total snowfall
- Sun hours
- UV index

## Tips

- Location can be a city name, airport code (e.g., "JFK"), or coordinates ("lat,lon")
- The API works worldwide
- No authentication required
- Data is updated regularly
