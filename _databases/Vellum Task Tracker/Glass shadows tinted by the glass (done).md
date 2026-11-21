---
id: z6uo1iscvw
title: "Glass shadows tinted by the glass (done)"
order: "b1Y"
created: 2026-11-03T22:19:43Z
updated: 2026-11-03T22:19:43Z
version: 1
Status: "Done"
Milestone: "M24 Transparency"
Area: "Shading"
Priority: "Low"
Due: 2026-11-03
Notes: "2026-11-03, asked for. GlassShadow Off/Solid/Tinted (ShotGlass.shadows, vellum_glass_shadows 1 / \"tinted\" / 2, Material tab choice). Shadow pass draws tinted glass into GLASS_SHADOW (1024² depth) + GLASS_SHADOW_TINT (mix(white, base, sqrt(opacity))); shade multiplies the key tint where the pane is nearer the light than the surface. Golden glass_tint; others unchanged. ~1.5 ms @1080p when used. Key light only; lamps and light groups not yet. Replaces the 'Glass shadows tinted by the glass' Not started row. Not committed."
---
