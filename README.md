# vneskills

Shared Cursor / Claude Code skills for the [VertexNova](https://github.com/vertexnova)
3D visualization stack: window, GPU (vnerhi), shaders (vneshaderc), scene (vnegfx),
XR (vnexr), and Gaussian splatting (vne3dgs).

A visualization change is finished when the picture is right, or when a CPU check
proves the data that feeds the picture. Compiling is not that check.

## Install

Install both plugins. pstack owns generic engineering rigor; this plugin owns
VertexNova visualization playbooks.

```bash
/add-plugin pstack
/add-plugin vneskills
```

Or clone next to the other VertexNova checkouts and point Cursor at this directory:

```bash
git clone https://github.com/vertexnova/vneskills.git ../vneskills
```

### As a submodule (preferred in VertexNova libraries)

In each library, delete the old shared skill copies and add this repo under
`skills/vneskills`:

```bash
git submodule add https://github.com/vertexnova/vneskills.git skills/vneskills
git submodule update --init skills/vneskills
```

Keep only domain skills next to the submodule (for example `vne-xr-session` in
vnexr). Symlink for Claude Code:

```bash
mkdir -p .claude/skills
for s in skills/vneskills/skills/*/; do
  ln -sfn "../../../$s" ".claude/skills/$(basename "$s")"
done
for s in skills/*/; do
  name=$(basename "$s")
  [ "$name" = "vneskills" ] && continue
  ln -sfn "../../$s" ".claude/skills/$name"
done
```

Claude Code: you can also install as a marketplace plugin when published.

## Get started

Type `/vne-mode` at the start of visualization work. It detects the current repo
(`vnerhi`, `vnegfx`, `vnexr`, `vnewindow`, `vne3dgs`, ...) and routes to the
matching playbooks.

For investigation, refactor, overnight landing, or PR babysit, use pstack's
`/poteto-mode`. Do not reimplement those playbooks here.

## Skills

Folders stay flat (`skills/<name>/SKILL.md`) so plugin loaders and review tools
find them; the groups below come from the name prefix.

### Entry points

| Skill | Use when |
|-------|----------|
| [vne-mode](skills/vne-mode/SKILL.md) | Starting visualization work (routes to the rest) |
| [vne-review](skills/vne-review/SKILL.md) | Reviewing a PR or diff (routes to `vne-review-*`) |

### Authoring standards (every edit)

| Skill | Use when |
|-------|----------|
| [plain-ascii-authoring](skills/plain-ascii-authoring/SKILL.md) | Any authored text or source |
| [vne-coding-style](skills/vne-coding-style/SKILL.md) | Writing or editing C++ / Objective-C++ |
| [vne-header-hygiene](skills/vne-header-hygiene/SKILL.md) | Editing includes or creating headers |
| [vne-testing](skills/vne-testing/SKILL.md) | Adding or changing GoogleTest coverage |

### Visualization workflow

| Skill | Use when |
|-------|----------|
| [vne-frame](skills/vne-frame/SKILL.md) | Naming frame inputs before editing a pass |
| [vne-backend-matrix](skills/vne-backend-matrix/SKILL.md) | Shader or binding changes across backends |
| [vne-cross-repo](skills/vne-cross-repo/SKILL.md) | Public API or binding changes across repos |
| [vne-gpu-benchmark](skills/vne-gpu-benchmark/SKILL.md) | Claiming a GPU speedup or regression |

### Verification (before finishing)

| Skill | Use when |
|-------|----------|
| [vne-build-verify](skills/vne-build-verify/SKILL.md) | Format, build, run, test |
| [vne-picture-check](skills/vne-picture-check/SKILL.md) | Proving the picture or the data behind it |

### Review lenses (loaded by vne-review)

| Skill | Reviews |
|-------|---------|
| [vne-review-cpp](skills/vne-review-cpp/SKILL.md) | C++ / ObjC++ correctness: lifetime, errors, integers, UB, threads |
| [vne-review-input](skills/vne-review-input/SKILL.md) | Loaders, parsers, FFI entry points (untrusted input) |
| [vne-review-medical](skills/vne-review-medical/SKILL.md) | Orientation, spacing, Hounsfield, window/level, measurement |
| [vne-review-shaders](skills/vne-review-shaders/SKILL.md) | GLSL / WGSL / MSL source and GPU struct layouts |
| [vne-review-gpu](skills/vne-review-gpu/SKILL.md) | Host GPU code: pipelines, bindings, sync, WebGPU limits |
| [vne-review-cmake](skills/vne-review-cmake/SKILL.md) | CMake targets, export, dependencies, platforms |
| [vne-review-scripts-ci](skills/vne-review-scripts-ci/SKILL.md) | bash, PowerShell, Python, GitHub Actions |
| [vne-review-bindings](skills/vne-review-bindings/SKILL.md) | Swift, C#, Kotlin, JNI over the C API |

The same rules feed the CodeRabbit settings in [coderabbit/](coderabbit/README.md).

Domain skills that name concrete paths stay in each library repo (for example
`vne-shader-pipeline` in vnerhi / vnegfx, `vne-xr-session` in vnexr). `/vne-mode`
opens the local skill when present.

## Use with pstack (no local copy)

`/poteto-mode`, `/how`, `/why`, `/teach`, `/interrogate`, `/tdd`, `/no-comments`,
`/technical-writing`, `/show-me-your-work`, `/reflect`, `/correct`.

Doxygen on non-obvious public APIs stays. `/no-comments` does not strip
`@brief` / `@param` / `@return`.

## Self-check

```bash
rg -nP "[^\x00-\x7F]" skills README.md plugin.json
```

A clean run prints nothing.

## License

Apache-2.0. See [LICENSE](LICENSE).
