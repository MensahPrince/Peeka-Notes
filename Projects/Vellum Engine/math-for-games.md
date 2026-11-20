---
id: l0adi79jhz
title: "Math for Games"
sort: 5000
version: 1
---

The maths a stylised renderer runs on, with glam and WGSL: vectors, matrices, quaternions, transforms, the camera, and the per-pixel maths NPR passes lean on (depth, normals, derivatives, luminance). Part of [Engine Overview](page:3vibzoegdx).

---

## 1\. glam in one table

| Type | Use |
| --- | --- |
| `Vec2`, `Vec3`, `Vec4` | Positions, directions, colours |
| `Vec3A` | A 16-byte aligned `Vec3`, faster with SIMD (for hot arrays) |
| `Quat` | Rotations |
| `Mat3`, `Mat4` | Transforms; **column-major** |
| `Affine3A` | A rotation + scale + translation without the projective row: cheaper than `Mat4` for model transforms |

glam is column-major and uses column vectors: `m * v` transforms `v`, and `a * b` applies `b` first, then `a`. WGSL matrices are column-major too, so `to_cols_array_2d()` goes straight to a uniform.

---

## 2\. Conventions to fix on day one

Pick them, write them down, never mix them:

| Choice | Vellum |
| --- | --- |
| Handedness | Right-handed (glam's `_rh` functions) |
| Up | \+Y |
| Forward | −Z (the camera looks down −Z) |
| Units | 1 unit = 1 metre (line widths and hatch scales are tuned in these) |
| Clip-space depth | Reverse-Z, infinite far plane: `Mat4::perspective_infinite_reverse_rh` |
| Normals in the prepass | View space, unit length |
| Angles | Radians in code; degrees only in UI |
| Colour | Linear in shaders; sRGB only in textures marked as such and the surface |

glTF files are right-handed, +Y up, metres, so they import without conversion ([Asset Pipeline](page:s79wv0aek9)).

---

## 3\. Vectors you will use every day

```rust
use glam::{Vec3, Quat, Mat4};

let to_light = (light_pos - position).normalize_or_zero(); // never divide by a zero length
let lit_side = normal.dot(to_light) > 0.0;                 // dot: facing the light?
let side = forward.cross(Vec3::Y);                         // cross: a perpendicular axis
let halfway = a.lerp(b, 0.5);                              // linear blend
let dist2 = position.distance_squared(camera);             // compare squared, skip the sqrt
```

- **Dot product**: projection and "how aligned"; 1 same direction, 0 perpendicular, −1 opposite. n·l is the input of all of [Toon Shading](page:m2km9l0szv); n·v finds silhouettes and rims.
- **Cross product**: the axis perpendicular to two vectors; face normals, tangent frames for hatching.
- Compare **squared** distances in hot loops ([Performance and Profiling](page:tmmt1jyud5)).

---

## 4\. Rotations: use quaternions

Euler angles (yaw, pitch, roll) are fine for input and UI but suffer **gimbal lock** and blend badly. Store rotations as `Quat`:

```rust
let yaw = Quat::from_rotation_y(angle);
let combined = yaw * pitch;                 // pitch first, then yaw
let rotated = combined * Vec3::NEG_Z;       // rotate a vector
let smooth = from.slerp(to, t);             // shortest-path blend
let renormalised = q.normalize();           // after many multiplications, fight drift
```

`slerp` is what animation sampling and fixed-step interpolation use for rotations ([Game Loop and Timing](page:kom1zxtzlw)).

---

## 5\. Transforms and the hierarchy

```rust
#[derive(Clone, Copy)]
pub struct Transform { pub translation: Vec3, pub rotation: Quat, pub scale: Vec3 }

impl Transform {
    pub fn matrix(&self) -> Mat4 {
        Mat4::from_scale_rotation_translation(self.scale, self.rotation, self.translation)
    }
}
```

- An entity's `Transform` is **local** (relative to its parent); `GlobalTransform` is computed: `parent_global * local`.
- A **transform propagation** system walks parents before children once per frame and writes `GlobalTransform` ([Entity Component System](page:vj3mm21ptz)).
- Normals transform by the **inverse transpose** of the model matrix; with uniform scale the model matrix itself is fine (the inverted-hull shader in [Outlines and Edges](page:sof6p2vevn) assumes it).
- Non-uniform scale on a parent skews children and their normals: avoid it, or pass a normal matrix.

---

## 6\. The camera

```rust
let view = Mat4::look_at_rh(eye, target, Vec3::Y);
let proj = Mat4::perspective_infinite_reverse_rh(fov_y_radians, width / height, z_near);
let view_proj = proj * view; // to the GPU as a uniform
```

- **Reverse-Z** (far at 0, near at 1, a float depth buffer) spreads precision evenly: edge detection on depth needs that precision far away.
- On resize, recompute `proj` with the new aspect ratio ([Rendering with wgpu](page:hpy2x2spuf)).
- An **orthographic** camera (`Mat4::orthographic_rh`) suits technical illustration and isometric looks; lines and hatching then keep their size at any depth.

---

## 7\. Per-pixel maths for NPR

| Need | Formula | Used in |
| --- | --- | --- |
| View distance from reverse-Z infinite depth | `z_near / depth` | [Outlines and Edges](page:sof6p2vevn) |
| Half-Lambert | `n·l * 0.5 + 0.5` | Ramp lookup, [Toon Shading](page:m2km9l0szv) |
| Silhouette test (face) | `dot(n, v) ≈ 0`; edge between faces with opposite signs | Mesh lines |
| Crease test | `dot(n1, n2) < cos(angle)` | Mesh lines |
| Rim | `pow(1 - saturate(dot(n, v)), p)` | [Toon Shading](page:m2km9l0szv) |
| Luminance (linear sRGB) | `dot(rgb, vec3(0.2126, 0.7152, 0.0722))` | Tone for hatching and halftone |
| Pixel footprint | `fwidth(x) = abs(dpdx(x)) + abs(dpdy(x))` | Antialiased thresholds everywhere |

### Derivatives, the NPR workhorse

`dpdx`, `dpdy` and `fwidth` tell a fragment shader how fast a value changes across one pixel. Any hard threshold (`step`) becomes an antialiased one by widening it by `fwidth`:

```wgsl
let w = fwidth(value);
let mask = smoothstep(threshold - w, threshold + w, value);
```

They are only valid in **uniform control flow**: compute them before any `if` that depends on per-pixel data.

### Normal encoding

A view-space unit normal fits in two numbers: **octahedral encoding** maps the sphere onto a square, so an `Rg16Float` target replaces `Rgba16Float` and halves the prepass bandwidth ([Performance and Profiling](page:tmmt1jyud5)).

---

## 8\. Floating point traps

- `0.1 + 0.2 != 0.3`: compare with a tolerance (`(a - b).abs() < 1e-5`) or glam's `abs_diff_eq`.
- `normalize` of a zero vector is NaN, and NaN in a normal target makes edges appear everywhere: guard with `normalize_or_zero` on the CPU and `max(length(v), 1e-6)` in WGSL.
- Golden-image tests need the same operations in the same order; GPUs still differ slightly, so compare with a tolerance ([Performance and Profiling](page:tmmt1jyud5)).
