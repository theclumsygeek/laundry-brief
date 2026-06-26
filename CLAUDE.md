# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file HTML app (`laundry-brief.html`) that gives a laundry-drying recommendation for Bedok, Singapore. No build step, no dependencies, no bundler — open the file directly in a browser or serve it with any static server.

To run locally:
```
npx serve .
# or
python -m http.server 8080
```

## Architecture

Everything lives in `laundry-brief.html`: styles, logic, and markup are all inline. The file is structured in three layers:

**CSS** (inline `<style>`) — uses CSS custom properties (`--paper`, `--ink`, `--rope`, etc.) for theming. Dark mode is handled by `[data-theme="dark"]` on `#laundry-root`, with a `@media (prefers-color-scheme: dark)` fallback.

**JS** (inline `<script>`, IIFE) — pure vanilla JS, no frameworks. Key pieces:
- `loadWeather()` — fetches from NEA/MSS APIs (v2 first, v1 fallback). Humidity and wind are non-fatal; only the 2-hour nowcast is required.
- `buildPlan(nowHour, hourlyRiskByHour, dryHours)` — core scheduling logic. Finds the next rain-risk hour, checks if drying completes before it, returns a `{verdict, startBy, hangBy, dryBy, riskAt}` object.
- `render(state)` — single render function keyed on `state.status` ('loading' | 'error' | 'ready'). Rebuilds `innerHTML` on every state change (no diffing).
- `boot()` — calls `loadWeather()`, assembles state, calls `render()`. Re-runs on Refresh and fabric chip clicks.

**Drying model** — `DRY_HOURS_BASELINE` (3.5h for light cotton) × fabric multiplier × `weatherMultiplier({rh, windKmh, conditionText})`. The weather multiplier is the cube root of three factors (humidity, wind, sky condition) so no single extreme dominates.

**Hourly risk profile** — built from the 24-hour forecast's per-period east-region text where available (`periodRiskByHour`), falling back to a heuristic based on `regionOutlook` if the periods array is empty. The live 2-hour nowcast is overlaid onto the nearest display slot regardless.

**State persistence** — `localStorage` for theme (`laundry-brief-theme`) and selected fabric IDs (`laundry-brief-fabrics`).

## Key constants to know

```js
DRY_HOURS_BASELINE = 3.5   // hours, light cotton, typical Bedok conditions
WASH_HOURS = 2             // wash cycle + hang time
LATEST_FINISH_HOUR = 19    // don't plan drying past 7pm
EARLIEST_START_HOUR = 6
```

Fabric multipliers live in the `FABRICS` array (`mult: 0.6` for synthetics → `2.3` for towels).

## External APIs

All from data.gov.sg / MSS. All read-only, no auth required.
- 2-hour nowcast (v2 then v1 fallback) — drives current condition and immediate-hour risk
- 24-hour forecast (v2 then v1 fallback) — drives period-based hourly risk and temp/RH range
- Relative humidity & wind speed (v1 only) — nearest station to Bedok (`1.3236, 103.9273`), non-fatal if unavailable
