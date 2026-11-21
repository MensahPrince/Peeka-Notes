---
id: 66q48ugb1j
title: "Preview: surface left unconfigured after a suboptimal frame (Windows error flood)"
order: "b3E"
created: 2026-11-10T17:13:20Z
updated: 2026-11-10T17:13:20Z
version: 1
Status: "Done"
Area: "Platform"
Priority: "High"
Due: 2026-11-10
Notes: "Commit 3a8ca86. The surface was configured with a frame still out, which wgpu refuses and which drops the configuration; now presented first, configured again when lost, logged once. Not yet confirmed on Windows."
---
