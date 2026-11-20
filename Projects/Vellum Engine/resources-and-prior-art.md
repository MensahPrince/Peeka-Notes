---
id: 9yb80j0oyb
title: "Resources and Prior Art"
sort: 13000
version: 1
---

Papers, talks, books, renderers and tools behind these notes, from NPR research to the Rust and wgpu ecosystem. Part of [Engine Overview](page:3vibzoegdx).

---

## 1\. NPR papers (the classics)

| Paper | For |
| --- | --- |
| Gooch, Gooch, Shirley, Cohen, *A Non-Photorealistic Lighting Model for Automatic Technical Illustration* (1998) | Warm-to-cool shading: [Toon Shading](page:m2km9l0szv) |
| Saito and Takahashi, *Comprehensible Rendering of 3-D Shapes* (1990) | G-buffers and edges from depth: the root of [Outlines and Edges](page:sof6p2vevn) |
| Praun, Hoppe, Webb, Finkelstein, *Real-Time Hatching* (2001) | Tonal art maps: [Hatching and Painterly Rendering](page:6vwu9pwosr) |
| Meier, *Painterly Rendering for Animation* (1996) | Surface-anchored brush strokes |
| DeCarlo, Finkelstein, Rusinkiewicz, Santella, *Suggestive Contours* (2003) | Lines beyond silhouettes |
| Kalnins et al., *WYSIWYG NPR* (2002) and *Coherent Stylized Silhouettes* (2003) | Stroke styles drawn on models; temporally coherent lines |
| Bousseau, Kaplan, Thollot, Sillion, *Interactive Watercolor Rendering with Temporal Coherence* (2006) | Watercolour pipeline |
| Kyprianidis, Kang, Döllner, *Image and Video Abstraction by Anisotropic Kuwahara Filtering* (2009) | Painterly filter |
| Bénard, Bousseau, Thollot, *State-of-the-Art Report on Temporal Coherence for Stylized Animations* (2011) | The motion problems, surveyed |

---

## 2\. Production talks

- **Junya Motomura, "GuiltyGearXrd's Art Style: The X Factor Between 2D and 3D"** (GDC 2015): edited normals, vertex-colour shading controls, inverted-hull lines, animating on held frames. The single most useful NPR talk.
- <b>Sony Pictures Imageworks on <i>Into the Spider-Verse</i></b>: halftones, line work drawn on top of 3D, animating on twos.
- <b>Lucas Pope's devlogs for <i>Return of the Obra Dinn</i></b>: stabilised 1-bit dithering, edge rendering.
- **miHoYo, *Genshin Impact* rendering talks**: cel shading at scale, face shadow maps.
- <b>Team Okami / Clover, <i>Okami</i></b>: sumi-e ink and paper as a whole-game look.

---

## 3\. Books

| Book | For |
| --- | --- |
| Gooch and Gooch, *Non-Photorealistic Rendering* (2001) | The field's first overview |
| Strothotte and Schlechtweg, *Non-Photorealistic Computer Graphics* (2002) | Lines, hatching, illustration, systematically |
| Rosin and Collomosse (eds.), *Image and Video-Based Artistic Stylisation* (2013) | Painterly and filter-based methods |
| *Real-Time Rendering* (Akenine-Möller et al.), chapter on NPR | Compact survey plus all the rendering foundations |
| Eric Lengyel, *Foundations of Game Engine Development* (vol. 1: Mathematics) | [Math for Games](page:l0adi79jhz) done properly |

---

## 4\. Renderers and tools to learn from

| Project | What to read it for |
| --- | --- |
| **Blender Freestyle** | Line extraction, chaining and stroke styles: [Outlines and Edges](page:sof6p2vevn) |
| **Blender EEVEE + Shader to RGB** | How artists build toon shaders node by node |
| **Pencil+ (PSOFT)** | Production line rendering for anime |
| **Godot / Unity toon shader communities** | Practical ramps, outlines, hatching in shader code |
| **Bevy** | An ECS engine in Rust: render graph, extract phase, `naga_oil` |
| **rend3**, **wgpu examples** | Rendering in Rust with wgpu: [Rendering with wgpu](page:hpy2x2spuf) |
| **hecs** | A minimal archetype ECS, readable in an afternoon |

---

## 5\. Tutorials and articles

- **Learn Wgpu** (sotrh.github.io/learn-wgpu): window to lighting and instancing. Check it matches the wgpu version in use.
- **WebGPU Fundamentals** (webgpufundamentals.org): the same concepts in the web API, very clear.
- **Roystan's toon shader tutorial**: ramps, rim and highlights step by step.
- **Alexander Ameye, "5 ways to draw an outline"**: inverted hull, edge detection, jump flood, compared.
- **Ben Golus, "The Quest for Very Wide Outlines"**: jump flood outlines in depth.
- **Glenn Fiedler, "Fix Your Timestep!"**: the accumulator in [Game Loop and Timing](page:kom1zxtzlw).
- **Catherine West, "Using Rust for Game Development"** (RustConf 2018): why ECS and generational indices suit Rust ([Rust Patterns for Engines](page:7nta66y9tk)).
- **Nathan Reed, "Depth Precision Visualized"**: why reverse-Z.

---

## 6\. Tools

- **Blender** for models, edited normals, vertex-colour style channels; exports glTF ([Asset Pipeline](page:s79wv0aek9)).
- **Krita** for ramps, hatch strokes and paper textures.
- **RenderDoc** for inspecting every pass; **Tracy** for the CPU ([Performance and Profiling](page:tmmt1jyud5)).
- **NVIDIA FLIP** for comparing golden images.
- **cargo-watch** to rebuild on save.
