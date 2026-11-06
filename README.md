# EXP Tracker

A lightweight, minimal EXP/hr and time-to-level browser tracker for
**MapleStory Classic World**, built primarily for personal use.

EXP Tracker reads MapleStory's EXP display from a window explicitly shared
through **Chrome or Edge**. Tesseract.js performs OCR in the browser, and
the tracker validates EXP readings, calculates recent EXP rates, estimates
time to the next level, and detects inactivity without accessing the game
process or memory.

The project intentionally focuses on a small set of EXP tracking features
for personal use.

> **Resolution requirement:** The current build is calibrated for
> **MapleStory Classic World at 1366×768**. Other resolutions require
> recalibration.

## Features

-   Displays current level, exact EXP, and EXP percentage
-   Calculates rolling EXP/hour
-   Calculates `%/hr` and `%/min`
-   Estimates time remaining until the next level
-   Displays cumulative capture session time
-   Uses a 5-minute rolling window to smooth normal grinding variance
-   Samples the EXP display every 15 seconds
-   Uses a 55-second warmup before displaying rate estimates
-   Detects inactivity from real elapsed time and displays `Idle` after
    3 minutes without EXP gain
-   Starts a fresh rate window when training resumes after being idle
-   Uses exact EXP as the authoritative progress measurement after startup
-   Validates startup readings against the MapleStory Classic World EXP
    table
-   Uses OCR percentage as an independent cross-check against exact EXP
-   Handles a known OCR ambiguity where MapleStory's `[` may be recognized
    as a leading `1` in low percentages
-   Holds or skips suspicious forward EXP readings until corroborated
-   Confirms backwards EXP movement before treating it as death/EXP loss
-   Detects level-ups from a near-cap EXP reset
-   Updates current percentage and ETA after confirmed EXP loss without
    subtracting the loss from earned EXP/hour
-   Verifies that the browser capture has advanced before OCR
-   Includes capture and OCR diagnostics for calibration/troubleshooting
-   Includes an optional compact display window
-   Runs capture, OCR, validation, and calculations in the browser
-   No session data is saved to disk

## How It Works

The capture pipeline is:

``` text
MapleStory
    ↓
Chrome / Edge screen sharing
    ↓
Browser video stream
    ↓
Fresh-frame verification
    ↓
Normalize game content to 1366×768
    ↓
Crop EXP text at x=718, y=734, 116×15
    ↓
Grayscale → 2.5× resize → threshold 150
    ↓
Tesseract.js OCR
    ↓
EXP validation / tracking
    ↓
Main tracker + optional compact window
```

The user explicitly selects the MapleStory window through the browser's
screen-sharing picker. The tracker normalizes the selected game-content
rectangle to `1366×768`, extracts the fixed EXP crop, preprocesses it, and
runs Tesseract.js OCR.

At startup, the tracker uses exact EXP together with the displayed
percentage and the EXP table to establish the character's level. A known
OCR failure mode can turn a low percentage such as `[2.25%]` into
`12.25%`; startup validation considers the plausible corrected
interpretation and accepts it only when it agrees with exact EXP and the
EXP table.

After the level is established, **exact EXP is authoritative**. OCR
percentage remains an independent cross-check. Implausible exact-EXP
readings are held or skipped rather than injected into the earned-EXP
history.

The tracker samples OCR every 15 seconds and calculates EXP/hr using up to
the most recent five minutes of validated earned EXP. `%/hr` converts that
same rolling rate into percentage points of the current level per hour;
`%/min` divides that rate by 60. Rate output has a 55-second warmup.

Idle status is based on real elapsed time since the last observed EXP gain.
After three minutes without progress the rate fields display `Idle`. When
EXP gain resumes, a fresh rate window begins so the AFK interval does not
depress the new training rate.

Before OCR, the tracker verifies that the browser capture has actually
advanced rather than treating every frame callback as proof of a new image.

### Level-ups and EXP loss

A level-up is detected when exact EXP resets to a small value after the
character was near the end of the current level. The tracker advances to
the next EXP-table entry and continues using exact EXP for progress.

A backwards exact-EXP reading at the same level is not accepted immediately
because OCR can occasionally misread digits. The tracker requires
corroboration before adopting the lower EXP state.

EXP/hour tracks **earned EXP**, not raw changes in the character's EXP
value. A confirmed death/EXP loss therefore updates current progress and
ETA without subtracting the lost EXP from earned EXP/hour.

## Using EXP Tracker

Open the GitHub Pages site in **Chrome** (recommended) or Microsoft Edge.

1.  Click **Share MapleStory Window**.
2.  Select MapleStory in the browser's sharing picker.
3.  Keep the MapleStory EXP display visible while tracking.
4.  Optionally click **Open Compact Window** for the smaller display.
5.  Stop browser sharing when finished.

### Compact window

The compact window receives already-calculated tracker state from the main
page. It does not request another screen share or run another OCR worker.
Keep the main tracker page open while using it.

### Calibration / diagnostics

The default game-content geometry is:

``` text
X:      1
Y:      31
Width:  1366
Height: 768
```

The fixed EXP crop is:

``` text
X:      718
Y:      734
Width:  116
Height: 15
```

The **Calibration / OCR diagnostics** panel shows the captured frame,
game-content rectangle, EXP crop, processed OCR image, raw OCR text, parsed
values, and validation status.

## Browser Support

**Chrome is recommended. Microsoft Edge is also supported.**

Firefox is not currently recommended. During project testing, Firefox could
continue reporting playback/frame activity while the visible shared image
remained stuck on the initial captured frame. The same stripped-down capture
test updated normally in Chrome.

## Requirements

-   Chrome (recommended) or Microsoft Edge
-   MapleStory Classic World at the calibrated `1366×768` resolution
-   Internet access to load the current Tesseract.js runtime and associated
    OCR assets

## Local Development

A clone can be served locally with:

``` powershell
uv run python -m http.server 8000
```

Then open:

``` text
http://localhost:8000/
```

in Chrome.

Python is only serving the static HTML/JavaScript files in this workflow.
Screen capture, OCR, validation, calculations, and display all occur in the
browser.

## Privacy

Tracking state is kept in memory for the current browser session. Closing or
reloading the tracker resets the session.

EXP Tracker does **not** intentionally:

-   access MapleStory process memory
-   modify or interact with game files
-   send input to the game
-   connect to MapleStory/Nexon accounts
-   upload captured screenshots or EXP data
-   collect telemetry
-   maintain persistent EXP/session logs

The browser version loads Tesseract.js from jsDelivr, so network requests
are required to load the OCR runtime and associated assets.

This project does not claim that its architecture guarantees compatibility
with a game's anti-cheat system or terms of service.

## EXP Data

Level EXP requirements are based on the [MeowDB MapleStory Classic World EXP Table](https://meowdb.com/msclassic/guides/exp-table-level-1-to-100).

The EXP table is used to establish the character's level at startup, derive
percentage from exact EXP, calculate time to level, and sanity-check OCR
transitions.

## Notes

The fixed crop was verified with MapleStory content at **1366×768**. If the
game resolution or browser/Windows capture geometry changes, recalibrate the
game-content rectangle before changing OCR thresholds or parser behavior.

Because OCR is sampled every 15 seconds, detection of EXP gain, level-ups,
and EXP loss has approximately that sampling resolution. Idle status itself
is evaluated from real elapsed time and the display refreshes independently.

EXP/hr, `%/hr`, `%/min`, and time-to-level are estimates based on recent
activity and naturally change with grinding efficiency, downtime, map
changes, party conditions, deaths, and other gameplay factors.

The site currently loads Tesseract.js v5 from jsDelivr. See
`THIRD_PARTY_NOTICES.md` for third-party information.

## Project History

This is a small personal-use project rather than a full-featured MapleStory
companion application. The browser implementation evolved from an earlier
private Python/OBS desktop prototype.

The public repository intentionally contains only the current static browser
implementation rather than the legacy desktop capture stack.

## Acknowledgements

The EXP-reading approach was inspired by the open-source
[Maple-Classic-Exp-Overlay](https://github.com/beanguri/Maple-Classic-Exp-Overlay)
project.

EXP Tracker intentionally implements only the small subset of functionality
needed for personal EXP-rate and time-to-level tracking.
