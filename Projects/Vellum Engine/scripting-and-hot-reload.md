---
id: sa5xujelgo
title: "Scripting and Hot Reload"
sort: 11000
version: 1
---

Changing the look without restarting: style presets with live sliders, shaders and textures reloaded on save, and (optionally) renderer code as a hot-reloaded library. Part of [Engine Overview](page:3vibzoegdx).

---

## 1\. Why

NPR is **art direction**. A look is found by nudging a shadow threshold, repainting a ramp, thinning a line, swapping a paper texture, hundreds of times. Every one of those should show within a frame, on the model, in motion.

| What changes | How it reloads | Speed |
| --- | --- | --- |
| **Style parameters** (thresholds, widths, colours) | egui sliders write the uniform; saved as RON presets | Instant |
| **Textures** (ramps, hatch maps, paper) | File watcher, asset reload ([Asset Pipeline](page:s79wv0aek9)) | Instant |
| **WGSL shaders** | File watcher, validate, rebuild pipelines | Under a second |
| **Rust renderer code** | `cdylib` hot reload, or restart | Seconds |

Start with the first three: they cover nearly all look development.

---

## 2\. Style presets

A style is data:

```rust
#[derive(serde::Serialize, serde::Deserialize, Clone)]
pub struct Style {
    pub shading: Shading,               // Toon { ramp, threshold, rim, highlight } or Gooch { warm, cool }
    pub edges: Option<EdgeParams>,      // depth / normal thresholds, width, colour
    pub hulls: bool,
    pub hatching: Option<Hatching>,     // tam path, scale, tone curve
    pub paint: Option<Paint>,           // Kuwahara radius, watercolour wobble, edge darkening
    pub paper: Option<String>,          // paper texture path
    pub boil_hz: f32,
}
```

```ron
Style(
    shading: Toon(ramp: "ramps/warm_two_tone.png", threshold: 0.1, rim: 0.3, highlight: 0.05),
    edges: Some(EdgeParams(depth_threshold: 0.02, normal_threshold: 0.4, width_px: 1.5, color: (0.1, 0.08, 0.12))),
    hulls: true,
    hatching: None,
    paint: None,
    paper: Some("paper/cold_press.png"),
    boil_hz: 0.0,
)
```

- An egui panel shows every field as a slider or picker; changes write the style uniform buffer that frame.
- **Save** writes the RON back; presets are swapped with Tab ([Input Handling](page:oaumtmnlq4)).
- Per-object overrides are components on entities ([Entity Component System](page:vj3mm21ptz)); the preset is the default.
- The preset decides which passes run: no `paint` means the render graph culls the paint pass ([Rendering with wgpu](page:hpy2x2spuf)).

---

## 3\. Shader hot reload

Watch the `shaders/` directory; on change, validate first and only then replace the pipeline:

```rust
fn reload_shader(device: &wgpu::Device, path: &std::path::Path) -> Result<wgpu::ShaderModule, String> {
    let source = std::fs::read_to_string(path).map_err(|e| e.to_string())?;
    // Validate with naga before handing it to wgpu: a bad shader would otherwise
    // hit wgpu's error handler, which panics by default.
    let module = naga::front::wgsl::parse_str(&source).map_err(|e| e.emit_to_string(&source))?;
    naga::valid::Validator::new(naga::valid::ValidationFlags::all(), naga::valid::Capabilities::all())
        .validate(&module)
        .map_err(|e| format!("{e:?}"))?;
    Ok(device.create_shader_module(wgpu::ShaderModuleDescriptor {
        label: path.to_str(),
        source: wgpu::ShaderSource::Wgsl(source.into()),
    }))
}
```

- On error: **keep the old pipeline**, show the message (with line and column) in the overlay. A typo never blanks the screen.
- Use the **same naga version** wgpu depends on, or validation and compilation disagree.
- Also wrap pipeline creation in `device.push_error_scope(wgpu::ErrorFilter::Validation)` / `pop_error_scope`: layout mismatches only show there.
- **Imports and variants**: WGSL has no `#include`. `naga_oil` (Bevy's) adds `#import` and `#ifdef`, so shared code (lighting, octahedral normals, `fwidth` thresholds) lives in one file and styles compile as variants (`HATCHING`, `RIM`).

---

## 4\. Hot-reloading Rust code (optional)

For iterating on renderer code itself: split the app into a host (window, device, scene memory) and a `cdylib` with passes or systems, reloaded with `libloading`:

```rust
type FrameFn = unsafe extern "C" fn(ctx: *mut FrameContext);

struct Plugin { lib: libloading::Library, frame: FrameFn }

fn load(path: &std::path::Path) -> Plugin {
    // Copy first: some platforms lock a loaded library, and the build would fail to overwrite it.
    let copy = path.with_extension("loaded");
    std::fs::copy(path, &copy).unwrap();
    unsafe {
        let lib = libloading::Library::new(&copy).unwrap();
        let frame = *lib.get::<FrameFn>(b"vellum_frame").unwrap();
        Plugin { lib, frame }
    }
}
```

### Why it is fragile

- **No stable Rust ABI**: host and library must be built by the same compiler with the same dependency versions.
- **State lives in the host**: GPU objects created by the library must be dropped before it unloads.
- `TypeId`s, vtables and function pointers from the old library dangle after unloading.

Development builds only; `hot-lib-reloader` wraps the pattern. Most of the time, shader reload is enough.

---

## 5\. Scripting (later)

For scene setup and procedural animation without recompiling: `rhai` (pure Rust, easy to embed) or Lua via `mlua`. Expose a narrow API (`spawn(mesh, transform)`, `set_style(entity, preset)`, `keyframe(…)`), run scripts at a fixed point in the frame ([Game Loop and Timing](page:kom1zxtzlw)), and check entity handles at the boundary ([Rust Patterns for Engines](page:7nta66y9tk)).

---

## 6\. What Vellum does

1. Style presets in RON, edited live with egui.
1. Ramps, hatch maps and paper reloaded on save.
1. WGSL reloaded on save, validated, the old pipeline kept on error.
1. Later, maybe: a dylib for passes, Rhai for scenes.

A reload must never cost a hitch in a release build, because release builds don't do it ([Performance and Profiling](page:tmmt1jyud5)).

## What Vellum does (M10a)

<callout icon="📌">

**Reload by rebuilding the renderer.** Any change (shot, glTF or shader) builds a whole new `Renderer` inside a wgpu validation error scope. It replaces the old one only if no error was raised, so a broken shader logs its parse error with line and column and the last good renderer keeps drawing. Building takes a fraction of a second, so there's no per-pass reload bookkeeping.

</callout>

- **Watching:** modification times are polled twice a second (no `notify` crate; its current release is a pre-release).
- **Shaders:** `ShaderLibrary` holds the WGSL sources. Release builds embed them; dev builds read `crates/vellum/src/shaders/`. A test checks the two are identical.
- **Shot files:** RON with implicit `Some`. That made `lines: None` ambiguous ("not set"), so the no-lines mask is spelled `Off`. Misspelt fields are errors; misspelt object or material names are warnings.
