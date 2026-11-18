---
name: vne-review-cmake
description: >
  Review checklist for CMake: target-based usage, scopes, subproject safety,
  install/export, dependency pinning, platform specifics, vnecmake helpers. Use
  when reviewing CMakeLists.txt, *.cmake, or *.cmake.in changes.
---

# Review: CMake

Build breakage is the largest fix category in several repos. Flag:

## Targets, not directories

- `include_directories`, `link_libraries`, `add_definitions`, `add_compile_options`
  at directory scope, or edits to `CMAKE_CXX_FLAGS`.
- Wrong scope: a dependency used only in `.cpp` marked `PUBLIC`, or one exposed in
  public headers marked `PRIVATE`.
- New target without `target_compile_features(... cxx_std_20)` or an alias
  `vne::<name>`.

## Works as a subproject

- `CMAKE_SOURCE_DIR` / `CMAKE_BINARY_DIR` where `PROJECT_SOURCE_DIR` or
  `CMAKE_CURRENT_*` is meant.
- Option without the `VNE_<LIB>_` prefix.
- Tests, samples, or install rules on by default when not
  `PROJECT_IS_TOP_LEVEL`.
- Sibling dependency added without `vne_use_dep` (duplicate target errors).

## Install and export

- Include dirs without `$<BUILD_INTERFACE:...>` / `$<INSTALL_INTERFACE:...>`.
- Absolute paths in exported targets.
- `install(EXPORT)` without the namespace, or a PUBLIC dependency missing a
  `find_dependency` in `*Config.cmake.in`.

## Dependencies

- Submodule pin that does not exist on the remote.
- `FetchContent` or `ExternalProject` not pinned to a tag or hash.
- Network access at configure time in the default build.

## Platform

- `if(CMAKE_BUILD_TYPE ...)` instead of generator expressions (breaks
  multi-config generators: Visual Studio, Xcode).
- OBJECT library without position-independent code under Emscripten pthreads.
- Shared Windows build without export macros for new public symbols.

## Hygiene

- Hand-rolled warnings, sanitizers, or coverage instead of the vnecmake helpers
  (`set_project_warnings`, `vne_configure_debug_target`).
- `file(GLOB)` for sources without `CONFIGURE_DEPENDS`.
- Unquoted variable expansions in conditions or paths.
- `message(FATAL_ERROR)` that does not say how to fix it.
- `CMAKE_CXX_CLANG_TIDY` set with `-checks=*`, which overrides the repo
  `.clang-tidy`.

Sources: CMake buildsystem docs; Craig Scott, Professional CMake; Effective
Modern CMake.
