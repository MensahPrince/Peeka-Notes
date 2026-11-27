---
id: 56h06s9n5l
title: "REMI M5 Change Detection Report"
sort: 14000
version: 1
---

<callout icon="🛰️">

**M5 first pass (2026-11-27):** a Siamese ResNet-18 compares a before and an after Sentinel-2 image and flags 160 m chips where **new mining** appeared. On the clean test chips it scores **F1 0.98**, but a linear model on simple chip statistics already gets **0.95**. That's because the test only holds clear-cut cases. On the full map, two-thirds of what it flags is **cloud, haze or change nobody has labelled**. So the model works, but the test is too easy and the training data has gaps. Code: `change.py`.

</callout>

## What it does

- **Pairs:** the nine-year baseline (2016 composite → 2025-02-04) and the one-year pair (2024-03-31 → 2025-02-04), from the M2 tiles. Chips use the M4 grid (16 × 16 px, 160 m) and the M2 split, so train, val and test never share a 7.68 km block.
- **Model:** one ResNet-18 encoder reads both dates (shared weights, started from the M4 land-class model). A small head looks at the before features, the after features and how far apart they are. Each epoch sees every new-mining chip plus 6,000 no-change chips. Half of those are "hard": bare on both dates (old pits, towns, river sand), which must **not** be flagged.
- **Alert threshold:** picked for the best F1 on val (0.97 probability). Test was scored once, at that threshold.

## Labels

A pixel **changed** if it was green before (NDVI ≥ 0.5) and is bare after (NDVI \< 0.35), and is clear on both dates. On the 2016 composite, only the two low-haze dates count. A changed pixel is **new mining** if it lies in a reviewed mining candidate (galamsey or large-scale) or a published mining outline (Tang & Werner, Maus et al.).

| Chip label | Rule |
| --- | --- |
| New mining | ≥ 25 new-mining pixels (0.25 ha) |
| No change | ≤ 2 changed pixels |
| Left out | anything else: under half the chip clear, a little change, or change of unknown cause |

The labelled chips per pair:

|  | Train | Val | Test |
| --- | --- | --- | --- |
| 2016 → 2025, new mining / no change | 1,114 / 14,373 | 197 / 2,774 | 231 / 3,285 |
| 2024 → 2025, new mining / no change | 701 / 23,258 | 170 / 4,232 | 95 / 5,645 |

<callout background="yellow" icon="⚠️">

**There is no "other clearing" class.** Of the 914 reviewed candidates, 706 are mining, 205 unsure and 3 anything else. No farms or roads were found. So the model has never been shown a new farm, and the test can't measure whether it would flag one.

</callout>

## Results (test, 326 new-mining and 8,930 no-change chips)

| Method | Avg. precision | Precision | Recall | F1 | Bare but unchanged, flagged |
| --- | --- | --- | --- | --- | --- |
| NDVI drop (share of chip down ≥ 0.2) | 0\.893 | 0\.70 | 0\.93 | 0\.80 | 1 of 783 |
| Linear model on chip statistics | 0\.997 | 0\.99 | 0\.91 | 0\.95 | 0 of 783 |
| **Siamese ResNet-18** | **0.997** | **0.99** | **0.96** | **0.98** | 2 of 783 |

Per pair, the Siamese model gets F1 0.97 (2016 → 2025) and 0.98 (2024 → 2025). Val AP was above 0.98 from epoch 1. Starting from the M4 encoder means it already knows what a pit looks like. The best epoch was 14.

**Why the scores are so high:** the labels come from the same NDVI thresholds a simple rule can measure, and every borderline chip is left out. Old mining that didn't change (the hard negatives) is easy too: everyone gets it right. So this test shows the model can find clear new pits. It does **not** show it beats the linear model on hard cases.

## The full map tells a different story

![Chips the Siamese model flags as new mining, 2024-03-31 → 2025-02-04. About a third are real mining labels; the rest are cloud and unlabelled change.](_assets/48895e0a59d18e7c052e46d59354956c.png)

*White squares: tiles left out in M2 (too little clear ground).*

Run on every chip, the model flags **11,000 ha** for 2016 → 2025 and **5,300 ha** for the single year 2024 → 2025. The labelled new mining is about 1,440 ha and 770 ha (on clear, low-haze pixels). Where the flags land:

| Flagged chips | 2016 → 2025 | 2024 → 2025 |
| --- | --- | --- |
| Labelled new mining (correct) | 1,506 | 934 |
| Labelled no change (false alarms) | 16 | 1 |
| Left out: under half clear (**cloud**) | 728 | 604 |
| Left out: clear, but 2016 is **hazy** | 1,000 | n/a |
| Left out: clear, ≥ 25 px changed, **cause unknown** | 713 | 223 |
| Left out: clear, 3–24 px changed | 320 | 325 |

**Cloud and haze** are the biggest source of false alerts. The model never saw a cloudy or hazy chip in training, so it reads them as change. The **unknown-cause change** chips (936 in all, about 815 ha) are the cases REMI exists for: unmapped galamsey, or a farm, road or town. They have no label yet.

## Next steps

1. **Mask cloud and haze before alerting** (quick). Score only chips that are at least half clear and low-haze, and report the rest as "can't see". Also add cloudy chips to training as no-change.
1. **Label the unknown-change chips** (the real test set). Review about 900 chips (Claude first, then you check the test split, as in M3). This gives an "other clearing" class and a hard test where NDVI alone can't decide.
1. **Then retrain and compare again.** The Siamese model only earns its keep if it beats the linear model on those chips.
1. Later: M6 segmentation (U-Net on Colab), and M7, which combines M4 (is it mining now?) with M5 (did it just change?).

## Files

- `change.py`: `prepare`, `train`, `baseline`, `map`
- `data/models/change_siamese.pt`, `change_metrics.json`, `change_baseline.json`, `change_train.log`
- `data/models/change_prob_<pair>.tif`: new-mining probability per 160 m chip (opens in QGIS)
- `previews/change_map_<pair>.png`
