---
title: "Godot 4 Features and Improvements"
date: 2025-08-18T10:00:00+02:00
author: Jan Mesarč
categories:
  - godot
tags:
  - godot
  - gamedev
toc: true
---

## TL;DR (Quick Summary)

- Big upgrade over Godot 3: faster, better graphics, smoother workflow
- Vulkan is default: modern lighting, more lights, steadier frame time
- Optional Direct3D 12 (Windows) and future Windows tech (still maturing)
- Jolt Physics (optional) adds more stable, faster 3D physics
- Editor upgrades like multi-window, embedded game view, pins, better profiling, this saves daily time
- Migration: fix shaders, review lighting (AgX tone mapping), profile again, add backend toggle

## Who Is This For?

![Who Is This For](https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExd2ZqdGFtM3gyNmp2YThoZWJmZnE5d2hpeDd1dHJ3MGk5dXM1cG55cSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/gjNtbBnTIY354ZHBKW/giphy.gif)

Developers moving from Godot 3, new users picking an open-source engine, and Windows focused teams deciding between Vulkan and Direct3D 12.

## Introduction

Godot 4 is a big step forward for game developers. It brings many new features that make creating games easier and more fun. One of the most important changes is the new Vulkan rendering system, which helps games look better and run faster on modern computers. Godot 4 also adds advanced graphics features, so your games can have more realistic lighting, shadows, and effects.

Another exciting update is the support for Direct3D 12 in official builds. This means Godot can run better on Windows computers, but there are still some things to watch out because Direct3D 12 support is new and may not be perfect yet. For example some scenes show a bit more CPU cost, a few drivers have rare render glitches like pink/fallback materials from resource binding issues, HDR is experimental, and performance can swing between GPU vendors. Developers should test their games carefully if they want to use this feature.

In this article, we will look at how Vulkan rendering, advanced graphics features, and Direct3D 12 support make Godot 4 a great choice for making games. Let's explore what makes this version special and how it compares to its predecessor.

## Godot 3 vs Godot 4: The Quick Comparison

Godot 4 represents a dramatic evolution from Godot 3, with the engine being almost completely rewritten from the ground up. This overhaul was necessary to address fundamental flaws in the 3.x architecture, incorporate user feedback, and prepare the engine for future development. While Godot 3 was already a capable engine, Godot 4 brings modern rendering capabilities, improved performance, and enhanced workflows across the board.

| Feature              | Godot 3                  | Godot 4                                              |
| -------------------- | ------------------------ | ---------------------------------------------------- |
| **Primary Renderer** | OpenGL ES 2.0 / 3.0      | **Vulkan** (with OpenGL compatibility renderer)      |
| **Windows Renderer** | OpenGL ES                | **Direct3D 12** (optional, new)                      |
| **3D Lighting**      | Limited lights, baked global illumination | Clustered lighting (many lights), Real-time global illumination (SDFGI) |
| **Physics**          | Built-in (Bullet)        | Godot Physics, **Jolt Physics** (optional)           |
| **Scripting**        | GDScript, C#, VisualScript | GDScript (improved), C# (.NET 8), GDExtension        |
| **Editor UI**        | Single Window            | **Multi-window support**, embedded game view         |

Overall, Godot 4 is faster, looks better, and gives you more tools to make great games.

### Jolt Physics: A Game-Changer for Physical Interactions

With Godot 4.4, the engine now integrates Jolt Physics as a native (optional) feature. You can still use the default Godot Physics backend if you prefer. Jolt brings several advantages:

- More accurate and stable physics simulations
- Better performance for complex physical interactions
- Improved 3D physics interpolation for smoother object movement
- Physics-based 3D object snapping for more intuitive level design

**Note:** Jolt is for 3D physics only. For 2D projects, you will continue to use the built-in Godot Physics 2D engine.

#### When to Choose Jolt over Godot Physics

Jolt is the recommended 3D physics engine for most projects, particularly those that involve:

*   **Many rigid bodies:** Jolt excels at handling hundreds or thousands of dynamic objects.
*   **Complex simulations:** Its stability helps prevent objects from jittering or passing through each other.
*   **Performance-critical applications:** Jolt is generally faster and uses multithreading more effectively.

You might stick with the default Godot Physics if your project relies on features not yet supported in Jolt, such as `SoftBody3D` nodes.

#### How to Enable Jolt

You can switch to Jolt with a few clicks:

1.  Go to **Project > Project Settings**.
2.  Navigate to **Physics > 3D**.
3.  Change the **Physics Engine** from "GodotPhysics" to "JoltPhysics".


### Vulkan Rendering in Godot 4: A Major Leap Forward

Godot 4's switch to Vulkan is one of its most important improvements. Vulkan is a modern graphics API that provides high-efficiency, cross-platform access to modern GPUs. It allows games to run faster and look better because it gives developers more control over how graphics are drawn on the screen.

#### Why Vulkan is better than Godot 3’s older systems (GLES2 and GLES3)

- Improved speed: makes better use of modern CPUs and GPUs so games run smoother
- Better graphics quality: supports advanced lighting, shadows, effects for more realistic scenes
- Efficient GPU use: cuts wasted work so you can push more detail
- Cross-platform: works on Windows, Linux, Android and more

Godot’s team worked carefully over the years to build a strong Vulkan backend. They started with great 2D support and then expanded to full 3D capabilities. This means you can expect solid performance and visuals whether you’re making a 2D or 3D game.

#### Metal Support for MacOS and iOS

Godot 4.4 adds a native Metal backend for macOS and iOS. Metal is a low-level graphics API like Vulkan or D3D12. Before this, Godot used MoltenVK to run Vulkan over Metal. The native path is faster and gives more control over features and future tuning. It currently works **only** on Apple Silicon (ARM) devices.

**Metal** is now the default renderer on Apple platforms. You can still pick Vulkan via **MoltenVK** in Project Settings by switching the rendering driver to **Vulkan**.

#### Vulkan Deep Dive: Architecture and Performance

Under the hood, the Vulkan renderer in Godot 4 was almost rewritten from scratch. The goals were: give the engine more direct control, move data through the GPU in a clean way, and cut down hidden work done by drivers.

Main building blocks (in simple terms):

- Render phases: groups work (shadows, lighting, post-processing) so the GPU does not stall
- Forward+ / clustered lighting: sorts lights into screen regions so many dynamic lights are affordable
- Packed uniform data: keeps object/material data tight to reduce state changes
- Multithreaded prep: culling, skinning, particles, light assignment spread across CPU cores
- Smart GPU memory: custom allocator reduces expensive allocations (helps mobile)
- Compute shaders for global illumination: SDFGI (Signed Distance Field Global Illumination) and probes run on GPU for bounce light without baking each change

What this gives you:

- More stable frame time (fewer random spikes)
- Scales up (enable effects) or down (disable) for different hardware
- Easier extension with native modules (GDExtension) without editing engine core

Simple performance tips:

1. Use the built-in Frame Profiler (find CPU vs GPU bound first)
2. If GPU bound: lower overdraw (particle clouds, large transparent quads) before shrinking textures
3. Use MultiMesh / instancing for many repeated meshes
4. Keep light radii reasonable (huge omni lights cost more)
5. Limit shader variants (prefer uniforms over many #ifdef blocks)
6. Add LOD or imposters for far meshes

Current limits (2025):

- Some Android Vulkan drivers are still weaker: test early on target phones
- SDFGI (Signed Distance Field Global Illumination) costs more than baked lightmaps in big indoor maps. Use a mix: baked for static, SDFGI for moving lights
- AgX tone mapping (a filmic color curve) changes color balance; check UI and color graded textures

Coming from GLES3? Do this:

- Remove custom depth pre-pass tricks; Forward+ handles most lighting cost now
- Update old shaders: built-in variables changed; fix warnings the shader compiler shows
- Re-tune color grading (contrast may look different with AgX)

### Direct3D 12 Support: A New Option for Windows Users

Godot 4 also introduced support for Direct3D 12 (D3D12) in its official builds on Windows. Direct3D 12 is a modern graphics API by Microsoft, similar in power to Vulkan but designed specifically for Windows.

#### What Direct3D 12 brings to the table

- Better performance on Windows: can use latest platform features
- Improved graphics effects: supports advanced lighting and shadows like Vulkan
- Low overhead: talks more directly to the GPU, reducing extra work

However, D3D12 support in Godot 4 is still fairly new. This means that while it has the potential to boost performance on Windows machines, developers need to test their games carefully to avoid bugs or crashes caused by the newer implementation.

#### Direct3D 12 Deep Dive: When and Why to Use It

The D3D12 backend is mainly for projects that will ship on Windows only and want Windows tools.

Advantages over Vulkan (Windows only):

- Tools: PIX and Visual Studio GPU tools work very well
- Some hardware drivers get D3D12 fixes a little earlier
- Future Windows tech (like DirectStorage) may integrate more smoothly

Current status (2025):

- Main visual features match Vulkan like lighting, SDFGI (Signed Distance Field Global Illumination), volumetrics, post-processing
- Compute jobs behave the same in most cases

Limits / risks:

- In some scenes CPU cost is a bit higher (still improving)
- Fewer users test this path, so rare bugs may appear
- HDR still experimental; double-check colors on HDR monitors

Choosing a backend (quick guide):

| Scenario | Prefer Vulkan | Prefer D3D12 |
|---------|---------------|--------------|
| Cross-platform (Linux/macOS/Android) | Yes | No |
| Windows-only PC title | Maybe | Yes |
| Need PIX GPU captures | Not applicable | Yes |
| Console support | Using compatibility layers (adds some overhead): Yes | Xbox |
| Team familiarity | If already using Vulkan tooling | If invested in DirectX tooling |

Switch / test plan:

1. Work mainly in Vulkan (portable)
2. Once a week run the project in D3D12
3. Keep simple test scenes (lighting stress, particles, global illumination, UI scale) and open them in both backends

Debug tips (D3D12):

- Use PIX to spot too many state changes
- Watch video memory use; big streamed textures can cause stutter
- If materials turn pink, check descriptor heap (resource binding) issues

Fallback:

If a crash happens only in D3D12 near release, let players force Vulkan with a launch flag and store the backend choice in a config file.

Quick backend launch tip: you can start the editor or game with `--rendering-driver` vulkan (or D3D12 on Windows) to test both paths.

### Advanced Rendering Features

Beyond the core rendering changes, Godot 4 introduces several specific improvements:

- AgX tone mapping (filmic color curve for natural midtones)
- Ubershader support (simpler shader variant handling)
- Real-time global illumination
- Volumetric fog effects
- Improved shadow rendering

These features bring Godot closer to the capabilities of commercial engines while maintaining its lightweight and accessible nature.

### Enhanced Developer Experience

Godot 4.4 also brings several workflow improvements that save significant time over a project's lifecycle. These quality-of-life updates reduce the friction between your ideas and their implementation.

**Key Workflow Upgrades:**

*   **Multi-window and Embedded Game View:** Undock editor panels to a second monitor or edit your scene while the game is running in a separate process.
*   **Inspector Pins:** Pin frequently accessed nodes or properties in the Inspector for quick access, ideal for tweaking balance values like speed, damage, or costs.
*   **GDScript Enhancements:** Enjoy better static typing, improved autocompletion, and support for typed Arrays and Dictionaries (`Array[int]`, `Dictionary[String, float]`), leading to clearer and often faster code.
*   **Modern .NET Support:** Godot 4 ships with .NET 8 compatibility, enabling modern C# features and faster build times.
*   **GDExtension:** Add high-performance native code using C++ or other languages without having to recompile the entire engine.
*   **Improved Profiler:** A unified profiler helps you visualize script, physics, rendering, and GPU performance in one place to easily identify bottlenecks.
*   **Better Search:** The editor's search functionality now finds signals and input map actions, speeding up navigation.

**Migration Checklist for Godot 3 Users:**

If you are moving a project from Godot 3, watch for these common issues:

- [ ] **Run the Converter:** Start by copying your project and running the built-in project converter.
- [ ] **Fix Shaders:** Update your shaders to fix syntax changes and compiler warnings.
- [ ] **Re-tune Lighting:** The new AgX tone mapping will change your color balance. Re-check lighting and color grading.
- [ ] **Review Physics:** Resource names and some behaviors in the physics system have changed.
- [ ] **Profile Again:** Your old performance bottlenecks may no longer be relevant. Use the profiler to find new hotspots.
- [ ] **Add a Backend Switch:** If targeting Windows, consider adding a settings option to let players switch between Vulkan and D3D12.


## Looking Forward

While Godot 4 represents a massive step forward, it's worth noting that the developers consider it "the beginning of a journey". The team continues to optimize performance and add features with each point release, as evidenced by the improvements in versions 4.2, 4.3, and 4.4.

For developers considering Godot 4 for their projects, the engine now offers a compelling package: modern rendering capabilities, improved physics, enhanced workflows, and even upcoming console support. Whether you're creating 2D or 3D games, Godot 4 provides the tools you need without the licensing costs or restrictions of commercial engines.

As you explore Godot 4 for your next project, remember that the transition from Godot 3 might require some adjustment, but the benefits in terms of capabilities and future-proofing make it well worth the effort.

## Conclusion

![Conclusion](https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExNGlvdjZiZjV6YjJ4dnllZzl0eHA5Z3hwcWJla283Mjc2bnBrZTZ0biZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/nwVUMvocGxuMTuRwkg/giphy.gif)

Godot 4 is a clear jump forward from Godot 3. The biggest wins are Vulkan rendering (speed, modern effects, stable frame times), optional Direct3D 12 on Windows (tools and future Windows tech), stronger physics (Jolt), and many small workflow upgrades that save time every day.

If you come from Godot 3, plan a short migration phase: fix shaders, re-check lighting (AgX tone mapping), review physics names, and profile again instead of trusting old bottlenecks. Add a switch for Vulkan / D3D12 if you target Windows users. Start simple, then turn on advanced features (SDFGI, volumetrics, clustered lights) as performance allows.

Use the editor improvements (multi‑window, pinned nodes, better profiler, typed GDScript) to iterate faster. Only drop into native code (GDExtension) when profiling shows a true hotspot.

In short: Godot 4 gives you modern graphics, better tools, and a smoother daily workflow—without licensing lock‑in. Learn the new rendering basics, test early on target hardware, and you can build sharper, faster games that age well.