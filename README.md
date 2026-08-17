# IntELiMon Plot Data App

An offline, installable field data collection app (PWA) for point-intercept, time-lag fuel,
fuel-depth, and prism-cruise data. Built for **Android / Chrome**. Outputs one CSV row per plot,
appended to a single file per site.

---

## What it does

- **Project screen** — enter a 5-character site code, then **New file** (starts a fresh dataset)
  or **Open existing** (imports a previously exported CSV so you can keep adding to it). The current
  file and its record count are shown, with a **Download CSV** button.
- **Plot start** — plot name (defaults to `0001`), Operator 1 and Operator 2 initials, a
  **Manage categories** button (rename or clear any of the 10 cover labels, anytime), then
  **Scan started**, which stamps the date (`YYYYMMDD`) and time (`HHMM`).
- **40 point-intercept screens** across two transects (N→S then W→E), points every 1 m from
  0.5 to 19.5 m. Multiple cover categories may be selected per point under Live fuel / Litter /
  Other. **Bare Ground is exclusive** — selecting it clears all other selections, and selecting
  any other category clears Bare Ground. A running 1-/10-/100-/1000-hr time-lag fuel tally sits at
  the bottom of every point screen, above a **plot-wide notes field**.
- **12 depth screens**, inserted right after the intercepts at 0.5, 3.5, 6.5, 13.5, 16.5, 19.5 m
  on each transect. Three inputs (fuel bed, litter, duff, cm) plus the same shared notes field.
- **Prism data** — BAF selector (default 10) and four in-tree tallies (hardwoods, conifers, snags,
  other). Basal area per acre is computed as `count × BAF` on export.
- **Add record** — appends the plot's row to the on-device dataset (a duplicate check runs first),
  then offers **Download CSV**, **Collect overstory data**, and **Start another plot**.
- **Overstory** (optional, after Add record) — four quadrant pages (NW, NE, SE, SW; chosen from a
  dropdown), each with up to 25 rows of species code, an optional status
  (Live / Snag / Scorched / Insect damage / Catface scar), and a count. These write to a separate
  `SITE_overstory.csv`, one row per species entry; blank rows are skipped.

The notes field is a single plot-wide note (the same text on the point and depth screens); it
exports as the final `notes` column of the main CSV. Editable category labels persist on the
device — long-press a category on a point screen, or use **Manage categories** on plot start.

### QAQC prompts (all can be overridden)
- Advancing from a point with **no cover and no tally touch** → "Nothing was added to the last point."
- On depth, an **empty field** or a value over **140 / 20 / 10 cm** (fuel bed / litter / duff) → a
  warning naming the issue.

---

## Host it (one-time, ~5 min)

The app must be served over HTTPS once so it can install and cache for offline use.
GitHub Pages is the easy path:

1. Create a new GitHub repo and upload **all files in this folder** to the repo root
   (`index.html`, `manifest.webmanifest`, `sw.js`, `logo.png`, the three icons, this README).
2. In the repo: **Settings → Pages → Build and deployment → Source: Deploy from a branch**,
   pick `main` / `/root`, save.
3. Wait ~1 minute. Pages gives you an HTTPS URL like
   `https://YOURNAME.github.io/YOUR-REPO/`.

(Netlify or Cloudflare Pages work identically — drag the folder in.)

## Install on the phone (one-time)

1. Open the Pages URL in **Chrome on Android** (needs internet for this step only).
2. Install one of two ways — Chrome does not always pop an automatic banner:
   - Tap the **Install app to home screen** button on the app's first screen, **or**
   - Tap Chrome's **⋮** menu (top-right) → **Install app** / **Add to Home screen**.
3. Launch from the new home-screen icon. It now runs fully offline.

If neither the in-app button nor an "Install app" menu item appears, the page is being viewed in a
context Chrome won't install (e.g. a private tab, or an in-app browser opened from another app).
Open the URL directly in the normal Chrome app.

## How data is stored (important)

Chrome on Android has no API that lets a web app keep editing one chosen file on disk (that's a
desktop-only capability). So this app stores your records **on the device itself**, and you export
the file when you want it:

- **Add record** saves the plot's row into on-device storage immediately — this survives closing or
  swiping away the app, so an added record is never lost.
- **Download CSV** writes the complete dataset (header + every record) to your Downloads folder as
  one file. Do this at the end of the day, or after each plot if you like.
- **Open existing** imports a previously downloaded CSV back into the app so you can keep adding to
  it, then download the updated file. This is how you continue a file across days or devices.
- An **in-progress plot** (before you tap Add record) is auto-saved continuously; if the app closes
  mid-scan, reopening offers **Resume plot in progress**.

Practical field routine: New file → run plots (each Add record is saved) → Download CSV before you
leave. Next day: Open existing (pick yesterday's CSV) → keep adding → Download again.

## Updating the app

Change any file, then bump the cache version in `sw.js` (`intelimon-v1` → `intelimon-v2`) and
re-upload. Phones pick up the new version the next time they open online.

---

## CSV schema

### Main file (33 columns, one row per plot)

```
site, plot, op1, date, time, op2,
grass, forbs, woody, fine_lit, decid_lit, SNLIT, LNLIT, BG, cat9, cat10,
fuel_1hr, fuel_10hr, fuel_100hr, fuel_1000hr,
BAF, HWPBA, CPBA, SPBA, OPBA, HW_in, CON_in, SNAG_in, OTH_in,
MFBD, MLD, MDD, notes
```

- **`op1` / `op2`** — Operator 1 and Operator 2 initials from the plot-start screen.
- **`notes`** — the plot-wide free-text note (same field shown on point and depth screens).
- **Cover columns** (`grass`…`cat10`) — count of the 40 points where that category was selected.
  Because multiple categories can be recorded per point, these need not sum to 40. Column names
  are fixed slugs; renaming a category changes the display label only, so CSVs stay aligned across
  plots. `cat9`/`cat10` are the two editable "Other" slots.
- **`fuel_*`** — running time-lag tally totals.
- **`BAF`** — selected basal area factor. **`HWPBA`/`CPBA`/`SPBA`/`OPBA`** — basal area per acre
  = in-tree count × BAF. **`HW_in`/`CON_in`/`SNAG_in`/`OTH_in`** — raw in-tree counts (kept for
  traceability; delete these four from `HEADER` and `buildRow()` in `index.html` if unwanted).
- **`MFBD`/`MLD`/`MDD`** — mean fuel bed / litter / duff depth (cm), pooled across all 12 depth
  locations, 1 decimal.

### Overstory file (`SITE_overstory.csv`, one row per species entry)

```
site, plot, date, quadrant, species, status, count
```

Written when you collect overstory after a plot. `quadrant` is NW / NE / SE / SW; `species` is a
free-typed code; `status` is optional (Live / Snag / Scorched / Insect damage / Catface scar, blank
if unset); `count` is the number of individuals with DBH > 4 inches. Rows with no species code are
skipped. Download it from the overstory finish screen or the project screen.

## Notes / limits

- Designed for Android Chrome (also works on desktop Chrome). Records live in the browser's local
  storage for this site — **download the CSV regularly**, and don't clear the browser's site data
  for the app without exporting first.
- **New file** clears the current on-device records to start fresh (it warns you first). Download
  the existing file before starting a new one if you still need it.
- The iOS build discussed separately will share this same storage-and-export model.
