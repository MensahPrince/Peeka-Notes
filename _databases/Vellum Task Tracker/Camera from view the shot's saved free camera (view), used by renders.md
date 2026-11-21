---
id: fpso8aksxe
title: "Camera from view: the shot's saved free camera (view), used by renders"
order: "b1R"
created: 2026-11-03T21:04:12Z
updated: 2026-11-03T21:04:12Z
version: 1
Status: "Done"
Milestone: "Release 0.1"
Area: "Iteration"
Priority: "High"
Due: 2026-11-03
Notes: "2026-11-03, asked for before video. ShotView (eye, target, fov) in shots and overrides; View → Camera from view (Ctrl+Alt+0) saves it and picks the free camera; 0 looks through the shot's camera; undoable. Headless: named scene camera, else view, else framing; vellum render prints which (camera_label); Render tab shows it. ShotRenderer::open_with takes overrides; a bare glTF now renders with its saved overrides. Branch r4-video, not committed."
---
