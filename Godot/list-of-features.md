---
id: d2x2dpnrwr
title: "List of features"
sort: 4000
version: 1
source: https://docs.godotengine.org/en/stable/about/list_of_features.html
clipped: 2026-11-19 17:05
tags: [game, development, features, godot]
---

This page aims to list **all** features currently supported by Godot.

Note

This page lists features supported by the current stable version of Godot. Some of these features are not available in the [3.x release series](https://docs.godotengine.org/en/3.6/about/list_of_features.html).

## Platforms[](#platforms)

Note

For information about console support, see the [Godot website](https://godotengine.org/consoles/).

**Can run both the editor and exported projects:**

- Windows (x86 and ARM, 64-bit and 32-bit).
- macOS (x86 and ARM, 64-bit only).
- Linux (x86 and ARM, 64-bit and 32-bit).

  >
  > - Binaries are statically linked and can run on any distribution if compiled on an old enough base distribution.
  > - Official binaries are compiled using the [Godot Engine buildroot](https://github.com/godotengine/buildroot), allowing for binaries that work across common Linux distributions.
- Android (editor support is experimental).
- [Web browsers](https://docs.godotengine.org/en/stable/tutorials/editor/using_the_web_editor.html#doc-using-the-web-editor). Experimental in 4.0, using Godot 3.x is recommended instead when targeting HTML5.

Note

Linux supports rv64 (RISC-V), ppc64 & ppc32 (PowerPC), and loongarch64. However you must compile the editor for that platform (as well as export templates) yourself, no official downloads are currently provided. RISC-V compiling instructions can be found on the [Compiling for Linux, \*BSD](https://docs.godotengine.org/en/stable/engine_details/development/compiling/compiling_for_linuxbsd.html#doc-compiling-for-linuxbsd) page.

**Runs exported projects:**

- iOS.

Godot aims to be as platform-independent as possible and can be [ported to new platforms](https://docs.godotengine.org/en/stable/engine_details/engine_api/custom_platform_ports.html#doc-custom-platform-ports) with relative ease.

Note

Projects written in C# using Godot 4 currently cannot be exported to the web platform. To use C# on that platform, consider Godot 3 instead. Android and iOS platform support is available as of Godot 4.2, but is experimental and [some limitations apply](https://docs.godotengine.org/en/stable/tutorials/scripting/c_sharp/index.html#doc-c-sharp-platforms).

## Editor[](#editor)

**Features:**

- Scene tree editor.
- Built-in script editor. Also supports editing text files and using custom syntax highlighters.
- Support for [external script editors](https://docs.godotengine.org/en/stable/tutorials/editor/external_editor.html#doc-external-editor) such as Visual Studio Code or Vim.
- GDScript [debugger](https://docs.godotengine.org/en/stable/tutorials/scripting/debug/debugger_panel.html#doc-debugger-panel).

  >
  > - Supports debugging in threads.
- Visual profiler with CPU and GPU time indications for each step of the rendering pipeline.
- Performance monitoring tools, including [custom performance monitors](https://docs.godotengine.org/en/stable/tutorials/scripting/debug/custom_performance_monitors.html#doc-custom-performance-monitors).
- Supports [tracing profilers](https://docs.godotengine.org/en/stable/engine_details/development/profiling/index.html#doc-tracing-profilers) like [Tracy](https://docs.godotengine.org/en/stable/engine_details/development/profiling/tracy.html#doc-profiler-tracy) and [Perfetto](https://docs.godotengine.org/en/stable/engine_details/development/profiling/perfetto.html#doc-profiler-perfetto) for deeper optimization tasks.
- Any script can be [run in the editor](https://docs.godotengine.org/en/stable/tutorials/plugins/running_code_in_the_editor.html#doc-running-code-in-the-editor) and provide custom functionality such as clickable buttons in the inspector, without needing to create an editor plugin.
- Live script reloading.
- Live scene editing.

  >
  > - Changes *will* reflect in the editor and *will* be kept after closing the running project.
- Live camera replication (disabled by default).

  >
  > - Move the in-editor camera and see the result in the running project.
- Remote inspector.

  >
  > - Changes *won't* reflect in the editor and *won't* be kept after closing the running project.
- Run multiple simultaneous project instances from a single editor instance (useful for client/server testing).
- Optional [game embedding](https://docs.godotengine.org/en/stable/tutorials/editor/game_embedding.html#doc-game-embedding) to run the project in a panel within the editor.
  - Select 2D and 3D nodes in the project's viewport to inspect them in the editor.
  - Move the camera within the project using the 2D and 3D camera overrides.
  - Supports time scale adjustments, pausing, and frame advance.
  - Supports muting the project.
- Ruler tool to measure distances in 2D and 3D.
- Vertex snapping support in 3D.
- Support for following the selection as it moves in 3D by using Focus Selection twice.
- Built-in offline class reference documentation.
- Use the editor in dozens of languages contributed by the community.

**Plugins:**

- Editor plugins can be downloaded from the [Asset Store](https://docs.godotengine.org/en/stable/community/asset_store/what_is_asset_store.html#doc-what-is-asset-store) to extend editor functionality.
- [Create your own plugins](https://docs.godotengine.org/en/stable/tutorials/plugins/editor/making_plugins.html#doc-making-plugins) using GDScript to add new features or speed up your workflow.
- [Download projects from the Asset Store](https://docs.godotengine.org/en/stable/community/asset_store/using_in_editor.html#doc-using-asset-store-editor) in the Project Manager and import them directly.

## Rendering[](#rendering)

Godot 4 includes three renderers:

- **Forward+**. The most advanced renderer, suited for desktop platforms only. Used by default on desktop platforms. This renderer uses **Vulkan**, **Direct3D 12**, or **Metal** as the rendering driver, and it uses the **RenderingDevice** backend.
- **Mobile**. Fewer features, but renders simple scenes faster. Suited for mobile and desktop platforms. Used by default on mobile platforms. This renderer uses **Vulkan**, **Direct3D 12**, or **Metal** as the rendering driver, and it uses the **RenderingDevice** backend.
- **Compatibility**, sometimes called **GL Compatibility**. The least advanced renderer, suited for low-end desktop and mobile platforms. Used by default on the web platform. This renderer uses **OpenGL** as the rendering driver.

## 2D graphics[](#d-graphics)

- Sprite, polygon and line rendering.

  >
  > - High-level tools to draw lines and polygons such as [Polygon2D](https://docs.godotengine.org/en/stable/classes/class_polygon2d.html#class-polygon2d) and [Line2D](https://docs.godotengine.org/en/stable/classes/class_line2d.html#class-line2d), with support for texturing.
- AnimatedSprite2D as a helper for creating animated sprites.
- Parallax layers.

  >
  > - Pseudo-3D support including preview in the editor.
- [2D lighting](https://docs.godotengine.org/en/stable/tutorials/2d/2d_lights_and_shadows.html#doc-2d-lights-and-shadows) with normal maps and specular maps.

  >
  > - Point (omni/spot) and directional 2D lights.
  > - Hard or soft shadows (adjustable on a per-light basis).
  > - Custom shaders can access a real-time SDF representation of the 2D scene based on [LightOccluder2D](https://docs.godotengine.org/en/stable/classes/class_lightoccluder2d.html#class-lightoccluder2d) nodes, which can be used for improved 2D lighting effects including 2D global illumination.
- [Font rendering](https://docs.godotengine.org/en/stable/tutorials/ui/gui_using_fonts.html#doc-gui-using-fonts) using bitmaps, rasterization using FreeType or multi-channel signed distance fields (MSDF).

  >
  > - Bitmap fonts can be exported using tools like BMFont, or imported from images (for fixed-width fonts only).
  > - Dynamic fonts support monochrome fonts as well as colored fonts (e.g. for emoji). Supported formats are TTF, OTF, WOFF1 and WOFF2.
  > - Dynamic fonts support optional font outlines with adjustable width and color.
  > - Dynamic fonts support variable fonts and OpenType features including ligatures.
  > - Dynamic fonts support simulated bold and italic when the font file lacks those styles.
  > - Dynamic fonts support oversampling to keep fonts sharp at higher resolutions.
  > - Dynamic fonts support subpixel positioning to make fonts crisper at low sizes.
  > - Dynamic fonts support LCD subpixel optimizations to make fonts even crisper at low sizes.
  > - Signed distance field fonts can be scaled at any resolution without requiring re-rasterization. Multi-channel usage makes SDF fonts scale down to lower sizes better compared to monochrome SDF fonts.
- [Oversampling](https://docs.godotengine.org/en/stable/tutorials/rendering/multiple_resolutions.html#doc-multiple-resolutions-font-and-image-oversampling) for SVG images using the [DPITexture](https://docs.godotengine.org/en/stable/classes/class_dpitexture.html#class-dpitexture) import type. This allows for sharper results when scaling up the texture by re-rasterizing the SVG source image to a new resolution at run-time.
  - Oversampling can optionally take individual [CanvasItem](https://docs.godotengine.org/en/stable/classes/class_canvasitem.html#class-canvasitem) scales into account for crisper rendering when scaling nodes.
- GPU-based [particles](https://docs.godotengine.org/en/stable/tutorials/2d/particle_systems_2d.html#doc-particle-systems-2d) with support for [custom particle shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/particle_shader.html#doc-particle-shader).
- CPU-based particles.
- Optional [2D HDR rendering](https://docs.godotengine.org/en/stable/tutorials/3d/environment_and_post_processing.html#doc-environment-and-post-processing-using-glow-in-2d) for better glow capabilities.
- Optional debanding to reduce banding artifacts in gradients.
- [HDR output](https://docs.godotengine.org/en/stable/tutorials/rendering/hdr_output.html#doc-hdr-output) on supported platforms and renderers.

## 2D tools[](#d-tools)

- [TileMaps](https://docs.godotengine.org/en/stable/tutorials/2d/using_tilemaps.html#doc-using-tilemaps) for 2D tile-based level design.
- 2D camera with built-in smoothing and drag margins.
- Path2D node to represent a path in 2D space.

  >
  > - Can be drawn in the editor or generated procedurally.
  > - PathFollow2D node to make nodes follow a Path2D.
- [2D geometry helper class](https://docs.godotengine.org/en/stable/classes/class_geometry2d.html#class-geometry2d).

## 2D physics[](#d-physics)

**Physics bodies:**

- Static bodies.
- Animatable bodies (for objects moving only by script or animation, such as doors and platforms).
- Rigid bodies.
- Character bodies.
- Joints.
- Areas to detect bodies entering or leaving it.
- [Physics interpolation](https://docs.godotengine.org/en/stable/tutorials/physics/interpolation/index.html#doc-physics-interpolation).

**Collision detection:**

- Built-in shapes: line, box, circle, capsule, world boundary (infinite plane).
- Collision polygons (can be drawn manually or generated from a sprite in the editor).

## 3D graphics[](#id1)

- Linear HDR internal lighting calculations.
- Optional debanding to reduce banding artifacts in gradients.
- [HDR output](https://docs.godotengine.org/en/stable/tutorials/rendering/hdr_output.html#doc-hdr-output) on supported platforms and renderers.
- Perspective, orthographic and frustum-offset cameras.
- When using the Forward+ renderer, a depth prepass is used to improve performance in complex scenes by reducing the cost of overdraw.
- [Variable rate shading](https://docs.godotengine.org/en/stable/tutorials/3d/variable_rate_shading.html#doc-variable-rate-shading) on supported GPUs in Forward+ and Mobile.

**Physically-based rendering (built-in material features):**

- Follows the Disney PBR model.
- Supports Burley, Lambert, Lambert Wrap (half-Lambert) and Toon diffuse shading modes.
- Supports Schlick-GGX, Toon and Disabled specular shading modes.
- Uses a roughness-metallic workflow with support for ORM textures.
- Uses horizon specular occlusion (Filament model) to improve material appearance.
- Normal mapping.
- Parallax/relief mapping with automatic level of detail based on distance.
- Detail mapping for the albedo and normal maps.
- Sub-surface scattering and transmittance.
- Screen-space refraction with support for material roughness (resulting in blurry refraction).
- Proximity fade (soft particles) and distance fade.
- Distance fade can use alpha blending or dithering to avoid going through the transparent pipeline.
- Dithering can be determined on a per-pixel or per-object basis.

**Real-time lighting:**

- [Directional lights](https://docs.godotengine.org/en/stable/tutorials/3d/lights_and_shadows.html#doc-lights-and-shadows-directional-light) (sun/moon).
- [Omnidirectional lights](https://docs.godotengine.org/en/stable/tutorials/3d/lights_and_shadows.html#doc-lights-and-shadows-omni-light).
- [Spot lights](https://docs.godotengine.org/en/stable/tutorials/3d/lights_and_shadows.html#doc-lights-and-shadows-spot-light) with adjustable cone angle and attenuation.
- [Rectangular area lights](https://docs.godotengine.org/en/stable/tutorials/3d/lights_and_shadows.html#doc-lights-and-shadows-area-light) with an optional texture to determine the shape and color.
- Specular, indirect light, and volumetric fog energy can be adjusted on a per-light basis.
- Adjustable light "size" for spherical omni and disc spot lights (will also make shadows blurrier with variable penumbra).
- Optional distance fade system to fade distant lights and their shadows, improving performance.
- When using the Forward+ renderer (default on desktop), lights are rendered with clustered forward optimizations to decrease their individual cost. Clustered rendering also lifts any limits on the number of lights that can be used on a mesh.
- When using the Mobile renderer, up to 8 omni lights, 8 spot lights, and 8 area lights can be displayed per mesh resource. Baked lighting can be used to overcome this limit if needed.

**Shadow mapping:**

- *DirectionalLight3D:* Orthogonal (fastest), PSSM 2-split and 4-split. Supports blending between splits.
- *OmniLight3D:* Dual paraboloid (fast) or cubemap (slower but more accurate). Supports colored projector textures in the form of panoramas.
- *SpotLight3D:* Single texture. Supports colored projector textures.
- *AreaLight3D:* Single texture with dual paraboloid distortion to approximate the light's shape.
- Shadow normal offset bias and shadow pancaking to decrease the amount of visible shadow acne and peter-panning.
- PCSS-like shadow blur based on the light size and distance from the surface the shadow is cast on. Supported for all light types.
- Adjustable shadow blur on a per-light basis.

**Global illumination with indirect lighting:**

- [Baked lightmaps](https://docs.godotengine.org/en/stable/tutorials/3d/global_illumination/using_lightmap_gi.html#doc-using-lightmap-gi) (fast, but can't be updated at runtime).

  >
  > - Supports baking indirect light only or baking both direct and indirect lighting. The bake mode can be adjusted on a per-light basis to allow for hybrid light baking setups.
  > - Supports lighting dynamic objects using automatic and manually placed probes.
  > - Optionally supports directional lighting based on spherical harmonics.
  > - Optionally supports baking a shadowmask for distant static directional shadows.
  > - Optional supersampling at bake-time to improve quality and reduce light leaking at the cost of increased bake times and memory usage during baking.
  > - Lightmaps are baked on the GPU using compute shaders (much faster than CPU lightmapping). Baking can only be performed from the editor, not in exported projects.
  > - Supports GPU-based [denoising](https://docs.godotengine.org/en/stable/tutorials/3d/global_illumination/using_lightmap_gi.html#doc-using-lightmap-gi-denoising) with JNLM out of the box, or higher-quality CPU/GPU-based denoising with OIDN (requires downloading OIDN separately).
  > - Lightmaps are rendered with bicubic filtering to reduce scaling artifacts.
- [Voxel-based GI probes](https://docs.godotengine.org/en/stable/tutorials/3d/global_illumination/using_voxel_gi.html#doc-using-voxel-gi). Supports dynamic lights *and* dynamic occluders, while also supporting reflections. Requires a fast baking step which can be performed in the editor or at runtime (including from an exported project).
- [Signed-distance field GI](https://docs.godotengine.org/en/stable/tutorials/3d/global_illumination/using_sdfgi.html#doc-using-sdfgi) designed for large open worlds. Supports dynamic lights, but not dynamic occluders. Supports reflections. No baking required.
- [Screen-space indirect lighting (SSIL)](https://docs.godotengine.org/en/stable/tutorials/3d/environment_and_post_processing.html#doc-environment-and-post-processing-ssil) at half or full resolution. Fully real-time and supports any kind of emissive light source (including decals).
- VoxelGI and SDFGI use a deferred pass to allow for rendering GI at half resolution to improve performance (while still having functional MSAA support).

**Reflections:**

- Voxel-based reflections (when using GI probes) and SDF-based reflections (when using signed distance field GI). Voxel-based reflections are visible on transparent surfaces, while rough SDF-based reflections are visible on transparent surfaces.
- Fast baked reflections or slow real-time reflections using ReflectionProbe. Parallax box correction can optionally be enabled.
- Screen-space reflections with support for material roughness.
- Reflection techniques can be mixed together for greater accuracy or scalability.
- When using the Forward+ renderer (default on desktop), reflection probes are rendered with clustered forward optimizations to decrease their individual cost. Clustered rendering also lifts any limits on the number of reflection probes that can be used on a mesh.
- When using the Mobile renderer, up to 8 reflection probes can be displayed per mesh resource. When using the Compatibility renderer, up to 2 reflection probes can be displayed per mesh resource.

**Decals:**

- [Supports albedo](https://docs.godotengine.org/en/stable/tutorials/3d/using_decals.html#doc-using-decals), emissive, ORM, and normal mapping.
- Texture channels are smoothly overlaid on top of the underlying material, with support for normal/ORM-only decals.
- Support for normal fade to fade the decal depending on its incidence angle.
- Does not rely on runtime mesh generation. This means decals can be used on complex skinned meshes with no performance penalty, even if the decal moves every frame.
- Support for nearest, bilinear, trilinear or anisotropic texture filtering (configured globally).
- Optional distance fade system to fade distant decals, improving performance.
- When using the Forward+ renderer (default on desktop), decals are rendered with clustered forward optimizations to decrease their individual cost. Clustered rendering also lifts any limits on the number of decals that can be used on a mesh.
- When using the Mobile renderer, up to 8 decals can be displayed per mesh resource.

**Sky:**

- Panorama sky (using an HDRI).
- Procedural sky and Physically-based sky that respond to the DirectionalLights in the scene.
- Support for [custom sky shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/sky_shader.html#doc-sky-shader), which can be animated.
- The radiance map used for ambient and specular light can be updated in real-time depending on the quality settings chosen.

**Fog:**

- Exponential depth fog.
- Exponential height fog.
- Support for automatic fog color depending on the sky color (aerial perspective).
- Support for sun scattering in the fog.
- Support for controlling how much fog rendering should affect the sky, with separate controls for traditional and volumetric fog.
- Support for making specific materials ignore fog.

**Volumetric fog:**

- Global [volumetric fog](https://docs.godotengine.org/en/stable/tutorials/3d/volumetric_fog.html#doc-volumetric-fog) that reacts to lights and shadows.
- Volumetric fog can take indirect light into account when using VoxelGI or SDFGI.
- Fog volume nodes that can be placed to add fog to specific areas (or remove fog from specific areas). Supported shapes include box, ellipse, cone, cylinder, and 3D texture-based density maps.
- Each fog volume can have its own custom shader.
- Can be used together with traditional fog.

**Particles:**

- GPU-based particles with support for subemitters (2D + 3D), trails (2D + 3D), attractors (3D only) and collision (2D + 3D).
  - 3D particle attractor shapes supported: box, sphere and 3D vector fields.
  - 3D particle collision shapes supported: box, sphere, baked signed distance field and real-time heightmap (suited for open world weather effects).
  - 2D particle collision is handled using a signed distance field generated in real-time based on [LightOccluder2D](https://docs.godotengine.org/en/stable/classes/class_lightoccluder2d.html#class-lightoccluder2d) nodes in the scene.
  - Trails can use the built-in ribbon trail and tube trail meshes, or custom meshes with skeletons.
  - Support for custom particle shaders with manual emission.
- CPU-based particles.

**Post-processing:**

- Tonemapping (Linear, Reinhard, Filmic, ACES, AgX).
- Automatic exposure adjustments based on viewport brightness (and manual exposure override).
- Near and far depth of field with adjustable bokeh simulation (box, hexagon, circle).
- Screen-space ambient occlusion (SSAO) at half or full resolution.
- Glow/bloom with optional bicubic upscaling and several blend modes available: Screen, Soft Light, Add, Replace, Mix.
- Glow can have a colored dirt map texture, acting as a lens dirt effect.
- Glow can be [used as a screen-space blur effect](https://docs.godotengine.org/en/stable/tutorials/3d/environment_and_post_processing.html#doc-environment-and-post-processing-using-glow-to-blur-the-screen).
- Color correction using a one-dimensional ramp or a 3D LUT texture.
- Roughness limiter to reduce the impact of specular aliasing.
- Brightness, contrast and saturation adjustments.

**Texture filtering:**

- Nearest, bilinear, trilinear or anisotropic filtering.
- Filtering options are defined on a per-use basis, not a per-texture basis.

**Texture VRAM compression:**

- BPTC (for high-quality compression targeting desktop platforms).
- ASTC (for high-quality compression targeting mobile platforms).
- ETC2 (for fast compression targeting mobile platforms).
- S3TC (for fast compression targeting desktop platforms).
- Basis Universal (slow, but only requires one encoding for all platforms).

**Antialiasing:**

- Temporal [antialiasing](https://docs.godotengine.org/en/stable/tutorials/3d/3d_antialiasing.html#doc-3d-antialiasing) (TAA).
- AMD FidelityFX Super Resolution 2.2 [antialiasing](https://docs.godotengine.org/en/stable/tutorials/3d/3d_antialiasing.html#doc-3d-antialiasing) (FSR2), which can be used at native resolution as a form of high-quality temporal antialiasing.
- Multi-sample antialiasing (MSAA), for both [2D antialiasing](https://docs.godotengine.org/en/stable/tutorials/2d/2d_antialiasing.html#doc-2d-antialiasing) and [3D antialiasing](https://docs.godotengine.org/en/stable/tutorials/3d/3d_antialiasing.html#doc-3d-antialiasing).
- Fast approximate antialiasing (FXAA).
- Super-sample antialiasing (SSAA) using bilinear 3D scaling and a 3D resolution scale above 1.0.
- Alpha antialiasing, MSAA alpha to coverage and alpha hashing on a per-material basis.

**Resolution scaling:**

- Support for [rendering 3D at a lower resolution](https://docs.godotengine.org/en/stable/tutorials/3d/resolution_scaling.html#doc-resolution-scaling) while keeping 2D rendering at the original scale. This can be used to improve performance on low-end systems or improve visuals on high-end systems.
- Resolution scaling uses nearest-neighbor filtering, bilinear filtering, AMD FidelityFX Super Resolution 1.0 (FSR1), or AMD FidelityFX Super Resolution 2.2.1 (FSR2).
- Texture mipmap LOD bias is adjusted automatically to improve quality at lower resolution scales. It can also be modified with a manual offset.

Most effects listed above can be adjusted for better performance or to further improve quality. This can be helpful when [using Godot for offline rendering](https://docs.godotengine.org/en/stable/tutorials/animation/creating_movies.html#doc-creating-movies).

## 3D tools[](#id2)

- Built-in meshes: cube, cylinder/cone, (hemi)sphere, prism, plane, quad, torus, ribbon, tube.
- [GridMaps](https://docs.godotengine.org/en/stable/tutorials/3d/using_gridmaps.html#doc-using-gridmaps) for 3D tile-based level design.
- [Constructive solid geometry](https://docs.godotengine.org/en/stable/tutorials/3d/csg_tools.html#doc-csg-tools) (intended for prototyping).
- Tools for [procedural geometry generation](https://docs.godotengine.org/en/stable/tutorials/3d/procedural_geometry/index.html#doc-procedural-geometry).
- Path3D node to represent a path in 3D space.

  >
  > - Can be drawn in the editor or generated procedurally.
  > - PathFollow3D node to make nodes follow a Path3D.
- [3D geometry helper class](https://docs.godotengine.org/en/stable/classes/class_geometry3d.html#class-geometry3d).
- Support for exporting the current scene as a glTF 2.0 file, both from the editor and at runtime from an exported project.

## 3D physics[](#id3)

**Physics bodies:**

- Static bodies.
- Animatable bodies (for objects moving only by script or animation, such as doors and platforms).
- Rigid bodies.
- Character bodies.
- Vehicle bodies (intended for arcade physics, not simulation).
- Joints.
- [Soft bodies](https://docs.godotengine.org/en/stable/tutorials/physics/soft_body.html#doc-soft-body).
- [Ragdolls](https://docs.godotengine.org/en/stable/tutorials/physics/ragdoll_system.html#doc-ragdoll-system).
- Areas to detect bodies entering or leaving it.
- [Physics interpolation](https://docs.godotengine.org/en/stable/tutorials/physics/interpolation/index.html#doc-physics-interpolation).

**Collision detection:**

- Built-in shapes: cuboid, sphere, capsule, cylinder, world boundary (infinite plane).
- Generate triangle collision shapes for any mesh from the editor.
- Generate one or several convex collision shapes for any mesh from the editor.

## Shaders[](#shaders)

- *2D:* Custom vertex, fragment, and light shaders.
- *3D:* Custom vertex, fragment, light, sky, and fog shaders.
- Custom shaders can procedurally generate and modify textures in real-time using [DrawableTexture2D](https://docs.godotengine.org/en/stable/classes/class_drawabletexture2d.html#class-drawabletexture2d).
- Text-based shaders using a [shader language inspired by GLSL](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/shading_language.html#doc-shading-language).
- Syntax highlighting is provided on GitHub by using `gdshader` as the language name in a Markdown code block.
- Visual shader editor.

  >
  > - Support for [visual shader plugins](https://docs.godotengine.org/en/stable/tutorials/plugins/editor/visual_shader_plugins.html#doc-visual-shader-plugins).

## Scripting[](#scripting)

**General:**

- Object-oriented design pattern with scripts extending nodes.
- Signals and groups for communicating between scripts.
- Support for [cross-language scripting](https://docs.godotengine.org/en/stable/tutorials/scripting/cross_language_scripting.html#doc-cross-language-scripting).
- Many 2D, 3D and 4D linear algebra data types such as vectors and transforms.

[GDScript:](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/index.html#doc-gdscript)

- [High-level interpreted language](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_basics.html#doc-gdscript-reference) with [optional static typing](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/static_typing.html#doc-gdscript-static-typing).
- Syntax inspired by Python. However, GDScript is **not** based on Python.
- Syntax highlighting is provided on GitHub by using `gdscript` as the language name in a Markdown code block.
- [Use threads](https://docs.godotengine.org/en/stable/tutorials/performance/using_multiple_threads.html#doc-using-multiple-threads) to perform asynchronous actions or make use of multiple processor cores.

[C#:](https://docs.godotengine.org/en/stable/tutorials/scripting/c_sharp/index.html#doc-c-sharp)

- Packaged in a separate binary to keep file sizes and dependencies down.
- Supports .NET 8 and higher.

  >
  > - Full support for the C# 12.0 syntax and features.
- Supports Windows, Linux, and macOS. Since Godot 4.2, experimental support for Android and iOS is also available.

  >
  > - On the iOS platform only some architectures are supported: `arm64`.
  > - The web platform is currently unsupported. To use C# on that platform, consider Godot 3 instead.
- Using an external editor is recommended to benefit from IDE functionality.

**GDExtension (C, C++, Rust, D, ...):**

- When you need it, link to native libraries for higher performance and third-party integrations.

  >
  > - For scripting game logic, GDScript or C# are recommended if their performance is suitable.
- Official GDExtension bindings for [C](https://github.com/godotengine/godot-headers) and [C++](https://github.com/godotengine/godot-cpp).

  >
  > - Use any build system and language features you wish.
- Actively developed GDExtension bindings for [D](https://github.com/godot-dlang/godot-dlang), [Swift](https://github.com/migueldeicaza/SwiftGodot), and [Rust](https://github.com/godot-rust/gdextension) bindings provided by the community. (Some of these bindings may be experimental and not production-ready).

## Audio[](#audio)

**Features:**

- Mono, stereo, 5.1 and 7.1 output.
- Non-positional and positional playback in 2D and 3D.

  >
  > - Optional Doppler effect in 2D and 3D.
- Support for re-routable [audio buses](https://docs.godotengine.org/en/stable/tutorials/audio/audio_buses.html#doc-audio-buses) and effects with dozens of effects included.
- Support for polyphony (playing several sounds from a single [AudioStreamPlayer](https://docs.godotengine.org/en/stable/classes/class_audiostreamplayer.html#class-audiostreamplayer) node).
- Support for random volume and pitch.
- Support for real-time pitch scaling.
- Support for sequential/random sample selection, including repetition prevention when using random sample selection.
- [AudioListener2D](https://docs.godotengine.org/en/stable/classes/class_audiolistener2d.html#class-audiolistener2d) and [AudioListener3D](https://docs.godotengine.org/en/stable/classes/class_audiolistener3d.html#class-audiolistener3d) nodes to listen from a position different from the camera.
- Support for [procedural audio generation](https://docs.godotengine.org/en/stable/classes/class_audiostreamgenerator.html#class-audiostreamgenerator).
- Audio input to record microphones.
- [Text to speech](https://docs.godotengine.org/en/stable/tutorials/audio/text_to_speech.html#doc-text-to-speech) using platform-provided TTS engines.
- MIDI input.

  >
  > - No support for MIDI output yet.

**APIs used:**

- *Windows:* WASAPI.
- *macOS:* CoreAudio.
- *Linux:* PulseAudio or ALSA.

## Import[](#import)

- Support for [custom import plugins](https://docs.godotengine.org/en/stable/tutorials/plugins/editor/import_plugins.html#doc-import-plugins).

**Formats:**

- *Images:* See [Importing images](https://docs.godotengine.org/en/stable/tutorials/assets_pipeline/importing_images.html#doc-importing-images).
- *Audio:*

  >
  > - WAV with optional QOA or IMA-ADPCM compression.
  > - Ogg Vorbis.
  > - MP3.
- *3D scenes:* See [Importing 3D scenes](https://docs.godotengine.org/en/stable/tutorials/assets_pipeline/importing_3d_scenes/index.html#doc-importing-3d-scenes).

  >
  > - glTF 2.0 *(recommended)*.
  > - `.blend` (by calling Blender's glTF export functionality transparently).
  > - FBX (by calling [FBX2glTF](https://github.com/godotengine/FBX2glTF) transparently).
  > - Collada (.dae).
  > - Wavefront OBJ (static scenes only, can be loaded directly as a mesh or imported as a 3D scene).
- Support for loading glTF 2.0 scenes at runtime, including from an exported project.
- 3D meshes use [Mikktspace](http://www.mikktspace.com/) to generate tangents on import, which ensures consistency with other 3D applications such as Blender.

## Input[](#input)

- [Input mapping system](https://docs.godotengine.org/en/stable/tutorials/inputs/input_examples.html#doc-input-examples) using hardcoded input events or remappable input actions.

  >
  > - Axis values can be mapped to two different actions with a configurable deadzone.
  > - Use the same code to support both keyboards and gamepads.
- Keyboard input.

  >
  > - Keys can be mapped in "physical" mode to be independent of the keyboard layout.
- Mouse input.

  >
  > - The mouse cursor can be visible, hidden, captured or confined within the window.
  > - The mouse cursor's appearance can be changed to a custom image or one of the system cursors.
  > - When captured, raw input is used on Windows and Linux to sidestep the OS' mouse acceleration settings.
- [Gamepad input](https://docs.godotengine.org/en/stable/tutorials/inputs/controllers_gamepads_joysticks.html#doc-controllers-gamepads-joysticks) (up to 8 simultaneous controllers).
  - Support for [changing the LED color](https://docs.godotengine.org/en/stable/tutorials/inputs/controller_features.html#doc-controller-features-led-color) on supported controllers.
  - Support for [reading motion sensors](https://docs.godotengine.org/en/stable/tutorials/inputs/controller_features.html#doc-controller-features-motion-sensors) on supported controllers (used to implement gyro aiming).
- Pen/tablet input with pressure and tilt support.

## Navigation[](#navigation)

- A\* algorithm in [2D](https://docs.godotengine.org/en/stable/classes/class_astar2d.html#class-astar2d) and [3D](https://docs.godotengine.org/en/stable/classes/class_astar3d.html#class-astar3d).
- Navigation meshes with dynamic obstacle avoidance in [2D](https://docs.godotengine.org/en/stable/tutorials/navigation/navigation_introduction_2d.html#doc-navigation-overview-2d) and [3D](https://docs.godotengine.org/en/stable/tutorials/navigation/navigation_introduction_3d.html#doc-navigation-overview-3d).
- Generate navigation meshes from the editor or at runtime (including from an exported project).

## Networking[](#networking)

- Low-level TCP networking using [StreamPeer](https://docs.godotengine.org/en/stable/classes/class_streampeer.html#class-streampeer) and [TCPServer](https://docs.godotengine.org/en/stable/classes/class_tcpserver.html#class-tcpserver).
- Low-level UDP networking using [PacketPeer](https://docs.godotengine.org/en/stable/classes/class_packetpeer.html#class-packetpeer) and [UDPServer](https://docs.godotengine.org/en/stable/classes/class_udpserver.html#class-udpserver).
- Low-level HTTP requests using [HTTPClient](https://docs.godotengine.org/en/stable/classes/class_httpclient.html#class-httpclient).
- High-level HTTP requests using [HTTPRequest](https://docs.godotengine.org/en/stable/classes/class_httprequest.html#class-httprequest).

  >
  > - Supports HTTPS out of the box using bundled certificates.
- [High-level multiplayer](https://docs.godotengine.org/en/stable/tutorials/networking/high_level_multiplayer.html#doc-high-level-multiplayer) API using UDP and ENet.

  >
  > - Automatic replication using remote procedure calls (RPCs).
  > - Supports unreliable, reliable and ordered transfers.
- [WebSocket](https://docs.godotengine.org/en/stable/tutorials/networking/websocket.html#doc-websocket) client and server, available on all platforms.
- [WebRTC](https://docs.godotengine.org/en/stable/tutorials/networking/webrtc.html#doc-webrtc) client and server, available on all platforms.
- Support for [UPnP](https://docs.godotengine.org/en/stable/classes/class_upnp.html#class-upnp) to sidestep the requirement to forward ports when hosting a server behind a NAT.

## Internationalization[](#internationalization)

- Full support for Unicode including emoji.
- Support for loading system fonts on Windows, macOS, and Linux.
  - By default, system fonts are used as a fallback to display unsupported characters. This allows proper display of multilingual text without having to bundle large font files with a project.
- Store localization strings using [CSV](https://docs.godotengine.org/en/stable/tutorials/i18n/internationalizing_games.html#doc-internationalizing-games) or [gettext](https://docs.godotengine.org/en/stable/tutorials/i18n/localization_using_gettext.html#doc-localization-using-gettext).
  - Support for generating gettext POT and PO files from the editor.
- Use localized strings in your project automatically in GUI elements or by using the `tr()` function.
- Support for pluralization and translation contexts.
- Support for [bidirectional typesetting](https://docs.godotengine.org/en/stable/tutorials/i18n/internationalizing_games.html#doc-internationalizing-games-bidi), text shaping and OpenType localized forms.
- Automatic UI mirroring for right-to-left locales.
- Support for [pseudolocalization](https://docs.godotengine.org/en/stable/tutorials/i18n/pseudolocalization.html#doc-pseudolocalization) to test your project for i18n-friendliness.

## Windowing and OS integration[](#windowing-and-os-integration)

- Spawn multiple independent windows within a single process.
- Move, resize, minimize, and maximize windows spawned by the project.
- Change the window title and icon.
- Create transparent windows to use as overlays, with polygon-based mouse passthrough support.
- Request attention (will cause the title bar to blink on most platforms).
- Fullscreen mode.

  >
  > - Uses borderless fullscreen by default on Windows for fast alt-tabbing, but can optionally use exclusive fullscreen to reduce input lag.
- Borderless windows (fullscreen or non-fullscreen).
- Keep a window always on top.
- Make a window ignore focus (useful for overlays).
- Declare a window as a popup (hidden from task switcher) or exclusive (prevents interacting with other windows from the same process).
- Native file dialog support on Windows, macOS, Linux, and Android.
- Tray icon support on Windows and macOS.
- Global menu integration on macOS.
- Client-side decorations on macOS.
- Execute commands in a blocking or non-blocking manner (including running multiple instances of the same project).
- Open file paths and URLs using default or custom protocol handlers (if registered on the system).
- Parse custom command line arguments.
- Support for screen readers on Windows, macOS, and Linux.
- Any Godot binary (editor or exported project) can be [used as a headless server](https://docs.godotengine.org/en/stable/tutorials/export/exporting_for_dedicated_servers.html#doc-exporting-for-dedicated-servers) by starting it with the `--headless` command line argument. This allows running the engine without a GPU or display server.

## Mobile[](#mobile)

- [Virtual joystick](https://docs.godotengine.org/en/stable/classes/class_virtualjoystick.html#class-virtualjoystick) and [buttons](https://docs.godotengine.org/en/stable/classes/class_touchscreenbutton.html#class-touchscreenbutton) for touch input.
- In-app purchases on [Android](https://docs.godotengine.org/en/stable/tutorials/platform/android/android_in_app_purchases.html#doc-android-in-app-purchases) and [iOS](https://github.com/godot-sdk-integrations/godot-storekit2).
- Support for advertisements using third-party modules.
- Support for picture-in-picture mode on Android.

## XR support (AR and VR)[](#xr-support-ar-and-vr)

- Out of the box [support for OpenXR](https://docs.godotengine.org/en/stable/tutorials/xr/setting_up_xr.html#doc-setting-up-xr).

  >
  > - Including support for popular desktop headsets like the Valve Index, WMR headsets, and Quest over Link.
- Support for [Android-based headsets](https://docs.godotengine.org/en/stable/tutorials/xr/deploying_to_android.html#doc-deploying-to-android) using OpenXR through a plugin.
  - Including support for popular standalone headsets like the Meta Quest 1/2/3 and Pro, Pico 4, Magic Leap 2, and Lynx R1.
- Out of the box limited support for visionOS Apple headsets.
  - Currently only exporting an application for use on a flat plane within the headset is supported. Immersive experiences are not supported.
- Other devices supported through an XR plugin structure.
- Various advanced toolkits are available that implement common features required by XR applications.

## GUI system[](#gui-system)

Godot's GUI is built using the same Control nodes used to make games in Godot. The editor UI can easily be extended in many ways using add-ons.

**Nodes:**

- Buttons.
- Checkboxes, check buttons, radio buttons.
- Text entry using [LineEdit](https://docs.godotengine.org/en/stable/classes/class_lineedit.html#class-lineedit) (single line), [TextEdit](https://docs.godotengine.org/en/stable/classes/class_textedit.html#class-textedit) (multiple lines), and [CodeEdit](https://docs.godotengine.org/en/stable/classes/class_codeedit.html#class-codeedit) (supports syntax highlighting, line numbers, and more).
- Dropdown menus using [PopupMenu](https://docs.godotengine.org/en/stable/classes/class_popupmenu.html#class-popupmenu) and [OptionButton](https://docs.godotengine.org/en/stable/classes/class_optionbutton.html#class-optionbutton) with support for an optional search bar.
- Scrollbars.
- Labels.
- RichTextLabel for [text formatted using BBCode](https://docs.godotengine.org/en/stable/tutorials/ui/bbcode_in_richtextlabel.html#doc-bbcode-in-richtextlabel), with support for animated custom effects.
- Trees (can also be used to represent tables).
- Color picker with RGB, HSV, and OKHSL modes, as well as custom color palettes.
- Controls can be rotated and scaled.
- Drag-and-drop support.

**Sizing:**

- Anchors to keep GUI elements in a specific corner, edge or centered.
- Containers to place GUI elements automatically following certain rules.

  >
  > - [Stack](https://docs.godotengine.org/en/stable/classes/class_boxcontainer.html#class-boxcontainer) layouts.
  > - [Grid](https://docs.godotengine.org/en/stable/classes/class_gridcontainer.html#class-gridcontainer) layouts.
  > - [Flow](https://docs.godotengine.org/en/stable/classes/class_flowcontainer.html#class-flowcontainer) layouts (similar to autowrapping text).
  > - [Margin](https://docs.godotengine.org/en/stable/classes/class_margincontainer.html#class-margincontainer), [centered](https://docs.godotengine.org/en/stable/classes/class_centercontainer.html#class-centercontainer) and [aspect ratio](https://docs.godotengine.org/en/stable/classes/class_aspectratiocontainer.html#class-aspectratiocontainer) layouts.
  > - [Draggable splitter](https://docs.godotengine.org/en/stable/classes/class_splitcontainer.html#class-splitcontainer) layouts.
  > - [Foldable section](https://docs.godotengine.org/en/stable/classes/class_foldablecontainer.html#class-foldablecontainer) layouts.
- Scale to [multiple resolutions](https://docs.godotengine.org/en/stable/tutorials/rendering/multiple_resolutions.html#doc-multiple-resolutions) using the `canvas_items` or `viewport` stretch modes.
- Support any aspect ratio using anchors and the `expand` stretch aspect.

**Theming:**

- Built-in theme editor.

  >
  > - Generate a theme based on the current editor theme settings.
- Procedural vector-based theming using [StyleBoxFlat](https://docs.godotengine.org/en/stable/classes/class_styleboxflat.html#class-styleboxflat).

  >
  > - Supports rounded/beveled corners, drop shadows, per-border widths and antialiasing.
- Texture-based theming using [StyleBoxTexture](https://docs.godotengine.org/en/stable/classes/class_styleboxtexture.html#class-styleboxtexture).

Godot's small distribution size can make it a suitable alternative to frameworks like Electron or Qt.

## Animation[](#animation)

- Direct kinematics and inverse kinematics.
- Support for animating any property with customizable interpolation.
- Support for calling methods in animation tracks.
- Support for playing sounds in animation tracks.
- Support for Bézier curves in animation.

## File formats[](#file-formats)

- Scenes and resources can be saved in [text-based](https://docs.godotengine.org/en/stable/engine_details/file_formats/tscn.html#doc-tscn-file-format) or binary formats.

  >
  > - Text-based formats are human-readable and more friendly to version control.
  > - Binary formats are faster to save/load for large scenes/resources.
- Read and write text or binary files using [FileAccess](https://docs.godotengine.org/en/stable/classes/class_fileaccess.html#class-fileaccess).

  >
  > - Can optionally be compressed or encrypted.
- Read and write [JSON](https://docs.godotengine.org/en/stable/classes/class_json.html#class-json) files.
- Read and write INI-style configuration files using [ConfigFile](https://docs.godotengine.org/en/stable/classes/class_configfile.html#class-configfile).

  >
  > - Can (de)serialize any Godot datatype, including Vector2/3, Color, ...
- Read XML files using [XMLParser](https://docs.godotengine.org/en/stable/classes/class_xmlparser.html#class-xmlparser).
- [Load and save images, audio/video, fonts and ZIP archives](https://docs.godotengine.org/en/stable/tutorials/io/runtime_file_loading_and_saving.html#doc-runtime-loading-and-saving) in an exported project without having to go through Godot's import system.
- Pack game data into a PCK file (custom format optimized for fast seeking), into a ZIP archive, or directly into the executable for single-file distribution.
- [Export additional PCK files](https://docs.godotengine.org/en/stable/tutorials/export/exporting_pcks.html#doc-exporting-pcks) that can be read by the engine to support mods and DLCs.

## Miscellaneous[](#miscellaneous)

- [Video playback](https://docs.godotengine.org/en/stable/tutorials/animation/playing_videos.html#doc-playing-videos) with built-in support for Ogg Theora.
- [Movie Maker mode](https://docs.godotengine.org/en/stable/tutorials/animation/creating_movies.html#doc-creating-movies) to record videos from a running project with synchronized audio and perfect frame pacing.
- [Low-level access to servers](https://docs.godotengine.org/en/stable/tutorials/performance/using_servers.html#doc-using-servers) which allows bypassing the scene tree's overhead when needed.
- [Command line interface](https://docs.godotengine.org/en/stable/tutorials/editor/command_line_tutorial.html#doc-command-line-tutorial) for automation.

  >
  > - Export and deploy projects using continuous integration platforms.
  > - [Shell completion scripts](https://github.com/godotengine/godot/tree/master/misc/dist/shell) are available for Bash, zsh and fish.
  > - Print colored text to standard output on all platforms using [print\_rich](https://docs.godotengine.org/en/stable/classes/class_%40globalscope.html#class-globalscope-method-print-rich).
- The editor can [detect features used in a project and create a compilation profile](https://docs.godotengine.org/en/stable/tutorials/editor/using_engine_compilation_configuration_editor.html#doc-engine-compilation-configuration-editor), which can be used to create smaller export template binaries with unneeded features disabled.
- Support for [C++ modules](https://docs.godotengine.org/en/stable/engine_details/engine_api/custom_modules_in_cpp.html#doc-custom-modules-in-cpp) statically linked into the engine binary.
  - Most built-in modules can be disabled at compile-time to reduce binary size in custom builds. See [Optimizing a build for size](https://docs.godotengine.org/en/stable/engine_details/development/compiling/optimizing_for_size.html#doc-optimizing-for-size) for details.
- Engine and editor written in C++17.

  >
  > - Can be [compiled](https://docs.godotengine.org/en/stable/engine_details/development/compiling/introduction_to_the_buildsystem.html#doc-introduction-to-the-buildsystem) using GCC, Clang and MSVC. MinGW is also supported.
  > - Friendly towards packagers. In most cases, system libraries can be used instead of the ones provided by Godot. The build system doesn't download anything. Builds can be fully reproducible.
- Licensed under the permissive MIT license.

  >
  > - Open development process with [contributions welcome](https://contributing.godotengine.org/en/latest/organization/how_to_contribute.html).

See also

The [Godot proposals repository](https://github.com/godotengine/godot-proposals) lists features that have been requested by the community and may be implemented in future Godot releases.
