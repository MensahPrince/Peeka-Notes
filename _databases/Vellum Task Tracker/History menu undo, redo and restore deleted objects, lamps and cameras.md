---
id: klw7s8v7f4
title: "History menu: undo, redo and restore deleted objects, lamps and cameras"
order: "b1h"
created: 2026-11-04T08:06:04Z
updated: 2026-11-04T08:06:04Z
version: 1
Status: "Done"
Area: "Iteration"
Priority: "High"
Due: 2026-11-04
Notes: "2026-11-04, asked for. Replaces the Edit menu and the Outliner's Deleted section. Window-added lamps and cameras go to Overrides.deleted_lamps / deleted_cameras on delete and come back as they were (Panel::restore_deleted, Bin); new names skip binned ones. File lamps are still switched off, file cameras still can't be deleted. Test deleted_lamps_and_cameras_wait_in_history. Commit 3d88a91."
---
