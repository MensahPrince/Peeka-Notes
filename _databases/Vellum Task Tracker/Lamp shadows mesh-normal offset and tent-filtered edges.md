---
id: b2grxwecwv
title: "Lamp shadows: mesh-normal offset and tent-filtered edges"
order: "b0m"
created: 2026-10-30T23:32:15Z
updated: 2026-10-30T23:32:15Z
version: 1
Status: "Done"
Milestone: "M19c Lamp shadow fixes"
Area: "Shading"
Priority: "Medium"
Notes: "2026-10-30. Lamp shadow test nudged along the mesh normal (not proxy/snapped shading normal), like the key light. Pool shadow edges stepped every 2-3 px at 1080p (spot texel ~2 px); 4 compare lookups half a texel apart (3x3 tent) round them off, no bigger atlas. Golden lamp_proxy; lamps re-blessed. Completes M19c."
---
