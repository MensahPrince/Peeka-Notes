---
id: 5fqpz0e5pw
title: "Physics and Collision"
sort: 8000
version: 1
---

> **Parked.** Vellum is now a non-photorealistic 3D renderer, not a game engine ([Engine Overview](page:3vibzoegdx)), so physics is out of scope. Picking uses the ID buffer instead of ray casts ([Input Handling](page:oaumtmnlq4)). These notes are kept in case Vellum grows into a stylised game later.

Bodies, colliders and contacts: how Vellum would run physics with rapier inside a fixed step and keep it in sync with the ECS.

---

## 1\. What a physics engine does each step

```
integrate forces ──► broad phase ──► narrow phase ──► solve contacts & joints ──► integrate positions ──► events
 (gravity, impulses)  (which pairs     (exact shapes:    (push apart, friction,
                       might touch:     contact points,   restitution)
                       AABB tree)       normals, depth)
```

| Stage | Question | Typical structure |
| --- | --- | --- |
| Broad phase | Which pairs are close? | Bounding-volume hierarchy, sweep and prune |
| Narrow phase | Do they touch, and where? | GJK / EPA, shape-specific tests |
| Solver | How do they push back? | Iterative impulse solver |

Writing this is a project of its own. The plan was **rapier** (pure Rust, 2D and 3D, by Dimforge), with Vellum's own job limited to the glue.

---

## 2\. Body types

| Type | Moved by | Use |
| --- | --- | --- |
| **Dynamic** | Forces and contacts | Crates, debris, ragdolls |
| **Kinematic** | You set its position or velocity; pushes dynamics, is not pushed | Moving platforms, character controllers |
| **Fixed** | Nothing | Level geometry |

Colliders (ball, cuboid, capsule, convex hull, trimesh) attach to bodies; a **sensor** collider reports overlaps without pushing (triggers, pickups).

---

## 3\. rapier in the loop

```rust
use rapier3d::prelude::*;

pub struct Physics {
    pipeline: PhysicsPipeline,
    params: IntegrationParameters,
    islands: IslandManager,
    broad_phase: DefaultBroadPhase,
    narrow_phase: NarrowPhase,
    bodies: RigidBodySet,
    colliders: ColliderSet,
    impulse_joints: ImpulseJointSet,
    multibody_joints: MultibodyJointSet,
    ccd: CCDSolver,
    gravity: Vector<Real>,
}

impl Physics {
    pub fn step(&mut self, dt: f32) {
        self.params.dt = dt; // the fixed DT, always the same
        self.pipeline.step(
            &self.gravity, &self.params, &mut self.islands, &mut self.broad_phase, &mut self.narrow_phase,
            &mut self.bodies, &mut self.colliders, &mut self.impulse_joints, &mut self.multibody_joints,
            &mut self.ccd, None, &(), &(),
        );
    }
}
```

> The exact `step` arguments change between rapier versions; the shape (sets of bodies and colliders, a pipeline, parameters) does not.

- **Always a fixed `DT`** ([Game Loop and Timing](page:kom1zxtzlw)); never the frame time.
- **CCD** (continuous collision detection) on small fast bodies prevents tunnelling.
- rapier can be built with `enhanced-determinism` for cross-platform identical results.

---

## 4\. Syncing with the ECS

rapier owns its own sets; entities hold **handles** into them (the pattern from [Rust Patterns for Engines](page:7nta66y9tk)):

```rust
pub struct RigidBody(pub rapier3d::prelude::RigidBodyHandle); // a component
```

Per fixed step ([Entity Component System](page:vj3mm21ptz)):

1. **Write in**: kinematic bodies and teleports copy `Transform` → rapier.
1. **Step** the pipeline.
1. **Read out**: dynamic bodies copy rapier position → `Transform` (keeping the previous one for interpolation).
1. **Events**: collision and contact-force events go to an event queue.

Units are metres and seconds, the same as [Math for Games](page:l0adi79jhz).

---

## 5\. Queries

- **Ray casts**: line of sight, mouse picking (Vellum now picks from the ID buffer instead).
- **Shape casts**: "would this capsule fit there?", the core of a character controller.
- **Intersection tests**: what is inside a radius.
- Collision **groups and filters** decide what can hit what.

---

## 6\. If it comes back

A stylised game would add a fixed-step physics system before animation, and could reuse the NPR passes unchanged: physics only moves `Transform`s, and extract does the rest ([Rendering with wgpu](page:hpy2x2spuf)).
