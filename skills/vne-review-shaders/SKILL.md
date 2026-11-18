---
name: vne-review-shaders
description: >
  Review checklist for shader source (GLSL compiled by vneshaderc to SPIR-V, MSL,
  WGSL, plus hand-written .metal / .wgsl): uniform control flow, numerical
  safety, loops, coordinates, blending, data layout, WGSL/MSL traps. Use when
  reviewing *.glsl, *.wgsl, *.metal, *.manifest.json, *_gpu.h, or *_bindings.h.
---

# Review: shaders

GLSL here must compile to three languages; WGSL is the strictest. Flag:

## Structure and layout

- Struct fields typed twice: the C++ `*_gpu.h` struct is the single definition,
  GLSL wraps it with `VNE_GLSL_UBO(...)`; each slot is owned by one
  `gpu/<family>/*_bindings.h` header.
- Hand-written `layout(set=, binding=)` where the macros apply.
- `*_gpu.h` struct without `VNE_VALIDATE_GPU_STRUCT_SIZE`.
- `vec3` in a uniform array, `bool` in a uniform block (use `uint`), std140
  array stride (16) or std430 padding not matched on the C++ side.
- GLSL or a struct changed but `gen/` bundles not regenerated in the same PR.
- No header comment naming the algorithm, units, and space of inputs/outputs.
- Copied helper that already exists in `shaders/common/`.

## Uniform control flow (WGSL rejects violations)

- Implicit-LOD `texture()`, `dFdx`/`dFdy`, `fwidth` inside per-pixel loops, after
  a per-pixel branch, or after `discard`; use `textureLod` or `texelFetch`.
- Compute `barrier()` in non-uniform control flow.
- Correctness that relies on early fragment tests.

## Numerical safety

- Unguarded division. Live case: `windowLevel()` in
  `vnegfx/src/vertexnova/gfx/shaders/common/volume_transfer.glsl` divides by the
  window width.
- `normalize` of a vector that can be zero; `pow` with a negative base; `sqrt`,
  `log`, `acos`, `asin` without clamped inputs; `clamp` with lo > hi.
- Unnamed epsilons, or epsilons in window depth instead of world or linear units.
- NaN able to reach a blended target.

## Loops and cost

- Loop without a compile-time cap constant (`kStepCap`) next to its dynamic bound.
- Arrays passed to functions: MSL and WGSL copy them by value (a 64-entry CSG
  window spilled registers and collapsed performance).
- Large or dynamically indexed local arrays.
- Missing early exit on saturated opacity (except integrals such as DRR).

## Coordinates

- A y flip in a shader; raster row 0 is NDC +1 on every backend.
- Depth outside [0,1]; per-vertex normalized ray directions (normalize per
  fragment); normals not using the inverse-transpose; winding that disagrees with
  the pipeline.

## Color and blending

- Premultiplied output with `SrcAlpha` blending (needs `One/OneMinusSrcAlpha`);
  each pass states which it writes.
- sRGB encode or tone mapping applied twice; alpha written outside [0,1].
- Float writes or blending on integer targets.
- Clinical colors passed through lighting, tint, or fog.

## Portability traps

- WGSL or MSL reserved words as identifiers (`meta` broke WGSL before).
- `defines` on the unnamed variant (vneshaderc ignores them).
- Filtering an `r32float` texture (not filterable on WebGPU by default).
- 3-component texture formats, point size > 1, geometry or tessellation stages.

## Evidence

- CPU mirror test for shader math where possible (closed-form Beer-Lambert in
  `vnegfx/tests/renderer/volume_test.cpp`), image goldens, and a WebGPU run.

Process (how to regenerate) lives in the local `vne-shader-pipeline` skill.

Sources: WGSL spec (uniformity analysis); GLSL std140/std430 rules; Vulkan,
Metal, and WebGPU limits.
