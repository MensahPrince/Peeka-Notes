---
id: aa3nxoptrp
title: "Band blur: blend core shadow and edge band into the shadow"
order: "b1I"
created: 2026-11-02T03:58:47Z
updated: 2026-11-02T03:58:47Z
version: 1
Status: "Done"
Milestone: "M19b Shadow-aware shading"
Area: "Shading"
Priority: "Medium"
Due: 2026-11-02
Notes: "2026-11-02, asked for. band_blur per material/object, vellum_band_blur in Blender, Material > Bands > blur in the panel. Widens band_cover's smoothstep from one pixel to the blur width (same units as the bands), in composite.wgsl and layers.rs, so layers still stack. Golden shadow_bands_blur; all other goldens unchanged. Not committed."
---
