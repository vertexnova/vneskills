---
name: vne-review
description: >
  Entry point for reviewing a VertexNova PR or diff: severity rubric, comment
  format, what to skip, required evidence, and which vne-review-* lens to load
  for each path. Use when reviewing, self-reviewing, or writing review-tool rules.
---

# vne-review

Approve a change that improves code health. Block only on a concrete failure.

## Severity

- **BLOCKER**: wrong image or measurement, crash, UB, leak, race, exception
  across a library boundary, unvalidated untrusted input, broken public API.
- **MAJOR**: behavior change without a test, rendering verified on Metal only,
  frame-path perf regression, ignored error or silent fallback, generated code
  out of sync.
- **MINOR**: clarity, missing doc on a non-obvious API, small duplication.

## Comment format

`file:line` + failing input or state + what happens + concrete fix (or the rule
broken). No failure scenario and no written rule -> no comment.

## Skip

Formatting (clang-format-17 in CI), naming (clang-tidy in CI), generated files
(`**/gen/**`, `*.vneshader/`, `*.g.cs`, `*.g.c`), `deps/`, `external/`,
`third_party/`, binary test data, and unbacked preferences.

## Lens by path

| Paths | Load |
|-------|------|
| `*.h *.hpp *.cpp *.cc *.mm` | [vne-review-cpp](../vne-review-cpp/SKILL.md) |
| loaders, parsers, `**/io/**`, `*_file.cpp`, C API entry points | [vne-review-input](../vne-review-input/SKILL.md) |
| volume, slice, picking, fiber, x-ray, DICOM, measurement | [vne-review-medical](../vne-review-medical/SKILL.md) |
| `*.glsl *.wgsl *.metal *.manifest.json *_gpu.h *_bindings.h` | [vne-review-shaders](../vne-review-shaders/SKILL.md) |
| render passes, `rhi/**` backends, pipelines, swapchains | [vne-review-gpu](../vne-review-gpu/SKILL.md) |
| `CMakeLists.txt *.cmake *.cmake.in` | [vne-review-cmake](../vne-review-cmake/SKILL.md) |
| `*.sh *.ps1 *.bat *.py`, `.github/workflows/**` | [vne-review-scripts-ci](../vne-review-scripts-ci/SKILL.md) |
| `bindings/**`, `*.swift *.cs *.kt *.kts`, JNI `*.c` | [vne-review-bindings](../vne-review-bindings/SKILL.md) |

Local repo skills add to these: `vne-c-api`, `vne-shader-pipeline` (vnegfx),
`vne-error-handling` (vnerhi), `vne-xwin-platforms`, `vne-xr-session`.

## Evidence (missing = MAJOR)

- Behavior change: a test that fails without it ([vne-testing](../vne-testing/SKILL.md)).
- Rendering: golden or named sample on a named backend ([vne-picture-check](../vne-picture-check/SKILL.md)).
- Shader, binding, format, sync: backends run ([vne-backend-matrix](../vne-backend-matrix/SKILL.md)).
- Public or C API: one consumer built ([vne-cross-repo](../vne-cross-repo/SKILL.md)).
- Perf claim: [vne-gpu-benchmark](../vne-gpu-benchmark/SKILL.md).

## PR hygiene

- Conventional Commit title (release-please builds the changelog from it).
- One purpose; no drive-by refactors or reformatting; `CHANGELOG.md` not hand-edited.
- Generated files regenerated with their inputs, never edited.
- Sample options and UI knobs are never removed by a refactor.
- ASCII only ([plain-ascii-authoring](../plain-ascii-authoring/SKILL.md)).

Sources: Google eng-practices code review guide; C++ Core Guidelines.
