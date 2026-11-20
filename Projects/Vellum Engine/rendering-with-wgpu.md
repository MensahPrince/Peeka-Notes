---
id: hpy2x2spuf
title: "Rendering with wgpu"
sort: 4000
version: 1
---

Getting pixels on screen with wgpu: the objects it is made of, what one frame does, and how Vellum organises its NPR passes into a render graph around a shared prepass. Part of [Engine Overview](page:3vibzoegdx).

---

## 1\. Why wgpu

wgpu implements the WebGPU API in Rust and runs on **Vulkan, Metal, DX12, OpenGL ES and WebGPU** in the browser. One renderer, every platform, with validation that catches mistakes before the driver does. Shaders are written in **WGSL** (naga translates them for each backend).

NPR leans on a few features: several render targets at once (the prepass), integer textures (IDs), `textureLoad` for exact pixel reads in post passes, and compute for mesh lines. All are core WebGPU.

> wgpu's API moves between releases (descriptors gain fields, names change). The snippets here show the shape; check the version's docs and the *Learn Wgpu* tutorial in [Resources and Prior Art](page:9yb80j0oyb).

---

## 2\. The objects

| Object | What it is | Made |
| --- | --- | --- |
| `Instance` | Entry point; picks backends | once |
| `Surface` | The window's drawable area | once per window (after the window exists, see [Game Loop and Timing](page:kom1zxtzlw)) |
| `Adapter` | A physical GPU | once |
| `Device` | Your connection to it; creates everything | once |
| `Queue` | Where command buffers and uploads go | once |
| `Buffer` | GPU memory: vertices, indices, uniforms, storage | per mesh / per frame data |
| `Texture` / `TextureView` | Images, depth buffers, render targets | per asset / per target |
| `Sampler` | How a texture is read (filtering, wrapping) | a few, shared |
| `ShaderModule` | Compiled WGSL | per shader |
| `RenderPipeline` | Shaders + vertex layout + blend + depth state, baked | per material style and pass |
| `ComputePipeline` | A compute shader, baked | per compute pass (edge extraction) |
| `BindGroup` | A set of resources a shader reads (camera uniform, texture + sampler) | per material / per frame |

**Bake early, bind cheaply**: pipelines are expensive to create, so make them at load time (and again on shader reload, [Scripting and Hot Reload](page:sa5xujelgo)); per frame you only set them and draw.

---

## 3\. One frame

```rust
let frame = surface.get_current_texture()?;               // may fail: Outdated/Lost → reconfigure
let view = frame.texture.create_view(&Default::default());
let mut encoder = device.create_command_encoder(&Default::default());
{
    let mut pass = encoder.begin_render_pass(&wgpu::RenderPassDescriptor {
        label: Some("main"),
        color_attachments: &[Some(wgpu::RenderPassColorAttachment {
            view: &view,
            resolve_target: None,
            ops: wgpu::Operations { load: wgpu::LoadOp::Clear(wgpu::Color::WHITE), store: wgpu::StoreOp::Store },
            // newer wgpu versions add fields here
        })],
        depth_stencil_attachment: None, // the prepass owns depth, see below
        ..Default::default()
    });
    pass.set_pipeline(&pipeline);
    pass.set_bind_group(0, &camera_bind_group, &[]);
    pass.set_vertex_buffer(0, mesh.vertices.slice(..));
    pass.set_index_buffer(mesh.indices.slice(..), wgpu::IndexFormat::Uint32);
    pass.draw_indexed(0..mesh.index_count, 0, 0..1);
}
queue.submit([encoder.finish()]);
frame.present();
```

- **Resize**: on `WindowEvent::Resized`, call `surface.configure` with the new size, and recreate every screen-sized target (the prepass, intermediate colour buffers).
- **Present mode**: `Fifo` (vsync) is the safe default; see [Game Loop and Timing](page:kom1zxtzlw).

---

## 4\. Data to the GPU

Uniforms and vertices are `#[repr(C)]` structs cast to bytes with `bytemuck`:

```rust
#[repr(C)]
#[derive(Clone, Copy, bytemuck::Pod, bytemuck::Zeroable)]
struct CameraUniform {
    view_proj: [[f32; 4]; 4], // glam's Mat4::to_cols_array_2d()
    view: [[f32; 4]; 4],      // for view-space normals in the prepass
    position: [f32; 3],
    z_near: f32,              // to linearise reverse-Z depth in post passes
    viewport: [f32; 2],       // pixel size, for constant-width lines
    _pad: [f32; 2],
}

queue.write_buffer(&camera_buffer, 0, bytemuck::bytes_of(&uniform));
```

- WGSL has **alignment rules** (a `vec3<f32>` takes 16 bytes in a uniform): pad structs, as above, or use `vec4`.
- The matrices come from [Math for Games](page:l0adi79jhz); Vellum uses an infinite **reverse-Z** projection, so the depth buffer is `Depth32Float`, cleared to 0, compared with `Greater`.

---

## 5\. The prepass: several targets at once

One pass writes everything the style passes need to know about each pixel ([Engine Overview](page:3vibzoegdx) lists the targets):

```rust
let mut prepass = encoder.begin_render_pass(&wgpu::RenderPassDescriptor {
    label: Some("prepass"),
    color_attachments: &[
        Some(target(&gbuf.normals, wgpu::Color::TRANSPARENT)), // view-space normals
        Some(target(&gbuf.ids, wgpu::Color::TRANSPARENT)),     // object id (r), material id (g), as u32
    ],
    depth_stencil_attachment: Some(wgpu::RenderPassDepthStencilAttachment {
        view: &gbuf.depth,
        depth_ops: Some(wgpu::Operations { load: wgpu::LoadOp::Clear(0.0), store: wgpu::StoreOp::Store }),
        stencil_ops: None,
    }),
    ..Default::default()
});
```

```wgsl
struct PrepassOut {
    @location(0) normal: vec4<f32>,
    @location(1) ids: vec2<u32>,
};

@fragment
fn fs_prepass(in: VertexOut) -> PrepassOut {
    var out: PrepassOut;
    out.normal = vec4(normalize(in.view_normal), 1.0);
    out.ids = vec2(draw.object_id, draw.material_id);
    return out;
}
```

- Integer targets (`Rg32Uint`) cannot be blended or filtered: read them with `textureLoad`, never `textureSample`.
- With the depth already written, the shading pass can use `CompareFunction::Equal` and no depth writes: every pixel is shaded once (no overdraw).
- A debug view that shows each target is the first tool to build: most NPR bugs are visible in the normals or IDs.

---

## 6\. Vellum's render graph

| \# | Pass | Reads | Writes | Notes |
| --- | --- | --- | --- | --- |
| 1 | **Shadow** | scene | shadow map | Thresholded later, not filtered softly ([Toon Shading](page:m2km9l0szv)) |
| 2 | **Prepass** | scene | depth, normals, IDs | Above |
| 3 | **Shading** | scene, shadow map, ramps, hatch maps | colour | Forward, one pipeline per material style ([Toon Shading](page:m2km9l0szv), [Hatching and Painterly Rendering](page:6vwu9pwosr)) |
| 4 | **Hull outlines** | scene | colour | Inverted hulls for meshes that ask for them ([Outlines and Edges](page:sof6p2vevn)) |
| 5 | **Mesh lines** | edge buffers, depth | line layer | Compute extraction + quad draw ([Outlines and Edges](page:sof6p2vevn)) |
| 6 | **Edge detect** | depth, normals, IDs | line layer | Full-screen ([Outlines and Edges](page:sof6p2vevn)) |
| 7 | **Paint** | colour, IDs | colour | Kuwahara, watercolour; optional, often half resolution |
| 8 | **Compose** | colour, line layer, paper | surface | Lines over paint over paper; tone map; egui on top |

- Passes declare what they read and write; the graph orders them, creates transient targets and drops passes whose outputs nobody uses (a style without paint skips pass 7). The Rust side is in [Rust Patterns for Engines](page:7nta66y9tk).
- Before the passes: **extract** drawables (world transform, mesh, material, style overrides) from the [Entity Component System](page:vj3mm21ptz) into flat lists; **sort** opaque by pipeline then material.
- Lines live on their own layer so the compose pass can give them paper grain, colour and wobble independently of the paint.

Frame cost and GPU timing (timestamp queries per pass, RenderDoc captures) are in [Performance and Profiling](page:tmmt1jyud5).

## wgpu 30 notes (checked against 30.0.1 source)

<callout background="yellow" icon="⚠️">

**Older snippets break on wgpu 30.** These are the changes Vellum ran into in M2.

- `surface.get_current_texture()` returns a `CurrentSurfaceTexture` enum (`Success`, `Suboptimal`, `Timeout`, `Occluded`, `Outdated`, `Lost`, `Validation`), not a `Result`.
- Presenting goes through the queue: `queue.present(frame)`.
- `DepthStencilState::depth_write_enabled` and `depth_compare` are `Option`s.
- `PipelineLayoutDescriptor` takes `bind_group_layouts: &[Some(&layout)]` and `immediate_size` (formerly push constants).
- Render pipelines and render passes take `multiview_mask: None`; color attachments take `depth_slice: None`.
- `RequestAdapterOptions` gained `apply_limit_buckets`: use `..Default::default()`.
- `InstanceDescriptor::new_without_display_handle_from_env()` is enough for Vulkan on Wayland.
- `buffer.map_async(mode, .., cb)` and `buffer.get_mapped_range(..)` work on the buffer directly; the latter returns a `Result`.

</callout>

<callout background="yellow" icon="⚠️">

**WGSL uniform padding (M5):** pad with scalars, not a `vec3`. A `vec3<u32>` aligns to 16 bytes, so `{ mode: u32, _pad: vec3<u32> }` is 32 bytes in WGSL but 16 in Rust, and wgpu rejects the buffer as too small.

</callout>
