# Per-repo build commands

Pick the section that matches `basename "$(git rev-parse --show-toplevel)"`.

## vnerhi

```bash
git submodule update --init --recursive
git submodule update --init testdata/vneresources

./scripts/build_macos.sh -t Debug -a configure_and_build --dev
# Linux: ./scripts/build_linux.sh -t Debug -a configure_and_build --dev

ctest --test-dir <build-dir> --output-on-failure
<build-dir>/bin/28_stencil_testing --backend vulkan --validation
```

`--dev` enables tests and samples. Build dir example:
`build/shared/Debug/build-macos-clang-17.0.0`. Backends: `null`, `vulkan`,
`metal`, `webgpu`. Shader changes: local `skills/vne-shader-pipeline/SKILL.md`.

Other platforms: `build_linux.sh`, `build_wasm.sh`, `build_ios.sh`,
`build_visionos.sh`, `build_android.sh`, `python scripts/build_windows.py`
(same flag set; see repo `scripts/README.md`).

## vnegfx

```bash
./scripts/init_submodules.sh full

./scripts/build_macos.sh -t Debug -a configure_and_build --dev
# or: cmake -S . -B build/phase5 -DCMAKE_BUILD_TYPE=Debug -DVNE_GFX_DEV=ON
#     cmake --build build/phase5 -j8

ctest --test-dir build/phase5 --output-on-failure
```

Private `vnerhi` is required. Shader / `gpu_layout.yaml` changes: local
`skills/vne-shader-pipeline/SKILL.md`. Image goldens live under
`testdata/vneresources` when initialized.

## vnexr

Primary host: Ubuntu 24.04. macOS builds Null / visionOS only (OpenXR blocked).

```bash
export VNE_VNERHI_PATH=../vnerhi   # optional local checkout
./scripts/init_submodules.sh full
./scripts/install_linux_openxr_deps.sh

./scripts/build_linux.sh -t Debug -a configure_and_build --with-openxr --with-examples
ctest --test-dir build/shared/Debug/build-linux-gcc-* --output-on-failure

systemctl --user start monado.service   # or monado-service &
export XR_RUNTIME_JSON=/usr/share/openxr/1/openxr_monado.json
./build/shared/Debug/build-linux-gcc-*/bin/samples/sample_01_null_simulator
./build/shared/Debug/build-linux-gcc-*/bin/samples/sample_02_openxr_stereo
```

Android / Galaxy XR:

```bash
./scripts/build_android.sh -t Debug -a install \
  -sample sample_02_openxr_stereo -abi arm64-v8a
./scripts/verify_android_vulkan.sh com.vertexnova.vnexr.openxr_stereo
adb logcat -s vne.xr:* OpenXR:*
```

Session and packaging rules: local `skills/vne-xr-session/SKILL.md` and
`skills/vne-xr-android/SKILL.md`.

## vnewindow

No shader regen step. No `VNE_GFX_*` / `build/phase3`.

```bash
git submodule update --init --recursive

./scripts/build_macos.sh -t Debug -a configure_and_build
# or: cmake -B build/shared -DCMAKE_BUILD_TYPE=Debug -DVNE_XWIN_DEV=ON
#     cmake --build build/shared

ctest -C Debug --test-dir build/shared --output-on-failure
```

Headless unit tests use `WindowAPI::eNullWindow`. Platform backends: local
`skills/vne-xwin-platforms/SKILL.md`.

## vne3dgs

CPU reference and math first; GPU viewer not required until that phase lands.

```bash
git submodule update --init --recursive

./scripts/build_macos.sh -t Debug -a configure_and_build
# Linux: ./scripts/build_linux.sh -t Debug -a configure_and_build

ctest --test-dir <build-dir> --output-on-failure
<build-dir>/bin/examples/example_00_hello_gs
```

## Fallback (other VertexNova repos)

Repos without a row above (vnecmake, vnecommon, vnelogging, vnemath, vnescene,
vneshaderc, vnevoice, ...):

```bash
git submodule update --init --recursive 2>/dev/null || true
python3 scripts/clang_formatter.py all --dry-run 2>/dev/null || true

# Prefer a platform script when present:
./scripts/build_macos.sh -t Debug -a configure_and_build
# or ./scripts/build_linux.sh -t Debug -a configure_and_build
# or: cmake -B build/shared -DCMAKE_BUILD_TYPE=Debug && cmake --build build/shared

ctest --test-dir <build-dir> --output-on-failure
```

Read the repo `README.md` and `scripts/README.md` when flags differ.
