# Using CUDA conda packages with Pixi

:::{tip} Resources
For extensive resources on reproducible CUDA workflows with Pixi see

* [SciPy 2025 tutorial on Reproducible Machine Learning Workflows for Scientists with Pixi](https://github.com/matthewfeickert-talks/reproducible-ml-for-scientists-with-pixi-scipy-2025), by Matthew Feickert, Ruben Arts, John Kirkham.
* Matthew Feickert, [Reproducible Machine Learning Workflows for Scientists](https://github.com/carpentries-incubator/reproducible-ml-workflows), 2025.
:::

Wrangling CUDA dependencies can be exceptionally difficult, especially across multiple machines and platforms.
As of 2022, the full CUDA stack from NVIDIA &mdash; from compilers to development libraries &mdash; is distributed as conda packages on conda-forge.
When combined with new technologies from Pixi, this allows for fully reproducible CUDA accelerated workflows to become possible.

## Pixi system requirements

Pixi allows for a [`system-requirements` table](https://pixi.sh/dev/workspace/system_requirements/), that allows for a workspace to define the system level machine configuration aspects that Pixi expects on the host system.
These system requirements are expressed through [conda virtual packages](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-virtual.html) and can be noted from `pixi info`

```bash
pixi info
```
```
System
------------
       Pixi version: 0.58.0
           Platform: linux-64
   Virtual packages: __unix=0=0
                   : __linux=6.8.0=0
                   : __glibc=2.39=0
                   : __cuda=13.0=0
                   : __archspec=1=skylake
          Cache dir: /home/<user>/.cache/rattler/cache
       Auth storage: /home/<user>/.rattler/credentials.json
   Config locations: No config files found

...
```

We can specify a system requirement for the existence of CUDA on a host system by

```bash
pixi workspace system-requirements add cuda 12
```

```{code} toml
:filename: pixi.toml
:linenos:
:emphasize-lines: 11-12
[workspace]
channels = ["conda-forge"]
name = "cuda"
platforms = ["linux-64"]
version = "0.1.0"

[tasks]

[dependencies]

[system-requirements]
cuda = "12"
```

::::{important} What does this do?

This system requirement does **not** mean that a minimum or maximum version of CUDA has been defined.
Adding this field effectively provides the `__cuda {version}` virtual package, ensuring that packages depending on `__cuda >= {version}` in their package metadata are resolved correctly.
To set specific versions of CUDA the **best** way is to define a `cuda-version` dependency.

:::{seealso} Caveat
The above is what **Pixi** implements &mdash; Pixi isn't able to make promises on what packages implement, but is able to guide resolution.
In modern (e.g. CUDA 12.9+ releases) of CUDA libraries NVIDIA has made it possible to differentiate at the **major** CUDA release, but not **minor** or finer.

* Setting a system requirement of CUDA 12 results in being bound to `v12` releases given upper bounds in `cuda-toolkit` dependencies interacting with the system-requirement.

```console
$ pixi workspace system-requirements add cuda 12
$ pixi add cuda
✔ Added cuda >=12.9.1,<13
```

* Setting a system requirement of CUDA 12.6 results in being bound to `v12` releases given upper bounds in `cuda-toolkit` dependencies interacting with the system-requirement, but does not restrict to `v12.6.*1.

```console
$ pixi workspace system-requirements add cuda 12.6
$ pixi add cuda
✔ Added cuda >=12.9.1,<13
```

* Setting a system requirement of CUDA 13 results in being bound to `v13` releases given lower bounds in `cuda-toolkit` dependencies interacting with the system-requirement.

```console
$ pixi workspace system-requirements add cuda 13
$ pixi add cuda
✔ Added cuda >=13.0.2,<14
```

* Setting a system requirement of CUDA 12.6 and specifying the `cuda-version` package results in being bound to `v12.6.*` releases given upper bounds in `cuda-toolkit` dependencies interacting with the system-requirement and `cuda-version`'s restrictions.

```console
$ pixi workspace system-requirements add cuda 12.6
$ pixi add cuda 'cuda-version 12.6.*'
✔ Added cuda >=12.6.3,<13
✔ Added cuda-version 12.6.*
```
:::

::::


:::{important} How to chose the CUDA version set

Knowing what version of CUDA to set for the `system-requirement` is going to require knowledge of the host platform's NVIDIA drivers.
The [NVIDIA System Management Interface](https://developer.nvidia.com/system-management-interface) (`nvidia-smi`) utility can provide information on a host machine's NVIDIA driver version and the **maximum supported version of CUDA that is known to work with the driver**.

```console
$ nvidia-smi --version
NVIDIA-SMI version  : 580.95.05
NVML version        : 580.95
DRIVER version      : 580.95.05
CUDA Version        : 13.0
```

CUDA has excellent backwards compatibility, so while for this driver version CUDA `13.0` is the known supported version it is probable that CUDA 12 releases will work with the driver, and possible that newer CUDA 13 releases will work as well.
The reported CUDA version is a **guarantee** but not a restriction.

For our software applications, if we know our dependencies have CUDA 13 support then we should provide a system requirement of CUDA 13.
If we know that dependencies only have support up to CUDA 12, or that our target host machine has a driver with support up to CUDA 12, then we should provide a system requirement of CUDA 12.
:::

All of this now allows us to do things like

```bash
pixi workspace system-requirements add cuda 12
pixi add pytorch-gpu 'cuda-version 12.9.*'
```

```
✔ Added pytorch-gpu >=2.8.0,<3
✔ Added cuda-version 12.9.*
```

and arrive at a fully specified and locked CUDA accelerated environment with PyTorch

```bash
pixi list torch
```
```
Package      Version  Build                           Size       Kind   Source
libtorch     2.8.0    cuda129_mkl_hc64f9c6_301        836.2 MiB  conda  https://conda.anaconda.org/conda-forge/
pytorch      2.8.0    cuda129_mkl_py313_h392364f_301  24 MiB     conda  https://conda.anaconda.org/conda-forge/
pytorch-gpu  2.8.0    cuda129_mkl_h43a4b0b_301        46.2 KiB   conda  https://conda.anaconda.org/conda-forge/
```

```bash
pixi list cuda
```
```
Package               Version  Build       Size       Kind   Source
cuda-crt-tools        12.9.86  ha770c72_2  28.5 KiB   conda  https://conda.anaconda.org/conda-forge/
cuda-cudart           12.9.79  h5888daf_0  22.7 KiB   conda  https://conda.anaconda.org/conda-forge/
cuda-cudart_linux-64  12.9.79  h3f2d84a_0  192.6 KiB  conda  https://conda.anaconda.org/conda-forge/
cuda-cuobjdump        12.9.82  hffce074_1  239.3 KiB  conda  https://conda.anaconda.org/conda-forge/
cuda-cupti            12.9.79  h676940d_1  1.8 MiB    conda  https://conda.anaconda.org/conda-forge/
cuda-nvcc-tools       12.9.86  he02047a_2  26.1 MiB   conda  https://conda.anaconda.org/conda-forge/
cuda-nvdisasm         12.9.88  hffce074_1  5.3 MiB    conda  https://conda.anaconda.org/conda-forge/
cuda-nvrtc            12.9.86  hecca717_1  64.1 MiB   conda  https://conda.anaconda.org/conda-forge/
cuda-nvtx             12.9.79  hecca717_1  28.7 KiB   conda  https://conda.anaconda.org/conda-forge/
cuda-nvvm-tools       12.9.86  h4bc722e_2  23.1 MiB   conda  https://conda.anaconda.org/conda-forge/
cuda-version          12.9     h4f385c5_3  21.1 KiB   conda  https://conda.anaconda.org/conda-forge/
```

```console
$ pixi run python -c 'import torch; print(torch.cuda.is_available())'
True
```

:::{tip} Solving CUDA environments without CUDA

What's very powerful about this functionality is that it allows for solving environments with CUDA dependencies when the platform that Pixi is doing the solve on **doesn't have CUDA support**.
This allows for targeting remote machines that will be the execution environment from whatever laptop you have.

**Example on `osx-arm64`**:

```console
% pixi init example && cd example
% pixi workspace system-requirements add cuda 12
% pixi add --platform linux-64 pytorch-gpu 'cuda-version 12.9.*'
% pixi list --platform linux-64 torch
Package      Version  Build                           Size       Kind   Source
libtorch     2.8.0    cuda129_mkl_hc64f9c6_301        836.2 MiB  conda  https://conda.anaconda.org/conda-forge/
pytorch      2.8.0    cuda129_mkl_py313_h392364f_301  24 MiB     conda  https://conda.anaconda.org/conda-forge/
pytorch-gpu  2.8.0    cuda129_mkl_h43a4b0b_301        46.2 KiB   conda  https://conda.anaconda.org/conda-forge/
```
:::

## Linux containerization

As covered in [Reproducible Machine Learning Workflows for Scientists](https://carpentries-incubator.github.io/reproducible-ml-workflows/pixi-deployment.html), Linux containerization can become trivially templated for deployment to remote workers with no shared filesystem or cache.
