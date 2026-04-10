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

## How the tiers are classified

Every numeric threshold the app uses comes from a published guideline.
The app cannot measure contraction *duration* (it only sees button
presses), so any part of a rule that depends on duration is noted as
"operator verified."

| Tier | Triggered when | Source |
|---|---|---|
| `idle` | 0 presses | — |
| `first` | 1 press | — |
| `drill` | avg of last 5 intervals > 10 min | Cleveland Clinic [3], NHS [1] |
| `warming` | 5 min < avg ≤ 10 min | Mayo Clinic [2] |
| `approach` | avg ≤ 5 min, but the cadence has NOT been sustained ~1 hr | Cleveland Clinic [3], Mayo Clinic [2] |
| `prep` | avg ≤ 5 min AND sustained over ~1 hour (≥55 min span) | Cleveland Clinic [3] (5-1-1 rule) |
| `imminent` | avg < 3 min (proxy for deep active labor, Mayo's 2–5 min active-labor range) | Mayo Clinic [2] |
| `critical` | last interval < 1.67 min OR avg < 1.67 min (equivalent to NHS "6+ contractions in 10 min") | NHS [1] |

`σ` (the standard deviation of the last 5 intervals) and the
"heuristic convergence" percentage shown on the page are display-only.
**No authoritative guideline publishes a numeric regularity / σ
threshold**, so the app does not use σ in classification.

## References

1. **NHS** — "Signs that labour has begun." Last reviewed
   2023-11-09; next review due 2026-11-09.
   <https://www.nhs.uk/pregnancy/labour-and-birth/signs-that-labour-has-begun/>
2. **Mayo Clinic** — "Stages of labor and birth: Baby, it's time!"
   <https://www.mayoclinic.org/healthy-lifestyle/labor-and-delivery/in-depth/stages-of-labor/art-20046545>
3. **Cleveland Clinic** — "Stages of Labor." Last reviewed 2025-04-23.
   <https://my.clevelandclinic.org/health/body/22640-stages-of-labor>
4. **Cleveland Clinic** — "Contractions." Last updated 2024-08-08.
   <https://my.clevelandclinic.org/health/symptoms/contractions>
5. **ACOG** — FAQ: "How to Tell When Labor Begins."
   <https://www.acog.org/womens-health/faqs/how-to-tell-when-labor-begins>
6. **ACOG / SMFM** — Obstetric Care Consensus No. 1: "Safe Prevention of
   the Primary Cesarean Delivery," *Obstet Gynecol*, March 2014
   (establishes 6 cm as the threshold for active labor). Full text via
   AJOG: <https://www.ajog.org/article/S0002-9378(14)00055-6/fulltext>
7. **WHO** — *WHO Labour Care Guide: User's Manual*, 2020 (active first
   stage from 5 cm).
   <https://iris.who.int/bitstream/handle/10665/337693/9789240017566-eng.pdf>
8. **NICE** — NG235, "Intrapartum care." Published 2023-09-29
   (established labour = progressive dilatation from 4 cm).
   <https://www.nice.org.uk/guidance/ng235/chapter/Recommendations>

Note on sources not cited: any popular "4-1-1 rule for first-time
mothers" convention is **not** present in ACOG, NHS, NICE, WHO, Mayo
Clinic, or Cleveland Clinic guidance that was verifiable, so the app
does not implement a primipara/multipara toggle.

## When to go to the hospital regardless of the app

Per NHS [1] and ACOG [5], contact your maternity unit urgently if any
of the following happen, whatever the app is showing:

- Your waters break (especially if the fluid is smelly or coloured)
- Vaginal bleeding
- You notice the fetus is moving less often
- Constant severe pain with no relief between contractions
- You are under 37 weeks and think you may be in labour
- Any contraction lasts longer than 2 minutes
- 6 or more contractions in 10 minutes
- A strong urge to push — in the UK, call 999

## App limitations

1. The app measures only the **interval** between button presses, not
   the **duration** of each contraction. The "1-minute long" part of
   5-1-1 must be confirmed by the operator.
2. The rolling statistics window is the last 5 intervals. The "sustained
   ~1 hour" check uses all presses in the last ~65 minutes.
3. No guideline publishes a numeric regularity/σ threshold — so σ and
   the heuristic convergence % are display-only and do not drive
   classification.
4. ACOG/WHO/NICE disagree on the cervical dilation marker for active
   labor (6 cm / 5 cm / 4 cm). The app cannot measure dilation and
   does not use this threshold.
5. The app does not know about override signals (see list above) — act
   on those independently.

## Disclaimer

This is a toy. It is **not** a medical device, **not** medical advice,
and **not** a replacement for an actual healthcare provider. The
thresholds above are drawn from published guidelines but the
classification logic is the author's own. If in doubt, call your
midwife, OB, or maternity unit.

## License

Beerware — see [LICENSE](LICENSE).
