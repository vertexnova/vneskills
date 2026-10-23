---
name: vne-build-verify
description: >
  Standard format, build, run, and test pipeline for VertexNova libraries. Use
  before finishing a C++, Objective-C++, shader, or packaging change. Detect the
  git toplevel name and follow the matching row in repos.md.
---

# Build and Verify

Run this pipeline before completing any change to source or shaders. Prefer
`./scripts/build_*.sh` (or the Windows scripts) over ad-hoc IDE binaries.

Detect the repo:

```bash
basename "$(git rev-parse --show-toplevel)"
```

Then open [repos.md](repos.md) and run that repo's commands. If the repo is not
listed, use the fallback at the bottom of `repos.md`.

## Shared steps (every repo)

### 1. Format

```bash
python3 scripts/clang_formatter.py all
python3 scripts/clang_formatter.py all --dry-run
```

CI enforces clang-format-17. `.clang-format` sets `SortIncludes: false`, so
include order is hand-maintained; see [vne-header-hygiene](../vne-header-hygiene/SKILL.md).

### 2. ASCII

```bash
rg -nP "[^\x00-\x7F]" src include samples tests scripts skills docs
```

A clean run prints nothing (fixtures and `deps/` may be exceptions; see
[plain-ascii-authoring](../plain-ascii-authoring/SKILL.md)).

### 3. Build, test, picture

Follow [repos.md](repos.md). Then, for rendering or XR changes, run
[vne-picture-check](../vne-picture-check/SKILL.md). Compiling alone is not done.

### 4. Shaders

If GLSL, manifests, `gpu_layout.yaml`, or `*_gpu.h` changed, open the local
`skills/vne-shader-pipeline/SKILL.md` when present (vnerhi, vnegfx).

## Done when

- Format dry-run is clean.
- Tests for the touched area pass (`ctest` or the repo script `-a test`).
- Picture or data check from [vne-picture-check](../vne-picture-check/SKILL.md)
  has evidence (test output, golden match, or a named sample run).
