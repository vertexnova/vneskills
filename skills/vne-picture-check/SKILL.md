---
name: vne-picture-check
description: >
  Prove a visualization change: descriptor or matrix test when the bug is data;
  image goldens when present; otherwise run a named sample on one real backend.
  Use before declaring a rendering, shader, XR, or sample change done.
---

# Picture Check

Compiling is not done. Null backend proves descriptors and lifetimes. It does
not prove the image.

## Pick the cheapest proof that fits

### A. Data bug (CPU-observable)

Vertex strides, binding layouts, reflection, poses, projections, session state.

1. Prefer a failing GoogleTest first (pstack `/tdd`).
2. Prefer the null backend when no real device is required.
3. Run via [vne-build-verify](../vne-build-verify/SKILL.md).

### B. Image goldens (vnegfx)

When `testdata/vneresources` is initialized and the suite has image comparison:

1. Run the matching image tests (`ctest` filter for the pass or technique).
2. Inspect failures; do not update goldens without confirming the new image is
   correct on a real backend.

### C. Named sample on a real backend

When there is no golden for the path:

1. Build with samples/examples enabled (see [repos.md](../vne-build-verify/repos.md)).
2. Run one named sample on one real backend (`vulkan`, `metal`, or `webgpu`).
3. Say which sample and backend you ran. A blank or wrong image fails the check.

Examples:

```bash
# vnerhi
<build-dir>/bin/28_stencil_testing --backend vulkan --validation

# vnexr desktop (Monado)
./build/shared/Debug/build-linux-gcc-*/bin/samples/sample_02_openxr_stereo

# vnexr null (no runtime)
./build/.../bin/samples/sample_01_null_simulator
```

### D. XR device path

1. Null session / null sample first.
2. Then Monado on Ubuntu, or Galaxy XR APK per local `skills/vne-xr-android/SKILL.md`.
3. Logcat / runtime logs for `vne.xr` and `OpenXR` when the headset path fails.

## Report

State which path (A/B/C/D), the command, and the result. If you only compiled,
the change is not finished.
