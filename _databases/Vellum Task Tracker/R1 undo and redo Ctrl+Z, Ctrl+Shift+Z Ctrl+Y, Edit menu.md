---
id: br785gpswi
title: "R1 undo and redo: Ctrl+Z, Ctrl+Shift+Z / Ctrl+Y, Edit menu"
order: "b1P"
created: 2026-11-02T19:54:01Z
updated: 2026-11-02T19:54:01Z
version: 1
Status: "In progress"
Milestone: "Release 0.1"
Area: "Iteration"
Priority: "High"
Due: 2026-11-02
Notes: "2026-11-02, asked for. history.rs keeps whole copies of the panel's Overrides (every window edit lands there) and makes a step once an edit is over: no mouse button held, no text field focused, so a drag is one step; 256 steps. Shortcuts skipped while a text field has focus (its own Ctrl+Z). Save keeps history, Save As carries it, Open starts fresh; undoing to the saved state clears the unsaved mark (save now prunes a copy). Undo re-applies the camera by name and drops a selection that left the scene. 8 tests. Branch r1-undo, not committed; not yet tried by hand. Supersedes the 'R1 undo and redo' row once checked."
---
