---
id: bu91z98ayi
title: "M25 glass: opt-in cel glass, streaks, own outline, glass layer"
order: "b1W"
created: 2026-11-03T21:59:43Z
updated: 2026-11-03T21:59:43Z
version: 1
Status: "Done"
Milestone: "M24 Transparency"
Area: "Render core"
Priority: "High"
Due: 2026-11-03
Notes: "2026-11-03. Material.glass (opacity, streak, shadows); vellum_glass(+_streak,_shadows) on BLEND; ShotGlass on looks/objects; Material tab Glass section. DrawMode.glass skipped by geometry/shadow passes (unless shadows). GlassPass: geometry.wgsl vs_main + fs_glass, sorted far->near per object, over FINAL keeping glow alpha, GLASS premult layer, GLASS_ID; writes DEPTH. GlassLinesPass edges GLASS_ID in line colour. Haze on glass; picking hits glass first; glass layer. 2.2 ms @1080p HD 620. Goldens glass, glass_stack; others unchanged. Not done: tinted shadows. Branch m25-transparency, not committed. Replaces the 'M24 transparency' row (milestone renumbered M25 in milestones.md)."
---
