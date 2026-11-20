---
id: 7nta66y9tk
title: "Rust Patterns for Engines"
cover: "_assets/8a559d333274f4392f5a4360ba3061b5.jpg"
cover_position: 50
sort: 12000
version: 1
---

The Rust-specific side of renderer design: ownership of the scene, borrowing during a frame, handles instead of references, a render graph that the borrow checker likes, threads, and the little `unsafe` an engine needs. Part of [Engine Overview](page:3vibzoegdx).

---

## 1\. The core tension

Scenes are graphs of things that refer to each other (a mesh uses a material, the material uses a ramp, a light is parented to the camera, an outline's colour follows its material). Object-oriented engines model that with pointers. Rust forbids shared mutable pointers, so the naive translation becomes `Rc<RefCell<T>>` everywhere, with run-time borrow panics instead of compile errors.

The engines that work in Rust (Bevy, Fyrox, rend3) take the same way out: **data in flat storage, relationships as ids**.

---

## 2\. Ids, not references

| Instead of | Use |
| --- | --- |
| `material: &Material` (lifetime hell) | `material: Handle<Material>` ([Asset Pipeline](page:s79wv0aek9)) |
| `Rc<RefCell<Texture>>` | `Handle<Texture>` into an asset store |
| `Weak<Node>` to a parent | `Parent(Entity)` component ([Math for Games](page:l0adi79jhz) hierarchy) |
| `&TextureView` held across passes | A graph resource id, resolved when the pass runs |
| Pointers into a `Vec` | Indices, with a generation ([Entity Component System](page:vj3mm21ptz)) |

Generational keys (`slotmap`, `thunderdome`, or our own `Entities`) make **use after free** a checked `None` instead of a bug:

```rust
let mut ramps: slotmap::SlotMap<slotmap::DefaultKey, Ramp> = slotmap::SlotMap::new();
let k = ramps.insert(Ramp::default());
ramps.remove(k);
assert!(ramps.get(k).is_none()); // the slot may be reused, the key no longer matches
```

---

## 3\. Borrowing the scene during a frame

### Split borrows

A system needing `&mut GlobalTransform` and `&Transform` borrows two different fields of the scene: legal. Keep component storages as separate fields (or behind a type map that hands out disjoint borrows) and most systems "just work".

### Deferred commands

Spawning, despawning or adding a style component while iterating would invalidate the iterator. Queue it:

```rust
pub enum Command { Spawn(Prefab, Transform), Despawn(Entity), Insert(Entity, Box<dyn FnOnce(&mut Scene)>) }

#[derive(Default)]
pub struct Commands(Vec<Command>);

impl Commands {
    pub fn despawn(&mut self, e: Entity) { self.0.push(Command::Despawn(e)); }
    pub fn apply(&mut self, scene: &mut Scene) {
        for c in self.0.drain(..) { scene.apply(c); }
    }
}
```

Applied after the systems run ([Game Loop and Timing](page:kom1zxtzlw)). Bevy's `Commands` is the same idea.

### Extract, don't share

The renderer copies what it needs out of the scene (the extract phase) instead of holding borrows into it while it draws ([Rendering with wgpu](page:hpy2x2spuf)). That also lets rendering run on another thread later: the copy is the boundary.

---

## 4\. A render graph the borrow checker likes

Passes want to share textures (the prepass normals are read by edges, paint and hatching). Holding `&TextureView`s in pass structs ties every pass to the lifetime of every target and breaks on resize. Instead, passes **declare** resources by id and receive them when they run:

```rust
#[derive(Clone, Copy, PartialEq, Eq, Hash)]
pub struct ResourceId(u32);

pub trait Pass {
    fn reads(&self) -> &[ResourceId];
    fn writes(&self) -> &[ResourceId];
    fn record(&self, encoder: &mut wgpu::CommandEncoder, res: &Resources);
}

pub struct Resources { views: Vec<wgpu::TextureView> } // indexed by ResourceId; rebuilt on resize

impl Resources {
    pub fn view(&self, id: ResourceId) -> &wgpu::TextureView { &self.views[id.0 as usize] }
}
```

- The graph sorts passes by their reads and writes, culls passes whose outputs nobody reads, and creates transient targets.
- Resize rebuilds `Resources`; passes hold only ids, so nothing dangles.
- `record` borrows `Resources` immutably: many passes can record in parallel into separate encoders.

---

## 5\. Threads

- **`Send` / `Sync` are design tools**: if the scene is `Send`, work can move across threads; if something isn't, the compiler says so. wgpu's `Device` and `Queue` are `Send + Sync` on native.
- **Channels** between threads: asset loading ([Asset Pipeline](page:s79wv0aek9)), file-watcher events for shaders and presets ([Scripting and Hot Reload](page:sa5xujelgo)).
- `rayon` for parallel work inside a system or an import job ([Performance and Profiling](page:tmmt1jyud5)).
- Avoid `Arc<Mutex<Scene>>`: it serialises everything and invites deadlocks. Give each thread its own data, and pass messages.

---

## 6\. Traits and generics

- Prefer **generic functions** in hot paths (monomorphised, inlined) and **trait objects** at boundaries (passes, an asset loader per file type, a style per material kind).
- A `Plugin` trait (`fn build(&self, app: &mut App)`) keeps subsystems (hatching, paint, mesh lines) decoupled from the core: a style can be a plugin that registers its passes and components.
- `Any` + `TypeId` give a type map for resources: `scene.resource::<SceneClock>()`.

---

## 7\. The `unsafe` an engine actually needs

| Where | Why | Keep it small by |
| --- | --- | --- |
| GPU buffers | Casting structs to bytes | `bytemuck` (`Pod`, `Zeroable`) instead of `transmute` |
| ECS internals | Handing out disjoint `&mut` to columns | One audited module, with tests and Miri |
| FFI | Dylib reloading | Wrappers with safe APIs ([Scripting and Hot Reload](page:sa5xujelgo)) |

Everything else (systems, passes, tools) stays safe Rust. Run `cargo miri test` on the ECS crate.

---

## 8\. Errors

- Asset and shader errors are **logged and survived** (keep the old asset or pipeline), never `unwrap`ped in the frame.
- `thiserror` for library error types, `anyhow` at the application edge.
- `tracing` for logs with spans that line up with profiling zones.
