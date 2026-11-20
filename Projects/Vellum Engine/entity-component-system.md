---
id: vj3mm21ptz
title: "Entity Component System"
cover: "_assets/8a188a0063c4ef058c8d2dab1b05137d.jpg"
cover_position: 50
sort: 3000
version: 1
---

Entities, components, systems: how Vellum keeps its scene (meshes, materials, lights, cameras, style overrides) as plain data, and why that suits Rust. Part of [Engine Overview](page:3vibzoegdx).

---

## 1\. Why not a scene graph of objects

The classic renderer has a `SceneNode` base class and subclasses (`MeshNode`, `LightNode`, `CameraNode`) pointing at parents and children. In Rust that turns into `Rc<RefCell<…>>` everywhere and fights with the borrow checker ([Rust Patterns for Engines](page:7nta66y9tk)). It is also slow to walk: each node's data is scattered in memory.

An **ECS** splits things differently:

| Part | What it is | Example |
| --- | --- | --- |
| **Entity** | Just an id | `Entity { index: 42, generation: 3 }` |
| **Component** | Plain data attached to an entity | `Transform`, `MeshRef(Handle<Mesh>)`, `Material`, `Light`, `Outline { width_px, color }` |
| **System** | A function over every entity with some components | "propagate transforms", "extract everything with a mesh and a material" |
| **Resource** | One global piece of data | `SceneClock`, the active `Style`, the asset stores |

A character is not a type; it is an entity that happens to have `Transform`, `MeshRef`, `Material`, `Outline` and `AnimateOnTwos`.

### Style as components

The ECS shines for NPR because looks are **per object and composable**:

| Component | Effect |
| --- | --- |
| `Material { style: Toon, ramp, … }` | Which shading pipeline draws it ([Toon Shading](page:m2km9l0szv)) |
| `Outline { width_px, color }` | Gets an inverted hull ([Outlines and Edges](page:sof6p2vevn)) |
| `NoEdges` | Skipped by screen-space edges (sky, glow effects) |
| `Hatched { tam }` | Hatching instead of flat tones ([Hatching and Painterly Rendering](page:6vwu9pwosr)) |
| `HeldAnimation { drawings_per_second }` | Animated on twos ([Game Loop and Timing](page:kom1zxtzlw)) |
| `LineWeight(f32)` | Heavier lines for important objects |

Adding a look to one object is inserting a component; no class hierarchy changes.

---

## 2\. Entities as generational indices

An entity is an index into arrays plus a **generation**, so a stale id to a despawned entity is detected instead of silently pointing at whatever reused its slot.

```rust
#[derive(Clone, Copy, PartialEq, Eq, Hash, Debug)]
pub struct Entity { index: u32, generation: u32 }

#[derive(Default)]
pub struct Entities {
    generations: Vec<u32>,
    alive: Vec<bool>,
    free: Vec<u32>,
}

impl Entities {
    pub fn spawn(&mut self) -> Entity {
        if let Some(index) = self.free.pop() {
            self.alive[index as usize] = true;
            return Entity { index, generation: self.generations[index as usize] };
        }
        self.generations.push(0);
        self.alive.push(true);
        Entity { index: self.generations.len() as u32 - 1, generation: 0 }
    }

    pub fn despawn(&mut self, e: Entity) -> bool {
        if !self.is_alive(e) { return false; }
        self.alive[e.index as usize] = false;
        self.generations[e.index as usize] += 1; // old ids no longer match
        self.free.push(e.index);
        true
    }

    pub fn is_alive(&self, e: Entity) -> bool {
        self.generations.get(e.index as usize) == Some(&e.generation) && self.alive[e.index as usize]
    }
}
```

The entity's index is also what the prepass writes as its **object ID** ([Rendering with wgpu](page:hpy2x2spuf)): a click on a pixel reads the ID back and finds the entity ([Input Handling](page:oaumtmnlq4)). The same idea is the `slotmap` crate, used for asset handles ([Asset Pipeline](page:s79wv0aek9)).

---

## 3\. Storage: archetypes or sparse sets

Where the components live decides what is fast.

### Archetypes (bevy\_ecs, hecs, flecs)

Entities with exactly the same set of components share a table, one column per component:

```
Archetype [Transform, MeshRef, Material]          Archetype [Transform, MeshRef, Material, Outline]
 Transform: t0 t1 t2 t3                            Transform: t4 t5
 MeshRef:   m0 m1 m2 m3                            MeshRef:   m4 m5
 Material:  a0 a1 a2 a3                            Material:  a4 a5
                                                   Outline:   o4 o5
```

- **Fast iteration**: a query walks dense columns, ideal for the cache ([Performance and Profiling](page:tmmt1jyud5)).
- **Slow structural change**: adding or removing a component moves the entity to another table.

### Sparse sets (EnTT, shipyard)

One dense array per component type, plus a sparse array from entity index to its slot:

- **Fast add and remove**: O(1), nothing moves. Good for toggling style components live from the style panel.
- **Slower multi-component queries**: iterate the smallest set and look the others up.

|  | Archetype | Sparse set |
| --- | --- | --- |
| Iterating many entities | Best | Good |
| Adding / removing components often | Costly | Best |
| Implementation effort | Higher | Lower |

**Vellum's choice**: sparse sets. Scenes have thousands of entities, not millions, and style components are toggled often; measure before changing.

---

## 4\. A minimal sparse set

```rust
pub struct SparseSet<T> {
    sparse: Vec<Option<u32>>, // entity index → slot in dense
    dense: Vec<Entity>,       // slot → entity
    data: Vec<T>,             // slot → component
}

impl<T> SparseSet<T> {
    pub fn insert(&mut self, e: Entity, value: T) {
        let i = e.index as usize;
        if self.sparse.len() <= i { self.sparse.resize(i + 1, None); }
        match self.sparse[i] {
            Some(slot) => self.data[slot as usize] = value,
            None => {
                self.sparse[i] = Some(self.dense.len() as u32);
                self.dense.push(e);
                self.data.push(value);
            }
        }
    }

    pub fn remove(&mut self, e: Entity) -> Option<T> {
        let slot = self.sparse.get_mut(e.index as usize)?.take()? as usize;
        // Swap-remove keeps the arrays dense; fix the moved entity's slot.
        let last = *self.dense.last().unwrap();
        self.dense.swap_remove(slot);
        let value = self.data.swap_remove(slot);
        if last != e { self.sparse[last.index as usize] = Some(slot as u32); }
        Some(value)
    }

    pub fn get(&self, e: Entity) -> Option<&T> {
        let slot = (*self.sparse.get(e.index as usize)?)?;
        (self.dense[slot as usize] == e).then(|| &self.data[slot as usize])
    }

    pub fn iter(&self) -> impl Iterator<Item = (Entity, &T)> {
        self.dense.iter().copied().zip(self.data.iter())
    }
}
```

---

## 5\. Systems and borrowing

A system wants `&mut GlobalTransform` and `&Transform` at once. Keeping each component type in its own storage makes that legal: two different fields of the scene, borrowed separately.

```rust
fn animate(transforms: &mut SparseSet<Transform>, tracks: &SparseSet<AnimationTrack>, held: &SparseSet<HeldAnimation>, time: f64) {
    for (e, track) in tracks.iter() {
        let t = held.get(e).map_or(time, |h| h.sample_time(time)); // on twos, or smooth
        if let Some(transform) = transforms.get_mut(e) {
            *transform = track.sample(t);
        }
    }
}
```

- **Order**: systems run in a fixed order each frame ([Game Loop and Timing](page:kom1zxtzlw)): animation, transform propagation, extract.
- **Structural changes during iteration** (spawn, despawn, add a style component) go into a **command buffer** applied after the systems, so no iterator is invalidated ([Rust Patterns for Engines](page:7nta66y9tk)).
- **Change detection**: a dirty flag per component (or a "changed this frame" list) lets extract re-upload only what moved or was restyled.

---

## 6\. What renders

The renderer never walks the ECS directly. An **extract** system copies what is drawable (world transform, mesh handle, material and style parameters, object ID) into flat lists each frame, and [Rendering with wgpu](page:hpy2x2spuf) sorts them per pass: everything into the prepass, each material style into its shading pipeline, `Outline` entities into the hull pass. Transforms and their parent-child hierarchy are in [Math for Games](page:l0adi79jhz).
