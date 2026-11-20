---
id: epzdv5qm9v
title: "REMI Test Imagery"
cover: "_assets/f3949d7fdc230cbc80d651834a54ceab.jpg"
cover_position: 79
sort: 5000
version: 1
Imagery: REMI Sentine-2 Images.
---

The first real images of the pilot box, downloaded on 2026-10-21 with the scripts in the [REMI Data Download Guide](page:sf7hcbolae). They are previews for looking at; the full-resolution data is on disk (see below).

<callout icon="📍">

**Pilot box:** `-2.20, 5.30, -1.90, 5.60` (lon/lat), about 33 × 33 km around Prestea, Western Region, Ghana. North is up in every image.

</callout>

<columns>

<column>

### 🌿 How to read false colour

Near-infrared is shown as red, so **healthy vegetation is bright red**. **Bare soil, mining pits and towns are cyan or grey**, and **water is dark blue or black**. Land cleared for mining stands out far more clearly than in true colour.

</column>

<column>

### 👀 What to look for

- **Cyan threads along the rivers** in the west and north-west: cleared, bare ground following the watercourses, the pattern REMI is meant to flag.
- **Large open pits and ponds at the bottom edge**, near Tarkwa: they look like a big industrial mine, a good test of the licence overlay in Layer 6.

</column>

</columns>

<callout background="yellow" icon="⚠️">

These are first impressions from looking at the pictures, **not verified sites**. Nothing here has been checked on the ground or against concession data.

</callout>

## 🛰️ Sentinel-2 · 2025-02-04 · 10 m

All four tiles over the box were captured that day, so the image has no gaps.

<columns>

<column>

![Sentinel-2 · 2025-02-04 · true colour (10 m)](_assets/538742719b31f1564803f998168933e7.png)

</column>

<column>

![Sentinel-2 · 2025-02-04 · false colour, NIR-red-green (10 m)](_assets/29aa320928161d9b6498f1efc93f4177.png)

</column>

</columns>

## 🌎 Landsat 8 · 2024-12-21 · 30 m

Scene grid square path 194 / row 056, the only one that covers the whole box. It has no gaps and no cloud flagged in the box.

<columns>

<column>

![Landsat 8 · 2024-12-21 · true colour (30 m)](_assets/a3d23c6b05c30dfc6429d35c00385bb9.png)

</column>

<column>

![Landsat 8 · 2024-12-21 · false colour, NIR-red-green (30 m)](_assets/3ea0ec57fb9c8ba389d4ea7e6c470172.png)

</column>

</columns>

## 💧 Rivers and water · 2025-02-04

Testing how well each source maps the box's rivers, for Layer 6's "near a river?" check. **Left:** open water detected from Sentinel-2 with the MNDWI water index (blue) against OSM rivers (red). **Right:** streams computed from the Copernicus 30 m elevation map (yellow) and OSM rivers (blue), over false colour.

<columns>

<column>

![Open water detected by MNDWI (blue) and OSM rivers (red), 2025-02-04; grey is cloud](_assets/8ef2229779c79833f6c91279ab70de97.png)

</column>

<column>

![Streams computed from the Copernicus DEM (yellow) and OSM rivers (blue), over Sentinel-2 false colour, 2025-02-04](_assets/ac4df853a1e3daa2b7f5dee44e994066.png)

</column>

</columns>

| Source | River length in box | Detected water within 150 m | Within 60 m |
| --- | --: | --: | --: |
| OSM rivers and ponds | 130 km | 42 % | 32 % |
| Streams from the DEM | 1,270 km | 64 % | 36 % |
| **Both combined** |  | **73 %** | **51 %** |

- **The DEM finds the missing tributaries.** In the north-west, the yellow lines run along the cyan mining scars, the streams OSM doesn't have.
- **It is weaker on the main river.** Only 38 % of OSM's Ankobra lies within 60 m of a computed channel (63 % within 150 m): the floodplain is flat, and the DEM measures the top of the forest canopy rather than the ground.
- **Channels cross the big Tarkwa pits**, because the DEM was captured around 2011–2015, before the pits took their current shape. Ignore channels inside active pits.
- **The other 27 % of water** is mostly mine-pit and isolated ponds, not rivers.

<callout icon="📌">

**Decision for Layer 6:** use **OSM for the main Ankobra channel and the DEM streams for the tributaries.** The DEM network starts a channel wherever 0.5 km² of land drains into it; raise this to 1–2 km² if it proves too dense.

</callout>

## 🧹 Notes for Layer 2 (Preprocessing)

- **Tile seams in Sentinel-2.** In true colour, the four tiles differ slightly in brightness, leaving straight lines (a vertical one on the left, a horizontal one near the bottom). Colour normalisation has to remove these, or change detection will read them as change.
- **The cloud layer misses small clouds.** The `SCL` band flagged only 0.9 % cloud or shadow, but the image clearly has scattered small clouds and their shadows, mostly in the east and south. A stricter mask (for example, growing the masked areas by a few pixels) is worth testing.
- **Landsat is coarser but cleaner** on these dates. Comparing it with Sentinel-2 means resampling to 30 m.

## 📁 Where the files are

All on disk in `~/Downloads/REMI-test-data/`:

| File | What it is |
| --- | --- |
| `data/raw/s2_2025-02-04.tif` | Sentinel-2 "after", 6 bands (blue, green, red, nir, swir16, scl), 10 m, 127 MB |
| `data/raw/s2_2024-03-31.tif` | Sentinel-2 "before", same bands, 133 MB |
| `data/raw/landsat_2024-12-21.tif` | Landsat 8 "after", 6 bands (blue, green, red, nir08, swir16, qa\_pixel), 30 m, 15 MB |
| `data/raw/landsat_2024-01-28.tif`, `landsat_2024-02-21.tif` | Landsat "before", same bands |
| `data/raw/landsat_*.tif` (5 more) | Other Landsat dates: four are 99 % empty, and 2025-01-30 is 74 % cloud |
| `data/raw/osm.gpkg` | OSM roads, rivers and water bodies |
| `data/raw/copdem_30m.tif` | Copernicus 30 m elevation map, with a 0.15° margin around the box |
| `data/derived/streams_dem.gpkg` | Streams computed from the elevation map |
| `previews/*.png` | The pictures on this page |
| `s2_earth_search.py`, `landsat_planetary.py`, `osm_download.py` | The download scripts |
| `dem_streams.py` | Downloads the elevation map and computes the streams |
| `make_previews.py` | Turns the GeoTIFFs into these previews |

The pictures on this page are also copied into the Peeka workspace, so they stay here even if the folder is moved.
