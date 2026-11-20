---
id: 6vwu9pwosr
title: "Hatching and Painterly Rendering"
sort: 15000
version: 1
---

When tone becomes marks: hatching with tonal art maps, halftones and dithering, painterly filters, watercolour, paper, and keeping it all stable while things move. Part of [Engine Overview](page:3vibzoegdx).

---

## 1\. Tone as marks

A drawing has no grey: it has **density of marks**. Every style here takes the lit value from shading ([Toon Shading](page:m2km9l0szv)) and turns it into ink coverage: more strokes, bigger dots, thicker paint.

| Style | Marks | Lives in |
| --- | --- | --- |
| Hatching, cross-hatching, engraving | Lines at one or more angles | Object space (textures on the surface) |
| Stippling | Dots | Object or screen space |
| Halftone, comic | Dots on a grid | Screen space |
| 1-bit dithering | Pixel patterns | Screen space (stabilised) |
| Painterly | Brush strokes | Screen space filters or surface-anchored particles |
| Watercolour | Pigment pooling, granulation, edges | Screen space with paper |

---

## 2\. Hatching with tonal art maps

Praun et al. (2001): a set of hatch textures, one per **tone**, where each darker tone contains every stroke of the lighter ones plus more. Blending between neighbours then fades strokes in and out instead of swapping patterns. Each tone also has hand-built **mip levels** where strokes keep their pixel width (generated mipmaps would blur lines into grey).

Six tones fit in two RGB textures:

```wgsl
@group(1) @binding(4) var tam_light: texture_2d<f32>; // ink coverage of tones 1–3 in r, g, b
@group(1) @binding(5) var tam_dark: texture_2d<f32>;  // tones 4–6
@group(1) @binding(6) var tam_samp: sampler;          // repeat, trilinear

fn hatch(uv: vec2<f32>, lit: f32) -> f32 {
    let a = textureSample(tam_light, tam_samp, uv).rgb;
    let b = textureSample(tam_dark, tam_samp, uv).rgb;
    var levels = array<f32, 7>(0.0, a.r, a.g, a.b, b.r, b.g, b.b);
    let t = saturate(1.0 - lit) * 6.0;   // 0 = paper, 6 = darkest tone
    let i = u32(floor(t));
    let j = min(i + 1u, 6u);
    let ink = mix(levels[i], levels[j], fract(t));
    return 1.0 - ink;                    // paper brightness, multiplied onto the paper colour
}
```

- **Stroke direction**: UV space makes hatching follow the parametrisation; artists can lay UVs along the form, or strokes follow principal curvature directions (computed at import) for an engraving look.
- No UVs? **Triplanar** mapping in object space, at the cost of seams where projections blend.
- The TAM textures are assets with special mip handling: [Asset Pipeline](page:s79wv0aek9).

---

## 3\. Halftone and dithering

Screen-space halftone dots (comic Ben-Day dots, *Spider-Man: Into the Spider-Verse*):

```wgsl
fn halftone(frag: vec2<f32>, tone: f32, cell: f32, angle: f32) -> f32 {
    let r = mat2x2<f32>(cos(angle), -sin(angle), sin(angle), cos(angle));
    let p = r * frag / cell;
    let d = length(fract(p) - 0.5);          // distance to the nearest dot centre
    let radius = sqrt(1.0 - tone) * 0.7071;  // dot area grows with darkness
    let w = fwidth(d);
    return smoothstep(radius - w, radius + w, d); // 1 = paper, 0 = ink
}
```

One halftone per ink (cyan, magenta, yellow, black at different angles) gives the printed look; slight misregistration sells it.

**1-bit dithering**: an ordered (Bayer) or blue-noise threshold per pixel. In screen space the pattern stays still while the world slides under it. *Return of the Obra Dinn* (Lucas Pope) fixed this by mapping the dither pattern onto a sphere around the camera, so it rotates with the view.

---

## 4\. Painterly

- **Kuwahara filter**: for each pixel, look at four quadrants around it and output the mean of the one with the lowest variance. Flattens texture into patches and keeps edges. The **anisotropic** Kuwahara (Kyprianidis et al., 2009) stretches the kernel along the local structure tensor: strokes follow the forms.
- **Brush-stroke particles** (Meier, 1996): scatter particles on surfaces at load, render each as an oriented brush-texture billboard coloured by the shaded image beneath it. Anchored to the surface, so strokes move with objects: the most coherent painterly technique, and the costliest.
- Kuwahara is a full-screen pass with a big kernel: run it at half resolution for 4K ([Performance and Profiling](page:tmmt1jyud5)).

---

## 5\. Watercolour

After Bousseau et al. (2006):

1. **Wobble**: offset the image's UVs by low-frequency noise, so shapes don't sit exactly on their geometry.
1. **Edge darkening**: pigment pools at the edges of a wash. Darken where the gradient of a blurred copy of the colour is large.
1. **Pigment density**: modulate colour with paper texture and low-frequency turbulence noise (`colour * (1 + density * (paper - 0.5))`).
1. **Granulation**: pigment settles in the paper's valleys: higher frequency, driven by the paper's height texture.
1. **Bleeding**: blur colour across edges where the ID buffer says regions are "wet", not across hard lines.

---

## 6\. Paper and media

- A **paper texture** multiplied over everything sells most styles instantly. Its grain also modulates lines and hatching: ink skips where the paper is high.
- Screen-space paper is still while the camera moves (shower-door); a small offset following camera motion feels like panning across a drawing.
- Different media per layer: pencil lines over watercolour washes over paper, composited in [Rendering with wgpu](page:hpy2x2spuf)'s compose pass.

---

## 7\. Temporal coherence

The hard part of NPR in motion.

| Problem | What it looks like | Fixes |
| --- | --- | --- |
| **Shower door** | The world slides behind a fixed screen-space texture | Object-space marks; or move the texture with the camera |
| **Swimming** | Marks slide over surfaces | Anchor to UVs or particles on the surface |
| **Popping** | Strokes appear or vanish abruptly | Nested tones (TAMs) and fades; hysteresis on thresholds |
| **Scale** | Strokes grow as the camera nears | Mip-level TAMs; fractal (infinite zoom) patterns |
| **Too still** | Looks like a 3D model with a filter | Deliberate **boil**: re-seed noise on a stepped clock |

A hand-drawn film redraws every frame, so lines tremble: reproduce it by changing the noise seed 8–12 times a second, on the stepped clock from [Game Loop and Timing](page:kom1zxtzlw), not every frame.
