---
id: hxgz4gzq3u
title: "M20 face map: texture loading + threshold vs light's turn around the head, mirrored by side"
order: "b0s"
created: 2026-10-31T19:20:18Z
updated: 2026-10-31T19:20:18Z
version: 1
Status: "Done"
Milestone: "M20 Face shadow maps"
Area: "Shading"
Priority: "High"
Notes: "2026-10-31. Items 1-2. Loaded via the line-texture loader into the texture array; a face pass after the G-buffer writes map + mirrored map to a new RG16F 'face' target; shade compares with acos(light·forward)/π. Replaces N·L and key cast shadows on that material. Shot-only setting face_map: (image, head) so it can be tried. Replaces 'Minimal texture loading' and the first half of 'SDF threshold face map'."
---
