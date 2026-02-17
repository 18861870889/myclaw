---
name: weather
description: Get current weather and forecasts (no API key required). Use for weather reports, checking conditions before going out, or getting weather for any city.
metadata: { "openclaw": { "emoji": "🌤️", "requires": { "bins": ["curl"] } } }
---

# Weather

Use **Open-Meteo** - free API, no key, works in China.

## Quick Query

```bash
# 上海 (Shanghai): latitude=31.23, longitude=121.47
# 绍兴 (Shaoxing): latitude=30.00, longitude=120.58
curl -s "https://api.open-meteo.com/v1/forecast?latitude=30.00&longitude=120.58&current_weather=true&timezone=Asia/Shanghai"
```

## Parameters

- `latitude`, `longitude`: Get from city name (search or use known coords)
- `current_weather=true`: Basic info (temp, wind, weathercode)
- `hourly=relativehumidity_2m,precipitation_probability`: Optional hourly data
- `timezone=Asia/Shanghai`: Local timezone

## Weather Codes (WMO)

| Code | Meaning |
|------|---------|
| 0    | Clear   |
| 1-3  | Partly cloudy |
| 45-48 | Fog |
| 51-67 | Rain/Drizzle |
| 71-77 | Snow |
| 80-82 | Rain showers |
| 95-99 | Thunderstorm |

## Finding Coordinates

Use Google Places API or search: "Shaoxing latitude longitude"

## Example Output

```json
{"current_weather":{"temperature":5.6,"windspeed":14.7,"weathercode":61}}
```

61 = 小雨 (light rain)

---

Note: wttr.in is blocked in China - always use Open-Meteo.
