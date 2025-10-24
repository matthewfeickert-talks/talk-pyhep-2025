# Introduction to Pixi basics

### Conceptual overview

[Pixi](https://www.pixi.sh/) is a cross-platform package and environment manager that can handle complex development workflows.
Importantly, Pixi automatically and non-optionally will produce or update a lock file &mdash; a structured file that contains a full list of all environments defined with a complete list of all packages, as well as definition of each packages with digest information on the binary &mdash; for the software environments defined by the user whenever any actions mutate the environment.
Pixi is written in Rust, and leverages the language's speed and technologies to solve environments fast.

Pixi addresses the concept of computational reproducibility by focusing on a set of main features

1. **Virtual environment management**: Pixi can create environments that contain conda packages and Python packages and use or switch between environments easily.
1. **Package management**: Pixi enables the user to install, update, and remove packages from these environments through the `pixi` command line.
1. **Task management**: Pixi has a task runner system built-in, which allows for tasks with custom logic and dependencies on other tasks to be created.

These features become powerful when combined with robust behaviors

1. **Automatic lock files**: Any changes to a Pixi workspace that can mutate the environments defined in it will automatically and non-optionally result in the Pixi lock file for the workspace being updated.
This ensures that any state of a Pixi project is trivially computationally reproducible.
1. **Solving environments for other platforms**: Pixi allows the user to solve environment for platforms other than the current user machine's.
This allows for users to solve and share environment to any collaborator with confidence that all environments will work with no additional setup.
1. **Pairity of conda and Python packages**: Pixi allows for conda packages and Python packages to be used together seamlessly, and is unique in its ability to handle overlap in dependencies between them.
Pixi will first solve all conda package requirements for the target environment, lock the environment, and then solve all the dependencies of the Python packages for the environment, determine if there are any overlaps with the existing conda environment, and the only install the missing Python dependencies.
This ensures allows for fully reproducible solves and for the two package ecosystems to compliment each other rather than potentially cause conflicts.
1. **Efficient caching**: Pixi uses an efficient global cache shared between all Pixi projects and globally installed tools on a machine.
The first time Pixi installs a package it will download the files to the global cache and [link the files into the environment](https://prefix.dev/blog/what-linking-means-when-installing-a-conda-package).
When Pixi has to reinstall the same package in a different environment, the package will be linked from the same cache, making sure internet bandwidth for downloads and disk space is used as efficiently as possible.

## First example project

Let's begin by creating a Pixi project in our `pyhep-2025-pixi-tutorial` Git repository

```bash
cd pyhep-2025-pixi-tutorial
```

with [`pixi init`](https://pixi.sh/latest/reference/cli/pixi/init/)

```bash
pixi init example
```
```
✔ Created <path>/pyhep-2025-pixi-tutorial/example/pixi.toml
```

We see that a directory has been created, so let's navigate to that directory and explore its contents

```bash
cd example
ls -1ap
```
```
./
../
.gitattributes
.gitignore
pixi.toml
```

The most interesting file is the Pixi manifest `pixi.toml`.

```{code} toml
:filename: pixi.toml
:linenos:
[workspace]
authors = ["Name <email>"]
channels = ["conda-forge"]
name = "example"
platforms = ["linux-64"]
version = "0.1.0"

[tasks]

[dependencies]
```

The Pixi manifest is where we declaratively specify the components of software environments that we need and Pixi then resolves into software environments for the specified platforms that can be installed.

### Workspace table

Let's explore the [`[workspace]` table](https://pixi.sh/latest/reference/pixi_manifest/#the-workspace-table) first

```{code} toml
:filename: pixi.toml
:linenos:
:emphasize-lines: 1-6
[workspace]
authors = ["Name <email>"]
channels = ["conda-forge"]
name = "example"
platforms = ["linux-64"]
version = "0.1.0"

[tasks]

[dependencies]
```

The `[workspace]` table defines both metadata for the project as well as information about what computing platforms to get solve the workspace for and where the packages will be installed from.

The required table entries are

* [`channels`](https://pixi.sh/latest/reference/pixi_manifest/#channels): Priority ordered list of conda channels to fetch packages from.

::: {tip} Example

```{code} toml
channels = ["conda-forge", "bioconda", "nvidia"]
```
:::

* [`platforms`](https://pixi.sh/latest/reference/pixi_manifest/#platforms): List of conda recognized computing platforms the workspace supports.
Pixi will resolve the dependencies for all platforms listed.
The entry will be autopopulated with your computer's platform.

::: {tip} Example

```{code} toml
platforms = ["linux-64", "osx-arm64", "win-64"]
```
:::

Given Pixi's use in multiple different computational fields there are over [20 supported platforms](https://docs.rs/rattler_conda_types/latest/rattler_conda_types/platform/enum.Platform.html).

The rest is optional metadata:

* [`authors`](https://pixi.sh/latest/reference/pixi_manifest/#authors-optional): Optional metadata about the author of the project.
* [`name`](https://pixi.sh/latest/reference/pixi_manifest/#name-optional): Name of the workspace.
If the name is not specified, the name of the directory that contains the workspace is used.
* [`version`](https://pixi.sh/latest/reference/pixi_manifest/#version-optional): Version of the workspace

### Dependencies table

The [`[dependencies]` table](https://pixi.sh/latest/reference/pixi_manifest/#the-dependencies-tables) allows for declarative specification of the requirements for our software environment.
As a first example, let's use `pixi add` to specify a dependency on Awkward.

```bash
pixi add awkward
```

```
✔ Added awkward >=2.8.9,<3
```

If we inspect our project directory system we now see additional files and directories have been created

```bash
ls -1ap
```

```
./
../
.gitattributes
.gitignore
.pixi/
pixi.lock
pixi.toml
```

and our Pixi manifest now has its `[dependencies]` table updated

```{code} toml
:filename: pixi.toml
:linenos:
:emphasize-lines: 9-10
[workspace]
channels = ["conda-forge"]
name = "example"
platforms = ["linux-64"]
version = "0.1.0"

[tasks]

[dependencies]
awkward = ">=2.8.9,<3"
```

If we now ask for information about our project with [`pixi info`](https://pixi.sh/latest/advanced/explain_info_command/) we can see that we have also created a "default" environment that has `awkward` as a dependency.

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
          Cache dir: /home/<username>/.cache/rattler/cache
       Auth storage: /home/<username>/.rattler/credentials.json
   Config locations: No config files found

Global
------------
            Bin dir: /home/<username>/.pixi/bin
    Environment dir: /home/<username>/.pixi/envs
       Manifest dir: /home/<username>/.pixi/manifests/pixi-global.toml

Workspace
------------
               Name: example
            Version: 0.1.0
      Manifest file: <path>/example/pixi.toml
       Last updated: 24-10-2025 16:28:44

Environments
------------
        Environment: default
           Features: default
           Channels: conda-forge
   Dependency count: 1
       Dependencies: awkward
   Target platforms: linux-64
    Prefix location: <path>/example/.pixi/envs/default
```

Pixi has also resolved all of Awkward's dependencies as well, which we can see if we list all of the software in the `default` environment with [`pixi list`](https://pixi.sh/latest/reference/cli/pixi/list/)

```bash
pixi list
```

```
Package                  Version    Build                 Size       Kind   Source
_libgcc_mutex            0.1        conda_forge           2.5 KiB    conda  https://conda.anaconda.org/conda-forge/
_openmp_mutex            4.5        2_gnu                 23.1 KiB   conda  https://conda.anaconda.org/conda-forge/
_x86_64-microarch-level  1          2_x86_64              7.6 KiB    conda  https://conda.anaconda.org/conda-forge/
awkward                  2.8.9      pyhcf101f3_0          454.1 KiB  conda  https://conda.anaconda.org/conda-forge/
awkward-cpp              50         py314h4a2b8d0_0       602.2 KiB  conda  https://conda.anaconda.org/conda-forge/
bzip2                    1.0.8      hda65f42_8            254.2 KiB  conda  https://conda.anaconda.org/conda-forge/
ca-certificates          2025.10.5  hbd8a1cb_0            152.3 KiB  conda  https://conda.anaconda.org/conda-forge/
fsspec                   2025.9.0   pyhd8ed1ab_0          142.3 KiB  conda  https://conda.anaconda.org/conda-forge/
importlib-metadata       8.7.0      pyhe01879c_1          33.8 KiB   conda  https://conda.anaconda.org/conda-forge/
ld_impl_linux-64         2.44       h1aa0949_4            725.1 KiB  conda  https://conda.anaconda.org/conda-forge/
libblas                  3.9.0      37_h4a7cf45_openblas  17.1 KiB   conda  https://conda.anaconda.org/conda-forge/
libcblas                 3.9.0      37_h0358290_openblas  17.1 KiB   conda  https://conda.anaconda.org/conda-forge/
libexpat                 2.7.1      hecca717_0            73.1 KiB   conda  https://conda.anaconda.org/conda-forge/
libffi                   3.5.2      h9ec8514_0            56.5 KiB   conda  https://conda.anaconda.org/conda-forge/
libgcc                   15.2.0     h767d61c_7            803.3 KiB  conda  https://conda.anaconda.org/conda-forge/
libgfortran              15.2.0     h69a702a_7            28.6 KiB   conda  https://conda.anaconda.org/conda-forge/
libgfortran5             15.2.0     hcd61629_7            1.5 MiB    conda  https://conda.anaconda.org/conda-forge/
libgomp                  15.2.0     h767d61c_7            437.4 KiB  conda  https://conda.anaconda.org/conda-forge/
liblapack                3.9.0      37_h47877c9_openblas  17.1 KiB   conda  https://conda.anaconda.org/conda-forge/
liblzma                  5.8.1      hb9d3cd8_2            110.2 KiB  conda  https://conda.anaconda.org/conda-forge/
libmpdec                 4.0.0      hb9d3cd8_0            89 KiB     conda  https://conda.anaconda.org/conda-forge/
libopenblas              0.3.30     pthreads_h94d23a6_2   5.7 MiB    conda  https://conda.anaconda.org/conda-forge/
libsqlite                3.50.4     h0c1763c_0            910.7 KiB  conda  https://conda.anaconda.org/conda-forge/
libstdcxx                15.2.0     h8f9b012_7            3.7 MiB    conda  https://conda.anaconda.org/conda-forge/
libuuid                  2.41.2     he9a06e4_0            36.3 KiB   conda  https://conda.anaconda.org/conda-forge/
libzlib                  1.3.1      hb9d3cd8_2            59.5 KiB   conda  https://conda.anaconda.org/conda-forge/
ncurses                  6.5        h2d0b736_3            870.7 KiB  conda  https://conda.anaconda.org/conda-forge/
numpy                    2.3.4      py314h2b28147_0       8.5 MiB    conda  https://conda.anaconda.org/conda-forge/
openssl                  3.5.4      h26f9b46_0            3 MiB      conda  https://conda.anaconda.org/conda-forge/
packaging                25.0       pyh29332c3_1          61 KiB     conda  https://conda.anaconda.org/conda-forge/
python                   3.14.0     h32b2ec7_102_cp314    35 MiB     conda  https://conda.anaconda.org/conda-forge/
python_abi               3.14       8_cp314               6.8 KiB    conda  https://conda.anaconda.org/conda-forge/
readline                 8.2        h8c095d6_2            275.9 KiB  conda  https://conda.anaconda.org/conda-forge/
tk                       8.6.13     noxft_hd72426e_102    3.1 MiB    conda  https://conda.anaconda.org/conda-forge/
typing_extensions        4.15.0     pyhcf101f3_0          50.5 KiB   conda  https://conda.anaconda.org/conda-forge/
tzdata                   2025b      h78e105d_0            120.1 KiB  conda  https://conda.anaconda.org/conda-forge/
zipp                     3.23.0     pyhd8ed1ab_0          22.4 KiB   conda  https://conda.anaconda.org/conda-forge/
zstd                     1.5.7      hb8e6e7a_2            554.3 KiB  conda  https://conda.anaconda.org/conda-forge/
```

We can now use the environment by either running operations inside of it with [`pixi run`](https://pixi.sh/latest/reference/cli/pixi/run/)

```bash
pixi run python
```

```
Python 3.14.0 | packaged by conda-forge | (main, Oct 22 2025, 23:24:08) [GCC 14.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

or by spawning an interactive shell with the `default` environment activated (similar to how `conda activate` works)

```bash
pixi shell
```

```console
(example) $
(example) $ command -v python
<path>/example/.pixi/envs/default/bin/python
(example) $ python --version
Python 3.14.0
(example) $ python -c 'import awkward; print(awkward.__version__)'
2.8.9
```

### Features and environments

Pixi allows for defining multiple different environments in a single workspace.
To do this efficiently, it has the concept of `features` which allow for defining LEGO-like **components** of environments.

```bash
pixi add --feature lab notebook jupyterlab
```

```
✔ Added notebook
✔ Added jupyterlab
Added these only for feature: lab
```

```{code} toml
:filename: pixi.toml
:linenos:
:emphasize-lines: 12-14
[workspace]
channels = ["conda-forge"]
name = "example"
platforms = ["linux-64"]
version = "0.1.0"

[tasks]

[dependencies]
awkward = ">=2.8.9,<3"

[feature.lab.dependencies]
notebook = "*"
jupyterlab = "*"
```

By themselves `features` don't exist in an instantiated space.
They become resolved when they are added to an `environment`.

```bash
pixi workspace environment add --feature lab interactive
```
```
✔ Added environment interactive
```

```{code} toml
:filename: pixi.toml
:linenos:
:emphasize-lines: 16-17
[workspace]
channels = ["conda-forge"]
name = "example"
platforms = ["linux-64"]
version = "0.1.0"

[tasks]

[dependencies]
awkward = ">=2.8.9,<3"

[feature.lab.dependencies]
notebook = "*"
jupyterlab = "*"

[environments]
interactive = ["lab"]
```

We can optionally resolve the `lab` feature to their specific versions by `upgrading` the `feature` dependencies

```bash
pixi upgrade --feature lab
```
```{code} toml
:filename: pixi.toml
:linenos:
:emphasize-lines: 12-14
[workspace]
channels = ["conda-forge"]
name = "example"
platforms = ["linux-64"]
version = "0.1.0"

[tasks]

[dependencies]
awkward = ">=2.8.9,<3"

[feature.lab.dependencies]
notebook = ">=7.4.7,<8"
jupyterlab = ">=4.4.10,<5"

[environments]
interactive = ["lab"]
```

[Unless otherwise specified](https://pixi.sh/latest/tutorials/multi_environment/#non-default-environments), all `environments` build off of the `default` environment, so dependencies of the `default` environment exist in the `interactive` environment as well.

To be able to run a Jupyter Lab instance and interact with Awkward we can run

```bash
pixi run --environment interactive jupyter lab
```

### Tasks table

Pixi has a built in task runner system.
We can use the [`[tasks]` table](https://pixi.sh/latest/reference/pixi_manifest/#the-tasks-table) to define tasks that we would like to execute.

```bash
pixi task add --feature lab --description "Launch Jupyter Lab" lab "jupyter lab"
```
```
✔ Added task `lab`: jupyter lab, description = "Launch Jupyter Lab"
```

::::{tab-set}

:::{tab-item} Pixi default syntax

```{code} toml
:filename: pixi.toml
:linenos:
:emphasize-lines: 16-17
[workspace]
channels = ["conda-forge"]
name = "example"
platforms = ["linux-64"]
version = "0.1.0"

[tasks]

[dependencies]
awkward = ">=2.8.9,<3"

[feature.lab.dependencies]
notebook = ">=7.4.7,<8"
jupyterlab = ">=4.4.10,<5"

[feature.lab.tasks]
lab = { cmd = "jupyter lab", description = "Launch Jupyter Lab" }

[environments]
interactive = ["lab"]
```

:::

:::{tab-item} Subtable syntax

```{code} toml
:filename: pixi.toml
:linenos:
:emphasize-lines: 16-18
[workspace]
channels = ["conda-forge"]
name = "example"
platforms = ["linux-64"]
version = "0.1.0"

[tasks]

[dependencies]
awkward = ">=2.8.9,<3"

[feature.lab.dependencies]
notebook = ">=7.4.7,<8"
jupyterlab = ">=4.4.10,<5"

[feature.lab.tasks.lab]
description = "Launch Jupyter Lab"
cmd = "jupyter lab"

[environments]
interactive = ["lab"]
```

:::

::::

Which now allows us to launch Jupyter Lab in the `interactive` environment by running

```bash
pixi run lab
```

As there is only one `task` named `lab` we don't need to specify the environment it exists in, Pixi is able to determine it automatically.

It is canonical for all Pixi workspaces to have a `start` task so that anyone encountering a Pixi project can explore by running

```bash
pixi run start
```

Let's add one by using the `depends-on` syntax to avoid redefining commands

```bash
pixi task add --feature lab --depends-on="lab" start ""
```
```
✔ Added task `start`: , depends-on = 'lab'
```

::::{tab-set}

:::{tab-item} Pixi default syntax

```{code} toml
:filename: pixi.toml
:linenos:
:emphasize-lines: 18
[workspace]
channels = ["conda-forge"]
name = "example"
platforms = ["linux-64"]
version = "0.1.0"

[tasks]

[dependencies]
awkward = ">=2.8.9,<3"

[feature.lab.dependencies]
notebook = ">=7.4.7,<8"
jupyterlab = ">=4.4.10,<5"

[feature.lab.tasks]
lab = { cmd = "jupyter lab", description = "Launch Jupyter Lab" }
start = [{ task = "lab" }]

[environments]
interactive = ["lab"]
```

:::

:::{tab-item} Subtable syntax

```{code} toml
:filename: pixi.toml
:linenos:
:emphasize-lines: 20-21
[workspace]
channels = ["conda-forge"]
name = "example"
platforms = ["linux-64"]
version = "0.1.0"

[tasks]

[dependencies]
awkward = ">=2.8.9,<3"

[feature.lab.dependencies]
notebook = ">=7.4.7,<8"
jupyterlab = ">=4.4.10,<5"

[feature.lab.tasks.lab]
description = "Launch Jupyter Lab"
cmd = "jupyter lab"

[feature.lab.tasks.start]
depends-on = ["lab"]

[environments]
interactive = ["lab"]
```

:::

::::

### Lock files

Once the workspace has been defined, any Pixi operation on the workspace will result in all environments in the workspace having their dependencies resolved and then fully specified ("locked") at the digest ("hash") level in a single `pixi.lock` Pixi lock file.
This happens **automatically and non-optionally**.
The lock file is a YAML file that contains two definition groups: `environments` and `packages`.
The `environments` group lists every environment in the workspace for every platform with a complete listing of all packages in the environment.
The `packages` group lists a full definition of every package that appears in the `environments` lists, including the package's URL and digests (e.g. sha256, md5).

```yaml
version: 6
environments:
  default:
    channels:
    - url: https://conda.anaconda.org/conda-forge/
    packages:
      linux-64:

...

      - conda: https://conda.anaconda.org/conda-forge/noarch/awkward-2.8.9-pyhcf101f3_0.conda
      - conda: https://conda.anaconda.org/conda-forge/linux-64/awkward-cpp-50-py314h4a2b8d0_0.conda

...

  interactive:
    channels:
    - url: https://conda.anaconda.org/conda-forge/
    packages:
      linux-64:

...

      - conda: https://conda.anaconda.org/conda-forge/noarch/awkward-2.8.9-pyhcf101f3_0.conda
      - conda: https://conda.anaconda.org/conda-forge/linux-64/awkward-cpp-50-py314h4a2b8d0_0.conda


...

packages:

...

- conda: https://conda.anaconda.org/conda-forge/noarch/awkward-2.8.9-pyhcf101f3_0.conda
  sha256: 4399adbde0b672bbb700c9ccbf771242ff22e1f9b3d118bc2c08a208ed7ec0fe
  md5: 542a5edd0954812e536b47a17d774586
  depends:
  - python >=3.10
  - awkward-cpp ==50
  - importlib-metadata >=4.13.0
  - numpy >=1.18.0
  - packaging
  - typing_extensions >=4.1.0
  - fsspec >=2022.11.0
  - python
  license: BSD-3-Clause
  license_family: BSD
  size: 465034
  timestamp: 1758425222640

...

```

These groups provide a full description of every package described in the Pixi workspace and its dependencies and constraints on other packages.
Versioning the lock file along with the manifest file in a version control system allows for workspaces to be fully reproducible to the byte level indefinitely into the future, conditioned on the continued existence of the package indexes the workspace pulls from (e.g. conda-forge, PyPI).
In the event that long term preservation and reproducibility are of importance, there are [community projects](https://github.com/quantco/pixi-pack) that allow for downloading all dependencies of a Pixi environment and generating a tar archive containing all of the packages, which can later be unpacked and installed.

### Multi-platform support

In addition to being multi-environment, Pixi allows for managing multiple platforms as well.
We can add additional platforms to be resolved with `pixi workspace platform`.
Let's add support for x86 Linux, Apple silicon macOS, and x86 Windows machines with

```bash
pixi workspace platform add linux-64 osx-arm64 win-64
```
```
✔ Added linux-64
✔ Added osx-arm64
✔ Added win-64
```

```{code} toml
:filename: pixi.toml
:linenos:
:emphasize-lines: 4
[workspace]
channels = ["conda-forge"]
name = "example"
platforms = ["linux-64", "osx-arm64", "win-64"]
version = "0.1.0"

[tasks]

[dependencies]
awkward = ">=2.8.9,<3"

[feature.lab.dependencies]
notebook = ">=7.4.7,<8"
jupyterlab = ">=4.4.10,<5"

[feature.lab.tasks.start]
depends-on = ["lab"]

[feature.lab.tasks.lab]
description = "Launch Jupyter Lab"
cmd = "jupyter lab"

[environments]
interactive = ["lab"]
```

With that one command Pixi has fully resolved the existing workspace for all platforms, as can be seen by checking `pixi list`

```console
$ pixi list --platform linux-64 awkward
Package      Version  Build            Size       Kind   Source
awkward      2.8.9    pyhcf101f3_0     454.1 KiB  conda  https://conda.anaconda.org/conda-forge/
awkward-cpp  50       py314h4a2b8d0_0  602.2 KiB  conda  https://conda.anaconda.org/conda-forge/
$ pixi list --platform osx-arm64 awkward
Package      Version  Build            Size       Kind   Source
awkward      2.8.9    pyhcf101f3_0     454.1 KiB  conda  https://conda.anaconda.org/conda-forge/
awkward-cpp  50       py314hb7f56de_0  457.9 KiB  conda  https://conda.anaconda.org/conda-forge/
$ pixi list --platform win-64 awkward
Package      Version  Build            Size       Kind   Source
awkward      2.8.9    pyhcf101f3_0     454.1 KiB  conda  https://conda.anaconda.org/conda-forge/
awkward-cpp  50       py314h396c588_0  470.6 KiB  conda  https://conda.anaconda.org/conda-forge/
```

or checking the lock file

```console
$ grep "awkward-cpp" pixi.lock
      - conda: https://conda.anaconda.org/conda-forge/linux-64/awkward-cpp-50-py314h4a2b8d0_0.conda
      - conda: https://conda.anaconda.org/conda-forge/osx-arm64/awkward-cpp-50-py314hb7f56de_0.conda
      - conda: https://conda.anaconda.org/conda-forge/win-64/awkward-cpp-50-py314h396c588_0.conda
      - conda: https://conda.anaconda.org/conda-forge/linux-64/awkward-cpp-50-py314h4a2b8d0_0.conda
      - conda: https://conda.anaconda.org/conda-forge/osx-arm64/awkward-cpp-50-py314hb7f56de_0.conda
      - conda: https://conda.anaconda.org/conda-forge/win-64/awkward-cpp-50-py314h396c588_0.conda
  - awkward-cpp ==50
- conda: https://conda.anaconda.org/conda-forge/linux-64/awkward-cpp-50-py314h4a2b8d0_0.conda
- conda: https://conda.anaconda.org/conda-forge/osx-arm64/awkward-cpp-50-py314hb7f56de_0.conda
  - awkward-cpp-p-0
- conda: https://conda.anaconda.org/conda-forge/win-64/awkward-cpp-50-py314h396c588_0.conda
  - awkward-cpp-p-0
```
