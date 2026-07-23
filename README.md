# COLMAP for Windows with CUDA and Ceres Solver

This repository tracks [COLMAP](https://colmap.github.io/) (currently synced to **4.1.1**) with a
small set of Windows-specific additions for building with **CUDA** and a
**CUDA-accelerated Ceres Solver**.

## Is a custom CUDA-Ceres build still necessary?

Mostly no, and it's worth being upfront about that:

- **CUDA-accelerated Ceres bundle adjustment ships in stock COLMAP already.**
  `bundle_adjustment_ceres.cc` sets `dense_linear_algebra_library_type = ceres::CUDA`
  and, when available, `ceres::CUDA_SPARSE` automatically whenever Ceres was built
  with CUDA support and a GPU is selected. No fork or patch is required for the
  application-level integration — only for how Ceres itself gets built.
- **vcpkg's `ceres` port now has a built-in `cuda` feature** ("Support for CUDA
  based dense solvers"). `vcpkg.json` in this repo now requests it automatically
  whenever COLMAP's own `cuda` feature is selected (i.e. `-DCUDA_ENABLED=ON`), so
  a CUDA-accelerated dense Ceres solver comes from the normal, cached vcpkg build
  — no manual Ceres clone/build/patch/`Ceres_DIR` juggling needed for that case.
- **The one real gap:** vcpkg's `ceres` port is currently pinned to Ceres 2.2.0,
  which predates cuDSS-backed `CUDA_SPARSE` (GPU sparse Cholesky) support. If you
  specifically need GPU-accelerated *sparse* Ceres BA (large problems, not just
  small/dense ones), you still need to build Ceres from source against cuDSS
  yourself — that's what `build_ceres_colmap.ps1` is for. It's a niche, advanced
  path now, not the default recommendation.
- **The bigger development to be aware of:** COLMAP 4.1.0 shipped **Caspar**, a
  purpose-built GPU bundle-adjustment backend (not based on Ceres at all) that
  the upstream changelog describes as **1-2 orders of magnitude faster** than
  Ceres BA. It's already vendored in this repository
  (`src/colmap/estimators/bundle_adjustment_caspar.*`,
  `src/thirdparty/Symforce-Caspar/`) and just gained an MSVC/CUDA compatibility
  fix upstream, so it now builds on Windows too. If your goal is "fast
  GPU-accelerated bundle adjustment on Windows," build with `-DCASPAR_ENABLED=ON`
  instead of chasing CUDA-Ceres — it's the bigger win by a wide margin.

In short: this repo still exists to keep a working, up-to-date, CUDA-enabled
Windows build of COLMAP with minimal friction — but "CUDA Ceres" specifically is
no longer something you need custom tooling for in the common case.

## What's actually custom here

- `vcpkg.json`: requests Ceres' `cuda` feature whenever COLMAP's own `cuda`
  feature is active, so CI/vcpkg builds get a CUDA-accelerated dense Ceres
  solver for free.
- `src/thirdparty/SiftGPU/SiftGPU.cpp`: disables SiftGPU's non-portable MSVC
  "charizing operator" (`#@`) macro path, which modern MSVC/CUDA toolchains
  fail to compile. Upstream COLMAP still ships the original macro; this patch
  is still needed for MSVC builds.
- `build_ceres_colmap.ps1`: optional, standalone script for hand-building Ceres
  with cuDSS (GPU sparse solver) support and wiring it into a from-source COLMAP
  build. Only needed if you want `CUDA_SPARSE`, since vcpkg's Ceres doesn't offer
  it yet.

## Pre-compiled Binaries
If you don't want to build from source, you can download the **pre-compiled Windows binaries** from the [Releases](../../releases) page.
Simply extract the `.zip` archive and run `colmap.exe` from the `bin` directory.

## Building from Source

### Recommended: vcpkg (matches upstream CI)

```powershell
git clone https://github.com/microsoft/vcpkg
./vcpkg/bootstrap-vcpkg.bat
mkdir build; cd build
cmake .. -GNinja `
  -DCMAKE_BUILD_TYPE=Release `
  -DGUI_ENABLED=ON `
  -DCUDA_ENABLED=ON `
  -DCASPAR_ENABLED=ON `
  -DCMAKE_CUDA_ARCHITECTURES=all-major `
  -DCMAKE_TOOLCHAIN_FILE="../vcpkg/scripts/buildsystems/vcpkg.cmake" `
  -DVCPKG_TARGET_TRIPLET=x64-windows-release `
  -DCMAKE_INSTALL_PREFIX=install
ninja
ninja install
```

This gets you CUDA-accelerated dense Ceres BA and Caspar GPU BA, both without
any manual Ceres build.

### Advanced: hand-built Ceres with cuDSS (CUDA_SPARSE)

Only needed if you specifically want GPU-accelerated *sparse* Ceres BA today.

#### Prerequisites
1. **Visual Studio** with C++ development tools installed.
2. **CUDA Toolkit** installed and configured in your system PATH.
3. **Git** installed.

#### Build Steps
1. Clone this repository:
   ```powershell
   git clone https://github.com/Malachite-Ore/colmap-windows-cuda.git
   ```
2. Run the automated build script (ensure your execution policy allows it):
   ```powershell
   Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force
   .\build_ceres_colmap.ps1
   ```
3. The script will automatically configure `vcpkg`, clone and build Ceres with
   CUDA/cuDSS support from source, apply the MSVC SiftGPU patch to a fresh
   COLMAP clone, and build COLMAP against that Ceres.

## License
This project inherits the original licenses of COLMAP (New BSD License) and its respective dependencies.
