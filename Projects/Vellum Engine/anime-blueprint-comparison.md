---
id: twet3v9vld
title: "Anime Blueprint Comparison"
sort: 27000
version: 1
---

<callout icon="📌">

**The *Anime Rendering Engine — technical blueprint* (6 pages) compared with Vellum on 2026-11-15.** Vellum already has its architecture, its compositor and its screen-space lines. The gaps became **M51–M58** in `milestones.md` and in the [Vellum Task Tracker](page:mq2uz8r1n6). Roadmap: phases 1, 2, 3 and 6 done, 4 partial, 5 mostly not started.

</callout>

## Section by section

| § | Blueprint | Vellum | Status |
| --- | --- | --- | --- |
| 1 | Separate stages, each passing defined data on | G-buffer → light value → lines → palette compositor → post, in `frame_graph.rs` | ✅ |
| 2 | Base, shadow and highlight colours, thresholds | `Palette` (8 slots), `Ramp.lit_edge` / `mid_edge` | ✅ |
| 2 | Highlight threshold | Through `roughness` and highlight shapes only | ⚠️ M55 |
| 2 | Ramp texture | `Tones` is `Two` or `Three` | ❌ M51 |
| 2 | Texture intensity | `image` on or off | ❌ M52 |
| 3 | N·L into bands; "do not hardcode only three levels"; hard or soft steps | Bands yes; 2 or 3 only; main step always hard | ⚠️ M51 |
| 4A | Inverted-hull silhouettes | Not built | ❌ M58 |
| 4B | Screen-space edges from depth, normals, IDs, materials | Yes, with creases, panels, taper, weight, boil | ✅ |
| 4 | Distant architecture thin or no lines | Width by distance clamped to 0.6–1.6× | ⚠️ M53 |
| 5 | Hair | Hair shadows on face maps, painted strands | ✅ |
| 5 | Cloth: folds, pleats, fold masks | Not built | ❌ M56 |
| 6 | Separate background pipeline: weathering, painterly blending, detail lost with distance | One material model; backdrop and stepped haze | ❌ M57 |
| 7 | Grade, bloom, rim, atmosphere, shadow tint, grain | All there (bloom defaults to 0.15, the blueprint's 15%) | ✅ |
| 7 | Edge cleanup | Thinnest lines fade only | ⚠️ M55 |
| 8 | One Anime Render Settings panel with presets | Spread over four tabs; presets only for metal and water | ❌ M54 |
| 10 | Rust, wgpu, WGSL, a render graph | Exactly this | ✅ |

## The new milestones, in order

1. **M51 Ramps:** any number of bands, each step hard or soft, or a ramp image.
1. **M52 Texture strength and simplification**, and less contrast and detail with distance.
1. **M53 Lines that thin and fade with distance.**
1. **M54 The look panel**, and presets for a whole look.
1. **M55 Highlight and rim thresholds** per material, and edge cleanup.
1. **M56 Cloth:** fold masks, pleat highlights, directional fold shading, a darker underside.
1. **M57 Background materials:** gradient lighting, weathering, painterly blending (needs M48b).
1. **M58 Outlines from an inverted hull**, off by default.

<toggle>

What Vellum has that the blueprint doesn't mention

Timing on twos and threes, holds and smears; perspective cheats; SDF face shadow maps and their studio; hatching, cross-hatching, screentone and stippling; cel metal, glass, water and mirrors; layered PSD and SVG for paint-over; Blender and VRM import; light linking and cookies; the timeline; painting in the view and on the image.

</toggle>

## Ambient light from the world: what it would cost

<callout icon="💡">

Vellum has a basic version already: the **World tab's fill** (sky colour, ground colour, strength), a hemisphere light added in `shade.wgsl`. It lifts colour but not tone, so hatching stays. Today it's set by hand and is off unless a shot sets it.

</callout>

| Step | Work | GPU cost (960×540, HD 620; estimates) |
| --- | --- | --- |
| A. Blender's World colour and strength exported as the fill | ½ day | None |
| B. The fill taken from the backdrop panorama: 9 spherical-harmonic numbers worked out on load from a small mip | 1–2 days | About 0.05 ms in shade (6.3 ms now) |
| C. Ambient occlusion, so creases don't fill up with light | 3–5 days | 1–3 ms at half size (5–10% of the frame) |
| D. Ambient stepped into the ramp's bands, so it doesn't put gradients into flat cels | 1 day after M51 | None |

⚠️ **The artistic cost is the real one:** ambient is smooth by nature and works against flat cel tones. B with D keeps the cel look. C is mostly covered by the existing contact shadows.

📌 **Chosen 2026-11-15: B and D, as M59.**
