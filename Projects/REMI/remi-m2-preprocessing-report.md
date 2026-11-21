---
id: 8u0jq0qub6
title: "REMI M2 Preprocessing Report"
sort: 7000
version: 1
---

Progress report for **M2 (Preprocessing)**: making the before/after images clean and comparable before any model sees them. Inputs are the images on [REMI Test Imagery](page:epzdv5qm9v); tracking is on [REMI Project Hub](page:imqxd436oq). *Started 2026-10-21.*

<callout icon="📌">

**Decision: don't use Sentinel-2's built-in cloud layer (SCL) for REMI.** It missed most of the small clouds, and it labelled the bright tailings at the Tarkwa mine as cloud, which would hide exactly the bare ground galamsey leaves behind. Sentinel-2 cloud masks now come from **omnicloudmask** only. Landsat keeps its own `qa_pixel` flags.

</callout>

<callout icon="📌">

**Decision: trust new water away from the rivers; treat change on the main river as noise.** New ponds off the river channels are the useful mining signal. Water "appearing" or "disappearing" along the Ankobra itself mostly reflects river width, silt and 30 m mixed pixels, not mining.

</callout>

## Progress

| Step | What it does | Status |
| --- | --- | --- |
| 1 · Cloud mask | Mark cloud and shadow pixels so they're never read as land change | ✅ Done 2026-10-21 |
| 2 · Water mask | MNDWI per date, using the cloud masks; maps of new water between dates | ✅ Done 2026-10-21 |
| 3 · Match brightness | Remove the Sentinel-2 tile seams, and match each "after" image to its "before" image on unchanged ground | ✅ Done 2026-10-27 |
| 4 · Tiles | Cut aligned before/after pairs into 256 × 256 patches for the models | ✅ Done 2026-11-21 |

---

## ☁️ Step 1 · Cloud mask

**Method.** `cloud_mask.py` runs omnicloudmask (an AI cloud detector) on the red, green and near-infrared bands at 20 m, which is about 4× faster on this laptop's CPU and within its 10–50 m range. It then widens every cloud and shadow by **50 m** to catch soft edges. Each image gets a mask on its own pixel grid: **0** clear, **1** cloud or shadow, **255** no data. All five images took about 30 seconds.

### Results

| Image | Built-in flags (SCL or QA) | omnicloudmask | Final mask (after widening) |
| --- | --: | --: | --: |
| Sentinel-2 · 2024-03-31 (before) | 3\.3 % | 1\.5 % | **2.4 %** |
| Sentinel-2 · 2025-02-04 (after) | 0\.9 % | 7\.2 % (4.8 thick, 2.5 shadow) | **10.8 %** |
| Landsat · 2024-01-28 (before) | 3\.0 % | — | **4.9 %** |
| Landsat · 2024-02-21 (before) | 0\.3 % | — | **0.4 %** |
| Landsat · 2024-12-21 (after) | 0\.0 % | — | **0.0 %** |

### Full view

Masked areas are tinted **magenta**. The mining scars along the rivers in the north-west stay unmasked on both dates, as they should.

<columns>

<column>

![Before · 2024-03-31 · cloud mask (magenta) over true colour](_assets/784314214e25afd79f75e750a815f9c9.png)

</column>

<column>

![After · 2025-02-04 · cloud mask (magenta) over true colour](_assets/fc12bf1922c265c536052d2b2e807d05.png)

</column>

</columns>

### Zoom: south-east quarter at full 10 m resolution

The area with the most small clouds, and the edge of the Tarkwa mine (bottom left of each view).

<columns>

<column>

![Before · 2024-03-31 · zoom on the south-east quarter, full 10 m resolution](_assets/2489725763e11e2a5638aa30944a55de.png)

</column>

<column>

![After · 2025-02-04 · zoom on the south-east quarter: small clouds caught, Tarkwa tailings (bottom left) left clear](_assets/ec50d58a1cdb253db2281a164757ed75.png)

</column>

</columns>

### Why SCL was dropped

<columns>

<column>

#### ❌ It misses real clouds

On 2025-02-04, SCL flagged **0.9 %** of the box. omnicloudmask found **7.2 %**: scattered small clouds and their shadows across the south-east, clearly visible in the zoom. **6.5 %** of the box was cloud that SCL missed.

</column>

<column>

#### ❌ It hides mines

At two spots on the Tarkwa tailings, SCL called **56 %** and **87 %** of the pixels "medium-probability cloud", where the image shows no cloud and omnicloudmask calls them clear. On 2024-03-31, **2.8 %** of the box was SCL "cloud" that omnicloudmask calls clear.

</column>

</columns>

<callout background="yellow" icon="⚠️">

**Why this matters for REMI.** Fresh mining scars and tailings are bright, bare ground, which is what SCL tends to mistake for cloud. A mask that hides bright ground would hide galamsey from every later layer, and REMI would never see the change it exists to find.

</callout>

<toggle>

Remaining caveats

- **Thin haze.** omnicloudmask reported 0 % thin cloud on both dates. The images look clear, but hazy days (common in the Harmattan, Dec–Feb) should be checked by eye before use.
- **Widening costs ground.** The 50 m buffer raises the masked area on 2025-02-04 by half (7.2 % → 10.8 %). That's a safe default; it can be reduced if too much land is lost.
- **Landsat QA hasn't been checked by eye.** Its flags are used as they come. The 2024-12-21 image looked clear, matching its 0 %, but the masks for the two "before" dates haven't been viewed or compared with omnicloudmask yet.

</toggle>

---

## 💧 Step 2 · Water mask

**Method.** `water_mask.py` computes the **MNDWI** water index, `(green − swir16) / (green + swir16)`, and calls a pixel water where it is above **0**. Cloud pixels from step 1 are marked "unknown" rather than land, so a cloud can never look like water disappearing. Water patches under 5 pixels are dropped as noise (0.05 ha at 10 m, 0.45 ha at 30 m). Each before/after pair then gets a change map: **water both dates**, **new water**, **water lost**, or **cloud on either date**. Both pairs ran in under 4 seconds.

### Results

| Pair | Water before | Water after | Both dates | 🔴 New | 🟡 Lost | Hidden by cloud |
| --- | --: | --: | --: | --: | --: | --: |
| Sentinel-2 · 2024-03-31 → 2025-02-04 | 813 ha | 432 ha | 279 ha | 144 ha | 480 ha | 14,024 ha (12.7 %) |
| Landsat · 2024-02-21 → 2024-12-21 | 370 ha | 327 ha | 168 ha | 159 ha | 201 ha | 489 ha (0.4 %) |

### Change maps

**Blue** water on both dates · **red** new water · **yellow** water lost · **grey** cloud on either date.

<columns>

<column>

![Sentinel-2 water change, 2024-03-31 → 2025-02-04: blue both dates, red new, yellow lost, grey cloud](_assets/53cf3066a707b07aaf428e6e25935961.png)

</column>

<column>

![Landsat water change, 2024-02-21 → 2024-12-21 (both dry season): blue both dates, red new, yellow lost, grey cloud](_assets/e719056ae18849160362ef29172e3237.png)

</column>

</columns>

![Zoom on the north-west mining corridor at 10 m: red new ponds in the scarred ground, yellow lost water along the river channels](_assets/0e46fb5a59c69dc5f05e7e7c98ab485c.png)

### What the maps show

<columns>

<column>

#### ✅ Detection is clean

Almost nothing is flagged as water in the forest or under cloud shadow. Water and its changes sit where they should: the **mining corridors along the rivers in the north-west**, and the ponds at the **Tarkwa mine**.

**New water off the rivers looks like the real signal:** small red ponds inside the scarred ground in the zoom, and isolated new ponds in the Landsat map. In the Landsat pair, **71 %** of new water lies away from any known channel.

</column>

<column>

#### ⚠️ River-channel change is mostly noise

The yellow "lost" water largely traces the **river channels themselves**. In the Sentinel-2 pair this is mostly the season: rivers were wider at the start of the rains in March than in dry February. But it happens in the **dry-season Landsat pair too**: the whole southern Ankobra channel shows as "lost". At 30 m, a river that narrow is mostly mixed pixels, and the Ankobra is heavily silted from upstream mining, so small changes in width or muddiness flip it in and out of "water".

</column>

</columns>

| Pair | New water near any channel (60 m) | Lost water near any channel (60 m) |
| --- | --: | --: |
| Sentinel-2 | 45 % | 51 % |
| Landsat | 29 % | 58 % |

"Any channel" means OSM rivers or the streams computed from the elevation map.

<callout icon="💡">

**For change detection (M5):** compare **same-season** dates where possible, as the Landsat pair does. The Sentinel-2 pair's March "before" date was forced by cloud (see the [REMI Data Download Guide](page:sf7hcbolae)), and it roughly doubles the water on the "before" side. Treat **new water away from the main channel** as a mining clue, and give little weight to change along the river itself.

</callout>

<toggle>

Remaining caveats

- **Threshold not tuned.** MNDWI > 0 is the textbook default. Very muddy water can fall below it, which may be part of why the silted Ankobra flickers. Tuning needs labelled water examples (M3).
- **No ground truth yet.** "Clean" means it looks right by eye. Whether the red ponds are new mining pits has not been checked.
- **Clouds hide 12.7 % of the Sentinel-2 pair.** Any change under those patches is unknown, not absent.
- **Sentinel-2 swir16 is natively 20 m**, resampled to 10 m, so water edges are softer than the 10 m grid suggests.

</toggle>

---

## 🎚️ Step 3 · Match brightness

**Why.** Change detection compares pixel values between dates. Anything that shifts those values without changing the ground looks like change: a hazier day, a different satellite, or a different haze correction on each Sentinel-2 tile. This step takes those out, so that what is left is change on the ground.

**Method.** `normalise.py` works in two parts, and both satellites' values are first converted to surface reflectance (0–1).

1. **Seams (Sentinel-2 only).** Each image is a mosaic of four tiles (30NWL, 30NXL, 30NWM, 30NXM). Sentinel-2's processor estimates the haze separately for each tile, so neighbouring tiles disagree about the same ground. The tiles overlap by several kilometres inside the box, so the script downloads each tile on its own and fits it to the reference tile, **30NXM**, band by band, using the pixels both tiles see. The fit is a straight line (gain and offset).
1. **Dates.** **IR-MAD**, a standard statistical method, finds pixels whose values didn't change between the two dates: **28,323** for Sentinel-2 and **2,935** for Landsat. The "after" image is matched to the "before" image band by band, using two thirds of those pixels. The last third is held back to check the result.

<callout icon="📌">

**Decision: 30NXM is the reference tile.** It covers 57 % of the box and agrees best with Landsat. Over dense forest, Landsat puts blue at about **0.028**. On 2025-02-04, 30NXM reads **0.022**, while the two west tiles read **0.005–0.012**, which means their haze correction overshot.

</callout>

<callout icon="📌">

**Decision: in the visible bands, the "after" image is only shifted, not stretched.** **93 %** of the unchanged ground is forest, so in blue, green and red it covers only the dark end of the range (red 0.04–0.07, while bare ground reaches 0.21). IR-MAD finds almost no unchanged bare or urban ground: 3 pixels at the 95 % cutoff and 38 at 50 %. On the Sentinel-2 pair, bright ground really did change between dry February and the first rains in March. A slope fitted on forest alone would rescale the bright bare ground that galamsey leaves behind, with nothing to check it against. On the first run, that gave a red gain of 1.30, which would have dimmed a 0.20 pixel to 0.16. So a band gets a slope only where the unchanged ground covers at least half its brightness range. In practice that's **NIR and SWIR**; the visible bands get the median shift.

</callout>

### Seams

<columns>

<column>

![Sentinel-2 · 2025-02-04 as downloaded: the west tiles are darker (magenta lines are tile edges)](_assets/867fb9aa6bbc576f81b4d411e7507306.png)

</column>

<column>

![Sentinel-2 · 2025-02-04 with seams removed: every tile matched to 30NXM](_assets/b1c3982be0dbb2eaf87112a262ec3602.png)

</column>

</columns>

The vertical seam on the left and the brighter strip at the bottom right are gone. Numbers are the median difference from 30NXM on the same pixels, before → after the fix:

| Tile | 2025-02-04 blue | 2025-02-04 NIR | 2024-03-31 blue | 2024-03-31 NIR |
| --- | --: | --: | --: | --: |
| 30NWL | 0\.0174 → **0.0015** | 0\.0059 → **0.0018** | 0\.0120 → **0.0008** | 0\.0064 → **0.0014** |
| 30NWM | 0\.0149 → **0.0010** | 0\.0046 → **0.0008** | 0\.0075 → **0.0004** | 0\.0029 → **0.0008** |
| 30NXL | 0\.0106 → **0.0022** | 0\.0022 → **0.0018** | 0\.0005 → 0.0008 | 0\.0012 → 0.0026 |

The worst seam, blue on 2025-02-04, is about **10× smaller**. 30NXL on 2024-03-31 hardly differed from 30NXM to begin with, so the fit left it slightly worse (NIR off by 0.0026 instead of 0.0012, under 1 %). It covers 9 % of the box.

### Dates

Held-out unchanged ground, median difference between the dates, before → after matching:

| Band | Sentinel-2 · how matched | Sentinel-2 | Landsat · how matched | Landsat |
| --- | --- | --: | --- | --: |
| Blue | shift −0.011 | 0\.0113 → **0.0016** | shift +0.002 | 0\.0015 → **0.0006** |
| Green | shift −0.010 | 0\.0102 → **0.0015** | shift +0.001 | 0\.0010 → **0.0010** |
| Red | shift +0.005 | 0\.0046 → **0.0018** | shift −0.001 | 0\.0018 → **0.0013** |
| NIR | gain 0.92, offset −0.023 | 0\.0499 → **0.0086** | gain 0.91, offset +0.003 | 0\.0266 → **0.0039** |
| SWIR | gain 1.16, offset −0.042 | 0\.0068 → **0.0021** | gain 0.86, offset +0.008 | 0\.0174 → **0.0021** |

The leftover bias is **0.0004 or less** in every band of both pairs. The biggest correction was **NIR on the Sentinel-2 pair**: unchanged forest read 0.05 lower in February than in March, which on its own would make the whole box look as if it had lost vegetation.

<columns>

<column>

![Before · 2024-03-31 · false colour (NIR-red-green)](_assets/ea103df3dbb7f21e0724fc88e7c2e029.png)

</column>

<column>

![After · 2025-02-04 · not matched (same colour scale as before)](_assets/6192ba3c1cc5f4cc1f03a69abc7bd01d.png)

</column>

<column>

![After · 2025-02-04 · matched to before](_assets/0318bff9ecd0dbc02f88e83041bd3e89.png)

</column>

</columns>

All three use the **same colour scale**, taken from the "before" image. Before matching, the February forest looks dull red next to March. After matching, it reads like March, and the mining corridors, towns and the Tarkwa mine stand out the same way on both dates.

#### Zoom: north-west mining corridor at full 10 m resolution

<columns>

<column>

![Zoom, north-west mining corridor · before · true colour, 10 m](_assets/3d0bd517897efc8653ccb6585c74043a.png)

</column>

<column>

![Zoom · after, not matched](_assets/f34d8978aed1605b423da9a0f382734f.png)

</column>

<column>

![Zoom · after, matched](_assets/e73d72a7a56f111c55194efb6650fe6a.png)

</column>

</columns>

Unmatched, the February forest looks brown next to March, so a model comparing them would see change everywhere. Matched, the forest looks alike on both dates, and what differs is the bare ground. The white mining scars along the streams look **larger in February**. That's the kind of change M5 should pick up (not yet measured).

### What it does to change detection

<columns>

<column>

![NDVI change, not matched: almost the whole box looks less green](_assets/b14608931a784e6c2a315911f9259137.png)

</column>

<column>

![NDVI change, matched: the false drop is gone; the March green-up patch and the river corridors remain](_assets/30ec23c870264783545caa54c42f7d29.png)

</column>

</columns>

NDVI measures greenness. Orange = less green in February, green = greener, grey = cloud on either date.

|  | Not matched | Matched |
| --- | --: | --: |
| Median NDVI change | −0.059 | **+0.001** |
| Box that looks clearly less green (\< −0.1) | 26\.2 % | **8.0 %** |
| Box that looks clearly greener (> +0.1) | 4\.6 % | 11\.8 % |

Before matching, a quarter of the box looks like it lost vegetation, which would bury any real clearing. After matching, the loss that remains is concentrated in the **river corridors in the north-west**, where the mining is. The large patch left of centre still shows as less green. That's the **March green-up** spotted on [REMI Test Imagery](page:epzdv5qm9v): real seasonal change in the vegetation, not haze, so matching correctly leaves it in.

<callout icon="💡">

**The March green-up can't be normalised away.** It's real change in the plants, and only a same-season "before" date removes it. The plan for M5 stands: prefer same-season pairs like the Landsat one, and treat broad, smooth greenness loss as season rather than mining.

</callout>

#### Landsat pair (both dry season)

<columns>

<column>

![Landsat · before 2024-02-21 · false colour](_assets/ec27e7139cb8233c46c91a3cf2939c1c.png)

</column>

<column>

![Landsat · after 2024-12-21 · not matched](_assets/98d059d5f147fec066e3be3b04926bf5.png)

</column>

<column>

![Landsat · after 2024-12-21 · matched](_assets/ceff690aaa6e309401b8da03ed44ff65.png)

</column>

</columns>

The Landsat dates already matched closely in the visible bands. The correction is mainly NIR and SWIR (gains 0.91 and 0.86).

<callout background="yellow" icon="⚠️">

**The water areas from step 2 shift after matching.** With the same MNDWI > 0 rule (before removing specks), water on the "after" date drops from **471 to 423 ha** for Sentinel-2 and from **450 to 378 ha** for Landsat, because matching changes SWIR. Step 2 ran on the unmatched images, so its change maps should be rebuilt on the matched ones before M5 uses them.

**✅ Rebuilt 2026-11-21.** `water_mask.py` now reads `<before>_sr` and `<after>_sr_matched`. New results (old in brackets):

| Pair | Water before | Water after | Both dates | 🔴 New | 🟡 Lost |
| --- | --: | --: | --: | --: | --: |
| Sentinel-2 | 801 ha (813) | 391 ha (432) | 246 (279) | **138** (144) | 502 (480) |
| Landsat | 370 ha (370) | 282 ha (327) | 152 (168) | **130** (159) | 217 (201) |

The "before" water barely moves: Landsat's isn't matched, and at a cutoff of 0, MNDWI only asks whether green is above SWIR, so converting to reflectance changes nothing. The "after" water shrinks because matching lifted SWIR. New water drops a little on both pairs. The change maps in the Step 2 section above are the old ones.

</callout>

<toggle>

Remaining caveats

- **Visible bands on bright ground.** The seam fits on the same ground show that haze differences also stretch values, not just shift them (tile gains 0.84–1.05). With a shift only, bright bare ground in the "after" image may still be off by up to about **0.02** in blue to red. Fixing that needs unchanged bright ground: roofs, roads or old tailings, ideally marked by hand in M3.
- **No ground truth.** "Unchanged ground" is IR-MAD's call. The held-out check shows the matching is consistent, not that it's right.
- **Absolute level isn't checked.** Everything is matched to 30NXM and to the "before" date, which is what change detection needs. Whether 30NXM's own haze correction is right is untested; it only agrees with Landsat best of the four.
- **Clouds.** Only pixels clear on both dates were used. Cloud pixels in the matched images still carry the date-wide correction and stay flagged by the step 1 masks.
- **Earth Search metadata is wrong about the offset.** It says to subtract 0.1, but these files already have it removed. Without the correction, the darkest blue pixel is 0.017 and forest matches Landsat; with it, most of the image would go negative. Re-check this if the download source changes.

</toggle>

---

## 🧩 Step 4 · Tiles

**Why.** The models (M4–M6) take fixed-size inputs, and change detection needs the "before" and "after" of exactly the same ground, side by side.

**Method.** `tiles.py` cuts each pair into **256 × 256 pixel patches**: **2.56 km** for Sentinel-2 and **7.68 km** for Landsat. Step 3 already wrote both dates on one grid, so a patch is the same window in every file and nothing is resampled. A check confirmed that every patch's bands are identical to that window of the source image. Each kept patch gets two GeoTIFFs:

- **img**: 10 bands. The "before" image (blue, green, red, NIR, SWIR), then the matched "after" image in the same order, as reflectance × 10000 (0 = no data).
- **mask**: 2 bands. **clear** (1 = data and cloud-free on both dates) and the rebuilt **water change** from step 2.

Each pair also gets an **index** GeoPackage: one square per patch, with its split, clear share, hectares of new and lost water, and whether it was kept. A patch is kept if at least **half** of its part inside the image is clear on both dates and at least a **quarter** of it lies inside the image. Edge patches are padded with no data. Both pairs took 30 seconds.

<callout icon="📌">

**Decision: split by 7.68 km block, not by patch.** Neighbouring patches share ground (a mining corridor runs across several), so splitting patch by patch would let a model be tested on ground it trained next to. Train / val / test (70 / 15 / 15 %) is assigned per 7.68 km block. The two satellites' grids share their top-left corner, so one block is exactly one Landsat patch and 3 × 3 Sentinel-2 patches, and **a place is in the same split for both satellites** (checked).

</callout>

### Results

| Pair | Patch size | Grid | Kept | Train | Val | Test | New water in kept patches |
| --- | --: | --: | --: | --: | --: | --: | --: |
| Sentinel-2 | 2\.56 km | 13 × 14 | **167** of 182 | 116 | 30 | 21 | 137 of 138 ha |
| Landsat | 7\.68 km | 5 × 5 | **24** of 25 | 17 | 4 | 3 | 130 of 130 ha |

Sentinel-2 dropped 13 slivers (the image is 3,329 px wide, so the last column is 1 px) and two patches less than half clear. Landsat dropped only its bottom-right corner, 84 × 84 px.

<columns>

<column>

![Sentinel-2 pair: 167 patches of 2.56 km kept, coloured by split; white lines are the 7.68 km split blocks; hatched = dropped](_assets/927da9ad111bc1e98d0573c3984d0593.png)

</column>

<column>

![Landsat pair: 24 patches of 7.68 km kept (each patch is one split block); only the bottom-right corner sliver is dropped](_assets/c161acadb00562532b162c0ee6f75828.png)

</column>

</columns>

<callout background="yellow" icon="⚠️">

**With only 25 blocks, a random split is lumpy.** The whole **Tarkwa mine** landed in **val**, so the training set never sees a large industrial mine, and one of the north-west mining corridors is in val too. That's fine for testing the pipeline, but once M3 has labelled galamsey sites, the split should be **stratified**, so that each split gets some mining ground, forest and town. The seed and block size are set at the top of `tiles.py`.

**✅ Fixed 2026-11-21.** The split is now stratified on land, mapped mining area (both sets of outlines) and candidate clearings, so every split holds riverside galamsey, and Tarkwa is in train. The counts in the table above are from the first run; the current ones are Sentinel-2 115 / 25 / 27 and Landsat 15 / 4 / 5. The mask files now have 4 bands (land class, and mining from both sets of outlines as bit flags). See [REMI M3 Labelled Data Report](page:spbr0bdedd).

</callout>

<toggle>

Remaining caveats

- **Landsat gives few patches.** 24 patches of 7.68 km, only 17 for training: too few to train a model from Landsat alone in this box. Landsat's role is the longer history (older "before" dates), or training on a larger area.
- **No overlap between patches.** That's right for the index and for testing. For training, random crops or overlapping windows from the same files add variety without new files.
- **Cloudy pixels stay in kept patches.** Up to half a patch can be cloud. Models must use the **clear** band to ignore those pixels in the loss.
- **Sentinel-2 SWIR is 20 m** resampled to 10 m, as in step 2.

</toggle>

---

## 📁 Files

All in `~/gitRepo/REMI/` (moved from `~/Downloads/REMI-test-data/`):

| File | What it is |
| --- | --- |
| `cloud_mask.py` | Builds the cloud masks: `python cloud_mask.py` for all five images, or name some, e.g. `python cloud_mask.py s2_2025-02-04` |
| `water_mask.py` | Builds the water masks and change maps: `python water_mask.py` for both pairs, or `python water_mask.py <before> <after>` |
| `data/interim/<image>_cloudmask.tif` | One cloud mask per image (5 files): 0 clear, 1 cloud or shadow, 255 no data |
| `data/interim/<image>_water.tif` | One water mask per image in a pair (4 files): 0 land, 1 water, 254 cloud, 255 no data |
| `data/interim/<before>__<after>_waterchange.tif` | One change map per pair (2 files): 0 land both, 1 water both, 2 new, 3 lost, 254 cloud, 255 no data |
| `previews/s2_<date>_cloudmask.png`, `..._zoom.png` | The cloud-mask pictures above |
| `previews/<before>__<after>_waterchange.png`, `..._zoom.png` | The water-change pictures above |
| `normalise.py` | Removes the seams and matches the dates: `python normalise.py` for both pairs, or `python normalise.py <before> <after>`. The first Sentinel-2 run downloads each tile (about 2.5 min per date); later runs reuse the saved fits |
| `data/interim/<image>_sr.tif` | Surface reflectance × 10000 with seams removed (4 files): bands blue, green, red, NIR, SWIR; 0 = no data |
| `data/interim/<after>_sr_matched.tif` | The "after" image matched to its "before" image (2 files). **Use these with the `_sr` "before" images for change detection** |
| `data/interim/s2_<date>_tiles.tif`, `s2_<date>_seams.json` | Which tile each pixel came from, and each tile's fitted gain and offset per band |
| `data/interim/<before>__<after>_nochange.tif` | The unchanged ground used for matching (2 files): 1 unchanged, 0 not, 255 cloud or no data |
| `previews/*_seams_*.png`, `*_step3_*.png` | The step 3 pictures above |
| `tiles.py` | Cuts the patches: `python tiles.py` for both pairs, or `python tiles.py <before> <after>`. Rewrites the index and the preview each run |
| `data/tiles/<before>__<after>/img/r<row>_c<col>.tif` | One per kept patch: 10 bands, before then after (blue, green, red, NIR, SWIR), reflectance × 10000, 0 = no data |
| `data/tiles/<before>__<after>/mask/r<row>_c<col>.tif` | One per kept patch: band 1 clear (1 = clear on both dates), band 2 water change (codes as in step 2) |
| `data/tiles/<before>__<after>_index.gpkg` | Every patch on the grid with its footprint, block, split, clear share, new and lost water (ha), and whether it was kept. Open in QGIS and colour by `split` |
| `previews/<before>__<after>_tiles.png` | The tile-grid pictures above |

**Git:** the code is at `github.com/MensahPrince/REMI`. `data/` and `previews/` are not in git: they're large and the scripts rebuild them.

To see a mask in QGIS, load it on top of its image, set **Symbology → Paletted/Unique values**, and give each value its own colour.
