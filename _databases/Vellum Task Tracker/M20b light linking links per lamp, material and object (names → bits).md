---
id: kuey2r2r1y
title: "M20b light linking: links per lamp, material and object (names → bits)"
order: "b13"
created: 2026-11-01T21:42:41Z
updated: 2026-11-01T21:42:41Z
version: 1
Status: "Done"
Milestone: "M20b Light linking and cookies"
Area: "Shading"
Priority: "High"
Due: 2026-11-01
Notes: "2026-11-01. links: [\"hero\"] on a lamp lights only materials/objects sharing a name; unlinked lamps light everything. Shots, panel (Light tab links, Material tab light links), Blender vellum_links on light/material/mesh object. Masks in spare vec4 slots; object links via composable per-object material copies (Scene::own_materials). Golden light_link; existing goldens unchanged. Commit dc07240. Replaces the open 'Light linking' row."
---
