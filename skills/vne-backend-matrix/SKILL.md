---
name: vne-backend-matrix
description: >
  List which GPU backends a shader, binding, or resource change must hit
  (Vulkan, Metal, WebGPU, Null). Use when editing pipelines, descriptors,
  swapchains, or cross-compiled shaders.
---

# Backend Matrix

vnerhi backends: **Vulkan**, **Metal**, **WebGPU**, **Null**.

Null is a lifetime and descriptor check. It is not a picture check. A shader or
binding change that only runs on Null is unfinished for visual work.

## When to open this skill

- Editing GLSL / MSL / WGSL, manifests, or `RenderPipelineDescriptor`.
- Changing resource set layouts, dynamic offsets, or format mapping.
- Touching swapchain or surface creation paths.

## Procedure

1. List the change (shader, binding, format, sync).
2. Mark required backends for this platform:

| Backend | Typical platforms | Role |
|---------|-------------------|------|
| Null | all CI hosts | Descriptors, lifetimes; skip image proof |
| Vulkan | Linux, Windows, Android, some macOS via MoltenVK if enabled | Primary desktop / XR GPU |
| Metal | macOS, iOS, visionOS | Apple GPU path |
| WebGPU | WASM / browser, Dawn native where enabled | Web and Dawn |

3. Run at least one real backend for picture proof
   ([vne-picture-check](../vne-picture-check/SKILL.md)).
4. If the change is cross-compiled (SPIR-V + MSL + WGSL), regenerate bundles via
   the local `skills/vne-shader-pipeline/SKILL.md` and spot-check one backend per
   shader language when practical.

## XR note

vnexr OpenXR paths are Vulkan-first on Android / Linux / Windows. visionOS uses
Metal. Null session covers CI without a runtime. Do not treat a Null session
pass as a headset picture pass.
