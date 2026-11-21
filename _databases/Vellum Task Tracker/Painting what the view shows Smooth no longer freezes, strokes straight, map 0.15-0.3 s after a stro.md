---
id: vsjv1wxg69
title: "Painting what the view shows: Smooth no longer freezes, strokes straight, map 0.15-0.3 s after a stroke"
order: "b4c"
created: 2026-11-13T21:43:12Z
updated: 2026-11-13T21:43:12Z
version: 1
Status: "In progress"
Milestone: "M46 Workspace tabs and Paint"
Area: "Iteration"
Priority: "High"
Due: 2026-11-13
Notes: "2026-11-13, built and tested, not committed yet (waiting on a try with the Wacom). brush_seen paints the texels under each window pixel of the brush; summed-area smoothing; BuildCache keeps keys' distance fields between builds, threaded, row-sweep transform. Suzanne, 7 keys, release: 1.15 s to 140 ms (one key changed), 290 ms worst."
---
