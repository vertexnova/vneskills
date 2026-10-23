---
name: vne-testing
description: >
  Conventions for writing and organizing GoogleTest tests in VertexNova
  libraries, including backend-gated and null-backend tests. Use this skill when
  adding or changing tests, or when a behavior change needs test coverage.
---

# Testing Conventions

VertexNova libraries use GoogleTest. A feature or bug fix ships with a test
unless the behavior can only be observed on a GPU. Tests must be deterministic
and fast.

## 1. Layout

- `tests/` - the main suite, one file per unit, built into a single
  `<repo>_tests` binary (for example `vnerhi_tests`, `vnexr_tests`).
- `tests/backend/{vulkan,metal,webgpu,null,...}/` - tests that need a specific
  backend, when the repo has that split.
- `tests/fixtures/` - shared helpers and fixtures.

## 2. File and case naming

- One test file per unit: `<subject>_test.cpp`, matching the header under test
  (`vertex_layout.h` -> `vertex_layout_test.cpp`).
- Follow project naming in test code too (PascalCase types, camelCase methods,
  snake_case locals); see [vne-coding-style](../vne-coding-style/SKILL.md).
- Register new files in the repo's `tests/CMakeLists.txt`.

## 3. Structure

- Arrange, Act, Assert, with a blank line between phases.
- One behavior per `TEST` / `TEST_F`; give the case a name that states the behavior.
- Keep mocks minimal and assert on observable behavior, not implementation details.

## 4. Determinism and hardware

- No `sleep`, no timeouts, no wall-clock or ordering assumptions.
- Fix random seeds; avoid data races in threaded tests.
- Prefer the null backend for anything that does not need a real device: it
  exercises descriptor validation and resource lifetimes without a GPU.
- A test that needs a real backend belongs under `tests/backend/<backend>/` (or
  the repo's equivalent) and must skip cleanly (`GTEST_SKIP()`) when the device
  or extension is unavailable, never fail.

## 5. What is worth a test

Descriptor and reflection logic is testable without a GPU and is where most silent
breakage lives: vertex layout strides and offsets, shader resource set validation,
dynamic-offset bookkeeping, format and capability mapping, swapchain descriptor
contracts, pose/projection math, session state machines. Prefer a null-backend
test over a manual visual check when the behavior can be observed through a
descriptor, reflection result, or matrix.

The Null backend proves descriptors and lifetimes. It does not prove the image.
Rendering output needs [vne-picture-check](../vne-picture-check/SKILL.md): image
goldens when the repo has them, otherwise a named sample on one real backend.

## 6. Running

```bash
ctest --test-dir <build-dir> --output-on-failure
ctest --test-dir <build-dir> -R <repo>_tests
<build-dir>/bin/<repo>_tests --gtest_filter=Suite.TestName
```

Build the test target with the repo's `--dev` / `VNE_*_DEV` / `--with-tests`
flag; see [vne-build-verify](../vne-build-verify/SKILL.md) and
[repos.md](../vne-build-verify/repos.md).

When the bug is CPU-observable, write the failing GoogleTest first (pstack
`/tdd`), then the fix.
