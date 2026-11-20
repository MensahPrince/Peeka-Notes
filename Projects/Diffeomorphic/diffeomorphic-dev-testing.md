---
id: cglhfqaus0
title: "Diffeomorphic Dev & Testing"
sort: 4000
version: 1
---

How the import\_daz changes are built, tested headless, and handed over for testing in Blender.

## Setup

<columns>

<column>

### Repo

- `~/gitRepo/import_daz` (DAZ Importer 5.3.0, Blender ≥ 4.2), branch `master`.
- All Genesis 9 work is **still uncommitted** (task in [Diffeomorphic Tasks](page:6eeklid8pz)).

</column>

<column>

### Blender

- Flatpak **Blender 5.1** (`org.blender.Blender`).
- CloudRig 2.2.31 installed as an extension under `~/.var/app/org.blender.Blender/config/blender/5.1/extensions/blender_org/cloudrig`.

</column>

<column>

### Test file

- `~/Downloads/Telegram Desktop/Mavis.blend`: Genesis 9, 133 bones, 44 face bones, no morph drivers.
- Always test on a copy.

</column>

</columns>

## Running the addon headless

Load the repo and CloudRig as add-ons without installing them, then drive the operators from a script:

```python
import sys, addon_utils
sys.path.insert(0, "/home/warhammer/gitRepo")
addon_utils.enable("rigify", default_set=True)
addon_utils.enable("import_daz", default_set=True, handle_error=None)
# CloudRig: add its extensions folder to sys.path, then
addon_utils.enable("cloudrig", default_set=True, handle_error=None)
```

```bash
flatpak run --filesystem=$SCRATCH --filesystem=/home/warhammer/gitRepo \
  org.blender.Blender -b --factory-startup copy.blend --python test.py
```

<toggle>

Gotchas

- DAZ operators catch their own errors; read `import_daz.error.LS.message` to see why one "failed".
- Create Metarig leaves the original rig active; find the metarig by its `DazMetaRig` property.
- Don't name a scratch script `inspect.py`: it shadows Python's module.
- Delete the `__pycache__` folders the test runs leave in the repo.

</toggle>

## What every change is checked against

1. **Rest pose:** the evaluated mesh with the ties on vs muted. The target is under 0.1 mm.
1. **Each control:** pose it and measure the DAZ bones it should move (and that nothing else moves).
1. **Renders:** Workbench renders of the head and body, front, side and three-quarter, to look at fits and deformations.
1. **Regression:** the old path (options off) still gives the baseline bone count (513 for Mavis with Rigify).

## Handing over a build

The installable zip goes to `~/Desktop/import_daz_rigify_face.zip` (tracked plus new files, top folder `import_daz/`). To install: uninstall DAZ Importer, Install from Disk, **restart Blender**, convert a fresh file.
