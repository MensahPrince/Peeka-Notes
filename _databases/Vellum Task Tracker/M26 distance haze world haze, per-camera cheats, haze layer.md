---
id: hhtugr21xq
title: "M26 distance haze: world haze, per-camera cheats, haze layer"
order: "b1V"
created: 2026-11-03T21:39:24Z
updated: 2026-11-03T21:39:24Z
version: 1
Status: "Done"
Area: "Compositing"
Priority: "High"
Due: 2026-11-03
Notes: "2026-11-03, asked for (improvements.md's first engine gap). haze.rs (Haze, amount, HazeOverride); ShotHaze on shot + overrides (World tab Haze section, cloud icon), CameraHaze on ShotCamera (Camera tab). HazePass after lines: depth -> distance via inverse projection -> Rgba8UnormSrgb HAZE (colour, cover); composite mixes it over colour and ink and dims glow; background untouched. Haze layer in render_layers when on. 1.5 ms @1080p HD 620 on, 0.05 off. Goldens haze, haze_steps; all others unchanged. Branch m26-haze, not committed."
---
