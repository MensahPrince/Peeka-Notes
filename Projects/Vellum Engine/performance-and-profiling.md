---
id: tmmt1jyud5
title: "Performance and Profiling"
sort: 10000
version: 1
---

Frame budgets for a GPU-heavy stylised renderer, memory and bandwidth, the tools that show where time goes, and golden-image tests. Part of [Engine Overview](page:3vibzoegdx).

---

## 1\. The budget

| Target | Frame budget |
| --- | --- |
| 30 Hz | 33\.3 ms |
| 60 Hz | 16\.6 ms |
| 120 Hz | 8\.3 ms |
| 144 Hz | 6\.9 ms |

**Measure in milliseconds**, never fps: going from 60 to 50 fps is 3.3 ms; from 200 to 150 fps is only 1.7 ms.

An NPR frame is **GPU-bound**: the CPU side (animation, extract, a few hundred draws) is small, while the GPU runs a prepass, shading and several full-screen passes. A rough split at 60 Hz, 1440p:

| Pass | Budget |
| --- | --- |
| Shadow + prepass | 2 ms |
| Shading | 3 ms |
| Lines (edges, hulls, mesh lines) | 2 ms |
| Paint (Kuwahara, watercolour) | 4 ms, or skip |
| Compose + UI | 1 ms |
| Slack | 4 ms |

Offline renders ([Game Loop and Timing](page:kom1zxtzlw)) have no budget: there, supersample and use the expensive styles freely.

---

## 2\. Full-screen passes cost pixels

A full-screen pass runs once per pixel: 1080p is 2.1 million, 1440p 3.7 million, 4K 8.3 million. A 9×9 Kuwahara samples 81 texels per pixel: at 4K that is 670 million reads.

- Run **paint passes at half resolution** and upsample guided by the ID buffer (so edges stay sharp).
- **Separable** filters (blurs) in two 1D passes: 2n samples instead of n².
- Read with `textureLoad` and share neighbourhood reads through **compute workgroup memory** for big kernels.
- Skip passes a style doesn't use: the render graph culls them ([Rendering with wgpu](page:hpy2x2spuf)).

---

## 3\. Bandwidth: keep the prepass slim

Every post pass reads the prepass targets, so their size multiplies:

| Target | Fat | Slim |
| --- | --- | --- |
| Normals | `Rgba16Float`, 8 B/px | Octahedral `Rg16Float`, 4 B/px ([Math for Games](page:l0adi79jhz)) |
| IDs | `Rg32Uint`, 8 B/px | `R32Uint` with object and material packed (20 + 12 bits), 4 B/px |
| Depth | `Depth32Float`, 4 B/px | Keep: edges need the precision |

At 1440p that is 74 MB down to 44 MB of targets, read several times a frame.

---

## 4\. Build settings first

```toml
# Cargo.toml: debug builds that are fast enough to look at
[profile.dev]
opt-level = 1

[profile.dev.package."*"]
opt-level = 3        # dependencies (wgpu, naga, glam, image) optimised, our code still quick to compile

[profile.release]
lto = "thin"
codegen-units = 1    # slower to build, faster to run

[profile.profiling]
inherits = "release"
debug = true         # symbols for the profiler
```

A Rust debug build can be 10–50× slower than release on the CPU side; asset import (smoothing normals, edge adjacency) suffers most. Peeka uses the same `opt-level = 3` for dependencies trick for Slint.

---

## 5\. Tools

| Tool | Shows | How |
| --- | --- | --- |
| **wgpu timestamp queries** | GPU time per pass | Needs `TIMESTAMP_QUERY`; write one before and after each pass, show them in the overlay |
| **RenderDoc** | One GPU frame: every pass, every target, every draw | The tool for "why is this line missing"; inspect the prepass targets |
| **Tracy** | CPU timeline of zones, threads, allocations | `tracy-client` or the `profiling` crate's macros |
| **puffin** | Same idea, in-game with `puffin_egui` | `puffin::profile_scope!("extract")` |
| `cargo flamegraph` / `perf` | Where CPU time goes, statistically | Asset import, extract |

```rust
fn frame(&mut self) {
    profiling::scope!("frame");
    { profiling::scope!("animate"); self.animate(); }
    { profiling::scope!("extract"); self.extract(); }
    { profiling::scope!("record"); self.render_graph.record(&mut encoder); } // GPU time: timestamp queries
}
```

---

## 6\. CPU side: layout and allocation

- **Structure of arrays**: component storages in the [Entity Component System](page:vj3mm21ptz) keep extract walking dense memory.
- **Indices, not pointers** ([Rust Patterns for Engines](page:7nta66y9tk)).
- **No allocation per frame**: reuse draw lists (`clear()` keeps capacity), a bump arena (`bumpalo`) for temporaries.
- **Few pipeline switches**: sort draws by pipeline then material; one pipeline per style, not per object.

---

## 7\. Parallelism

- Asset import on workers ([Asset Pipeline](page:s79wv0aek9)); `rayon` for big per-mesh jobs (normal smoothing, edge building).
- Extract can be parallel per component set; recording command buffers per pass can be too (`CommandEncoder`s on several threads, submitted together).
- Rust's `Send` / `Sync` make data races compile errors, not heisenbugs.

---

## 8\. Golden-image tests

NPR regressions are visual: a threshold off by 0.01 doubles the lines. Catch them with images:

1. A set of test scenes, fixed camera paths ([Input Handling](page:oaumtmnlq4)), fixed scene times.
1. Render each offline ([Game Loop and Timing](page:kom1zxtzlw)) and compare with a stored reference image.
1. Compare with a **tolerance**: GPUs and drivers differ in the last bits. A per-pixel threshold plus a maximum share of differing pixels, or NVIDIA's **FLIP** metric.
1. On failure, save the diff image as a CI artifact.

Record GPU timings in the same run and fail if a pass grows past its budget.
