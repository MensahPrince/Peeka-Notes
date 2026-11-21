---
id: 9x6pbzb1k3
title: "Emission adds over the lit surface, as in EEVEE"
order: "b1M"
created: 2026-11-02T18:51:20Z
updated: 2026-11-02T18:51:20Z
version: 1
Status: "Done"
Milestone: "M16 Unlit and emissive"
Area: "Compositing"
Priority: "High"
Due: 2026-11-02
Notes: "f6894dc. glTF emission no longer makes a material unlit or drops its texture: factor × strength (through the emissive texture) is added over the lit colour. Value shoulder past 0.6 keeps hue instead of clipping to white; overflow in FINAL alpha feeds bloom. Emission section in Look panel, open by default; shot keys emission / emission_strength."
---
