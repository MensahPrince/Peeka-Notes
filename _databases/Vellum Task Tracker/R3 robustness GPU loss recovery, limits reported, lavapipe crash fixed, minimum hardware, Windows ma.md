---
id: c7dslok0zr
title: "R3 robustness: GPU loss recovery, limits reported, lavapipe crash fixed, minimum hardware, Windows/macOS CI"
order: "b1Q"
created: 2026-11-02T20:18:08Z
updated: 2026-11-02T20:18:08Z
version: 1
Status: "In progress"
Milestone: "Release 0.1"
Area: "Platform"
Priority: "High"
Due: 2026-11-02
Notes: "2026-11-02. Lost device: preview restarts its GPU side on the same window (tried with VELLUM_LOSE_GPU_AT in debug), exits 1 with a message after 3 losses; OOM rebuilds the renderer; oversize Render Image reported. limits.rs: lamps>64, shadow tiles>32, face maps>32, textures>256, images>1024² as warnings, shown behind a header ⚠ count. Golden segfault: Vulkan loader race in vkSetDebugUtilsObjectNameEXT; software instance drops debug flags; 20/20 runs pass (2/10 crashed before). HD 620: 17-33 ms at 960x540, 44-102 ms at 1080p, in README. CI builds + CPU tests on Windows and macOS. Open: a run on real Windows/NVIDIA/AMD hardware. Branch r3-robustness, not committed. Replaces the 'Golden tests sometimes segfault' row."
---
