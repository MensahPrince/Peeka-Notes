---
id: sof6p2vevn
title: "Outlines and Edges"
sort: 14000
version: 1
---

Ink lines: which edges to draw, three ways to find them (inverted hulls, screen-space detection, mesh edges), and how to give lines weight and character. Part of [Engine Overview](page:3vibzoegdx).

---

## 1\. Which lines

| Line | Where it is | Found from |
| --- | --- | --- |
| **Silhouette** | Where the surface turns away from the eye | Front-face / back-face boundary, or a depth jump |
| **Outline** | The silhouette of a whole object against what is behind | Object ID change, depth jump |
| **Crease** | Sharp edges inside the shape (a box's corners) | Dihedral angle between faces, or a normal jump |
| **Material boundary** | Where colour regions meet (a sleeve's cuff) | Material ID change |
| **Suggestive contour** | Almost-silhouettes that convey shape (DeCarlo et al., 2003) | Curvature in the view direction |
| **Painted lines** | Detail the artist wants (wrinkles, seams) | Textures: lines drawn into UV space |

Most looks need silhouettes, outlines and creases; the rest are refinements.

---

## 2\. Inverted hull

Draw the mesh a second time, pushed outward along its normals, with **front faces culled** and a flat line colour. Only the rim of the bigger, inside-out copy shows around the real one.

```wgsl
@vertex
fn vs_outline(@location(0) pos: vec3<f32>, @location(1) outline_normal: vec3<f32>) -> @builtin(position) vec4<f32> {
    let clip = camera.view_proj * model.matrix * vec4(pos, 1.0);
    let world_n = normalize((model.matrix * vec4(outline_normal, 0.0)).xyz); // uniform scale only
    let clip_n = (camera.view_proj * vec4(world_n, 0.0)).xy;
    // Offset in clip space, times w: a constant width in pixels at any distance.
    let offset = normalize(clip_n) / camera.viewport * outline.width_px * 2.0 * clip.w;
    return vec4(clip.xy + offset, clip.zw);
}
```

- Pipeline: `cull_mode: Some(Face::Front)`, depth test on, drawn after the mesh.
- **Split normals break it**: at a hard edge the two copies of a vertex move apart and the hull cracks. Bake **smoothed normals** into a separate attribute (`outline_normal`) at import ([Asset Pipeline](page:s79wv0aek9)).
- Width can come from a vertex colour channel: thin at fingertips, thick at the jaw (the *Guilty Gear Xrd* approach, see [Toon Shading](page:m2km9l0szv)).
- Cheap, per object, colourable per material; but no creases, and lines poke through nearby geometry.

---

## 3\. Screen-space edge detection

A full-screen pass reads the prepass targets (depth, normals, IDs, see [Rendering with wgpu](page:hpy2x2spuf)) and marks pixels where they change sharply.

```wgsl
@group(0) @binding(0) var depth_tex: texture_depth_2d;
@group(0) @binding(1) var normal_tex: texture_2d<f32>;
@group(0) @binding(2) var id_tex: texture_2d<u32>;

fn view_depth(p: vec2<i32>) -> f32 {
    let q = clamp(p, vec2(0), params.size - vec2(1));
    return params.z_near / max(textureLoad(depth_tex, q, 0), 1e-6); // infinite reverse-Z → view distance
}

fn normal_at(p: vec2<i32>) -> vec3<f32> {
    return textureLoad(normal_tex, clamp(p, vec2(0), params.size - vec2(1)), 0).xyz;
}

fn id_at(p: vec2<i32>) -> u32 {
    return textureLoad(id_tex, clamp(p, vec2(0), params.size - vec2(1)), 0).r;
}

@fragment
fn fs_edges(@builtin(position) frag: vec4<f32>) -> @location(0) vec4<f32> {
    let p = vec2<i32>(frag.xy);
    let a = p; let b = p + vec2(1, 1); let c = p + vec2(1, 0); let d = p + vec2(0, 1);

    // Roberts cross on depth, relative to distance: far objects need bigger jumps.
    let dz = sqrt(pow(view_depth(b) - view_depth(a), 2.0) + pow(view_depth(d) - view_depth(c), 2.0)) / view_depth(a);
    let nb = normal_at(b) - normal_at(a);
    let nd = normal_at(d) - normal_at(c);
    let dn = sqrt(dot(nb, nb) + dot(nd, nd));
    let did = id_at(a) != id_at(b) || id_at(c) != id_at(d);

    let edge = max(max(step(params.depth_threshold, dz), step(params.normal_threshold, dn)), select(0.0, 1.0, did));
    return vec4(params.line_color, edge);
}
```

- **Grazing angles** make false depth edges on floors and walls seen edge-on: scale the depth threshold by `1 / max(dot(n, v), 0.1)`.
- **Line width** is one pixel and aliased: widen by sampling a bigger kernel (Sobel at a distance of `width` pixels), or render edges at 2× and downsample.
- **Wide object outlines** (selection, a thick comic border): the **jump flood** algorithm computes distance to the nearest edge pixel in log₂(width) passes, then any width is a threshold on that distance.
- Works on anything that writes the prepass, including terrain and particles; no per-mesh setup.

---

## 4\. Mesh edges as geometry

For lines that can be stylised as strokes, find them on the mesh:

- **Silhouette edge**: an edge whose two faces disagree about facing the eye: `dot(n1, v) * dot(n2, v) < 0`.
- **Crease edge**: the angle between the two faces' normals above a threshold (`dot(n1, n2) < cos(30°)`); fixed per mesh, so computed once at load.
- **Border edge**: an edge with only one face.

Per frame, a compute shader tests every edge (each edge stores its two vertices and two faces, built at import) and appends visible ones to a buffer. A vertex shader expands each into a screen-space **quad** of the chosen width; an instanced draw with `draw_indirect` draws them all.

- **Visibility**: depth test against the prepass with a small bias, so lines on the surface are not hidden by it.
- **Strokes**: chaining edges into long paths (on the CPU) allows taper, brush textures along the length, and wobble. This is what Blender's **Freestyle** and Pencil+ do; it costs more and needs care to stay stable frame to frame ([Hatching and Painterly Rendering](page:6vwu9pwosr)).

---

## 5\. Line weight

Ink lines carry meaning through their weight:

| Cue | Rule of thumb |
| --- | --- |
| Silhouette vs interior | Outer silhouettes thicker, creases thinner |
| Distance | Thinner far away, but clamp to ≥ 1 px or lines vanish |
| Light | Thicker on the shadow side, thinner or broken where light hits |
| Importance | Characters heavier than background |
| Tapering | Strokes thin at their ends, as a brush lifts |

---

## 6\. Choosing

|  | Inverted hull | Screen-space | Mesh edges |
| --- | --- | --- | --- |
| Silhouettes | Yes | Yes | Yes |
| Creases | No | Yes (normal jumps) | Yes |
| Constant pixel width | Yes | Yes | Yes |
| Stylised strokes | No | Hard | **Yes** |
| Cost | One extra draw per mesh | One full-screen pass | Compute + many quads |
| Setup per mesh | Smoothed normals | None | Edge adjacency |

**Vellum's plan**: screen-space edges for every object (cheap, catches everything), inverted hulls for characters that want art-directed width and colour, mesh strokes later for pencil and ink styles.

## What Vellum does (M7)

<callout icon="📌">

**Screen-space only.** Inverted hulls were skipped: uneven width, and screen-space silhouettes already cover them.

</callout>

| Kind | Detected by | Width at 1080p |
| --- | --- | --- |
| Silhouette | Neighbour depth breaks the planar prediction `2d − d_behind` by more than 2% | 2\.5 px |
| Object border | Object ID changes; marked on the nearer side | 2 px |
| Material border | Material ID changes within one object | 1\.5 px |
| Crease | Neighbour normals 25°–60° apart; strength grows with the angle | 0\.6–1.5 px (tapers) |

- **Width** also scales with distance (×0.6 to ×1.6, against the scene centre) and with output height.
- **Masks:** the Blender material custom property `vellum_lines` (0 = outlines only, −1 = none) is exported as glTF extras.
- **Pen:** round and antialiased, up to 8 px. The 2D search is split into rows then columns (`line_rows`), 18 reads instead of 81. That took a 1080p frame from 36 ms to 19 ms on an Intel HD 620.
- **Wobble:** smooth value noise shifts the pen up to 0.6 px; the pattern holds 5 frames until M9's clock.
