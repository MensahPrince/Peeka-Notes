---
id: cig1amq06k
title: "REMI Identify a Place"
sort: 17000
version: 1
---

`identify.py` answers one question: *what can REMI say about this spot?* Give it a coordinate in the pilot box, and it reports on the 160 m square under it. This is the first piece of M7 (decisions and alerts). Tracking is on [REMI Project Hub](page:imqxd436oq). *Added 2026-11-27.*

<callout icon="⚖️">

Every mining answer reads **"suspected mining activity"**, plus "verify before acting; licence status unknown". The REMI rule: a person checks every alert before anyone acts on it.

</callout>

## How to run it

From `~/gitRepo/REMI`:

```bash
.venv/bin/python identify.py 5.4566 -2.1301              # latitude longitude (e.g. from Google Maps)
.venv/bin/python identify.py --utm 596287 603236         # x y in UTM 30N, as QGIS shows them
.venv/bin/python identify.py 5.43 -2.14 5.52 -2.17       # several places in one go
.venv/bin/python identify.py --json 5.4566 -2.1301       # for other programs
```

It takes about 8 seconds, mostly loading the models. Several places in one call cost little more than one. In QGIS, the status bar shows the coordinate under the mouse (set the project to EPSG:32630 to get UTM).

## What it reports

| Line | Where it comes from | How far to trust it |
| --- | --- | --- |
| Summary | the lines below, combined | as good as its weakest input |
| Land, 2025-02-04 | M4 ResNet-18: vegetation, bare ground, built-up, mining, with probabilities | mining F1 0.72 on test: about 2 in 3 "mining" squares really are mining |
| New mining, 2016 → 2025 and 2024 → 2025 | M5 Siamese model: probability, flagged or not (threshold 0.972), and the hectares that went green → bare | reliable on clear-cut cases; overflags under cloud and haze, which is why visibility is shown |
| Visible share | cloud mask on both dates; for 2016, low-haze dates only | under 50 % → **not observed**, never "no" |
| Mining maps | share of the square inside Tang & Werner 2023 and Maus et al. 2022 | maps are from about 2019–2021 and drawn loosely |
| Reviewed candidate, chip review | the M3 labels and the M4 chip review touching the square | 903 of 914 candidate labels were made by Claude; the test split was checked by you |
| Nearest river and road | OSM rivers + DEM streams; OSM roads | OSM misses many small roads |
| Reported site | news and research reports within 2 km | each report is only placed to within \~2 km |

## Four test places

| Place | Summary | Why it's right (or not) |
| --- | --- | --- |
| Rank 43, north-west stream | Suspected mining, new clearing 2024 → 2025; inside both maps | Land model 97 % mining; 2016 not observed (hazy) |
| Rank 2, Tarkwa mine | Suspected mining, new clearing 2024 → 2025 | 0\.42 ha of forest cleared within the year, though the square's average NDVI didn't move. The 2016 pair isn't flagged: that ground was bare in 2016, grew back, and was dug again |
| Himan raid site (MLNR, July 2025) | Suspected mining (looks like mining in 2025), outside the published maps | 94 % mining; 2016 → 2025 probability 0.93, just under the threshold; the raid report sits 0 km away |
| Prestea town centre | No sign of mining: built-up (99 %) | Correct; the 2014–16 survey names Prestea as a galamsey community, but the workings are on the rivers around it |
| A forest square in the north | No sign of mining: vegetation (100 %) | 2016 not observed (hazy), and it says so |

## Limits

- Only inside the pilot box and on tiles kept in M2. Elsewhere it says "outside the pilot box" or "not observed".
- The answers come from the saved model outputs, for the dates already processed. A new date has to go through M2 and the models first.
- It can't tell galamsey from a new farm yet, and it can't say whether a site is licensed (that needs the concession data).
- The change maps it reads (`data/models/change_prob_*.tif`) must be rebuilt with `change.py map` after the change model is retrained.
