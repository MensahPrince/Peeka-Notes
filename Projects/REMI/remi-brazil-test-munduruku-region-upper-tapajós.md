---
id: 7q6zbf0s6f
title: "REMI Brazil Test: Munduruku Region, Upper Tapajós"
sort: 19000
version: 1
---

First real test of REMI outside Ghana: Prestea's models, unchanged, on a 33 km box in the upper Tapajós garimpo region of Pará, centred on **-7.136923, -57.174711**. How to set up an area is on [REMI Areas: Running the Pipeline on a New Place](page:fpk4rpe5tv). *Run 2026-11-27.*

<callout icon="🎯">

**The change model transfers well on the clear cases.** On labels built the same way as in Prestea, it finds 68 % of new-mining chips with **no false alarms** (precision 1.00, AP 0.979). Most of its "extra" flags look like **new garimpo the published maps don't have**, and most misses are real mining just under a strict threshold. The labels are weaker than Prestea's, and the visual check is a small sample, so this is promising, not proven.

</callout>

## Setup

|  |  |
| --- | --- |
| Box | \-57.325, -7.287 to -57.025, -6.987 (0.3°, about 33 × 33 km), UTM 21S (EPSG:32721) |
| Dates | **2025-08-06 → 2026-08-06**: same day one year apart, dry season, 0 % cloud on both tiles (21MVM, 21MVN) |
| Land cover (WorldCover 2021) | forest 96.5 %, other vegetation 3.2 %, bare 0.2 %, water 0.2 % |
| Mining maps | Maus et al. 2022: 8 outlines, **21 % of the box**. Tang & Werner 2023: 19 outlines, 0.7 % |
| OSM | 8 rivers, 41 roads, no ponds |
| Models | Prestea's M4 and M5, not retrained |
| Run time | about 15 minutes, mostly downloads |

Preprocessing behaved normally: seams were small (blue 0.0008 → 0.0001), and IR-MAD found unchanged ground across a wide brightness range. It didn't lean on water, as it seemed to at Itaituba.

## Labels here

As in Prestea, a chip is **new mining** if at least 0.25 ha went from green to bare inside a mining outline, and **no change** if at most 2 pixels changed. The difference: no reviewed candidates exist here, so "mining" means "inside a published outline" only. Result: **84 new-mining chips** and 42,315 no-change chips; 865 chips are left out.

## How the change model did

At Prestea's threshold (0.972):

|  | Value |
| --- | --: |
| Precision | **1.00** (57 of 57 flags with a label are right; 0 of 42,315 no-change chips flagged) |
| Recall | 0\.68 (27 labelled chips missed) |
| F1 | 0\.81 |
| Average precision | **0.979** (NDVI-drop rule: 0.899) |
| All flags | 90 chips, 230 ha |
| … on labelled new mining | 57 |
| … on clear change outside every mining outline | 25 |
| … on small change (under 0.25 ha) | 8 |

The threshold was chosen on Prestea's validation set, where every positive was clear-cut. Here it's stricter than it needs to be:

| Threshold | Precision | Recall | F1 | Chips flagged |
| --- | --: | --: | --: | --: |
| 0\.5 | 0\.96 | 0\.88 | 0\.92 | 169 |
| 0\.7 | 0\.97 | 0\.86 | 0\.91 | 149 |
| 0\.9 | 0\.99 | 0\.80 | 0\.88 | 122 |
| **0.972 (Prestea)** | **1.00** | **0.68** | **0.81** | **90** |

<callout background="yellow" icon="⚖️">

**Don't pick a new threshold from this table.** Choosing it on the same chips it's scored on would flatter it. A fair choice needs a separate validation set here, ideally reviewed sites (see next steps).

</callout>

## What the flags look like

![Munduruku region, Aug 2025 → Aug 2026: random samples of flags that match a label (top), flags outside the mining maps (middle), and misses (bottom). 480 m crops; yellow = the 160 m chip.](_assets/99a6ae8ed4e00b9ca23c18fdf8e1d2a8.png)

Four random chips per group, judged by eye at 10 m (Claude's reading, not checked on the ground):

- **Flags with a label (top):** all mining spreading: white sand, a silted river channel, turquoise ponds.
- **Flags outside the maps (middle):** three look like **new garimpo** (a fresh pit in forest, sand spreading, a turquoise pond); the fourth is sand spreading into pink cleared land, probably mining too. The maps are from about 2019–2021, so new workings outside them are expected.
- **Misses (bottom):** also real mining: stream channels being dug, pits beside the river. They scored 0.37–0.88, under the strict threshold.

![Chips flagged as new mining, 2025-08-06 → 2026-08-06, Munduruku box (33 km). Your point is at the centre.](_assets/7593cc3c5d7987b39c4710913facec37.png)

The flags cluster in the **south and south-east** of the box.

## Around your point

`identify.py -7.136923 -57.174711`: the 160 m square at the point itself is **forest** (NDVI 0.89, no change, outside both maps). Nearby:

| Distance | Flagged chips | Area |
| --- | --: | --: |
| within 2 km | 0 | 0 ha |
| within 5 km | 2 | 5 ha |
| within 10 km | 12 | 31 ha |

The two nearest, **2.3 km and 2.9 km north-north-east** (-7.11939, -57.16355 and -7.11215, -57.16500): new clearings of 0.4–0.5 ha beside a river, inside the Maus et al. mining area, 13–14 km from the nearest mapped road.

## What doesn't transfer: the land model

The M4 land model calls the first of those two squares **built-up (47 %)**, so `identify.py` adds "could be other clearing, e.g. building". That's misleading here, 13 km from any road: it's far more likely a garimpo pit or camp. In Ghana, galamsey and villages are close neighbours, and the model learned to call bright mixed ground built-up. **Its land classes shouldn't be trusted in Brazil** until it's tested on labels from there.

## Next steps for Brazil

1. **Review a sample of chips here**, as in M3: the 25 flags outside the maps, the 27 misses, and a random set of no-change chips. That gives honest labels, a validation set for the threshold, and a test of the land model.
1. **Add a second pair** (for example 2024-08-01 → 2026-08-06) to see two years of change.
1. **Add the Indigenous land boundaries** (FUNAI) as the overlay that concession data is in Ghana: under Brazilian law, mining isn't permitted on Indigenous land, so "inside a TI" changes what an alert means.
1. Then decide whether to fine-tune the models on Brazilian labels.

## Files

`areas/munduruku.json`; data in `data/areas/munduruku/` (raw images, interim, labels, tiles, `models/change_prob_*.tif`, `models/change_transfer.json`); pictures in `previews/munduruku/`. Score again with `REMI_AREA=munduruku .venv/bin/python change.py transfer`.
