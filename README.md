# Instant Meshes (Apple Silicon Port)

This repository is a maintenance port of the original **Instant Meshes** software, optimized for **Apple Silicon (M1/M2/M3)** Macs and updated for compatibility with the latest version of macOS, CMake, and modern compilers.

## Overview

**Instant Meshes** is an interactive software for generating field-aligned surface meshes from 3D input. 

This version addresses specific issues encountered on modern hardware:
- Fixes GLSL shader compilation errors (null-termination issues in resource generation).
- Updates the build system (`CMakeLists.txt`) for Apple Silicon compatibility.
- Streamlines the resource handling for modern environments.

## Credits & Original Work

All original research, core algorithms, and the initial software implementation are the work of the authors of the paper:

> **Instant Field-Aligned Meshes**  
> Wenzel Jakob, Marco Tarini, Daniele Panozzo, Olga Sorkine-Hornung  
> In *ACM Transactions on Graphics (Proceedings of SIGGRAPH Asia 2015)*

The original repository can be found at: [wjakob/instant-meshes](https://github.com/wjakob/instant-meshes)

**Note:** This repository is strictly a port and maintenance effort. No contributions have been made to the underlying algorithms or research beyond what was necessary for modern hardware compatibility.

## Building on macOS

Building requires CMake and a recent version of Xcode (or Command Line Tools).

```bash
git clone --recursive https://github.com/csv610/InstantMesh
cd InstantMesh
cmake .
make -j
```

## License

This project is licensed under the same 3-clause BSD license as the original software. See `LICENSE.txt` for the full text.
