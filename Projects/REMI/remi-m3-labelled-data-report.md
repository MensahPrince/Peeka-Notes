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

**Method.** `candidates.py` finds every patch that is **bare on 2025-02-04** (NDVI \< 0.35, clear of cloud) but that **WorldCover 2021 called forest or other vegetation**. It drops 1-pixel lines such as road edges, keeps patches of **0.5 ha** or more, and writes them as polygons, biggest first, with a description of each and an **empty `label` column** to fill in.

<callout icon="📌">

**Rules loosened on 2026-11-21, before any labelling.** The first version (NDVI \< 0.30, and a filter that deleted anything under 3 pixels wide) missed **narrow riverbed workings**: in the north-west, **69 %** of the bare ground left out was cut by the width filter, and many pale valleys sat just above the cutoff, at NDVI 0.30–0.45. The filter now keeps strips down to **2 pixels (20 m)**, and the cutoff is **0.35**. That took the list from 639 to **914** patches, while the share inside the published mining maps barely moved (72 % → 68 %), so the extra patches are mostly mining, not farms.

</callout>

### Results

|  | Patches | Area |
| --- | --: | --: |
| All candidates (≥ 0.5 ha) | **914** | **2,467 ha** |
| … 2 ha or more | 248 | 1,814 ha |
| … cleared **between** the two images (mostly green on 2024-03-31) | 263 | 499 ha |
| … within 150 m of a river (OSM or DEM stream) | 687 | 2,115 ha (86 %) |
| … mostly inside a Tang & Werner mining outline | 345 | 1,250 ha |

**86 % of the candidate area lies within 150 m of a river.** That's the pattern of riverside galamsey, not of farming. *(The pictures below are from the first version; the biggest patches are the same.)*

![The six largest candidates, 2024-03-31 (top) and 2025-02-04 (bottom), outlined in magenta](_assets/b89201669b011ce591dc326b3f79708a.png)

By eye: **#3 and #4** look like classic galamsey, scattered pits with green-tinged ponds on the streams. **#1, #2 and #6** are industrial pits at the Tarkwa mine spreading into ground that was green in March 2024 (#6 cuts a new arm into the forest). **#5** is a tailings dam wall. **None of this is confirmed**; that's step 3.

<toggle>

Remaining caveats

- **Season.** The "before" image is from the start of the rains (March) and the "after" from the dry season (February). Some farms and burned fields will be green in one and bare in the other. The review sorts these out ("farm").
- **Settlements are excluded** only where WorldCover calls them built-up. New building since 2021 will show up as candidates.
- **Cloud.** 12.7 % of the Sentinel-2 pair is cloud on one date or the other, so candidates there are missing, not absent.
- **Thresholds untuned.** NDVI \< 0.35 and 0.5 ha are first guesses; the reviewed labels will show whether to move them.
- **The narrowest workings are still missed.** Many dug-out stream channels are narrower than one 10 m pixel, so each pixel mixes sand with the vegetation beside it, and only a few fall below the cutoff. That's a limit of 10 m imagery. During the review, **draw these in by hand** (see step 3).

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

**Open the ready-made project:** `~/gitRepo/REMI/remi_labelling.qgz` (double-click it, or in QGIS **Project → Open**). Everything is loaded and coloured.

![What the QGIS project shows: candidates in cyan with their rank number, Tang & Werner outlines in pink, the Ankobra in blue, a reported site as a yellow star](_assets/30388e14af3ed58d2177c5090c9ff90a.png)

**What's in it** (Layers panel, top to bottom; tick a box to show a layer):

| Group | Layer | Shown at start |
| --- | --- | --- |
| Labelling | **Candidates: label me**: cyan outline = not labelled yet; fills with a colour once labelled (red mining, green farm, grey road, orange settlement, brown quarry, white cloud, purple other, yellow unsure). Rank numbers appear when you zoom in | ✅ |
|  | **Reported galamsey sites**: yellow stars with place and date (each is only accurate to \~2 km) | ✅ |
| Mining maps | **Tang & Werner 2023** (pink) | ✅ |
|  | **Maus et al. 2022** (yellow) |  |
| Context | Rivers (OSM, blue) ✅; roads; streams from elevation; water change 2024→2025; tiles by split | Rivers only |
| Imagery | **2025-02-04 true colour** (the "after" image) ✅; **2024-03-31 true colour** (before: tick it to flip between dates); 2025 false colour (plants show red, bare ground blue-grey); **Esri World Imagery** (much sharper, needs internet, date varies, so use it to *see detail*, not to judge *when*) | 2025 only |

**How to label:**

1. Right-click **Candidates: label me** → **Open Attribute Table**. Click the `rank` column header to sort: 1 is the biggest patch.
1. Click the **pencil** (Toggle Editing) at the top of the table.
1. Click a row's number at the far left to select it, then the **magnifier** button ("Zoom map to selected rows") to jump there.
1. Look at it: flip the 2024 image on and off, and tick Esri World Imagery for a sharper view.
1. In that row's `label` cell, **pick from the dropdown**: mining, farm, road, settlement, quarry, cloud/shadow, other, unsure. Type anything useful in `note` (e.g. "pits with ponds", "cocoa farm", "new since 2024"). The other columns are locked so they can't be changed by mistake.
1. **Save often:** the **Save Edits** button (disk with pencil), or **Ctrl+S** in the table.

### 🖼️ What things look like: picture guide

Real places from our own 2025 image, exactly as they look in QGIS. Each square is **900 m × 900 m**; the white bar is 200 m.

![What things look like in our 2025 image: galamsey mining, industrial mining, quarry, road, river, settlement, cloud and shadow, and a stream worked by galamsey (each square 900 m across)](_assets/f5fbae0137eac127a5e0575c5714d9dd.png)

|  | What to look for | Label |
| --- | --- | --- |
| **Galamsey** (top left) | Cream sand heaps, holes, **green or brown ponds**, ragged shape, on a stream | **mining** |
| **Industrial mine** (Tarkwa) | **Huge** terraced pit walls like giant steps, wide pale haul roads | **mining** + note "industrial pit" |
| **Tailings dam** (part of an industrial mine) | A **huge square or rounded pond** of brown or grey mine waste, inside **high white earth walls** with a road along the top. Tens of hectares. Example: **rank 4** | **mining** + note "tailings dam" |
| **Quarry** | **One compact** bright white or grey rock face cut into a hill, **next to a road**. Small next to a mine | **quarry** |
| **Road** | **Thin line of the same width all along**, smooth curves, runs a long way and **joins places**. Grey (tarmac) or **orange** (dirt) | **road** |
| **River** | **Winding** band whose width changes, with trees along the banks. ⚠️ In this box the Ankobra is so silted and mined that it often looks like **pale sand**, not water. Tick **Rivers (OSM)** in QGIS to see where it runs | (not a candidate itself; mining *on* it is **mining**) |
| **Settlement** (Prestea) | Grainy speckle of **tiny bright and grey squares** (roofs) with a grid of streets | **settlement** |
| **Cloud + shadow** | Bright white **fuzzy** puff; its shadow is a darker patch of the same shape nearby | **cloud/shadow** |
| **Stream worked by galamsey** | A **thin sandy strip following a valley**, wider in places, with pits. Easy to confuse with a road: a road keeps the **same width** and **goes somewhere**; a mined stream **wiggles with the valley** and has blobs | **mining** |

<callout icon="💡">

**Everything that belongs to a gold mine is "mining"**: the pit, waste rock dumps, **tailings dams**, haul roads inside the mine, processing plants. **"Quarry" is only for small building-stone or gravel pits.** In the top 30, the **industrial mines around Tarkwa** (south of the box) are ranks **2, 3, 4, 5, 7, 13, 18, 19 and 25** (pits, waste dumps, a tailings dam). **All the others**, including rank 1, are along the rivers in the **north** of the box: **galamsey**. A quick hint: if the patch is near the bottom of the map, think industrial; if it's in the north on a river, think galamsey.

**Road or mined stream?** Ask two questions. *Does it keep the same width?* Roads do; mined streams bulge and narrow. *Does it connect two places?* Roads do; mined streams follow the valley bottom and end in blobs of sand. Tick **Roads (OSM)** or **Rivers (OSM)** in QGIS: if an OSM road runs along it, it's a road.

</callout>

<callout icon="🪨">

**Fun fact: OpenStreetMap makes the same mistake.** It tags the whole Gold Fields **Tarkwa mine** as a "quarry", and calls a galamsey-worked stretch of river near Prestea a "quarry" too. That's why REMI defines the labels itself.

</callout>

### 🔎 How to tell mining from a farm (and the rest)

<callout icon="🎨">

**Ignore the pink and yellow outlines when you decide.** The published mining maps are rough envelopes: they take in farms, forest and villages around the mining. Judge **only what you see inside the cyan patch**. A farm inside a yellow outline is still a farm. Those cases are worth finding, because they show where the published maps are wrong.

</callout>

<columns>

<column>

#### ⛏️ Mining

- **Messy shape:** ragged edges, fingers that follow a stream
- **Busy surface:** white or cream sand heaps, holes, mounds
- **Ponds** that are green, turquoise or brown
- **On or next to a river or stream**
- Often **green forest in 2024, dug up in 2025**

</column>

<column>

#### 🌾 Farm

- **Neat shape:** square or rectangular, straight edges and corners
- **Even surface:** one flat colour, brown or tan, **dark grey if burnt** (February is burning season)
- **No** pits, mounds or ponds
- **Near a village or road**, in a patchwork of other rectangles
- **Small:** usually under 2 ha

</column>

</columns>

<callout background="yellow" icon="💡">

**Clear farms will be rare in this list.** Most farming here is cocoa, oil palm and plantain, which stay **green** all year, so they almost never become candidates (candidates are bare ground). Expect mostly **mining, roads, villages and building work**.

</callout>

| Label | What it looks like |
| --- | --- |
| **mining** | **All gold mining**, illegal galamsey and big legal mines alike (legality comes later, from the concession data). Galamsey: pits, sand heaps, coloured ponds, ragged shape, along a stream. Industrial (e.g. the Tarkwa mine): huge terraced pits, wide haul roads, tailings dams; add "industrial pit" in `note` |
| **farm** | Neat rectangle, even brown, tan or burnt surface, near a village, no ponds |
| **road** | Long, thin, steady-width line joining places |
| **settlement** | Roofs: small bright or grey squares packed together, with streets. Also new building plots at a village edge |
| **quarry** | Stone, gravel or sand dug **for building**: usually small, by a road or town. ⚠️ **Not** the big terraced gold pits near Tarkwa: those are **mining** (note "industrial pit") |
| **cloud/shadow** | White puff, or a dark patch beside one |
| **other** | You know what it is but it fits nothing above: say what in `note` |
| **unsure** | You can't tell: say why in `note` |

**Examples from our images.** Each pair is the same 900 m square in March 2024 and February 2025; cyan is the candidate patch.

![Top row: clear mining (ranks 43, 27, 58), 2024 then 2025. Bottom row: puzzling cases away from rivers: orange excavation (ranks 103, 390) and a village-edge strip (rank 644)](_assets/1b5496e69c426ab12ea73ca0679e19d7.png)

- **Top row, ranks 43, 27, 58: clear mining.** Cream sand, holes and ponds, right on a stream, and spreading between the two dates.
- **Bottom row: the puzzling ones.** These were picked as "away from rivers and outside both mining maps", expecting farms, but they aren't. **Ranks 103 and 390** are **orange** dug ground with fingers into the forest: freshly exposed laterite soil, so excavation, possibly mining away from a river. **Rank 644** is a small strip at a **village edge**: probably building work (settlement). If you can't decide on cases like 103, pick **unsure** and note what you see, e.g. "orange excavation, no ponds".

**When the 10 m image is too blurry**, tick **Esri World Imagery**. A farm shows crop rows or a clean cleared field; mining shows a cratered "moonscape" with ponds. **Esri's photo is from an unknown date** (not a sharper version of our 2024 or 2025 images), so use it to see *what* a thing is, and our two dates to see *when* it changed. If Esri and our 2025 image disagree, pick **unsure** and say so in `note`. To find Esri's date for a spot, use **Esri World Imagery Wayback** (livingatlas.arcgis.com/wayback).

**Practice first:** look at ranks **43, 27, 58** (mining), then **103** (unsure: orange excavation) in QGIS before starting from rank 1.

**Tips**

- **Start with ranks 1–248** (2 ha or more): they're 74 % of the candidate area. Labelling 50 already tells us a lot, so send word whenever you've done a batch.
- **When unsure, pick "unsure"** and write why in `note`. A wrong "mining" does more harm than an honest "unsure".
- **Mining with no cyan outline?** Narrow riverbed workings are often missed (see the caveats above). With editing on, use **Add Polygon Feature** (Ctrl+.) on the candidates layer, draw around it, finish with a right-click, and pick **mining** in the form. That's very useful: it shows what the rules miss.
- **Your labels are safe:** `candidates.py` now refuses to overwrite a file that has any labels in it (only with `--force`, which keeps a dated backup first). Rebuilding the project (`make_qgis_project.py`) never touches your labels.

---

## 📰 Step 4 · Reported galamsey sites

**Why.** The candidates and the mining outlines are REMI's guesses and other people's maps. Reports of real mining (raids, arrests, journalists' visits) are an independent check: places where someone saw galamsey on the ground. Full citations are on [REMI Sources](page:841cvgtq57).

**Method.** Reports name villages, not coordinates. `reported_sites.py` places each report on its village as OpenStreetMap has it, and **moves it onto the Ankobra** when the report says the site is on the river. So each point means "somewhere within about **2 km**", never an exact site. It then measures how much candidate clearing and mapped mining lies within 2 km, compared with an average 2 km circle in the box (**28 ha** of candidates, **137 ha** of mining outlines).

### Results

| Report | Date | Located? | Candidates within 2 km | Mining outlines within 2 km |
| --- | --- | --- | --: | --: |
| **Himan, "Amponsah" site on the Ankobra** (MLNR: 8 changfan destroyed, 8 pumps seized) | 2025-07-10 | ✅ On the Ankobra by Himan | **81 ha** (43 patches), 2.9× average *(46 ha, 2.6×, with the first candidate rules)* | 186 ha |
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
| `remi_labelling.qgz`, `make_qgis_project.py` | The QGIS project for labelling, and the script that builds it (`QT_QPA_PLATFORM=offscreen /usr/bin/python3 make_qgis_project.py`; it uses the system Python, which has QGIS, not `.venv`). The project isn't in git: rebuild it on another machine |
| `data/labels/candidates_s2.gpkg` | The 914 candidates with `area_ha`, `new_since_2024`, `river_m`, `road_m`, `mining_tw2023`, `new_water_ha`, `split`, `rank`, and the empty `label` and `note` |
| `previews/labels_s2_overview.png`, `candidates_s2_top6.png` | The pictures above |
| `reported_sites.py` | Places the reported sites and measures what's within 2 km. The reports themselves are listed in the script (`REPORTS`), so add new ones there |
| `data/labels/reported_sites.gpkg` | One point per report and place: date, what, source, URL, how precise, and candidate and mining area within 2 km. Unlocated reports have no geometry |
| `previews/reported_himan.png` | The Himan picture above |
