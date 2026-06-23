# Weather-app rubric

Identical functional checklist scored for every arm (1 point each, 12 total).
Score from loading the built `index.html` and reading the source.

| # | Criterion | Pass condition |
|---|-----------|----------------|
| 1 | Geolocation on load | Calls `navigator.geolocation`; shows weather for current position |
| 2 | Current temperature | Renders current temp |
| 3 | Current wind | Renders wind speed |
| 4 | Current humidity | Renders relative humidity |
| 5 | Current description | Renders a text weather description (mapped from WMO code) |
| 6 | Hourly forecast | Shows hourly entries for the rest of today |
| 7 | 7-day daily forecast | Shows 7 daily entries |
| 8 | Condition icon/visual | Each forecast/current block has an icon or visual cue |
| 9 | City search | Text input → Open-Meteo geocoding → loads that city |
| 10 | C/F toggle | A control that switches all temps between °C and °F |
| 11 | Open-Meteo, no key | Uses open-meteo.com endpoints, no API key in source |
| 12 | Clean responsive design | Layout is scannable and adapts to mobile/desktop widths |

## Quality parity guardrail
The YAGNI comparison (C vs D) is only meaningful at quality parity: if one arm
scores materially lower on this rubric, the token/LOC delta is not a clean
"leaner at parity" result and must be reported as a quality difference instead.

## YAGNI / output-size signal (the video's real metric)
Tracked alongside the rubric, NOT scored:
- `index.html` LOC (non-blank)
- byte size of `index.html`
- file count produced
