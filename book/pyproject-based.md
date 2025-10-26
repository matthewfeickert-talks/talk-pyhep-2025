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
...

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64"]

[tool.pixi.pypi-dependencies]
rosen-cpp = { path = ".", editable = true }

[tool.pixi.tasks]
```
