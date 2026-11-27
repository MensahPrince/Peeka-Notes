---
id: fpk4rpe5tv
title: "REMI Areas: Running the Pipeline on a New Place"
sort: 18000
version: 1
---

Since 2026-11-27 REMI can run on any box, not only the Prestea pilot. Each place is an **area**: one small file in `areas/` with its box and dates. Code: `area.py`. Tracking is on [REMI Project Hub](page:imqxd436oq); looking up a place is on [REMI Identify a Place](page:cig1amq06k).

<callout background="yellow" icon="⚠️">

**Running in a new place works. The models haven't been tested there.** They learned from Prestea only. In a new area, every `identify.py` answer ends with "models trained in prestea, not yet tested here". The first test in Brazil (below) already shows why that matters.

</callout>

## Add an area

```bash
# 1. the box (west south east north, in degrees) and a before and an after date
.venv/bin/python area.py new tapajos -56.03 -4.33 -55.94 -4.24 2023-07-23 2024-07-22

# 2. everything else runs with REMI_AREA set to that name
export REMI_AREA=tapajos
.venv/bin/python s2_earth_search.py 2023-07-23     # once per date (s2_planetary.py before 2017)
.venv/bin/python s2_earth_search.py 2024-07-22
.venv/bin/python osm_download.py
.venv/bin/python dem_streams.py
.venv/bin/python cloud_mask.py
.venv/bin/python normalise.py
.venv/bin/python water_mask.py
.venv/bin/python labels.py
.venv/bin/python tiles.py
.venv/bin/python change.py prepare
.venv/bin/python change.py map

# 3. then look up places; the area is found from the coordinate
.venv/bin/python identify.py -4.2857 -55.9515
```

A 10 × 10 km box took about 10 minutes end to end on this laptop, most of it downloads.

**Choosing dates.** Pick two clear dates in the **same season**, a year or so apart. In the Brazilian Amazon that's the dry season, roughly June to September. To find clear dates, search Earth Search or [Copernicus Browser](https://browser.dataspace.copernicus.eu) for scenes under 10 % cloud.

## What changes with the area

| Thing | Pilot box (Prestea) | A new area |
| --- | --- | --- |
| Files | `data/`, `previews/`, as before | `data/areas/<name>/`, `previews/<name>/` |
| UTM zone | 30N (EPSG:32630) | from the box centre; south of the equator too (e.g. 21S = EPSG:32721) |
| Seam reference tile | 30NXM (chosen by hand, see the M2 report) | the tile covering most of the box, unless set |
| Train / val / test split | balanced by 7.68 km block | **all test**: the models were trained elsewhere, so the whole area is unseen ground |
| Models | trained here | Prestea's (`models_from`), until labels exist here |
| 2015/16 haze composite | yes | none, unless a `baseline` is added to the area file |
| Mining outlines, WorldCover | clipped to the box | the same global datasets, clipped to the new box |

## The check that nothing changed for Prestea

The old code (from git) and the new code were both run on the same Prestea inputs, and their outputs compared pixel by pixel: **761 files, all identical**. That covers the reflectance images, cloud and water masks, the 2016 composite, every tile and index, and both sets of change chips.

<toggle>

A small finding from the check: reruns don't reproduce the M2 numbers exactly

Rerunning `normalise.py` gives NIR gain **0.927** where the M2 report has **0.92** (and SWIR 1.151 against 1.16). The old code does the same, so this isn't from the refactor. The first run computed the tile seams, which draws random numbers before IR-MAD takes its sample; later runs read the saved seams, so IR-MAD gets a different random sample. The effect is small: 2025 water 393 ha against 391 ha, and 5 of 21,974 labelled change chips differ. The saved data (and every reported number) are from the first run. Fixing it would mean giving IR-MAD its own random generator, which would shift the numbers slightly once more.

</toggle>

## First test: Itaituba, Brazil

A 10 × 10 km **test box** on the Tapajós at Itaituba, Pará (area `itaituba_test`, UTM 21S). It was picked only to test the code: it's mostly the town and the river, and **no published mining outline touches it**. Dates 2023-07-23 and 2024-07-22, both cloud-free.

| Step | Result |
| --- | --- |
| Download, cloud mask | 2 Sentinel-2 tiles (21MWR, 21MXR); 0 % cloud on both dates |
| Seams | reference 21MXR picked automatically; seams were small (blue 0.0013 → 0.0001) |
| Matching | IR-MAD kept only 0.1 % of the box as unchanged ground, all in a narrow brightness range: probably the river. ⚠️ Not checked |
| Land cover (WorldCover) | forest 39 %, water 31 %, other vegetation 19 %, built-up 10 % |
| Change model (Prestea's) | flagged **33 chips (84 ha)** as new mining |

**What it flagged.** The three strongest flags (probability 1.0) are fresh clearings of 0.5–1.5 ha **on the edge of town**. The land model calls them built-up (59–80 %), so they're probably new building plots, not mining. That's the gap already noted in the [REMI M5 Change Detection Report](page:56h06s9n5l): the change model has never seen a clearing that isn't mining, so it calls every new clearing mining. `identify.py` now says so when the two models disagree: *"… (but the land model sees built-up, 80 %: could be other clearing, e.g. building)"*.

## Before trusting REMI in a new place

1. **Pick a real study box** where mining is known: for Brazil, inside a Tang & Werner or Maus et al. outline, or near a reported site.
1. **Check the matching**, since IR-MAD may lean on water where a big river fills the box.
1. **Label a test set there** (a few hundred reviewed sites, as in M3) and score the Prestea models on it.
1. **Teach the change model non-mining clearing** (the M5 next step); without it, town growth and new farms will read as mining everywhere.
