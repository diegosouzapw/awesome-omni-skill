---
name: fuel-prices
description: Monitor fuel prices in Germany using Tankerkönig API. Use when user asks about fuel/gas prices, wants to find cheap stations nearby, or needs refueling recommendations.
---

# Fuel Price Skill ⛽

Monitor fuel prices in Germany using the free Tankerkönig API.

## 1. Quick Price Check

Get current prices near a location:

```bash
cd fuel-skill
TANKERKOENIG_API_KEY=xxx npx ts-node fuel-monitor.ts check
```

Or via curl:

```bash
curl -s "https://creativecommons.tankerkoenig.de/json/list.php?lat=LAT&lng=LNG&rad=5&sort=price&type=diesel&apikey=API_KEY"
```

### Fuel Types

- `diesel` — Diesel
- `e5` — Super E5
- `e10` — Super E10

## 2. Script Usage

```bash
# Check current prices (saves to history)
npx ts-node fuel-monitor.ts check

# View price history
npx ts-node fuel-monitor.ts history

# Analyze patterns (best/worst days)
npx ts-node fuel-monitor.ts analyze
```

### Configuration

Edit `CONFIG` in `fuel-monitor.ts`:

```typescript
const CONFIG = {
  lat: 48.137,        // Your latitude
  lng: 11.575,        // Your longitude
  radius: 5,          // Search radius in km
  fuelType: 'diesel', // diesel, e5, or e10
  apiKey: process.env.TANKERKOENIG_API_KEY,
};
```

## 3. Output Format

```
⛽✅ **Diesel now: €1.639**
📍 JET — Hauptstraße 42, München (2.4 km)
📊 Area average: €1.672

Price 3.3¢ below weekly average (€1.672)

👉 Recommend filling up today!
```

### Recommendations

| Symbol | Meaning | Condition |
|--------|---------|-----------|
| ⛽✅ | Fill up | Price ≥3¢ below weekly avg |
| ⏳ | Wait | Price ≥3¢ above weekly avg |
| ➖ | Neutral | Price near average |

## 4. API Reference

Tankerkönig API (free, CC BY 4.0):

```
Base: https://creativecommons.tankerkoenig.de/json

GET /list.php?lat=X&lng=Y&rad=R&type=TYPE&sort=price&apikey=KEY
```

Get API key: https://creativecommons.tankerkoenig.de/

## 5. Heartbeat Integration

Add to HEARTBEAT.md:

```markdown
### ⛽ Fuel Prices (morning)
- Check prices in the morning
- Show cheapest + nearby option
- Recommend if price is below average
- Calculate cost for typical fill (e.g., 40L)
```

Example morning output:

```
⛽ Diesel
- **V-Markt** — €1.638 ✓ (2.4 km, open)
- Jet — €1.649 (1.2 km)
- Average ~€1.68

40L = €65.52 — Good price, fill up today!
```

## 6. Requirements

```bash
npm install
# or
npm install typescript ts-node @types/node
```

Environment:
```bash
export TANKERKOENIG_API_KEY=your-api-key
```

## Files

- `fuel-monitor.ts` — Main script
- `price-history.json` — Local price history (auto-created)
- `package.json` — Dependencies

## Tips

- Prices often lowest on **Monday/Tuesday**
- Prices often highest on **Friday/weekend**
- Check before holidays (prices spike)
- Evening prices sometimes lower than morning
