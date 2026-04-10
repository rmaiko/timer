# EIS Countdown

> Payload launch monitor for human delivery systems.

A single-page web app for timing labor contractions. Press the big red
`CONTRACTION` button each time one starts. Starting from the second press,
the graph plots the interval (Δ, in minutes) between consecutive presses
against wall-clock time. When the intervals converge to roughly 5 minutes —
the "EIS window" — it's time to head to the launch facility.

Live at: https://rmaiko.github.io/timer/

## Features

- **One button.** Hit it every time a contraction starts.
- **Convergence graph.** Y-axis is Δ minutes (capped at 60), X-axis is
  wall-clock time. A dashed green line marks the 5-minute EIS target, a
  shaded band marks the 4–6 min window, and a funnel shows where the
  points *should* be trending.
- **Color-coded points.** Green inside the target band, amber when close,
  blue when far.
- **Robot-humor status line.** Scales from "payload has no intention of
  leaving the garage" through "PROCEED TO LAUNCH FACILITY" to
  "GOOD LUCK HUMAN" depending on the last interval.
- **Rolling telemetry.** Shows the latest Δ and a rolling average of the
  last five intervals.
- **Local storage.** All data stays in your browser. No servers, no
  accounts, no tracking.
- **Export.** Download the full log as JSON or CSV at any time.
- **Purge.** Wipe the log when you're done (or testing).

## Usage

1. Open https://rmaiko.github.io/timer/ on your phone or laptop.
2. Press `CONTRACTION` at the start of each contraction.
3. Watch the points funnel toward the 5-minute line.
4. When the status line starts yelling in all caps, act accordingly.
5. Download the telemetry afterwards if you want to keep a record.

## Running locally

It's a single `index.html` file with no build step and no dependencies.

```sh
git clone https://github.com/rmaiko/timer.git
cd timer
open index.html        # macOS — or just double-click it
# or serve it:
python3 -m http.server
```

## A note on data

Everything is stored in `localStorage` under the key `now_presses_v1`.
That means:

- Data persists across reloads on the same browser.
- Data is **not** synced across devices. Pick one device and stick with it.
- Clearing your browser's site data will wipe the log. Export first if
  you care.

## Disclaimer

This is a toy. It is not a medical device, not medical advice, and
definitely not a replacement for an actual healthcare provider. If in
doubt, call your midwife / OB / hospital.

## License

Beerware — see [LICENSE](LICENSE).
