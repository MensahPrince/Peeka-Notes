---
id: oaumtmnlq4
title: "Input Handling"
sort: 7000
version: 1
---

From key presses and mouse drags to "orbit", "next style" and "pick this object": events, per-frame state, action maps, camera controllers and picking from the ID buffer. Part of [Engine Overview](page:3vibzoegdx).

---

## 1\. Three layers

| Layer | What it holds | Example |
| --- | --- | --- |
| **Events** | What the OS said, in order | `MouseInput { button: Left, state: Pressed }` |
| **State** | What is down now, what changed this frame | `held(MouseLeft)`, `pressed(Tab)`, mouse delta |
| **Actions** | What the viewer means by it | `Orbit(Vec2)`, `Zoom(f32)`, `NextStyle`, `ToggleDebugView`, `Screenshot` |

Camera and tool code only ever asks about **actions**. Rebinding keys or adding a 3D mouse then never touches them.

---

## 2\. Events from winit

winit delivers `WindowEvent::KeyboardInput`, `MouseInput`, `CursorMoved`, `MouseWheel` to `window_event` ([Game Loop and Timing](page:kom1zxtzlw)); raw mouse motion for camera control comes as `DeviceEvent::MouseMotion`.

- **egui first**: pass every event to `egui_winit::State::on_window_event`; if egui consumed it (a slider is being dragged), the camera must not see it.
- **Physical vs logical keys**: `PhysicalKey::Code(KeyCode::KeyW)` is the key *position*: use it for fly controls. `Key::Character("w")` is what the key *types*: use it for shortcuts shown to the user.
- **Trackpads**: `MouseWheel` comes as `PixelDelta` (smooth) or `LineDelta` (notches); scale them differently.

---

## 3\. Per-frame state

```rust
#[derive(Default)]
pub struct ButtonState<K: std::hash::Hash + Eq + Copy> {
    held: std::collections::HashSet<K>,
    pressed: std::collections::HashSet<K>,  // went down this frame
    released: std::collections::HashSet<K>, // went up this frame
}

impl<K: std::hash::Hash + Eq + Copy> ButtonState<K> {
    pub fn press(&mut self, k: K) { if self.held.insert(k) { self.pressed.insert(k); } }
    pub fn release(&mut self, k: K) { if self.held.remove(&k) { self.released.insert(k); } }
    pub fn end_frame(&mut self) { self.pressed.clear(); self.released.clear(); }

    pub fn held(&self, k: K) -> bool { self.held.contains(&k) }
    pub fn just_pressed(&self, k: K) -> bool { self.pressed.contains(&k) }
}
```

The camera updates once per frame on real time, not scene time ([Game Loop and Timing](page:kom1zxtzlw)): paused animation must not freeze the view.

---

## 4\. Camera controllers

| Controller | Feels like | Controls |
| --- | --- | --- |
| **Orbit** | Turning a model in your hands | Drag to orbit, scroll to zoom, middle-drag or Shift-drag to pan |
| **Fly** | Walking through a set | WASD + mouse look, Shift faster |
| **Turntable** | A presentation | Constant yaw around the target; for renders |

```rust
pub struct Orbit { pub target: Vec3, pub distance: f32, pub yaw: f32, pub pitch: f32 }

impl Orbit {
    pub fn apply(&mut self, delta: Vec2, zoom: f32) {
        self.yaw -= delta.x * 0.005;
        self.pitch = (self.pitch - delta.y * 0.005).clamp(-1.5, 1.5); // never flip over the pole
        self.distance = (self.distance * (1.0 - zoom * 0.1)).max(0.1);
    }

    pub fn eye(&self) -> Vec3 {
        let rot = Quat::from_rotation_y(self.yaw) * Quat::from_rotation_x(self.pitch);
        self.target + rot * Vec3::new(0.0, 0.0, self.distance)
    }
}
```

`Mat4::look_at_rh(orbit.eye(), orbit.target, Vec3::Y)` gives the view matrix ([Math for Games](page:l0adi79jhz)). Smooth the motion with a short exponential damp so drags don't feel jerky.

---

## 5\. Style controls

- **Cycle presets**: Tab / Shift-Tab steps through the style presets ([Scripting and Hot Reload](page:sa5xujelgo)): cel, ink, pencil, watercolour.
- **Debug views**: number keys show one prepass target or layer full-screen: depth, normals, IDs, edges, lines only ([Rendering with wgpu](page:hpy2x2spuf)). The fastest way to see why a line is missing.
- **Freeze boil**: pause line boil and stepped animation to compare frames.
- **Screenshot**: render the current frame offline at 2× and save a PNG ([Game Loop and Timing](page:kom1zxtzlw)).

Bindings load from a RON file ([Asset Pipeline](page:s79wv0aek9)).

---

## 6\. Picking from the ID buffer

The prepass already knows which object is under every pixel. A click reads it back instead of ray-casting:

1. Copy the one pixel under the cursor from the ID target into a small mappable buffer (`copy_texture_to_buffer` with a 1×1 extent; rows still padded to 256 bytes).
1. Map it asynchronously; a frame or two later, the `u32` is the entity index ([Entity Component System](page:vj3mm21ptz)).
1. Check the generation, select the entity, show its style components in the egui panel.

Exact to what is **drawn** (outlines and hatching included), and no physics needed.

---

## 7\. Recording camera paths

Record the camera per frame and replay it: the same shot rendered with different styles, golden-image tests that always look at the same thing, and turntable renders ([Performance and Profiling](page:tmmt1jyud5)).
