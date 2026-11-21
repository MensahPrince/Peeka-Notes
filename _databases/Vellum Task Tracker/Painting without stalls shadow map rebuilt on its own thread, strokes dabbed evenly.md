---
id: b4fn4n7knj
title: "Painting without stalls: shadow map rebuilt on its own thread, strokes dabbed evenly"
order: "b4X"
created: 2026-11-13T19:27:13Z
updated: 2026-11-13T19:27:13Z
version: 1
Status: "Done"
Milestone: "M46 Workspace tabs and Paint"
Area: "Iteration"
Priority: "High"
Due: 2026-11-13
Notes: "2026-11-13, ebd2468 on main. map_job.rs builds the map off the window thread; Renderer::replace_texture writes it into its array layer in place; surface read once at pen-down; dabs a quarter brush apart with pressure blended. Release bench on Suzanne: 0.27 ms a move, 1 ms lift (was ~0.5 s freeze), no slow frames after. Touch still 36 ms (whole-view read)."
---
