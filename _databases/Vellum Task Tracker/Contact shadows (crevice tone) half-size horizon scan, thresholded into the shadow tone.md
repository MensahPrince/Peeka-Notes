---
id: 582b7a6muh
title: "Contact shadows (crevice tone): half-size horizon scan, thresholded into the shadow tone"
order: "b1b"
created: 2026-11-04T05:49:04Z
updated: 2026-11-04T05:49:04Z
version: 1
Status: "Done"
Milestone: "M18b Hard surface"
Area: "Shading"
Priority: "High"
Due: 2026-11-04
Notes: "2026-11-04, asked for. ContactPass (contact.wgsl scan 6x3 + 3x3 depth-aware blur, Rg32Float half size) -> shade contact_shadow depth-aware upsample, smoothstep at threshold, light *= 1 - c*darkness. Shot contact (World tab), material contact:false (Surface), V view contact, vellum render --view. Golden contact. 5 ms @1080p HD 620 when on. Replaces the 'Crevice tone' Not started row. Not committed."
---
