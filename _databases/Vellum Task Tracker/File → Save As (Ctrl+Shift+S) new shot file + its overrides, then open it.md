---
id: bp0bmtfm7n
title: "File → Save As (Ctrl+Shift+S): new shot file + its overrides, then open it"
order: "b0p"
created: 2026-10-31T11:16:15Z
updated: 2026-10-31T11:16:15Z
version: 1
Status: "Done"
Area: "Iteration"
Priority: "Medium"
Notes: "2026-10-31. Panel edits (saved or not) go into the new shot; the original keeps what it last saved. Same folder: shot copied as written; elsewhere or a bare glTF: written afresh (Shot::to_ron, unset fields left out) with every path re-pointed (Shot/Overrides::map_paths). Never overwrites. Commit 985b409."
---
