---
name: vne-review-medical
description: >
  Review checklist for medical-imaging correctness: patient frames (LPS/RAS),
  spacing and mm units, Hounsfield intensity, window/level, measurement
  precision, clinical color fidelity. Use when reviewing volume, slice/MPR,
  picking, measurement, DRR, fiber, x-ray, or DICOM code.
---

# Review: medical imaging correctness

A plausible-looking image can still be mirrored, mis-scaled, or mis-windowed.
Flag:

## Frames and orientation

- A position whose frame is not stated: voxel index, normalized box, world, or
  patient mm.
- Implicit LPS (DICOM, ITK) vs RAS (NIfTI, Slicer) mixing; the x and y sign flip
  must be explicit and in one place.
- Direction matrix row-major vs column-major converted at more than one point.
- Orientation "fixed" by rolling a sample camera instead of fixing the engine
  (the y-flip that mirrored every volume was hidden this way).
- Changes to orientation without `VolumeIsOrientedLikeAMesh` or an equivalent
  test still passing.

## Spacing and units

- Spacing not finite and > 0; negative spacing silently mirrors the box.
- Origin narrowed from `double` to `float`.
- Distances reported from the normalized volume box without `world_to_mm`.
- Slice position or measurement API in normalized units without a mm accessor.

## Intensity

- Raw values discarded: keep raw plus rescale slope and intercept
  (HU = raw * slope + intercept).
- 16-bit data auto-windowed to 8-bit without saying so and keeping the mapping.
- Signed 16-bit treated as unsigned.
- Window width allowed to reach 0 (DICOM VOI LUT needs width >= 1).
- Non-linear default LUT used for MIP, AIP, or DRR.

## Measurement-grade results

- CPU math in `float`; use `double`.
- Pick ray not built from the inverse view-projection.
- Unrefined results not flagged (`VNE_GFX_PICK_FLAG_APPROXIMATE`).
- Units missing from the result type or docs.
- Tests without closed-form expected values.

## Rendering fidelity

- Clinical colors (authored or direction-encoded) lit, tinted, or fogged.
- Image depends on step count (missing opacity correction per mm).
- DRR attenuation not integrated in mm.
- Silent fallback for an unsupported format or missing mips.

## Parity

- C#, Swift, Kotlin orientation matrices and loaders diverge from the C++ code.

Sources: DICOM PS3.3 C.7.6.2 (Image Plane), C.11.2 (VOI LUT), Modality LUT
rescale; NIfTI-1 qform/sform; ITK and 3D Slicer coordinate conventions.
