---
name: vne-gpu-benchmark
description: >
  Vet a GPU speedup or regression before reporting it. Same scene, camera, and
  resolution; warmup; repeat; name the limiter (submit, bandwidth, overdraw, or
  the pass). Use for volume isosurface, splat sort, OIT, or any frame-time claim.
---

# GPU Benchmark Checklist

Do not trust, report, or act on a GPU timing number until this checklist passes.

## 1. Same work

- Same scene, camera, and resolution.
- Same backend and validation settings (validation off for timing runs).
- Same power / thermal state when comparing device runs (desktop vs headset).

## 2. Method

- Warmup frames before measuring.
- Repeat enough times to see variance; report median or mean with a note on
  spread.
- Prefer end-to-end frame time the user feels, plus a pass timer when isolating
  a hypothesis.

## 3. Name the limiter

Say which one the number is measuring:

| Limiter | Symptom |
|---------|---------|
| Submit / CPU | GPU idle while CPU queues work |
| Bandwidth | Large buffers or textures; bandwidth-bound copies |
| Overdraw | Fill-rate heavy; order-independent transparency, volumes |
| The pass itself | Compute or raster cost inside one named pass (isosurface, splat sort) |

If you cannot name the limiter, the number is not ready to guide a change.

## 4. Examples

- Volume isosurface: fix camera, volume, and iso value; compare before/after on
  one backend.
- Splat sort (vne3dgs when GPU lands): same splat count and view; separate sort
  time from raster time.

## 5. Report

Include: scene id, resolution, backend, warmup count, repeat count, before/after
(or absolute), and the limiter. Reject claims that skip any of these.
