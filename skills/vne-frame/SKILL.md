---
name: vne-frame
description: >
  Name the frame inputs (camera, lights, per-pass GPU struct, bindings, target
  texture) before editing a pass, shader, material, or swapchain path. Use when
  changing rendering, volume, mesh, OIT, picking, CSG, splat, or XR view draws.
---

# Frame Inputs First

Before writing code that crosses a pass or shader boundary, write down the data
shape for one frame. Downstream edits become obvious once these names are fixed.

## Checklist (fill before editing)

1. **Camera / views** - view and projection matrices; XR: per-eye `View` from the
   session frame; desktop: window camera.
2. **Lights / scene inputs** - what the pass reads from ECS, uniforms, or buffers.
3. **Per-pass GPU struct** - the `*Gpu` layout or RHI push/uniform block; in
   vnegfx this is `gpu_layout.yaml` / `*_gpu.h`.
4. **Bindings** - descriptor sets / resource sets / binding slots; must match
   the shader reflection and the pipeline descriptor.
5. **Target texture** - color (and depth) the pass writes; XR: swapchain wrap
   from the host session, not a sample-created device.
6. **Who owns the device** - host session / `RenderSession` / sample app. Samples
   call into that device; they do not invent a second one for compositor images.

## How to settle unknowns

- Read the existing pass and its callers. Prefer one concrete sample that already
  exercises the path.
- In vnegfx, open local `skills/vne-shader-pipeline/SKILL.md` for schema sync.
- In vnerhi, open local `skills/vne-shader-pipeline/SKILL.md` for manifests and
  `RenderPipelineDescriptor`.
- In vnexr, open local `skills/vne-xr-session/SKILL.md` for host-owned swapchains.

## After the names are fixed

Edit the smallest set of files that implement that data shape. Then prove the
picture with [vne-picture-check](../vne-picture-check/SKILL.md). If the change
touches bindings or shaders, list backends with
[vne-backend-matrix](../vne-backend-matrix/SKILL.md).
