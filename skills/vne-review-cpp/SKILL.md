---
name: vne-review-cpp
description: >
  Review checklist for production C++ and Objective-C++ correctness: lifetime,
  errors, integer safety, UB, concurrency, GPU resource lifetime, API, hot paths.
  Use when reviewing *.h, *.cpp, *.mm changes. Style is out of scope.
---

# Review: C++ production

Style and naming belong to CI and [vne-coding-style](../vne-coding-style/SKILL.md).
Flag these.

## Lifetime and ownership

- Raw `new`/`delete` or manual resource release; use RAII and `unique_ptr`.
- `string_view` / `span` / reference that can outlive its owner (returned from a
  temporary, stored in a member, captured by a deferred lambda).
- Async, GPU, or platform callbacks capturing `this` or references; capture a
  handle or weak owner. (WebGPU callbacks fire after the object can be gone.)
- Use after `std::move` (clang-tidy `bugprone-use-after-move` is off here).
- Two owners that both destroy the same GPU object (past double-destroy in
  MeshDeferredRenderPass).

## Errors

- Exception that can leave a library API; that includes `bad_alloc` from an
  unbounded `resize` driven by input.
- `[[nodiscard]]` result ignored or discarded with `(void)`; log or propagate.
- Silent fallback: an empty texture, default object, or `true` returned for an
  unsupported case. Return an error. (`uploadVolume` once returned an empty
  texture for non-R8 data.)
- Empty `catch` without an `@expected` comment.

## Integer and float safety

- Products computed in `int` then widened: `static_cast<size_t>(w * h * c)`.
  Cast each operand first, check overflow before allocating.
- `size_t` is 32-bit on wasm; 64-bit products narrow silently.
- Unchecked `size_t` -> `int`/`uint32_t`, signed/unsigned compares, `atoi`.
- Float inputs from outside not checked with `std::isfinite`.

## Undefined behavior

- Uninitialized members (use in-class defaults), unaligned or type-punned reads
  (use `memcpy`, see `Volume::readVoxelAt`), shifts >= width, iterator
  invalidation, range-for over a member of a temporary.

## Concurrency

- Shared state touched off the owning thread without atomics or a lock; fence
  values must be atomic.
- New threading without `@threadsafe` docs; the stack is single-thread first.
- AppKit / UIKit calls off the main thread.

## GPU resource lifetime

- Buffer, texture, or pipeline destroyed while a frame in flight may use it.
- Swapchain recreated while the GPU is busy (past resize race).
- `waitIdle` / blocking map in a per-frame path.

## API and hot paths

- Missing `explicit`; `noexcept` that can throw; descriptor field not appended at
  the end; `friend` coupling instead of a narrow interface.
- Allocation, `std::string` formatting, or logging inside per-frame or per-voxel
  loops.
- Logs that print patient identifiers or DICOM tag values.

Sources: C++ Core Guidelines (R, ES, CP, I); CERT C++ INT30-C, INT31-C,
EXP54-CPP, ERR55-CPP; Chromium base/numerics.
