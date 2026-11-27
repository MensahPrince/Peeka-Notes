---
id: hrtvqupqy9
title: "REMI Tuning Log: How Each Value Was Chosen"
sort: 16000
version: 1
---

Every threshold and setting that shapes the numbers in [REMI Working Paper: Mapping New Galamsey from Sentinel-2](page:x6ahznsl6e): where it started, what it ended at, and the evidence that moved it. Code: `github.com/MensahPrince/REMI`. *Written 2026-11-27.*

<callout icon="📌">

**Few of these values are optimal in the strict sense.** No parameter was found by a grid or random search. Each one started as a textbook default or a first guess, and was changed only when a check showed it was wrong: a picture, a count, or a validation score. The Status column says which kind each value is: **changed on evidence**, **set once** (reasoned, never tested against alternatives), or **chosen on val** (picked by a validation score). Values marked *set once* are the obvious places to try a sweep.

</callout>

## Rules that held throughout

- **Test was scored once per model, at the end.** Epochs, alert thresholds and every comparison used validation only. No setting was changed after looking at a test score, with one exception: the M4 v1 test errors were inspected by eye, and that led to the v2 label fixes (see M4). The fair v1 against v2 comparison leaves out the chips reviewed as a result.
- **Splits are by 7.68 km block**, so a tweak can't win by memorising ground next to a test tile.
- **Seed 0** everywhere, so reruns give the same numbers.
- **Every model has a simple baseline** scored the same way, so a tweak to the model is measured against something that doesn't change.

## M2 · Preprocessing

| Setting | First | Final | What moved it | Status |
| --- | --- | --- | --- | --- |
| Sentinel-2 cloud mask | SCL (built-in) | OmniCloudMask | SCL flagged 0.9 % of 2025-02-04 where OmniCloudMask found 7.2 %, and called 56 % and 87 % of the Tarkwa tailings "cloud" | Changed on evidence |
| OmniCloudMask resolution | 10 m | 20 m | About 4× faster on CPU; inside the tool's stated 10–50 m range | Set once |
| Cloud buffer | none | 50 m | Soft cloud edges left unmasked; buffer raised the mask from 7.2 % to 10.8 % | Set once (could be cut if too much land is lost) |
| Seam reference tile | none | 30NXM | Covers 57 % of the box and matches Landsat best over forest (blue 0.022 against Landsat's 0.028; west tiles 0.005–0.012) | Changed on evidence |
| IR-MAD no-change cutoff |  | 0\.95 probability | Gives 28,323 unchanged S2 pixels; 2/3 to fit, 1/3 held out | Set once |
| Gain in visible bands | gain + offset | offset only (`MIN_SPAN` 0.5) | First run gave red gain 1.30, dimming bare ground 0.20 → 0.16. Unchanged pixels are 93 % forest and cover only the dark end. Now a band gets a slope only where unchanged ground spans at least half its brightness range: NIR and SWIR do, visible bands don't | Changed on evidence |
| Earth Search 0.1 offset | subtract (per metadata) | don't subtract | Subtracting drove most pixels negative; without it forest matches Landsat | Changed on evidence |
| Water index | MNDWI > 0 | MNDWI > 0 | Textbook default; never tuned (needs labelled water) | Set once |
| Water speck size |  | 5 px | Removes single-pixel noise | Set once |
| Water change maps | on raw images | on matched images | Matching changes SWIR; "after" water fell 432 → 391 ha (S2) | Changed on evidence |

**Result of the M2 tweaks:** median NDVI change across the box went from −0.059 to +0.001, and the area that looks clearly less green from 26.2 % to 8.0 %. Leftover bias on held-out unchanged ground is ≤ 0.0004 in every band.

### 2015/16 baseline

| Setting | First | Final | What moved it | Status |
| --- | --- | --- | --- | --- |
| Baseline image | Landsat 2015-12-29 | Sentinel-2 composite of 5 dates | The Landsat scene was too hazy for believable bare or water figures, raw or matched | Changed on evidence |
| Date order |  | least hazy first | Haze ranked by clear-forest red and NDVI against 2025 (0.054 / 0.69). On a hazy date, matching swung bare ground from 5,519 ha to 2,544 ha | Changed on evidence |
| Pixels used to measure change | all clear | low-haze only (dates 1–2) | Hazy pixels over-correct bare ground and water. Low-haze gives 1,820 ha new clearing; all clear pixels give 2,509 ha | Changed on evidence |

### Tiles and split

| Setting | First | Final | What moved it | Status |
| --- | --- | --- | --- | --- |
| Tile size |  | 256 px (2.56 km) | Fixed model input; one Landsat tile = 3 × 3 S2 tiles | Set once |
| Keep rule |  | ≥ 50 % clear, ≥ 25 % inside image | Drops edge slivers and mostly-cloud tiles (167 of 182 kept) | Set once |
| Split unit | per tile | per 7.68 km block | Neighbouring tiles share mining corridors | Changed on evidence |
| Split method, v1 | random |  | Put the whole Tarkwa mine in val: 45 % of mining in val, 6 % in test | Rejected |
| v2 | stratified on land + mining area |  | Put every riverside galamsey corridor in train | Rejected |
| v3 | \+ candidate clearing area |  | Val got no large-scale mining at all | Rejected |
| v4 (current) |  | galamsey and large-scale balanced separately; 20,000 tries, then hill-climb; ≥ 5 % of each per split | Every split now holds both kinds. Large-scale ends 82 / 12 / 5 % because one block holds 57 % of it | Changed on evidence |

## M3 · Candidate clearings

| Setting | First | Final | What moved it | Status |
| --- | --- | --- | --- | --- |
| Bare cutoff | NDVI \< 0.30 | NDVI \< 0.35 | Pale riverbed workings sat at NDVI 0.30–0.45 and were missed | Changed on evidence |
| Morphological opening | 3 × 3 | 2 × 2 | 3 × 3 removed 69 % of the bare ground in narrow dug-out stream channels; 2 × 2 keeps 20 m strips but still drops 1 px road edges | Changed on evidence |
| Minimum patch |  | 0\.5 ha | 50 px at 10 m | Set once |
| Effect | 639 patches | 914 patches | Share mostly inside mining maps barely moved (72 % → 68 %), so the extra patches are mostly mining, not farms |  |
| Label set | mining, farm, road … | \+ mining (large-scale) | Industrial pits and tailings were mixed in with galamsey; they behave differently (95 % inside Maus et al., median 200 m from a river, against 48 % and 0 m) | Changed on evidence |

## M4 · Land classification

| Setting | First | Final | What moved it | Status |
| --- | --- | --- | --- | --- |
| Chip size | 320 m | 160 m | At 320 m almost only forest chips were pure: val had 3 other-vegetation and 4 bare chips | Changed on evidence |
| Water class | included | dropped | 1 water chip in val; water comes from the pixel-level water mask instead | Changed on evidence |
| Forest per epoch | all 22,060 | random 6,000 | Epoch time \~4 min → \~1.5 min on 4 CPU cores | Changed on evidence (speed) |
| Class weights |  | 1 / √frequency | Standard choice for imbalance; not compared with alternatives | Set once |
| Mining chip rule |  | ≥ 30 % of labelled pixels | Galamsey strips rarely fill a chip | Set once |
| Other classes |  | majority ≥ 60 % | Mixed chips left out | Set once |
| Stale pixels (WorldCover green, NDVI \< 0.35 now, no candidate) | ignored | ignored, **and** a chip with ≥ 30 % stale pixels can't be vegetation | Bug: in mining ground, ignoring the bare pixels left only the green between pits, so the chip came out "forest" | Changed on evidence |
| Forest / other vegetation | separate | merged into vegetation | Other vegetation F1 0.16; the line can't be seen in cocoa and plantation country | Changed on evidence |
| Chip review | none | 347 chips by eye | Of 76 test chips labelled forest but predicted mining, about two-thirds were mining | Changed on evidence |
| Epochs, LR, batch | 25, 2e-3, 128 | same | Not varied | Set once |
| Epoch kept |  | best val macro F1 (v1: 18, v2: 16) |  | Chosen on val |

**How much each change bought (test mining F1):** v1 0.60 → v1 scored on v2 labels 0.77 → v2 retrained 0.72 (linear baseline 0.65). Leaving out the reviewed chips, which flatter v1: v1 0.68, v2 0.71. **Most of the gain came from the labels**, not the model.

## M5 · Change detection

All M5 values are first-pass. Only the epoch and the alert threshold were picked on data.

| Setting | Value | Reason | Status |
| --- | --- | --- | --- |
| Green before / bare after | NDVI ≥ 0.5 / \< 0.35 | Same as `candidates.py`, so labels and candidates agree | Set once |
| New-mining chip | ≥ 25 px (0.25 ha) | A 0.25 ha new pit is worth an alert | Set once |
| No-change chip | ≤ 2 changed px | Strict, so no-change really means none | Set once |
| Chip must be clear | ≥ 50 % | As tiles | Set once |
| 2016 pixels | low-haze dates only | From the M2 baseline finding | Changed on evidence |
| Encoder start | M4 weights | Val AP 0.989 after one epoch | Set once (from-scratch run not tried) |
| Negatives per epoch | 6,000, half "hard" (≥ 10 % bare after) | Old pits and towns must not be flagged | Set once |
| Epochs, LR | 15, 1e-3 | Lower LR than M4 because the encoder is pre-trained | Set once |
| Epoch kept | 14 (val AP 0.999) |  | Chosen on val |
| Alert threshold | 0\.972 probability | Best val F1; the baselines get theirs the same way | Chosen on val |

<callout background="yellow" icon="⚠️">

**M5's high scores aren't the product of tuning.** The test reaches F1 0.98 because the labels come from the same NDVI thresholds and every borderline chip is left out, so even a linear model gets 0.95. Tuning these values against this test would only fit the labelling rule. The next tweaks need a harder test first: mask cloud and haze, and label the \~900 chips of unknown-cause change (see [REMI M5 Change Detection Report](page:56h06s9n5l)).

</callout>

## Where a sweep would pay off

Ranked by how much the value moves the results and how little evidence backs it now:

1. **Bare and green NDVI cutoffs** (0.35 / 0.5). They define both the candidates and the M5 labels. A sweep needs labels made without NDVI, so it waits on the unknown-change review.
1. **M4 mining chip rule** (30 %) and **M5 new-mining size** (25 px). They trade small sites against false alerts.
1. **Cloud buffer** (50 m). It decides how much ground is "unobserved".
1. **MNDWI cutoff** (0). Silted water may fall below it; needs labelled water.
1. **Model settings** (LR, epochs, class weights). Least urgent: label fixes moved M4 far more than the model did.
