---
id: f9uwkb5lkz
title: "Smooth key light shadow edges: box fitted to the view, 3x3 tent filter"
order: "b1N"
created: 2026-11-02T19:37:52Z
updated: 2026-11-02T19:37:52Z
version: 1
Status: "Done"
Area: "Shading"
Priority: "High"
Due: 2026-11-02
Notes: "2026-11-02, asked for ('the shadows have jagged edges'). key_shadow(): ortho box as wide as what the camera sees of the scene (frustum clipped to the scene box), still as deep as the scene so off-screen casters cast; width in 1/8-octave steps, centre snapped to texels so edges don't crawl. Soft key blur and search kept in world size. Key test is now four compare lookups half a texel apart, as the lamps'. Truck side shadow on fox_and_truck no longer saw-toothed. Goldens shadow_bands* re-blessed. Commit 311c8d4 on r2-world-tab, not pushed."
---
