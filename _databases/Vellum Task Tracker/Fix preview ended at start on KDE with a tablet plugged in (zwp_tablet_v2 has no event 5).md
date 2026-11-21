---
id: my1x8dmz9p
title: "Fix: preview ended at start on KDE with a tablet plugged in (zwp_tablet_v2 has no event 5)"
order: "b4S"
created: 2026-11-13T15:11:41Z
updated: 2026-11-13T15:11:41Z
version: 1
Status: "Done"
Milestone: "M46 Workspace tabs and Paint"
Area: "Platform"
Priority: "High"
Due: 2026-11-13
Notes: "octotablet bound the tablet protocol at the compositor's version 2 but only knew version 1, so KDE's bustype event killed the program. Replaced by pen_wayland.rs on wayland-client/protocols, bound at the lower version. Checked with the Wacom One: found and runs. Uncommitted; main still has the crash until pushed"
---
