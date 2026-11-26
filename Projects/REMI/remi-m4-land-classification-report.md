---
id: hyxwfpn7xd
title: "REMI M4 Land Classification Report"
sort: 13000
version: 1
---

Progress report for **M4 (Land classification)**: a model that says what each piece of ground is. Labels come from [REMI M3 Labelled Data Report](page:spbr0bdedd); tracking is on [REMI Project Hub](page:imqxd436oq). *Started 2026-11-26.*

<callout icon="🎯">

**First result: the ResNet works, but it isn't better than a simple model yet, and the labels are why.** It finds galamsey along the rivers, towns and forest well. It scores no higher than a linear model on average colours, and most of its "mistakes" on mining are mining that the 2021 labels call forest. **Better labels will help more than a bigger model.**

</callout>

## ✅ v2 (2026-11-26): labels fixed, now it beats the baseline

**What changed.** The v1 results below showed that the labels were the problem, so v2 fixes them and changes nothing in the model:

1. **A bug in the chip labels.** In mining ground, the bare pixels were dropped as stale (WorldCover 2021 green, 2025 bare), so only the green between the pits was left to label the chip, and it came out "forest". Now a chip where stale pixels cover 30 % or more **is not called vegetation**.
1. **347 chips reviewed by eye**: every chip labelled vegetation that v1 called mining. **243 are mining**, 36 built-up, 12 really vegetation, 56 unsure. Of the 78 in test, **68 are mining**: v1 was mostly right. Kept in `data/labels/chip_review_s2.gpkg` (`labelled_by` = claude), separate from the candidates file.
1. **Forest and other vegetation merged** into **vegetation**.

| Test | v1 (old labels) | v1 (v2 labels) | **v2** | Linear baseline |
| --- | --: | --: | --: | --: |
| Accuracy | 0\.90 | 0\.96 | **0.96** | 0\.96 |
| Macro F1 | 0\.57 | 0\.70 | **0.70** | 0\.69 |
| **Mining F1** | 0\.60 | 0\.77 | **0.72** | 0\.65 |
| Mining precision / recall | 0\.47 / 0.82 | 0\.68 / 0.88 | **0.67 / 0.79** |  |
| Built-up F1 | 0\.74 | 0\.70 | **0.75** | 0\.65 |
| Bare ground F1 | 0\.39 | 0\.37 | 0\.35 | **0.47** |

<callout background="yellow" icon="⚖️">

**The fair comparison leaves out the chips reviewed by eye.** They were chosen *because* v1 called them mining, so they flatter v1. Without them (6,275 test chips): **v2 mining F1 0.71, v1 0.68**; macro F1 **0.71 against 0.68**. So most of the jump from v1's 0.60 comes from **fixing the labels**, and v2 adds a little on top.

</callout>

**The ResNet now beats the simple model** on mining (0.72 against 0.65) and built-up (0.75 against 0.65), the two classes that matter for alerts. Bare ground stays weak: only 54 test chips, mostly mine ground at Tarkwa that sits between "bare" and "mining".

![v2 land classes, 2025-02-04, 160 m chips: vegetation, bare ground, built-up, mining](_assets/9b2e6977ac76c98e9e226aff9bef4454.png)

![v2 confusion matrices, val and test (rows = label, columns = prediction)](_assets/a9f3368a168122be772d26b981acff49.png)

**Where v2 still errs on test:** 54 vegetation chips called mining (precision 0.67), and 19 mining chips called built-up (villages built among the workings). Test labels are still mostly Claude's, so check them in QGIS before trusting the last digit.

---

*Below: v1, the first run (kept for the record; its files are `data/models/landclass_v1_*` and `previews/landclass_v1_*`).*

## Method

`landclass.py` cuts the **2025-02-04 Sentinel-2 image** into **160 m chips** (16 × 16 pixels, five bands) and trains a **ResNet-18** to label each one.

|  |  |
| --- | --- |
| **Classes** | forest, other vegetation, bare ground, built-up, **mining** |
| **Labels** | WorldCover 2021, corrected with the reviewed candidates from M3: *mining* and *mining (large-scale)* become **mining**, *settlement* becomes built-up, *unsure* is left out |
| **Stale labels left out** | Pixels where WorldCover 2021 and the 2025 image clearly disagree: "green" but bare now (NDVI \< 0.35) outside any candidate, or "bare" but green now |
| **Chip rule** | **Mining** if mining covers 30 % of the chip; otherwise the majority class if it covers 60 %; mixed chips are not used |
| **Split** | The tile split (by 7.68 km block), so test is ground the model never saw |
| **Model** | ResNet-18 from scratch, 5 input bands, 3 × 3 first convolution for small chips; class-weighted loss; flips and turns; 25 epochs; best epoch on val kept (18) |
| **Speed** | CPU only, about 90 s per epoch: each epoch uses all 2,199 non-forest chips and a fresh random 6,000 of the 22,060 forest chips |

<toggle>

Why not water, and why 160 m

- **Water is not a class.** Water here is narrow rivers and small ponds, so it almost never fills 60 % of a chip: there was **1** water chip in val. REMI takes water from the **pixel-level 2025 water mask** (M2 step 2), which is better for it anyway.
- **320 m chips were tried first.** At that size only forest was pure: val had 3 "other vegetation" and 4 "bare" chips, too few to score. 160 m gives val 63, 25, 122 and 165 chips of the other classes.

</toggle>

| Chips used | Forest | Other vegetation | Bare | Built-up | Mining |
| --- | --: | --: | --: | --: | --: |
| Train | 22,060 | 481 | 281 | 533 | 904 |
| Val | 4,353 | 63 | 25 | 122 | 165 |
| Test | 5,555 | 130 | 54 | 167 | 126 |

## Results

![v1: ResNet-18 land classes for the whole box, 2025-02-04, 160 m chips](_assets/a2a0e8be8c6d62ba7a2d2908d8af883d.png)

**The map makes sense.** Galamsey (magenta) follows the north-west streams and the Ankobra; Prestea, Bogoso and Tarkwa are built-up (red); the Tarkwa mine pit is bare ground (tan), because outside the reviewed patches WorldCover calls it bare.

| Test (6,032 chips) | Precision | Recall | F1 |
| --- | --: | --: | --: |
| Forest | 0\.99 | 0\.91 | 0\.95 |
| Other vegetation | 0\.11 | 0\.31 | 0\.16 |
| Bare ground | 0\.36 | 0\.43 | 0\.39 |
| Built-up | 0\.62 | 0\.91 | 0\.74 |
| **Mining** | **0.47** | **0.82** | **0.60** |
| *Accuracy / macro F1* |  |  | *0.90 / 0.57* |

*Precision: of the chips the model called X, how many the label calls X. Recall: of the chips labelled X, how many the model found.*

![Confusion matrices, val and test (rows = label, columns = prediction)](_assets/245a9a2d5332c8c7ac6c200abee9d191.png)

### Compared with a simple model

A **linear model on each chip's average band values** (and their spread, and NDVI) is the bar to clear (`landclass.py baseline`):

|  | ResNet-18 | Linear baseline |
| --- | --: | --: |
| Test accuracy | 0\.90 | 0\.94 |
| Test macro F1 | 0\.57 | 0\.58 |
| Mining F1 (test) | 0\.60 | 0\.60 |
| Built-up F1 (test) | **0.74** | 0\.71 |

<callout background="yellow" icon="⚠️">

**The ResNet doesn't beat it yet.** A 160 m chip has little shape for a ResNet to use, so colour carries most of the information, and colour is what the linear model sees. And when labels are noisy, a bigger model mostly learns the noise.

</callout>

### Most "false alarms" on mining are real mining

76 test chips **labelled forest** were **predicted mining**, which is what drags mining precision down to 0.47. A random 24 of them:

![24 test chips labelled forest but predicted mining (yellow square = the 160 m chip)](_assets/c8c604f6400a956ae06ab552acd1f326.png)

**About two-thirds sit in mining ground**: sand, pits and workings with patches of green between them. WorldCover 2021 still calls those pixels forest, and they were too small to become candidates. The rest are villages, roads and a tailings edge. **So the model's true mining precision is clearly higher than 0.47**: it finds mining the labels miss.

<callout background="yellow" icon="⚠️">

**"Other vegetation" can't be learned from these labels** (F1 0.16). In this cocoa and plantation landscape, WorldCover's line between forest and other vegetation is hard to see even at 10 m, and its 2021 labels differ from the 2025 image. **Merge it into "vegetation"**, unless a better source turns up.

</callout>

## What next

1. **Fix the labels, not the model.** Mark the forest-labelled chips the model calls mining (the 76 in test first) as mining where they are, like the M3 review. Then retrain.
1. **Merge forest and other vegetation**, or keep them apart only where a better source exists.
1. **More context.** A 160 m chip can't show a mine's shape. A U-Net on whole 2.56 km tiles (M6) sees shape and labels every pixel. That's where **Colab's GPU** starts to pay off.
1. **Check Claude's test labels** (M3): the test scores are against labels that are 97 % Claude's.

## 📁 Files

| File | What it is |
| --- | --- |
| `landclass.py` | `prepare` (chips), `train`, `baseline`, `map` |
| `data/models/landclass_chips.npz` | 42,752 chips with labels (−1 = mixed), split and position |
| `data/models/landclass_resnet18.pt` | The trained model, with the band means and spreads it needs |
| `data/models/landclass_metrics.json`, `landclass_baseline.json`, `landclass_train.log` | Scores per epoch, val and test; the baseline's scores |
| `previews/landclass_map.png`, `landclass_confusion.png`, `landclass_forest_predicted_mining.png` | The pictures above |
