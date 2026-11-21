---
id: pq7j201csf
title: "Cameras in the window: Add → Camera, active camera for every render, Camera tab"
order: "b1U"
created: 2026-11-03T21:18:20Z
updated: 2026-11-03T21:18:20Z
version: 1
Status: "Done"
Milestone: "Release 0.1"
Area: "Iteration"
Priority: "High"
Due: 2026-11-03
Notes: "2026-11-03, asked for. Replaces the 'Camera from view: the shot's saved free camera (view)' row: `view` became named `cameras` in shots/overrides, resolved into scene camera nodes (ids 0xFFFD_0000+). shot `camera` = active camera; F12 now renders through it too (the view, with a note, when none). Header/C only change the view (no longer saved). camera_marker.rs draws frustums, click selects, gizmo moves eye+target; Camera tab (fov, position, target, Set active, Look through, Move to view); Render tab active-camera picker; Outliner rows with look-through icon and right-click Set active; Ctrl+Alt+0 moves active or adds; Delete removes window cameras. Tests: shot cameras resolve, marker frame/hit. Branch r4-video, not committed; not tried by hand in the window."
---
