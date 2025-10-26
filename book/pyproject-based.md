# Using Pixi with `pyproject.toml` based Python packages

If you initialize a Pixi project in the same directory as a `pyproject.toml`, it will prompt you to create the Pixi workspace as a `[tool.pixi]` subtable in the `pyproject.toml`

```bash
pixi init
```
```
A 'pyproject.toml' file already exists. Do you want to extend it with the '[tool.pixi]' configuration? yes
✔ Added package 'rosen-cpp' as an editable dependency.
```
