# IntELiMon Plot Data App

An offline, installable field data collection app (PWA) for point-intercept, time-lag fuel,
fuel-depth, and prism-cruise data. Built for **Android / Chrome**. Outputs one CSV row per plot,
appended to a single file per site.

---

## What it does

- **Project screen** — enter a 5-char site code, then **New file** (creates a CSV with the header
  row where you choose) or **Open existing** (append to a CSV you pick). The active file is shown.
- **Plot start** — plot name (4), observer initials, operator initials, then **Scan started**,
  which stamps the date (`YYYYMMDD`) and time (`HHMM`).
- **40 point-intercept screens** across two transects (N→S then W→E), points every 1 m from
  0.5 to 19.5 m. One cover category per point (radio) under Live fuel / Litter / Other. A running
  1-/10-/100-/1000-hr time-lag fuel tally sits at the bottom of every point screen.
- **12 depth screens**, inserted right after the intercepts at 0.5, 3.5, 6.5, 13.5, 16.5, 19.5 m
  on each transect. Three inputs: fuel bed, litter, duff (cm).
- **Prism data** — BAF selector (default 10) and four in-tree tallies (hardwoods, conifers, snags,
  other). Basal area per acre is computed as `count × BAF` on export.
- **Add record** — appends the plot's single row to the active file, with a duplicate check.
  Then **Start another plot** to keep filling the same file.

Editable category labels (the two blank cover slots, plus any cover label, plus the prism "Other"
label) persist on the device. **Long-press a category to rename it.**

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
2. Chrome shows **Install app** / **Add to Home screen** — tap it.
3. Launch from the new home-screen icon. It now runs fully offline.

## Where the CSV lives

When you tap **New file**, Android's file dialog lets you choose the location (Downloads,
a Drive folder, etc.) and suggests `SITE_YYYYMMDD.csv`. **Add record** rewrites that same file
with the new row appended. When you reopen the app another day, Chrome asks once to re-confirm
permission to edit that file — tap allow. A backup copy of every record is also kept inside the
app in case a file permission is ever lost.

## Updating the app

Change any file, then bump the cache version in `sw.js` (`intelimon-v1` → `intelimon-v2`) and
re-upload. Phones pick up the new version the next time they open online.

---

## CSV schema (32 columns, one row per plot)

```
site, plot, obs, date, time, operator,
grass, forbs, woody, fine_lit, decid_lit, SNLIT, LNLIT, BG, cat9, cat10,
fuel_1hr, fuel_10hr, fuel_100hr, fuel_1000hr,
BAF, HWPBA, CPBA, SPBA, OPBA, HW_in, CON_in, SNAG_in, OTH_in,
MFBD, MLD, MDD
```

- **Cover columns** (`grass`…`cat10`) — count of the 40 points where that category was the hit.
  Column names are fixed slugs; renaming a category on-screen changes the display label only, so
  CSVs stay aligned across plots. `cat9`/`cat10` are the two editable "Other" slots.
- **`fuel_*`** — running time-lag tally totals.
- **`BAF`** — selected basal area factor. **`HWPBA`/`CPBA`/`SPBA`/`OPBA`** — basal area per acre
  = in-tree count × BAF. **`HW_in`/`CON_in`/`SNAG_in`/`OTH_in`** — raw in-tree counts (kept for
  traceability; delete these four from `HEADER` and `buildRow()` in `index.html` if unwanted).
- **`MFBD`/`MLD`/`MDD`** — mean fuel bed / litter / duff depth (cm), pooled across all 12 depth
  locations, 1 decimal.

## Notes / limits

- Designed for Android Chrome. On iOS the single-file append is not possible (WebKit lacks the
  File System Access API) — that's the separate iOS build discussed for later.
- An in-progress scan is held in the browser on the device; it is not a file until you add the
  record. Finish a plot and add the record before closing.
