# Using Pixi with `pyproject.toml` based Python packages

:::{note} Example project being used
The example project being shown is taken from the September 2023 talk [Distributing your Science: Turning analyses into scientific tools](https://github.com/matthewfeickert-talks/talk-odsl-forum-seminar-2023) by Matthew Feickert.
The example project has the following directory tree structure

```bash
$ tree .
.
├── CMakeLists.txt
├── LICENSE
├── pyproject.toml
├── README.md
├── src
│   ├── basic_math.cpp
│   └── rosen_cpp
│       ├── example.py
│       └── __init__.py
└── tests
    └── test_example.py

4 directories, 8 files
```

You can clone the talk repository to get access to those files, or get them from this tutorial's source repository.
:::

## Initializing a Pixi project

If you initialize a Pixi project in the same directory as a [`pyproject.toml`](https://github.com/matthewfeickert-talks/talk-odsl-forum-seminar-2023/blob/d23b038df8969c02b89b9682ccb71199df88aa7e/examples/compiled_packaging/pyproject.toml), it will prompt you to create the Pixi workspace as a `[tool.pixi]` subtable in the `pyproject.toml`

```bash
pixi init
```
```
A 'pyproject.toml' file already exists. Do you want to extend it with the '[tool.pixi]' configuration? yes
✔ Added package 'rosen-cpp' as an editable dependency.
```

```{code} toml
:filename: pyproject.toml
[build-system]
requires = [
  "scikit-build-core>=0.5.0",
  "pybind11>=2.10.0",
  ]
build-backend = "scikit_build_core.build"


[project]
name = "rosen-cpp"
dynamic = ["version"]
description = "Example package for demonstration"
...
requires-python = ">=3.10"

dependencies = [
    "scipy>=1.14.0",
    "numpy",  # compatible versions controlled through scipy
]

...

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64"]

[tool.pixi.pypi-dependencies]
rosen-cpp = { path = ".", editable = true }

[tool.pixi.tasks]
```

Which at `pixi install` time, will:

* Automatically add `python` as a dependency
* Install the Python `project.dependencies` from PyPI
* Install the project as a local Python project (as an editable project by default) using the tools that exist already on the system

:::{note} editable install are for non-compiled extension projects
We can remove the editable install as `rosen-cpp` has compiled extensions, and so won't benefit from editable installs.

```toml
[tool.pixi.pypi-dependencies]
rosen-cpp = { path = "." }
```
:::

## Extending as a normal Pixi project

Beyond that nothing is special about the Pixi workspace, so we can add additional platforms

```bash
pixi workspace platform add linux-64 osx-arm64
```
```
✔ Added linux-64
✔ Added osx-arm64
```

```{code} toml
:filename: pyproject.toml
...

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64"]

[tool.pixi.pypi-dependencies]
rosen-cpp = { path = "." }

[tool.pixi.tasks]
```

add build tools

```bash
pixi add cxx-compiler
```
```
✔ Added cxx-compiler >=1.11.0,<2
```

```{code} toml
:filename: pyproject.toml
...

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64"]

[tool.pixi.pypi-dependencies]
rosen-cpp = { path = "." }

[tool.pixi.tasks]

[tool.pixi.dependencies]
cxx-compiler = ">=1.11.0,<2"
```

## Using conda packages over Python packages for local builds

Our Python dependencies are currently being installed as Python packages from PyPI (which is fine)

```bash
pixi list
```
```
Package                   Version    Build               Size       Kind   Source
_libgcc_mutex             0.1        conda_forge         2.5 KiB    conda  https://conda.anaconda.org/conda-forge/
_openmp_mutex             4.5        2_gnu               23.1 KiB   conda  https://conda.anaconda.org/conda-forge/
binutils                  2.44       h4852527_4          34.2 KiB   conda  https://conda.anaconda.org/conda-forge/
binutils_impl_linux-64    2.44       h9d8b0ac_4          3.5 MiB    conda  https://conda.anaconda.org/conda-forge/
binutils_linux-64         2.44       h4852527_4          35.2 KiB   conda  https://conda.anaconda.org/conda-forge/
bzip2                     1.0.8      hda65f42_8          254.2 KiB  conda  https://conda.anaconda.org/conda-forge/
c-compiler                1.11.0     h4d9bdce_0          6.5 KiB    conda  https://conda.anaconda.org/conda-forge/
ca-certificates           2025.10.5  hbd8a1cb_0          152.3 KiB  conda  https://conda.anaconda.org/conda-forge/
conda-gcc-specs           14.3.0     hb991d5c_7          32.5 KiB   conda  https://conda.anaconda.org/conda-forge/
cxx-compiler              1.11.0     hfcd1e18_0          6.5 KiB    conda  https://conda.anaconda.org/conda-forge/
gcc                       14.3.0     h76bdaa0_7          30.3 KiB   conda  https://conda.anaconda.org/conda-forge/
gcc_impl_linux-64         14.3.0     hd9e9e21_7          66.7 MiB   conda  https://conda.anaconda.org/conda-forge/
gcc_linux-64              14.3.0     h298d278_12         27.3 KiB   conda  https://conda.anaconda.org/conda-forge/
gxx                       14.3.0     he448592_7          29.7 KiB   conda  https://conda.anaconda.org/conda-forge/
gxx_impl_linux-64         14.3.0     he663afc_7          14.5 MiB   conda  https://conda.anaconda.org/conda-forge/
gxx_linux-64              14.3.0     h95f728e_12         26.4 KiB   conda  https://conda.anaconda.org/conda-forge/
kernel-headers_linux-64   4.18.0     he073ed8_8          1.2 MiB    conda  https://conda.anaconda.org/conda-forge/
ld_impl_linux-64          2.44       h1aa0949_4          725.1 KiB  conda  https://conda.anaconda.org/conda-forge/
libexpat                  2.7.1      hecca717_0          73.1 KiB   conda  https://conda.anaconda.org/conda-forge/
libffi                    3.5.2      h9ec8514_0          56.5 KiB   conda  https://conda.anaconda.org/conda-forge/
libgcc                    15.2.0     h767d61c_7          803.3 KiB  conda  https://conda.anaconda.org/conda-forge/
libgcc-devel_linux-64     14.3.0     h85bb3a7_107        2.6 MiB    conda  https://conda.anaconda.org/conda-forge/
libgomp                   15.2.0     h767d61c_7          437.4 KiB  conda  https://conda.anaconda.org/conda-forge/
liblzma                   5.8.1      hb9d3cd8_2          110.2 KiB  conda  https://conda.anaconda.org/conda-forge/
libmpdec                  4.0.0      hb9d3cd8_0          89 KiB     conda  https://conda.anaconda.org/conda-forge/
libsanitizer              14.3.0     hd08acf3_7          4.9 MiB    conda  https://conda.anaconda.org/conda-forge/
libsqlite                 3.50.4     h0c1763c_0          910.7 KiB  conda  https://conda.anaconda.org/conda-forge/
libstdcxx                 15.2.0     h8f9b012_7          3.7 MiB    conda  https://conda.anaconda.org/conda-forge/
libstdcxx-devel_linux-64  14.3.0     h85bb3a7_107        12.6 MiB   conda  https://conda.anaconda.org/conda-forge/
libuuid                   2.41.2     he9a06e4_0          36.3 KiB   conda  https://conda.anaconda.org/conda-forge/
libzlib                   1.3.1      hb9d3cd8_2          59.5 KiB   conda  https://conda.anaconda.org/conda-forge/
ncurses                   6.5        h2d0b736_3          870.7 KiB  conda  https://conda.anaconda.org/conda-forge/
numpy                     2.3.4                          55.2 MiB   pypi   numpy-2.3.4-cp314-cp314-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl
openssl                   3.5.4      h26f9b46_0          3 MiB      conda  https://conda.anaconda.org/conda-forge/
python                    3.14.0     h32b2ec7_102_cp314  35 MiB     conda  https://conda.anaconda.org/conda-forge/
python_abi                3.14       8_cp314             6.8 KiB    conda  https://conda.anaconda.org/conda-forge/
readline                  8.2        h8c095d6_2          275.9 KiB  conda  https://conda.anaconda.org/conda-forge/
rosen_cpp                 0.1.dev12                                 pypi
scipy                     1.16.2                         112.9 MiB  pypi   scipy-1.16.2-cp314-cp314-manylinux2014_x86_64.manylinux_2_17_x86_64.whl
sysroot_linux-64          2.28       h4ee821c_8          23.1 MiB   conda  https://conda.anaconda.org/conda-forge/
tk                        8.6.13     noxft_hd72426e_102  3.1 MiB    conda  https://conda.anaconda.org/conda-forge/
tzdata                    2025b      h78e105d_0          120.1 KiB  conda  https://conda.anaconda.org/conda-forge/
zstd                      1.5.7      hb8e6e7a_2          554.3 KiB  conda  https://conda.anaconda.org/conda-forge/
```

though if we want to harmonize and use as much of Pixi's abilities as possible, we can additionally add our Python package dependencies to our Pixi dependency table

:::{note}
NumPy is a dependency of SciPy, so we don't need to tell Pixi to add it.
:::

```bash
pixi add 'scipy >=1.14.0'
```
```
✔ Added scipy >=1.14.0
```

```{code} toml
:filename: pyproject.toml
...

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64"]

[tool.pixi.pypi-dependencies]
rosen-cpp = { path = "." }

[tool.pixi.tasks]

[tool.pixi.dependencies]
cxx-compiler = ">=1.11.0,<2"
scipy = ">=1.14.0"
```

```bash
pixi list
```
```
Package                   Version    Build                 Size       Kind   Source
_libgcc_mutex             0.1        conda_forge           2.5 KiB    conda  https://conda.anaconda.org/conda-forge/
_openmp_mutex             4.5        2_gnu                 23.1 KiB   conda  https://conda.anaconda.org/conda-forge/
binutils                  2.44       h4852527_4            34.2 KiB   conda  https://conda.anaconda.org/conda-forge/
binutils_impl_linux-64    2.44       h9d8b0ac_4            3.5 MiB    conda  https://conda.anaconda.org/conda-forge/
binutils_linux-64         2.44       h4852527_4            35.2 KiB   conda  https://conda.anaconda.org/conda-forge/
bzip2                     1.0.8      hda65f42_8            254.2 KiB  conda  https://conda.anaconda.org/conda-forge/
c-compiler                1.11.0     h4d9bdce_0            6.5 KiB    conda  https://conda.anaconda.org/conda-forge/
ca-certificates           2025.10.5  hbd8a1cb_0            152.3 KiB  conda  https://conda.anaconda.org/conda-forge/
conda-gcc-specs           14.3.0     hb991d5c_7            32.5 KiB   conda  https://conda.anaconda.org/conda-forge/
cxx-compiler              1.11.0     hfcd1e18_0            6.5 KiB    conda  https://conda.anaconda.org/conda-forge/
gcc                       14.3.0     h76bdaa0_7            30.3 KiB   conda  https://conda.anaconda.org/conda-forge/
gcc_impl_linux-64         14.3.0     hd9e9e21_7            66.7 MiB   conda  https://conda.anaconda.org/conda-forge/
gcc_linux-64              14.3.0     h298d278_12           27.3 KiB   conda  https://conda.anaconda.org/conda-forge/
gxx                       14.3.0     he448592_7            29.7 KiB   conda  https://conda.anaconda.org/conda-forge/
gxx_impl_linux-64         14.3.0     he663afc_7            14.5 MiB   conda  https://conda.anaconda.org/conda-forge/
gxx_linux-64              14.3.0     h95f728e_12           26.4 KiB   conda  https://conda.anaconda.org/conda-forge/
kernel-headers_linux-64   4.18.0     he073ed8_8            1.2 MiB    conda  https://conda.anaconda.org/conda-forge/
ld_impl_linux-64          2.44       h1aa0949_4            725.1 KiB  conda  https://conda.anaconda.org/conda-forge/
libblas                   3.9.0      37_h4a7cf45_openblas  17.1 KiB   conda  https://conda.anaconda.org/conda-forge/
libcblas                  3.9.0      37_h0358290_openblas  17.1 KiB   conda  https://conda.anaconda.org/conda-forge/
libexpat                  2.7.1      hecca717_0            73.1 KiB   conda  https://conda.anaconda.org/conda-forge/
libffi                    3.5.2      h9ec8514_0            56.5 KiB   conda  https://conda.anaconda.org/conda-forge/
libgcc                    15.2.0     h767d61c_7            803.3 KiB  conda  https://conda.anaconda.org/conda-forge/
libgcc-devel_linux-64     14.3.0     h85bb3a7_107          2.6 MiB    conda  https://conda.anaconda.org/conda-forge/
libgfortran               15.2.0     h69a702a_7            28.6 KiB   conda  https://conda.anaconda.org/conda-forge/
libgfortran5              15.2.0     hcd61629_7            1.5 MiB    conda  https://conda.anaconda.org/conda-forge/
libgomp                   15.2.0     h767d61c_7            437.4 KiB  conda  https://conda.anaconda.org/conda-forge/
liblapack                 3.9.0      37_h47877c9_openblas  17.1 KiB   conda  https://conda.anaconda.org/conda-forge/
liblzma                   5.8.1      hb9d3cd8_2            110.2 KiB  conda  https://conda.anaconda.org/conda-forge/
libmpdec                  4.0.0      hb9d3cd8_0            89 KiB     conda  https://conda.anaconda.org/conda-forge/
libopenblas               0.3.30     pthreads_h94d23a6_2   5.7 MiB    conda  https://conda.anaconda.org/conda-forge/
libsanitizer              14.3.0     hd08acf3_7            4.9 MiB    conda  https://conda.anaconda.org/conda-forge/
libsqlite                 3.50.4     h0c1763c_0            910.7 KiB  conda  https://conda.anaconda.org/conda-forge/
libstdcxx                 15.2.0     h8f9b012_7            3.7 MiB    conda  https://conda.anaconda.org/conda-forge/
libstdcxx-devel_linux-64  14.3.0     h85bb3a7_107          12.6 MiB   conda  https://conda.anaconda.org/conda-forge/
libuuid                   2.41.2     he9a06e4_0            36.3 KiB   conda  https://conda.anaconda.org/conda-forge/
libzlib                   1.3.1      hb9d3cd8_2            59.5 KiB   conda  https://conda.anaconda.org/conda-forge/
ncurses                   6.5        h2d0b736_3            870.7 KiB  conda  https://conda.anaconda.org/conda-forge/
numpy                     2.3.4      py314h2b28147_0       8.5 MiB    conda  https://conda.anaconda.org/conda-forge/
openssl                   3.5.4      h26f9b46_0            3 MiB      conda  https://conda.anaconda.org/conda-forge/
python                    3.14.0     h32b2ec7_102_cp314    35 MiB     conda  https://conda.anaconda.org/conda-forge/
python_abi                3.14       8_cp314               6.8 KiB    conda  https://conda.anaconda.org/conda-forge/
readline                  8.2        h8c095d6_2            275.9 KiB  conda  https://conda.anaconda.org/conda-forge/
rosen_cpp                 0.1.dev12                                   pypi
scipy                     1.16.2     py314he7377e1_0       16.5 MiB   conda  https://conda.anaconda.org/conda-forge/
sysroot_linux-64          2.28       h4ee821c_8            23.1 MiB   conda  https://conda.anaconda.org/conda-forge/
tk                        8.6.13     noxft_hd72426e_102    3.1 MiB    conda  https://conda.anaconda.org/conda-forge/
tzdata                    2025b      h78e105d_0            120.1 KiB  conda  https://conda.anaconda.org/conda-forge/
zstd                      1.5.7      hb8e6e7a_2            554.3 KiB  conda  https://conda.anaconda.org/conda-forge/
```

We similarly can add other features and environments

```console
$ pixi add --feature test pytest
✔ Added pytest
Added these only for feature: test
$ pixi workspace environment add --feature test test
✔ Added environment test
$ pixi task add --feature test test "pytest tests"
✔ Added task `test`: pytest tests
$ pixi run test
✨ Pixi task (test in test): pytest tests
====================================================== test session starts ...
platform linux -- Python 3.14.0, pytest-8.4.2, pluggy-1.6.0
rootdir: /home/feickert/Code/GitHub/talks/talk-pyhep-2025/book/code/packaging/compiled_packaging
configfile: pyproject.toml
collected 2 items

tests/test_example.py ..                                  [100%]

...

======================================================== 2 passed, 1 warning in 0.60s ...
```
