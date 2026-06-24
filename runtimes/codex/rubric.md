# Weather-app rubric (Codex runtime)

Identical functional checklist scored for every arm (1 point each, 12 total).
This is the same rubric as weather-bench — held fixed so the Codex numbers are
comparable to the Claude Code numbers. Score from loading the built `index.html`
and reading the source.

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
A cost or size delta only counts as "leaner at the same quality" if both arms
score the same on this rubric. If an arm scores materially lower, the delta is a
quality difference, not a concision win, and must be reported as such.

## Cross-session reuse (benchmark B, scored separately)
The session-2 extension is scored on whether it REUSED the session-1 build
(reuse rate) and on how much NEW code it wrote (avg new LOC). A with-memory arm
should reuse the existing component; a no-memory arm rebuilds it.
