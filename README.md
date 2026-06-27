# Laundry Brief

A tiny weather app that answers one question: **should I hang my laundry out to dry right now?**

It reads Singapore's live weather (from NEA / MSS) for your location, factors in what's in your load, and tells you whether there's a clean drying window before rain rolls in — with a suggested schedule and an hour-by-hour outlook.

## [→ Open the app](https://theclumsygeek.github.io/laundry-brief/)

No account needed, no API key required.

## Reading the verdict

The big card at the top is the recommendation:

- **🟢 Start by [time]** — there's a comfortable drying window. Wash now and you'll be dry with margin to spare.
- **🟡 Start by [time] — tight** — it'll work, but the window is narrow. Hang promptly and keep an eye on the sky.
- **🔴 Not today** — no clean drying window before showers are expected, or before the evening light fades (clothes barely dry after ~7pm). Use a dryer or an indoor rack.

If you open the app after 7pm, it automatically shows **tomorrow's** forecast instead.

Below the verdict:

- **Timeline (6 AM – 8 PM)** — the top bar shows your laundry stages (wash & hang → drying → safe buffer) in brown/blue; the bar beneath it is the rain-risk heat strip, coloured clear → low risk → showers → thundery. The two bars use separate colour schemes, and the legend below explains each. The vertical "now" line marks the current time.
- **Schedule** — the concrete steps: when to start washing, when to hang, when it should be dry, and when rain risk rises.
- **Hour-by-hour outlook** — a simple icon strip across the day.

An **"estimated"** badge on the timeline and outlook means no detailed hourly forecast was available, so the rain-risk profile is a coarse estimate from the day's general outlook rather than a true per-hour forecast — treat it as a rougher guide.

## What's in the load?

Tap the **fabric chips** to match your wash. Each fabric dries at a different rate (synthetics are quick; towels and denim are slow), and the app plans around your **slowest** pick — the realistic constraint for a mixed load. The one driving the plan is tagged **"sets the plan."**

Your selection is remembered for next time.

## Location

On first load the app asks for **location permission**.

- **Allow it** → the masthead shows your actual area (e.g. "Tampines, SG") and the forecast is tailored to it.
- **Deny it** → the app falls back to Bedok.

### Saving a home location

While on GPS, **tap your area label** (it's shown with a dotted underline) to save it as your **home**. You'll see a brief "🏠 Home saved!" confirmation.

You won't see a toggle right after saving — because you're *at* home, there's nothing to switch between. That's expected.

### Switching between Here and Home

When you're **away** from home (your GPS area differs from your saved home), a toggle appears in the masthead, labelled with the two area names:

```
📍 Tampines  |  🏠 Bedok
```

- **📍 [your area]** — forecast for where you currently are.
- **🏠 [home]** — forecast for your saved home, so you can decide whether to rush back for the washing.

Your choice is remembered between visits.

### Changing your home

To replace an existing home with somewhere new: while on **📍** (your current location) and away from your saved home, a small **"Set [area] as home"** link appears under the toggle. Tap it to overwrite the saved home with where you are now. The toggle then disappears, since you're once again *at* home.

## Refreshing

- **Refresh** button (next to "Live · MSS/NEA · fetched …") — fetches the latest weather **and** a fresh GPS fix. Use this after you've moved.
- Tapping fabric chips or the location toggle re-runs the plan using your **cached** location (no GPS re-prompt), so it's instant.

> If you physically move and the location looks stale, hit **Refresh** (or reload the page) to grab a new GPS fix.

## Dark mode

Use the **☀ / ☾** toggle in the top-right, or it follows your system theme by default. Your choice is remembered.

## Notes & limitations

- Drying-time estimates are a model (baseline drying time × fabric × humidity/wind/sky), not a guarantee. Singapore's afternoon thundery showers can build faster than forecast — **bring laundry in early if clouds darken.**
- Humidity and wind refine the estimate but are optional; if those feeds are unavailable the app still works from the sky condition alone.
- All weather data is read-only and public, from [data.gov.sg](https://data.gov.sg) / MSS. Nothing you do is sent anywhere — your home location and preferences are stored only in your browser's `localStorage`.

### Resetting saved data

To clear your saved home, fabric picks, theme, and location mode, clear this site's data in your browser settings, or run in the browser console:

```js
['home','fabrics','theme','mode'].forEach(k => localStorage.removeItem('laundry-brief-' + k));
```
