---
name: vne-cross-repo
description: >
  Map blast radius across VertexNova repos when changing a public API, C API,
  native window handle, or shader bundle. Use before editing interfaces that
  other libraries or bindings consume.
---

# Cross-Repo Blast Radius

A small-looking change in one library can break siblings. Prove what else is
affected by naming the consumers and checking at least one of them.

## Fan-out map

| Change in | Likely consumers |
|-----------|------------------|
| vnerhi public `I*` API / types | vnegfx, vnexr, vne3dgs, samples that create devices or pipelines |
| vnegfx C API (`types.h`, `gfx.h`) | `bindings/{csharp,swift,kotlin}`, `samples_{csharp,swift,kotlin}`; local `vne-c-api` / `vne-csharp-*` |
| vnewindow native handles / surface | vnerhi surface / swapchain creation |
| vneshaderc bundle or reflection | vnerhi samples and vnegfx shader pipeline |
| vnexr `ISession` / `Frame` / surfaces | samples and any app drawing into XR views |
| vnecommon / vnelogging / vnemath | most libraries via `deps/internal/` |

Sibling checkouts usually live next to each other under `../vne/`. Prefer a
local path override (`VNE_VNERHI_PATH=../vnerhi`) when testing a stack of
changes.

## Procedure

1. List the public symbols or files you are changing.
2. Search consumers (`rg` across sibling repos or the current repo's samples /
   bindings). Prefer evidence from code over memory.
3. If the change is in vnegfx C API, follow local `skills/vne-c-api/SKILL.md` and
   regenerate every binding (C#, Swift, Kotlin/JNI) in the same change; see
   [vne-review-bindings](../vne-review-bindings/SKILL.md).
4. Build and test the producer, then at least one consumer path (sample or
   binding test).
5. Document the one fact that makes the change safe (for example: "only Null
   backend callers use this overload" with a search that proves it).

## Do not claim safety from compile alone

A green producer build does not mean vnegfx or vnexr still link. Run a consumer
when the public surface moved.
