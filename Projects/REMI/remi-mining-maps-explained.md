---
id: eezqbdbynt
title: "REMI Mining Maps Explained"
sort: 12000
version: 1
---

A plain explanation, with pictures, of the two published mining maps REMI uses and what "a pixel is mining" means. The numbers and method are on [REMI M3 Labelled Data Report](page:spbr0bdedd); the citations are on [REMI Sources](page:841cvgtq57).

## 1 · The two mining maps

Two research teams each drew, **by hand**, the outline of every mining area they could see on satellite images:

- **Tang & Werner (2023)**, shown in **pink**: 170 outlines in our box. Tighter, closer to the actual pits.
- **Maus et al. (2022)**, shown in **yellow**: 28 outlines in our box, drawn on 2019 images. Wider, with whole river corridors as one shape.

<image src="_assets/a8aaa279a508ae06e1fe3cc5660c323f.png" width=1.0>

The pilot box: the satellite image, Tang & Werner's mining map (pink), and Maus's mining map (yellow)

</image>

**What to notice:** both maps mark the same places, the **riverside galamsey** in the west and north-west (the thin, branching light lines in the left image) and the big **Tarkwa mine** at the bottom. They mostly agree; Maus just draws fatter shapes.

## 2 · What "a pixel is mining" means

A satellite image is a grid of small squares called **pixels**. In our Sentinel-2 images, each pixel is **10 m × 10 m** of ground, about the size of a house plot. A model doesn't see "outlines"; it looks at each pixel and decides something about it.

So to teach a model, every pixel needs an answer: **is this pixel mining or not?** REMI gets that answer from the two maps. If a pixel falls inside an outline, the map says "mining" for that pixel. Each pixel gets one number:

| Number | Meaning |
| --- | --- |
| **0** | Neither map says mining |
| **1** | Only Tang & Werner says mining |
| **2** | Only Maus says mining |
| **3** | Both maps say mining |

Anything from **1 to 3 counts as mining**. Keeping 1, 2 and 3 separate means we always know which map said so, and **3** marks where both agree: the most trustworthy pixels.

![Close-up of 600 m × 600 m: the satellite pixels with both outlines (A), and the mining number each pixel gets (B)](_assets/5c450363e8f4cf1e909b27777ff932ba.png)

**How to read this picture:**

- **Left (A)** is the real image, zoomed in until you can see the pixel squares. The light, sandy squares are **bare ground**: dug-up mining land. The green squares are trees and bush. The pink and yellow lines are the two maps' outlines.
- **Right (B)** is the same ground, with each pixel coloured by its number. **Orange (3)** is the strip inside both outlines; **yellow (2)** is the wider band only Maus included; **dark green (0)** is outside both.

<callout background="yellow" icon="⚠️">

**The maps are not perfect, and this picture shows both problems.**

- **They miss new mining.** The top-left corner of A is clearly bare, dug-up ground, but in B it's **0**, because neither map drew it. It's probably newer than the images the maps were drawn on. That's what the **candidate list** is for: places that look bare now but aren't on the maps.
- **They include some forest.** Inside the outlines there are green patches of trees, still marked 2 or 3. The outlines are drawn around whole mining areas, not around each pit.

So the maps are a **starting point**, and your review of the candidates in QGIS is what turns them into reliable labels.

</callout>

## 3 · Where this is used

Each tile (a 2.56 km square cut from the big image) has a **mask** file with four layers. The fourth layer is this mining number. The models in M4–M6 will learn from it. The other layers mark cloud, water change, and land type (forest, farm, bare, town, water).

The location of the close-up: UTM 30N, about 600,300 E, 617,010 N, in the north-west mining corridor.
