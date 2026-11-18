---
name: vne-review-input
description: >
  Review checklist for code that parses bytes it did not produce: volume, image,
  mesh, fiber, font, video, shader-bundle loaders, C API descriptors from FFI
  callers, and the C#, Swift, Kotlin loader ports. Use when reviewing any loader,
  reader, parser, or public entry point.
---

# Review: untrusted input

Every file, network blob, and FFI argument is hostile until validated. Flag:

## Bounds and sizes

- Reads without a remaining-length check; use a bounded reader.
- No named caps (`kMax*`) on dimensions, counts, header bytes, total bytes.
- Size math not overflow-checked before allocation (vneio `image.cpp` multiplies
  `width * height * channels` in `int`).
- Declared sizes not compared exactly to the payload (truncated or trailing data).
- Clamping a bad value instead of rejecting it.

## Paths

- Path from file content (MHD `ElementDataFile`, NRRD data file) used without
  rejecting `..`, absolute paths, and symlink escape from the base directory.

## Semantics

- Metadata not validated: dims > 0, spacing finite and > 0, direction rows unit
  length, |det| above epsilon. `Volume::isMetadataValid()` exists; call it.
- Endianness handled for 2-byte types only; every element width needs it.
- Floats (points, spacing, origin) not checked for NaN and Inf.
- Indices (mesh faces, bundle offsets) not checked against their array size.

## Failure shape

- Partial object returned on error; return an error with the byte offset.
- Exception or `bad_alloc` escaping a no-throw loader.

## Third-party decoders

- stb, assimp, NrrdIO, FFmpeg: enable only needed formats; assimp needs
  `aiProcess_ValidateDataStructure` and index checks against `mNumVertices`.

## Parity and tests

- The same caps and checks in the C++, C#, Swift, and Kotlin ports (C# `checked`
  casts, Kotlin explicit archive limits, Swift `isFinite`).
- Negative tests exist: truncated, oversize dims, overflowing dims, bad magic,
  NaN, path traversal.

Reference implementation: `vnegfx/samples/framework/renderer/fiber/fiber_file.cpp`
(magic, version, counts checked against file size without overflow).

Sources: Chromium Rule of 2; CWE-190, CWE-787, CWE-22, CWE-400; OSS-Fuzz.
