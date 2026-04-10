# EIS Countdown

> Payload launch monitor for human delivery systems.

A single-page web app for timing labor contractions. Every time a
contraction starts, tap a button; the graph plots the interval
(Δ, in minutes) between consecutive presses and when intervals converge
to roughly 5 minutes — the "EIS window" — it's time to head to the
launch facility.

Live at: https://rmaiko.github.io/timer/

## Input modes

The app supports two ways to log a contraction. Both write to the same
record schema, so you can mix them freely.

1. **Duration buckets (primary).** Four colored buttons that bucket the
   contraction by how long it lasted. Fastest to tap, no preparation
   needed, good for early labor when the phone might not even be
   in hand:

   | Label | Range | Meaning (cited) |
   |---|---|---|
   | `Ow` | < 30 s | Early/latent range — Cleveland Clinic "20 to 30 seconds" [4] |
   | `Oooow` | 30 – 60 s | Intermediate, not yet active |
   | `Oooooow` | 60 – 120 s | Active labor range — Mayo "60 to 90 seconds" [2] |
   | `Oooo….w!` | > 120 s | NHS urgent override [1] — contractions lasting longer than 2 minutes |

   Each bucket stores the **midpoint** of its range as an estimated
   `duration_sec` (20 / 45 / 90 / 150 s) and tags the record with a
   `bucket:*` source so you can tell measured vs. estimated apart
   later.

2. **`BEGIN SCREAM` / `END SCREAM` (secondary).** Tap once when the
   contraction starts, tap again when it ends — the app measures the
   exact duration in seconds and records the true start time. More
   precise, but requires the app to be open at the start of the
   contraction, so it's most useful later in labor when you're already
   timing deliberately.

## Features

- **Convergence graph.** Y-axis is Δ minutes (capped at 60), X-axis is
  wall-clock time. A dashed green line marks the 5-minute EIS target, a
  shaded band marks the 4–6 min window, and a funnel shows where the
  points *should* be trending.
- **Color-coded points.** Green inside the target band, amber when close,
  blue when far.
- **Robot-humor status line.** Scales from "payload has no intention of
  leaving the garage" through "PROCEED TO LAUNCH FACILITY" to
  "GOOD LUCK HUMAN" depending on the recent pattern.
- **Telemetry line.** Last Δ, rolling average of the last 5 intervals,
  σ, last duration + source, rolling avg duration, whether the 5-1-1
  cadence has been sustained for ~1 hour, and a heuristic convergence
  %.
- **Local storage.** All data stays in your browser under the key
  `eis_contractions_v2`. Old `now_presses_v1` data is auto-migrated on
  first load and tagged with source `legacy`. No servers, no accounts,
  no tracking.
- **Export.** Download the full log as JSON or CSV at any time — each
  record includes timestamp, ISO time, Δ minutes, duration_sec, and
  source.
- **Purge.** Wipe the log when you're done (or testing).

## Usage

1. Open https://rmaiko.github.io/timer/ on your phone or laptop.
2. Tap a bucket button each time a contraction *finishes*, picking the
   range it felt like. (Or use `BEGIN SCREAM` / `END SCREAM` for an
   exact measurement if the app is already open.)
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

Everything is stored in `localStorage` under the key
`eis_contractions_v2`. Each record is `{ start_ts, duration_sec, source }`
where `source` is one of `bucket:hey | bucket:ow | bucket:ouch |
bucket:yikes | measured | legacy`. Old `now_presses_v1` data (bare
timestamps) is auto-migrated on first load and tagged with source
`legacy`.

- Data persists across reloads on the same browser.
- Data is **not** synced across devices. Pick one device and stick with it.
- Clearing your browser's site data will wipe the log. Export first if
  you care.

## How the tiers are classified

Every numeric threshold the app uses comes from a published guideline.
Duration-dependent checks are enforced whenever duration data is
available (from the SCREAM button or a bucket tap); otherwise the
duration half of each rule is noted as "operator verified."

| Tier | Triggered when | Source |
|---|---|---|
| `idle` | 0 presses | — |
| `first` | 1 press | — |
| `drill` | avg of last 5 intervals > 10 min | Cleveland Clinic [3], NHS [1] |
| `warming` | 5 min < avg ≤ 10 min | Mayo Clinic [2] |
| `approach` | avg ≤ 5 min, but the cadence has NOT been sustained ~1 hr | Cleveland Clinic [3], Mayo Clinic [2] |
| `prep` | avg ≤ 5 min AND sustained ≥1 hr AND (avg duration ≥ 55 s when data exists) | Cleveland Clinic [3] (5-1-1 rule) + Mayo Clinic [2] duration |
| `imminent` | avg < 3 min (proxy for deep active labor, Mayo's 2–5 min active-labor range) | Mayo Clinic [2] |
| `critical` | last interval < 1.67 min OR avg < 1.67 min OR last duration > 120 s | NHS [1] — "6+ contractions in 10 min" OR "contractions lasting longer than 2 minutes" |

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

1. Duration data is either **measured** (via `BEGIN SCREAM` / `END
   SCREAM`) or **bucketed** (via a tap on one of the 4 Ow buttons, in
   which case the stored `duration_sec` is the midpoint of the selected
   range — 20 / 45 / 90 / 150 s). When duration data is missing, the
   "~1 minute long" half of 5-1-1 falls back to operator confirmation.
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
6. `BEGIN SCREAM` requires the app to stay open between start and end
   taps — if you leave the page mid-contraction the timer is lost.
   Use a bucket tap after the fact in that case.

## Disclaimer

This is a toy. It is **not** a medical device, **not** medical advice,
and **not** a replacement for an actual healthcare provider. The
thresholds above are drawn from published guidelines but the
classification logic is the author's own. If in doubt, call your
midwife, OB, or maternity unit.

## License

Beerware — see [LICENSE](LICENSE).
