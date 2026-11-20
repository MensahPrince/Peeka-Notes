---
id: pwa7tbk5vg
title: "G9 Rigify Face"
sort: 2000
version: 1
---

<callout icon="💡">

**What it does:** Create Metarig / Convert To Rigify now keep Rigify's full face rig for **Genesis 9**, fit it to the character's face, and make the DAZ face bones follow the Rigify face controls. The DAZ weights are not changed. Tested on Mavis in Blender 5.1; the user confirmed it "behaves perfectly".

</callout>

## How to use it

1. Select the Genesis 9 armature.
1. **Convert To Rigify** (one click), or **Create Metarig** then DAZ's **Rigify Metarig**.
1. Keep **Rigify Face** and **Upgrade Face Rig** ticked (both on by default).

<callout background="yellow" icon="⚠️">

**Don't use Rigify's own Generate Rig button.** It only builds the control rig and never moves the meshes over, so nothing is bound. DAZ's Rigify Metarig button runs the same generate step and then does the binding.

</callout>

## How it works

```mermaid
flowchart LR
  A[Rigify human metarig<br/>with face] --> B[Store default face]
  B --> C[Thin plate spline<br/>23 landmarks]
  C --> D[Snap surface bones<br/>to head mesh]
  D --> E[Jaw pivot = DAZ lowerjaw]
  E --> F[Upgrade to modular face]
  F --> G[Generate + bind]
  G --> H[DAZ face bones follow<br/>Rigify DEF bones]
```

<columns>

<column>

### Fitting the metarig

- `adjustMetaBones` stores the default face, and the face is no longer deleted (`delete_children` of the head is skipped).
- `fitFace` warps it with a **thin plate spline** built from landmark pairs in `data/rigify/genesis9.json` (left side mirrored in code; duplicate center points removed, they made the system singular).
- Bones on the skin are then **snapped to the closest point of the body mesh** (BVH tree of the mesh with the most head-weighted vertices). Interior bones (face, jaw, eyes, teeth, tongue) are not snapped.
- `heads`: the Rigify `jaw` head is set to the DAZ `lowerjaw` hinge so the jaw opens at the right pivot.

</column>

<column>

### Following the face

- `addFaceFollow` puts an **Armature constraint** on each DAZ face bone, targeting one or two Rigify DEF bones (`follow` in the data file). Vertices then deform as if weighted to those DEF bones.
- The Armature constraint double-transforms with a parent, so followers are **unparented** (in `hideFaceFollowers`, after the face renaming).
- **Eyelids** rotate around the eye: an MCH track bone damped-tracks the Rigify lid, and the DAZ lid copies its rotation (`LOCAL_OWNER_ORIENT`, mix BEFORE).
- The addon's own gaze and tongue IK are skipped; followers move to the hidden Help collection.

</column>

</columns>

## Files changed

| File | Change |
| --- | --- |
| `data/rigify/genesis9.json` | new `face` section: landmarks, interior, heads, follow, track |
| `rigify_tools/rigify.py` | Rigify Face + Upgrade Face Rig options, FaceWarp, fitFace, addFaceFollow, hideFaceFollowers |
| `fix.py` | face renames skip names that already exist (no more `eye.L.001`) |

<toggle collapsed=true>

Bugs found along the way

- **Name clashes:** the DAZ extra `chin` clashed with Rigify's `chin` control, and extra-bone parenting grabbed the Rigify bone. Extras that clash now get a `_` prefix, like the existing `_root`.
- The DAZ `chin` weights end up on Rigify's `DEF-chin`, because the metarig has a bone named `chin`. Harmless: the chin still follows Rigify.
- Mavis's tongue bones sit about 15 cm above the tongue mesh in the original file. The Rigify tongue is placed in the mouth and still drives the mesh correctly.

</toggle>

## Known limits

- [ ] FACS or expression morphs that rotate `lowerjaw` no longer carry the lip bones, because the followers are unparented. Each bone's own morph motion still works. Not tested with real FACS.
- [ ] Keep DAZ Rig with the face is untested.
- [ ] Only Mavis has been tested.

See also [G9 CloudRig Conversion](page:tqg3n192rl) and [Diffeomorphic Dev & Testing](page:cglhfqaus0).
