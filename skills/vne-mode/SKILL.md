---
name: vne-mode
description: >
  Default entry for VertexNova 3D visualization work. Detects the current repo
  and routes to frame, picture-check, backend-matrix, cross-repo, build-verify,
  and local domain skills. Use at the start of pass, shader, GPU, XR, or sample
  tasks. For investigation, refactor, overnight landing, or PR babysit, use
  pstack /poteto-mode instead.
---

# vne-mode

Entry playbook for the VertexNova visualization stack. Install pstack alongside
this plugin. Do not reimplement poteto-mode playbooks here.

## 0. Detect the repo

```bash
basename "$(git rev-parse --show-toplevel)"
```

Known names: `vnerhi`, `vnegfx`, `vnexr`, `vnewindow`, `vne3dgs`, `vneshaderc`,
`vnevoice`, and other `vne*` libraries under the sibling checkout tree.

## 1. Match the task

| Task | Route |
|------|-------|
| Picture or pass (shader, material, volume, mesh, OIT, picking, CSG, splat) | [vne-frame](../vne-frame/SKILL.md), then local `skills/vne-shader-pipeline/SKILL.md` if present, then [vne-picture-check](../vne-picture-check/SKILL.md) |
| GPU resource or swapchain | [vne-frame](../vne-frame/SKILL.md) + [vne-backend-matrix](../vne-backend-matrix/SKILL.md) |
| Public API or C API | [vne-cross-repo](../vne-cross-repo/SKILL.md), then local `vne-c-api` / `vne-csharp-*` if present |
| XR session or Android APK | local `skills/vne-xr-session/SKILL.md` and `skills/vne-xr-android/SKILL.md` |
| Window / native handle | local `skills/vne-xwin-platforms/SKILL.md` |
| Bug with CPU-observable result | pstack `/tdd` (failing GoogleTest first), then fix |
| Perf number | [vne-gpu-benchmark](../vne-gpu-benchmark/SKILL.md) |
| Review a PR or diff (or self-review before opening one) | [vne-review](../vne-review/SKILL.md), which loads the matching `vne-review-*` lenses |
| How / why / teach / overnight / babysit | pstack `/how`, `/why`, `/teach`, `/interrogate`, `/poteto-mode` |

Every source edit also applies:

1. [plain-ascii-authoring](../plain-ascii-authoring/SKILL.md)
2. [vne-coding-style](../vne-coding-style/SKILL.md)
3. [vne-header-hygiene](../vne-header-hygiene/SKILL.md) when headers or includes change
4. [vne-testing](../vne-testing/SKILL.md) when tests change
5. [vne-build-verify](../vne-build-verify/SKILL.md) before finishing

## 2. Session ownership (short)

Host owns the device and swapchains. Samples and layers draw into textures the
host provides; they do not create a second graphics device for compositor images.

**XR (vnexr):**

```
pollEvents -> beginFrame (Frame + SurfaceTextures)
  -> sample / layer onUpdate
  -> endFrame (composition layers)
```

**gfx (vnegfx):** `RenderSession` / pass pipeline owns the frame context; passes
read named GPU structs and write named targets. Name those before editing.

## 3. Done

A visualization change is finished when [vne-picture-check](../vne-picture-check/SKILL.md)
has evidence. Null backend proves descriptors and lifetimes, not the image.
Compiling is not that check.
