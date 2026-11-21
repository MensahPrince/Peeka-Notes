---
id: spbr0bdedd
title: "REMI M3 Labelled Data Report"
sort: 10000
version: 1
---

Progress report for **M3 (Labelled data)**: examples of mining scars and land classes for the models to learn from. Inputs are the M2 outputs on [REMI M2 Preprocessing Report](page:8u0jq0qub6); tracking is on [REMI Project Hub](page:imqxd436oq). *Started 2026-11-21.*

<callout icon="🎯">

**Approach: start from open datasets, then have a person correct them.** Labelling the whole box by hand would take weeks. Two published datasets give rough labels straight away (**weak labels**), and a script lists the places where they are most likely wrong, so the hand labelling goes where it matters.

</callout>

## Progress

| Step | What it does | Status |
| --- | --- | --- |
| 1 · Weak labels | Land classes from ESA WorldCover 2021; mining outlines from Tang & Werner (2023) | ✅ Done 2026-11-21 |
| 2 · Candidate list | Places green in 2021 but bare now, ranked, for a person to label | ✅ Done 2026-11-21 |
| 3 · Review in QGIS | Label the candidates: mining, farm, road, settlement … | ⏳ Next (you) |
| 4 · Known galamsey sites | Sites from news, NGO and government reports, to check the labels | ✅ First pass 2026-11-21 |

---

## 🗺️ Step 1 · Weak labels

**Method.** `labels.py` puts two datasets onto the exact pixel grids of the images (Sentinel-2 at 10 m, Landsat at 30 m), so they line up with every tile.

1. **ESA WorldCover 2021** (10 m, from the Planetary Computer), regrouped into REMI's classes: **forest**, **other vegetation** (farms, shrub, grass), **bare ground**, **built-up**, **water**.
1. <b>Tang & Werner (2023), <i>Global mining footprint mapped from high-resolution satellite imagery</i></b> (Zenodo, CC BY 4.0): **170 outlines** touch the box, covering **9,350 ha** inside it (8.5 %). They were drawn by hand, and include pits, tailings, ponds, waste dumps and plant.
1. <b>Maus et al. (2022), <i>Global-scale mining polygons v2</i></b> (PANGAEA, CC BY-SA 4.0), as a cross-check: drawn by hand on the **2019** Sentinel-2 cloudless mosaic, within 10 km of known mines. **28 outlines, 13,041 ha** in the box (11.8 %). The first download failed (PANGAEA's file store was down); the retry worked on 2026-11-21.

### The two sets of mining outlines compared

|  | Tang & Werner 2023 | Maus et al. 2022 | Either |
| --- | --: | --: | --: |
| Outlines touching the box | 170 | 28 |  |
| Area in the box | 9,350 ha | 13,041 ha | 14,132 ha |
| Bare in 2025 (NDVI \< 0.3) inside | 31 % | 26 % | 24 % |
| Green in 2025 (NDVI ≥ 0.6) inside | 37 % | 45 % | 46 % |
| **Share of all 2025 bare ground they cover** | 49 % | **58 %** | **60 %** |
| Candidate clearings mostly inside | 273 patches, 855 ha | 372 patches, 1,100 ha | 396 patches, 1,135 ha |

They **agree well**: 88 % of Tang & Werner's area lies inside Maus (overlap 0.58 of the combined area). Maus draws **wider corridors** along the north-west rivers, so it holds more green land, but it also covers more of today's bare ground. Tang & Werner is **tighter** around individual pits and splits big sites into parts (170 outlines against 28).

![2025-02-04: Maus et al. 2022 outlines (yellow) against Tang & Werner 2023 (magenta)](_assets/512cc26b4ef705d1bb467f74c83c5d7c.png)

<callout icon="📌">

**Decision (updated 2026-11-21): the tiles' mining band uses both sets**, as bit flags: **1** Tang & Werner only, **2** Maus only, **3** both. Anything from 1 to 3 counts as mining, and the source stays recoverable, e.g. "3" for where both agree. For a **mining-area** label (Layer 6: is this inside a known mining zone?), use **either set**. For a **scar** mask (M6 segmentation), use *either set AND bare*. Even both together miss **40 %** of today's bare ground, which is why the candidate review (step 3) matters.

</callout>

| Class (WorldCover 2021) | Share of the box | Median NDVI in 2025 |
| --- | --: | --: |
| Forest | 85\.2 % | 0\.74 |
| Other vegetation | 10\.1 % | 0\.68 |
| Bare ground | 1\.6 % | 0\.23 |
| Built-up | 2\.7 % | 0\.33 |
| Water | 0\.3 % | 0\.22 |

<callout background="yellow" icon="⚠️">

**WorldCover misses most of the bare ground REMI cares about.** Of the land that is **bare in 2025** (NDVI \< 0.3, clear of cloud), WorldCover calls **40 % "forest"** and only 22 % "bare". That's land cleared since 2021, or simply mislabelled, and it's exactly the ground REMI exists to find. **Use WorldCover for green land, not for bare ground or mining.**

</callout>

<callout background="yellow" icon="⚠️">

**The mining outlines are envelopes, not scars.** Inside them, only **31 %** of the land is bare in 2025 and **37 %** is green (NDVI ≥ 0.6): old pits growing back, and forest the outline takes in between pits. They also miss a lot: **half of all bare ground in 2025 lies outside them**, much of it along the north-west streams. So treat them as **"mining area"**, and take the scar itself as *inside an outline AND bare*.

</callout>

![2025-02-04 true colour: magenta = Tang & Werner mining outlines, cyan = candidate clearings (green in 2021, bare now)](_assets/d11edcc5f94605391e4715bd90bead26.png)

---

## 🔍 Step 2 · Candidate list

**Method.** `candidates.py` finds every patch that is **bare on 2025-02-04** (NDVI \< 0.3, clear of cloud) but that **WorldCover 2021 called forest or other vegetation**. It drops thin lines such as road edges, keeps patches of **0.5 ha** or more, and writes them as polygons, biggest first, with a description of each and an **empty `label` column** to fill in.

### Results

|  | Patches | Area |
| --- | --: | --: |
| All candidates (≥ 0.5 ha) | **639** | **1,597 ha** |
| … 2 ha or more | 179 | 1,151 ha |
| … cleared **between** the two images (mostly green on 2024-03-31) | 199 | 394 ha |
| … within 150 m of a river (OSM or DEM stream) | 475 | 1,277 ha (80 %) |
| … mostly inside a Tang & Werner mining outline | 273 | 855 ha |

**80 % of the candidate area lies within 150 m of a river.** That's the pattern of riverside galamsey, not of farming.

![The six largest candidates, 2024-03-31 (top) and 2025-02-04 (bottom), outlined in magenta](_assets/b89201669b011ce591dc326b3f79708a.png)

By eye: **#3 and #4** look like classic galamsey, scattered pits with green-tinged ponds on the streams. **#1, #2 and #6** are industrial pits at the Tarkwa mine spreading into ground that was green in March 2024 (#6 cuts a new arm into the forest). **#5** is a tailings dam wall. **None of this is confirmed**; that's step 3.

<toggle>

Remaining caveats

- **Season.** The "before" image is from the start of the rains (March) and the "after" from the dry season (February). Some farms and burned fields will be green in one and bare in the other. The review sorts these out ("farm").
- **Settlements are excluded** only where WorldCover calls them built-up. New building since 2021 will show up as candidates.
- **Cloud.** 12.7 % of the Sentinel-2 pair is cloud on one date or the other, so candidates there are missing, not absent.
- **Thresholds untuned.** NDVI \< 0.3 and 0.5 ha are first guesses; the reviewed labels will show whether to move them.

</toggle>

---

## 🧩 Labels in the tiles, and a fairer split

Each tile's **mask** file now has **4 bands**: clear, water change, **land class** and **mining**. The mining band uses **both sets of outlines** as bit flags (1 Tang & Werner only, 2 Maus only, 3 both, 255 unknown). The tile index has `mining_ha` (either set) and `mining_both_ha` columns.

<callout icon="📌">

**Decision: stratify the split on land, mining area and candidate clearings.** The random split had put **45 %** of the mapped mining area in val (the whole Tarkwa mine) and only **6 %** in test. Balancing on mining area alone then went wrong the other way: with both sets of outlines it put **every riverside galamsey corridor in train**, because the big industrial mines alone could fill val and test's share. So `tiles.py` now also balances the **candidate clearing area** (fresh bare ground, mostly riverside). It tries 20,000 seeded block assignments and keeps the one closest to 70 / 15 / 15 % on all three. Splits stay per 7.68 km block and the same for both satellites (checked).

</callout>

| Split | Sentinel-2 patches | Mining area (either set) | Share of mining | Share of candidate clearings |
| --- | --: | --: | --: | --: |
| Train | 115 | 10,368 ha | 73 % | 68 % |
| Val | 25 | 1,208 ha | 9 % | 16 % |
| Test | 27 | 2,537 ha | 18 % | 15 % |

![Sentinel-2 patches, split balanced on land, both mining outline sets and candidate clearings: riverside galamsey in every split](_assets/7ba8d95edc923393b173203e03ce6ba8.png)

**Every split now holds riverside galamsey.** **Val:** a north-west corridor and Bogoso. **Test:** another north-west corridor and the southern strip. **Train:** the Tarkwa mine, the Himan–Prestea stretch of the Ankobra (where the July 2025 raid was), and the rest. Three targets can't all be met exactly with only 25 blocks: mining comes out 73 / 9 / 18 %.

<callout background="yellow" icon="⚠️">

**Redo the split once the candidates are reviewed.** It balances on candidates that aren't confirmed yet. After step 3, balance on the patches labelled **mining** instead. `tiles.py` would need a small change to read the `label` column.

</callout>

---

## ✋ Step 3 · How to review the candidates in QGIS

1. Open `data/labels/candidates_s2.gpkg` over `data/interim/s2_2025-02-04_sr_matched.tif`, and add `s2_2024-03-31_sr.tif` underneath to flip between dates. Optionally add `mining_tang2023.gpkg` and a Google or Bing satellite layer (QuickMapServices plugin) for a sharper view.
1. **Layer → Toggle Editing**, then open the attribute table, sorted by `rank`.
1. For each patch, zoom to it (right-click → **Zoom to Feature**) and type one of these into `label`: **mining**, **farm**, **road**, **settlement**, **quarry**, **cloud/shadow**, **other**, **unsure**. Add anything useful to `note`.
1. Save edits. The top **179 patches (≥ 2 ha)** cover 72 % of the area, so start there.

Your labels stay in the file: `candidates.py` overwrites it, so don't rerun it after you start labelling without saving a copy first.

---

## 📰 Step 4 · Reported galamsey sites

**Why.** The candidates and the mining outlines are REMI's guesses and other people's maps. Reports of real mining (raids, arrests, journalists' visits) are an independent check: places where someone saw galamsey on the ground. Full citations are on [REMI Sources](page:841cvgtq57).

**Method.** Reports name villages, not coordinates. `reported_sites.py` places each report on its village as OpenStreetMap has it, and **moves it onto the Ankobra** when the report says the site is on the river. So each point means "somewhere within about **2 km**", never an exact site. It then measures how much candidate clearing and mapped mining lies within 2 km, compared with an average 2 km circle in the box (**18 ha** of candidates, **137 ha** of mining outlines).

### Results

| Report | Date | Located? | Candidates within 2 km | Mining outlines within 2 km |
| --- | --- | --- | --: | --: |
| **Himan, "Amponsah" site on the Ankobra** (MLNR: 8 changfan destroyed, 8 pumps seized) | 2025-07-10 | ✅ On the Ankobra by Himan | **46 ha** (33 patches), 2.6× average | 186 ha |
| Bogoso (Ghanaian Times: Blue Water Guards operations) | 2025-10 | Town centre | 7 ha | 39 ha |
| Prestea (2018 field survey community) | 2014–16 | Town centre | 10 ha | **193 ha** |
| Aboso, Bompieso, Huni Valley (2018 field survey communities) | 2014–16 | Town centres | 0–6 ha | 0–25 ha |
| Prestea-Hemang bridge, Awodua, Ehyeamu (JoyNews: Ankobra blocked by miners) | 2025-03-02 | ❌ Not in OSM | — | — |
| Tarkwa Bremang (MyJoyOnline: 5 changfan burned on the Ankobra) | 2025-11-21 | ❌ Not in OSM | — | — |
| Kadadwen (GhanaWeb: navy arrests 56) | unknown | ❌ Not in OSM | — | — |

![Himan on the Ankobra, where Blue Water Guards raided a site (MLNR, 2025-07-10): March 2024, February 2025, and the candidates (cyan) and mining outlines (magenta)](_assets/4735e4c09cbecd233f8bad8cd8350964.png)

<callout background="green" icon="✅">

**The one report precise enough to test agrees with the candidates.** Just north of the Himan anchor, a large galamsey field sits on a bend of the Ankobra, with pits and green-tinged ponds, and it **grows visibly between March 2024 and February 2025**. The raid came five months after the "after" image. Most of the field is already in the candidate list.

</callout>

<callout background="yellow" icon="⚠️">

**Town-level reports can't test anything.** A town centre is not a mining site: the activity is usually on the rivers around it, and built-up land is excluded from the candidates. So Bogoso, Aboso, Bompieso and Huni Valley sitting at or below the box average says nothing either way. The 2018 survey is also from 2014–16, and didn't publish its site coordinates (only district totals: **Prestea-Huni Valley 1,130 sites, Tarkwa-Nsuaem 3,648**).

</callout>

<callout icon="🙋">

**You can place the missing ones.** Ehyeamu, Awodua, the Prestea-Hemang bridge, Tarkwa Bremang and Kadadwen aren't in OpenStreetMap. If you can find them (Google Maps, local knowledge, or the JoyNews video), open `data/labels/reported_sites.gpkg` in QGIS, toggle editing, and move each point onto the site. Or send me the coordinates and I'll add them. The JoyNews report puts Ehyeamu and Awodua about **40 minutes upstream by boat** from the Ankobra estuary.

</callout>

---

## 📁 Files

All in `~/gitRepo/REMI/`. Code is in git (`github.com/MensahPrince/REMI`); `data/` is not.

| File | What it is |
| --- | --- |
| `labels.py` | Builds the weak labels. Needs the Tang & Werner shapefile in `data/raw/external/tang2023/` (downloaded 2026-11-21 from Zenodo record 7894216, 320 MB) |
| `candidates.py` | Builds the candidate list from the Sentinel-2 pair |
| `data/labels/<grid>_landclass.tif` | WorldCover classes on the `s2` or `landsat` grid: 1 forest, 2 other vegetation, 3 bare, 4 built-up, 5 water, 255 none |
| `data/labels/<grid>_mining.tif` | 1 = inside a Tang & Werner outline |
| `data/labels/mining_tang2023.gpkg` | The 170 mining outlines that touch the box, in UTM 30N |
| `data/labels/mining_maus2022.gpkg`, `<grid>_mining_maus.tif` | The 28 Maus et al. outlines that touch the box, and their mask (1 = inside). Source file: `data/raw/external/maus2022/global_mining_polygons_v2.gpkg` |
| `data/labels/candidates_s2.gpkg` | The 639 candidates with `area_ha`, `new_since_2024`, `river_m`, `road_m`, `mining_tw2023`, `new_water_ha`, `split`, `rank`, and the empty `label` and `note` |
| `previews/labels_s2_overview.png`, `candidates_s2_top6.png` | The pictures above |
| `reported_sites.py` | Places the reported sites and measures what's within 2 km. The reports themselves are listed in the script (`REPORTS`), so add new ones there |
| `data/labels/reported_sites.gpkg` | One point per report and place: date, what, source, URL, how precise, and candidate and mining area within 2 km. Unlocated reports have no geometry |
| `previews/reported_himan.png` | The Himan picture above |
