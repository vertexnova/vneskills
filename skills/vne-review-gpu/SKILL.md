---
name: vne-review-gpu
description: >
  Review checklist for host-side GPU code across Vulkan, Metal, and WebGPU:
  pipeline keys, attachments, bindings, uniform rings, hazards, sync, WebGPU
  limits. Use when reviewing render passes, rhi backends, pipelines,
  descriptors, swapchains, or readback code.
---

# Review: GPU host code

Metal accepting something proves nothing; WebGPU is the strictest backend. Flag:

## Known "Metal tolerated, WebGPU rejected" bugs

- Hardcoded depth format (`Depth32Float`) instead of the target's format
  (WebGPU uses D24S8 there).
- Bind group or pipeline not rebuilt after MSAA was toggled.
- Depth attachment sample count different from the color attachments.
- Binding indices past the Metal buffer limit.
- Blending enabled on an integer target.

## Pipelines and bindings

- Pipeline or cache key missing a color format, depth format, or sample count.
- Dynamic offsets that do not match the layout's declared dynamic bindings;
  `frame::markDynamicBindings` must run before `initialize()`.
- Uniform ring offsets reset or shared between passes (opaque and transparent
  passes once overwrote each other's data).

## Hazards and sync

- Texture sampled and written in the same pass; use a snapshot copy.
- Fence values or cached GPU state written from two threads without atomics.
- Resource destroyed while an in-flight frame can still use it.
- Swapchain recreated while the GPU is busy.
- `waitIdle`, blocking map, or `emscripten_sleep` in a per-frame path.

## WebGPU host limits

- Copy row pitch not a multiple of 256 bytes.
- More than 8 storage bindings per stage; push constants.
- Assuming `generateMipmaps` (absent: mips silently stay at level 0).
- Synchronous buffer mapping; readback must be async.

## Conventions

- Backend to math API mapping not via `graphicsApiFromBackend` (it was once
  copied into three files).
- vnemath projection helpers called without the graphics API (the default is
  `GraphicsApi::eOpenGL`, depth -1..1).

## Evidence

- Backends actually run are listed; Metal-only verification of a rendering
  change is MAJOR. See [vne-backend-matrix](../vne-backend-matrix/SKILL.md).

Error-handling rules for vnerhi backends live in its local `vne-error-handling`.

Sources: Vulkan, Metal, and WebGPU specs and limits; WebGPU validation rules.
