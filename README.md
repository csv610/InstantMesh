# Instant Meshes (Apple Silicon Port - 2026)

This repository is a maintenance port of the original **Instant Meshes** software, optimized for **Apple Silicon (M1/M2/M3)** Macs. It has been updated in **2026** for compatibility with the latest version of macOS, **CMake 4.2**, and modern compilers.

## Overview

**Instant Meshes** is an interactive software for generating field-aligned surface meshes from 3D input. 

This version addresses specific issues encountered on modern hardware:
- Fixes GLSL shader compilation errors (null-termination issues in resource generation).
- Updates the build system (`CMakeLists.txt`) for Apple Silicon compatibility.
- Streamlines the resource handling for modern environments.

## Why this Port was Non-Trivial

Porting this legacy codebase to modern Apple Silicon hardware involved solving several deep-seated technical issues:

1.  **GLSL Buffer Overflows:** The original resource generation system (`bin2c.cmake`) did not null-terminate shader source strings. On modern Apple Silicon drivers, this caused the GLSL compiler to read past the intended buffer into adjacent memory (often hitting binary resources or XML plists), resulting in cryptic syntax errors like `ERROR: 0:164: '<'`.
2.  **Resource Pipeline Refactoring:** Solving this required a precise modification of the internal resource pipeline to ensure shader files are null-terminated while binary files (like PNGs) remain untouched, all while maintaining the integrity of the `ext/nanogui` submodules.
3.  **Refactoring with AI:** The extensive refactoring of the shader initialization system in `src/viewer.cpp` to use size-aware `std::string` constructors was performed using **Gemini 3.1**, ensuring a robust and modernized implementation.

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

## CLI Usage

The compiled binary supports a powerful batch mode for automated processing.

```bash
"./Instant Meshes" -o output.obj -f 5000 input.obj
```

### Common Options:
*   `-o, --output <path>`: Output path (PLY/OBJ).
*   `-f, --faces <count>`: Target face count.
*   `-v, --vertices <count>`: Target vertex count.
*   `-s, --scale <scale>`: Target edge length.
*   `-r, --rosy <2|4|6>`: Rotation symmetry (4 for quads).
*   `-p, --posy <4|6>`: Position symmetry (4 for quads).
*   `-b, --boundaries`: Align to boundaries.
*   `-D, --dominant`: Generate tri/quad dominant mesh instead of pure.

## Calling from Python

The most robust way to use Instant Meshes in a Python workflow is via the `subprocess` module. Using a `dataclass` is recommended for clean parameter management.

```python
import subprocess
from dataclasses import dataclass, asdict
from typing import Optional

@dataclass
class RemeshOptions:
    faces: Optional[int] = None      # Target face count
    vertices: Optional[int] = None   # Target vertex count
    scale: Optional[float] = None    # Target edge length
    rosy: int = 4                    # Rotation symmetry (2, 4, 6)
    posy: int = 4                    # Position symmetry (4, 6)
    crease: Optional[float] = None   # Crease angle threshold
    smooth: int = 2                  # Smoothing iterations
    knn: int = 10                    # kNN for point clouds
    threads: Optional[int] = None    # Thread count
    deterministic: bool = False      # Deterministic algorithms
    intrinsic: bool = False          # Intrinsic optimization
    boundaries: bool = False         # Align to boundaries
    dominant: bool = False           # Tri/Quad dominant output

def instant_meshes_remesh(input_mesh: str, output_mesh: str, options: RemeshOptions):
    cmd = ["./Instant Meshes", "-o", output_mesh]

    if options.faces: cmd += ["-f", str(options.faces)]
    elif options.vertices: cmd += ["-v", str(options.vertices)]
    elif options.scale: cmd += ["-s", str(options.scale)]

    cmd += ["-r", str(options.rosy), "-p", str(options.posy)]
    if options.crease is not None: cmd += ["-c", str(options.crease)]
    if options.smooth != 2: cmd += ["-S", str(options.smooth)]
    if options.knn != 10: cmd += ["-k", str(options.knn)]
    if options.threads: cmd += ["-t", str(options.threads)]

    if options.deterministic: cmd.append("-d")
    if options.intrinsic: cmd.append("-i")
    if options.boundaries: cmd.append("-b")
    if options.dominant: cmd.append("-D")

    cmd.append(input_mesh)
    subprocess.run(cmd, check=True)

# Usage
opts = RemeshOptions(faces=5000, dominant=True, boundaries=True)
instant_meshes_remesh("input.obj", "output.obj", opts)
```

### Detailed Options Explanation:

| Option | CLI Flag | Description |
| :--- | :--- | :--- |
| **Faces** | `-f`, `--faces` | The approximate number of faces in the generated mesh. |
| **Vertices** | `-v`, `--vertices` | The approximate number of vertices in the generated mesh. |
| **Scale** | `-s`, `--scale` | The desired world-space length of edges in the output. |
| **RoSy** | `-r`, `--rosy` | **Rotation Symmetry**: 2 for line-aligned, 4 for quads (default), 6 for triangles. |
| **PoSy** | `-p`, `--posy` | **Position Symmetry**: 4 for quads/tri-dominant (default), 6 for triangles. |
| **Crease** | `-c`, `--crease` | Dihedral angle threshold (in degrees) to identify and preserve sharp features. |
| **Smooth** | `-S`, `--smooth` | Number of post-extraction smoothing and reprojection steps (default: 2). |
| **kNN** | `-k`, `--knn` | For point cloud inputs, the number of adjacent points to consider for surface estimation. |
| **Threads** | `-t`, `--threads` | Limits the number of CPU threads used for parallel computation. |
| **Deterministic** | `-d` | Ensures bit-for-bit identical results on every run at a slight performance cost. |
| **Intrinsic** | `-i` | Performs optimization in tangent space rather than 3D world space. |
| **Boundaries** | `-b` | Forces the field to align with the open boundaries of the input mesh. |
| **Dominant** | `-D` | Produces a tri/quad dominant mesh instead of forcing a pure tri or quad structure. |

## License

This project is licensed under the same 3-clause BSD license as the original software. See `LICENSE.txt` for the full text.
