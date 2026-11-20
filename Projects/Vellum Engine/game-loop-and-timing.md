---
id: kom1zxtzlw
title: "Game Loop and Timing"
cover: "_assets/6dc5a7d6c00936313762c2b73327b725.jpg"
cover_position: 50
sort: 2000
version: 1
---

How a frame happens: the OS's event loop, the clocks animation reads, stepped time for a hand-drawn feel, and an offline mode that renders exact frames. Part of [Engine Overview](page:3vibzoegdx).

---

## 1\. The loop under winit

On desktop the OS owns the loop: winit calls you. Since winit 0.30 that is the `ApplicationHandler` trait.

```rust
struct App { /* window, renderer, scene, clocks */ }

impl winit::application::ApplicationHandler for App {
    fn resumed(&mut self, el: &winit::event_loop::ActiveEventLoop) {
        // Create the window (and the wgpu surface) here: on some platforms there is none before.
    }

    fn window_event(&mut self, el: &winit::event_loop::ActiveEventLoop, _id: winit::window::WindowId, event: winit::event::WindowEvent) {
        match event {
            winit::event::WindowEvent::CloseRequested => el.exit(),
            winit::event::WindowEvent::RedrawRequested => self.frame(),
            other => self.input.handle(&other),
        }
    }

    fn about_to_wait(&mut self, el: &winit::event_loop::ActiveEventLoop) {
        if self.needs_redraw() { self.window.request_redraw(); }
    }
}
```

- Vellum is a **viewer**, not a game: when nothing moves (no animation, no boil, no input) it can stop drawing. Use `ControlFlow::Wait` and request redraws only when something changed; `Poll` while animating.
- **Present mode** decides pacing: `Fifo` is vsync (always available, no tearing); `Mailbox` is low latency without tearing where supported. See [Rendering with wgpu](page:hpy2x2spuf).

---

## 2\. Two clocks

| Clock | What it is | Who reads it |
| --- | --- | --- |
| **Real time** | `std::time::Instant`, monotonic | Frame timing, the camera, UI |
| **Scene time** | Seconds into the animation; can pause, scrub, slow down, or advance exactly 1/24 s per frame | Animation, line boil, anything stylised in time |

```rust
pub struct SceneClock { pub time: f64, pub speed: f64, pub paused: bool }

impl SceneClock {
    pub fn advance(&mut self, real_dt: f64) {
        if !self.paused { self.time += real_dt * self.speed; }
    }
}
```

Nothing in the scene reads `Instant` directly: that is what makes offline rendering and scrubbing work.

---

## 3\. Animating on twos

Hand-drawn animation holds each drawing for two frames (12 drawings a second at 24 fps), sometimes three. Smooth 60 Hz motion on a drawn-looking character reads as "3D with a filter"; stepped motion reads as **drawn**. *Into the Spider-Verse* animated characters on twos and moved the camera on ones.

```rust
/// The time a held pose shows: animation is sampled at this, not at the clock's time.
pub fn held(time: f64, drawings_per_second: f64) -> f64 {
    (time * drawings_per_second).floor() / drawings_per_second
}

let pose_time = held(clock.time, 12.0); // on twos at 24 fps
let camera_time = clock.time;           // the camera stays smooth
```

- The rate is **per entity** (a style override in the [Entity Component System](page:vj3mm21ptz)): characters on twos, effects on ones, background held.
- **Line boil**: the noise seed that wobbles lines and strokes changes on the same kind of stepped clock, `floor(time * boil_hz)`, at 8–12 Hz ([Hatching and Painterly Rendering](page:6vwu9pwosr)).

---

## 4\. Fixed steps, when something simulates

Anything simulated (brush-stroke particles drifting, cloth, a procedural sway) should advance in **fixed** steps so it looks the same at any frame rate and in offline renders. The accumulator from Glenn Fiedler's *Fix Your Timestep!*:

```rust
const DT: f64 = 1.0 / 120.0;
const MAX_FRAME: f64 = 0.25; // never simulate more than this per frame

accumulator += frame_dt.min(MAX_FRAME);
while accumulator >= DT {
    sim.step(DT);
    accumulator -= DT;
}
let alpha = accumulator / DT; // blend previous and current state when drawing
```

Interpolate with `lerp` and `slerp` ([Math for Games](page:l0adi79jhz)): `previous.translation.lerp(current.translation, alpha as f32)`.

---

## 5\. Offline rendering

For animation output, real time does not exist:

1. Scene time = `frame_index / fps`, exactly.
1. Render into an offscreen texture (any size: 4K, or 2× and downsample for cleaner lines).
1. `copy_texture_to_buffer` into a mappable buffer; rows padded to `COPY_BYTES_PER_ROW_ALIGNMENT` (256 bytes).
1. Map, strip the padding, write a PNG with `image`.
1. Next frame, however long that one took.

The same path makes **golden images** for tests ([Performance and Profiling](page:tmmt1jyud5)).

---

## 6\. Measuring time

- `std::time::Instant` is monotonic: use it, never wall-clock time.
- Show frame time in **milliseconds**, not fps: 16.6 ms is the 60 Hz budget. Budgets are in [Performance and Profiling](page:tmmt1jyud5).
- GPU time is measured separately (timestamp queries): an NPR frame is usually GPU-bound.

---

## 7\. Order inside a frame

1. Collect input; update the camera ([Input Handling](page:oaumtmnlq4)).
1. Advance the scene clock; run fixed steps if anything simulates.
1. Sample animation at each entity's held time; propagate transforms ([Math for Games](page:l0adi79jhz)).
1. Apply reloaded shaders, textures and style presets ([Scripting and Hot Reload](page:sa5xujelgo)).
1. **Extract** what to draw, then run the render graph ([Rendering with wgpu](page:hpy2x2spuf)).

Extracting means drawing never borrows the scene while it changes ([Rust Patterns for Engines](page:7nta66y9tk)).
