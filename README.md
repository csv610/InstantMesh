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

The most robust way to use Instant Meshes in a Python workflow is via the `subprocess` module. Below is a comprehensive wrapper supporting all common CLI flags:

```python
import subprocess
import os

def instant_meshes_remesh(
    input_mesh, 
    output_mesh, 
    faces=None,      # -f, --faces <count>
    vertices=None,   # -v, --vertices <count>
    scale=None,      # -s, --scale <scale>
    rosy=4,          # -r, --rosy <2|4|6>
    posy=4,          # -p, --posy <4|6>
    crease=None,     # -c, --crease <degrees>
    smooth=2,        # -S, --smooth <iter>
    knn=10,          # -k, --knn <count>
    threads=None,    # -t, --threads <count>
    deterministic=False, # -d, --deterministic
    intrinsic=False,     # -i, --intrinsic
    boundaries=False,    # -b, --boundaries
    dominant=False       # -D, --dominant
):
    """
    Comprehensive Python wrapper for Instant Meshes CLI.
    Note: Provide exactly one of [faces, vertices, scale].
    """
    cmd = ["./Instant Meshes", "-o", output_mesh]

    # Target size constraints (exclusive)
    if faces: cmd += ["-f", str(faces)]
    elif vertices: cmd += ["-v", str(vertices)]
    elif scale: cmd += ["-s", str(scale)]

    # Symmetry and optimization
    cmd += ["-r", str(rosy), "-p", str(posy)]
    if crease is not None: cmd += ["-c", str(crease)]
    if smooth != 2: cmd += ["-S", str(smooth)]
    if knn != 10: cmd += ["-k", str(knn)]
    if threads: cmd += ["-t", str(threads)]

    # Boolean flags
    if deterministic: cmd.append("-d")
    if intrinsic: cmd.append("-i")
    if boundaries: cmd.append("-b")
    if dominant: cmd.append("-D")

    # Input file
    cmd.append(input_mesh)

    print(f"Running: {' '.join(cmd)}")
    subprocess.run(cmd, check=True)

# Example Usage:
# Generate a tri-quad dominant mesh with 5000 faces and sharp crease handling
instant_meshes_remesh(
    "input.obj", "output.obj", 
    faces=5000, 
    crease=45, 
    dominant=True,
    boundaries=True
)
```

## License

This project is licensed under the same 3-clause BSD license as the original software. See `LICENSE.txt` for the full text.
