# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file HTML app (`index.html`) that gives a laundry-drying recommendation for the user's location in Singapore. It resolves location from the browser's GPS (falling back to Bedok when denied), and supports saving a "home" location to check conditions there while away. No build step, no dependencies, no bundler — open the file directly in a browser or serve it with any static server.

User-facing usage is documented in `README.md`.

To run locally:
```
npx serve .
# or
python -m http.server 8080
```

## Architecture

Everything lives in `index.html`: styles, logic, and markup are all inline. The file is structured in three layers:

**CSS** (inline `<style>`) — uses CSS custom properties (`--paper`, `--ink`, `--rope`, etc.) for theming. Dark mode is handled by `[data-theme="dark"]` on `#laundry-root`, with a `@media (prefers-color-scheme: dark)` fallback.

**JS** (inline `<script>`, IIFE) — pure vanilla JS, no frameworks. Key pieces:
- `loadWeather(lat, lng)` — fetches from NEA/MSS APIs (v2 first, v1 fallback). The coords pick the nearest humidity/wind station. Humidity and wind are non-fatal; only the 2-hour nowcast is required.
- `findNearestArea(lat, lng, areaMetadata)` — resolves the named forecast area closest to a coordinate, using the `area_metadata` list embedded in the 2-hour nowcast response. No external geocoding API needed.
- `buildPlan(nowHour, hourlyRiskByHour, dryHours)` — core scheduling logic. Finds the next rain-risk hour, checks if drying completes before it, returns a `{verdict, startBy, hangBy, dryBy, riskAt}` object.
- `render(state)` — single render function keyed on `state.status` ('loading' | 'error' | 'ready'). Rebuilds `innerHTML` on every state change (no diffing).
- `boot(forceGPS)` — resolves location coords (Phase A: GPS/home/fallback), calls `loadWeather()`, resolves the area name (Phase B), computes the location toggle/save-home UI state (Phase C), assembles state, calls `render()`. Re-runs on Refresh, location toggle, and fabric chip clicks. `forceGPS === true` (only the Refresh button) drops the session GPS cache and requests a fresh fix.

**Location resolution** — `gpsCoords` is fetched once per session and cached (so chip taps and toggles don't re-prompt the OS). `boot()` attempts GPS regardless of mode so the "📍 Here" toggle option always has a label. The active coords feeding the weather are chosen by `locationMode` ('gps' | 'home'): home mode uses the saved home, otherwise GPS, otherwise saved home, otherwise the Bedok fallback. The area name is resolved from the API's own `area_metadata` via `findNearestArea`. The `📍 <gpsArea> | 🏠 <homeArea>` toggle shows only when the current GPS area differs from the saved home area; tapping the plain location label (when on GPS and not already at home) saves the current area as home. When the toggle is showing and the user is on GPS mode, a "Set <area> as home" button also appears beneath it to *replace* an existing saved home with the current GPS area (the only way to re-home while a home is already set). `locationMode` is deliberately not destructured inside `render()` — it stays the module-level `let` so the toggle click handlers can reassign it.

**Drying model** — `DRY_HOURS_BASELINE` (3.5h for light cotton) × fabric multiplier × `weatherMultiplier({rh, windKmh, conditionText})`. The weather multiplier is the cube root of three factors (humidity, wind, sky condition) so no single extreme dominates.

**Hourly risk profile** — built from the 24-hour forecast's per-period east-region text where available (`periodRiskByHour`), falling back to a heuristic based on `regionOutlook` if the periods array is empty. The live 2-hour nowcast is overlaid onto the nearest display slot regardless.

**State persistence** — `localStorage` for theme (`laundry-brief-theme`), selected fabric IDs (`laundry-brief-fabrics`), saved home (`laundry-brief-home` — `{lat, lng, label}`), and location mode (`laundry-brief-mode` — `'gps' | 'home'`).

## Key constants to know

```js
DRY_HOURS_BASELINE = 3.5   // hours, light cotton, typical Bedok conditions
WASH_HOURS = 2             // wash cycle + hang time
LATEST_FINISH_HOUR = 19    // don't plan drying past 7pm
EARLIEST_START_HOUR = 6
```

Fabric multipliers live in the `FABRICS` array (`mult: 0.6` for synthetics → `2.3` for towels).

`FALLBACK_LAT/LNG` (`1.3236, 103.9273`, Bedok) is used only when GPS is denied and no home is saved.

## External APIs

All from data.gov.sg / MSS. All read-only, no auth required. The APIs return nationwide data with no location parameter — location only affects which area forecast is extracted and which humidity/wind station is treated as nearest.
- 2-hour nowcast (v2 then v1 fallback) — drives current condition and immediate-hour risk; its `area_metadata` also drives GPS→area-name resolution
- 24-hour forecast (v2 then v1 fallback) — drives period-based hourly risk and temp/RH range
- Relative humidity & wind speed (v1 only) — nearest station to the active location, non-fatal if unavailable
