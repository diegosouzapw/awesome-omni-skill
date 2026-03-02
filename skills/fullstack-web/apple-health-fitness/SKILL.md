---
name: apple-health-fitness
description: Query Health and Fitness data from Apple Health app including activity, workouts, heart rate, sleep, and health metrics. Use when user asks about health stats, fitness activity, workouts, sleep data, or health metrics.
allowed-tools:
  - Bash
  - Read
  - Write
---

# Apple Health & Fitness

Access Health and Fitness data from Apple Health app.

## Status

**UNDER DEVELOPMENT** - Helper scripts not yet implemented.

Apple Health lacks AppleScript support. Current workarounds:
1. Manual export from Health app (Profile > Export All Health Data)
2. `healthexport` CLI tool (`pip3 install healthexport`)
3. Parse exported XML/CSV with standard tools

## Planned Commands

```bash
health-export --days 7 --type steps ~/health-export/
health-query ~/health-export/ --metric steps --days 7 --summary
fitness-summary --week
```

## Available Data Types

Activity, Workouts, Heart Rate, Sleep, Body Measurements, Vitals

## Privacy & Security

Health data is **HIGHLY SENSITIVE**:
- Read-only access
- Always ask before accessing
- Aggregated summaries only
- Never share externally

## Development Roadmap

See [reference/roadmap.md](reference/roadmap.md) for implementation plans and technical details.

## Related Skills

- `apple-productivity` - Calendar, Contacts, Mail, Messages
- `apple-shortcuts` - Could trigger Health exports
