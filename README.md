# COLMAP for Windows with CUDA and Ceres Solver

This repository contains a custom build of [COLMAP](https://colmap.github.io/), specifically patched and compiled for **Windows** environments using **CUDA 13.3** and **Ceres Solver**.

## Features
- **CUDA 13.3 Integration**: Take full advantage of GPU acceleration for dense reconstruction, feature extraction, and matching.
- **Ceres Solver**: Pre-configured with the powerful Ceres Solver for robust bundle adjustment and non-linear least squares problems.
- **Automated Build Script**: Includes an easy-to-use PowerShell script (`build_ceres_colmap.ps1`) to compile COLMAP and its dependencies from scratch using `vcpkg`.
- **SiftGPU Patches**: Includes a custom patch to `SiftGPU.cpp` to resolve charizing operator (`#@`) compilation errors under modern MSVC/CUDA compilers.

## Pre-compiled Binaries
If you don't want to build from source, you can download the **pre-compiled Windows binaries** from the [Releases](../../releases) page. 
Simply extract the `.zip` archive and run `colmap.exe` from the `bin` directory.

## Building from Source

### Prerequisites
1. **Visual Studio** with C++ development tools installed.
2. **CUDA Toolkit 13.3** installed and configured in your system PATH.
3. **Git** installed.

### Build Steps
1. Clone this repository:
   ```powershell
   git clone https://github.com/Brainrotisreal/colmap-windows-cuda.git
   ```
2. Run the automated build script (ensure your execution policy allows it):
   ```powershell
   Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force
   .\build_ceres_colmap.ps1
   ```
3. The script will automatically configure `vcpkg`, install dependencies (including Ceres), apply the MSVC patch, and build COLMAP.

## License
This project inherits the original licenses of COLMAP (New BSD License) and its respective dependencies.
