---
id: m2km9l0szv
title: "Toon Shading"
sort: 16000
version: 1
---

Turning light into a few flat tones: ramps, hard terminators that don't alias, rim light, stylised highlights, Gooch shading and the normal editing that makes faces read. Part of [Engine Overview](page:3vibzoegdx).

---

## 1\. The idea

Photoreal shading answers "how much light reaches this point". Toon shading asks the same question and then **quantises** the answer: lit, shadowed, maybe a mid-tone and a highlight. The art lives in *where* the boundaries fall and *what colours* sit either side of them, not in the lighting maths.

| Ingredient | What it controls |
| --- | --- |
| **Ramp** | How n·l maps to tone: bands, their colours, how soft the boundaries are |
| **Terminator** | Where light turns to shadow; often biased per vertex or per material |
| **Shadow colour** | A hue-shifted colour, not black (purple-blue shadows on warm skin) |
| **Rim light** | A bright edge on the silhouette, separating figures from background |
| **Highlight** | A hard-edged specular shape, sometimes painted as a texture |
| **Normals** | Edited so shadows fall as an illustrator would draw them |

---

## 2\. Ramps

The simplest flexible form: compute n·l, remap it to 0..1 and look up a 1D **ramp texture**.

```wgsl
struct Camera { view_proj: mat4x4<f32>, position: vec3<f32>, _pad: f32 };
struct Light { direction: vec3<f32>, intensity: f32, color: vec3<f32>, _pad: f32 };
@group(0) @binding(0) var<uniform> camera: Camera;
@group(0) @binding(1) var<uniform> light: Light;

@group(1) @binding(0) var albedo: texture_2d<f32>;
@group(1) @binding(1) var albedo_samp: sampler;
@group(1) @binding(2) var ramp: texture_2d<f32>;
@group(1) @binding(3) var ramp_samp: sampler; // clamp-to-edge; nearest for hard bands, linear for soft

@fragment
fn fs_toon(in: VertexOut) -> @location(0) vec4<f32> {
    let n = normalize(in.world_normal);
    let l = normalize(-light.direction);
    let v = normalize(camera.position - in.world_pos);

    let ndl = dot(n, l);                                                  // −1..1
    let tone = textureSample(ramp, ramp_samp, vec2(ndl * 0.5 + 0.5, 0.5)).rgb; // half-Lambert remap

    let rim = pow(1.0 - saturate(dot(n, v)), 4.0) * smoothstep(0.0, 0.1, ndl); // only on the lit side
    let base = textureSample(albedo, albedo_samp, in.uv).rgb;
    return vec4(base * tone * light.color + rim * 0.3, 1.0);
}
```

- A ramp is an **asset**, painted by an artist: a few pixels wide is enough ([Asset Pipeline](page:s79wv0aek9) covers sampling and colour space).
- A 2D ramp (n·l on X, a second value on Y: material, distance, time of day) gives one texture for many looks.
- `ndl * 0.5 + 0.5` (half-Lambert) keeps the shadow side from collapsing into one flat value.

---

## 3\. A hard terminator without jaggies

A plain `step(threshold, ndl)` aliases: the boundary crawls as the model moves. Widen the step by the pixel's own rate of change:

```wgsl
let w = fwidth(ndl);                                      // how much ndl changes across one pixel
let lit = smoothstep(threshold - w, threshold + w, ndl);  // a one-pixel-wide transition at any distance
let color = mix(shadow_color, lit_color, lit);
```

The same trick antialiases every hard edge in this page and in [Hatching and Painterly Rendering](page:6vwu9pwosr). `fwidth` and friends are in [Math for Games](page:l0adi79jhz).

---

## 4\. Highlights and rim

```wgsl
let h = normalize(l + v);
let ndh = dot(n, h);
let wh = fwidth(ndh);
let spec = smoothstep(1.0 - size - wh, 1.0 - size + wh, ndh); // a hard-edged blob, its size an art parameter
```

- **Stylised highlights**: stretch or shape them with a texture lookup in tangent space (anime hair "angel ring" bands).
- **Rim light** multiplied by the lit term reads as backlight; unmasked it reads as a glow. Give artists both.

---

## 5\. Several lights and shadows

Banding each light separately and adding the results makes stripes. Instead:

1. Sum the light that reaches the point (keep it scalar, or luminance plus a colour).
1. Band **once**.
1. Cast shadows from a shadow map act as another input to the same band: `min(lit, shadow)`, so a cast shadow uses the shadow tone, never a darker extra one.

Hard bands make **shadow acne** and shimmering obvious: use a generous slope-scaled bias, and PCF filtered then thresholded with the same `fwidth` trick.

---

## 6\. Normals are art

The face of an anime character shaded with its true normals gets blotchy shadows around the nose and mouth. Studios fix the **normals**, not the shader:

- **Normal transfer**: copy normals from a simpler proxy (a sphere or ellipsoid around the head) so shadows sweep cleanly (Blender's Data Transfer modifier, exported as glTF normals).
- **Per-vertex shading offsets**: Arc System Works' *Guilty Gear Xrd* stored a shadow-threshold bias in a vertex colour channel, letting artists paint where shadow always or never falls; another channel drove outline width ([Outlines and Edges](page:sof6p2vevn)).
- **Face-forward light**: drive character shading from a per-character key light that follows the camera a little, so faces stay readable whatever the scene light does.

---

## 7\. Gooch shading

For technical illustration (Gooch et al., 1998): instead of light to dark, shade **warm to cool**, keeping the darkest values free for lines.

```wgsl
let k = (1.0 + dot(n, l)) * 0.5;
let cool = vec3(0.0, 0.0, 0.55) + 0.25 * base;
let warm = vec3(0.3, 0.3, 0.0) + 0.5 * base;
let color = mix(cool, warm, k);
```

Pair it with silhouettes and creases from [Outlines and Edges](page:sof6p2vevn).

---

## 8\. Where it runs

Toon shading is a **material**, chosen per mesh: it runs in the forward shading pass after the prepass ([Rendering with wgpu](page:hpy2x2spuf)). Its parameters (ramp, threshold, rim, highlight size) are a style preset that reloads live ([Scripting and Hot Reload](page:sa5xujelgo)).

## What Vellum does (M6)

<callout icon="📌">

**Light value, not colour.** The shade pass writes a `light` texture: r = ramped light (0 / 0.5 / 1), g = cast-shadow mask, b = rim mask, a = highlight mask. Palettes (M8) turn it into colour.

</callout>

- **Ramp:** N·L cut at 0.1 (2-tone), or at 0.05 and 0.5 (3-tone). `aa_step` antialiases each edge over about a pixel using `fwidth`, so the fullscreen shader has no early returns.
- **Cast shadows:** a 2048² orthographic shadow map, fitted to the drawn objects' bounds. Surfaces are pushed 1.5 texels along the normal, plus a small depth bias. The hardware 2×2 comparison filter gives a soft edge, which the same threshold turns into a hard but smooth shape.
- **Rim:** `1 − n·v` alone flags whole floors at grazing angles. Vellum also requires the pixel to be in front of a silhouette: the second difference of reverse-Z depth over ±4 px is clearly negative. Reverse-Z depth is linear across the screen on any plane, so floors never qualify.
- **Highlight:** the Blinn-Phong lobe cut at 0.97, so it reads as a painted spot.
- **Proxy normals:** done in Blender with a Data Transfer modifier (custom normals from a simplified mesh). Vellum uses exported normals as they are. Steps are in `docs/blender-export.md` in the repo.
