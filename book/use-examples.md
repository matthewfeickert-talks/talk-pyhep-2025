# Various use examples

## Install ROOT globally from conda-forge with a `pyroot` alias

```bash
pixi global install root
pixi global expose add --environment root pyroot=python
```

```console
$ root --version
ROOT Version: 6.36.04
Built for linuxx8664gcc on Oct 02 2025, 08:33:44
From tags/6-36-04@6-36-04
$ command -v root
/home/<user>/.pixi/bin/root
$ pyroot
Python 3.13.9 | packaged by conda-forge | (main, Oct 16 2025, 10:31:39) [GCC 14.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import ROOT
>>> ROOT
<module 'ROOT' from '/home/<user>/.pixi/envs/root/lib/python3.13/site-packages/ROOT/__init__.py'>
>>>
```

## Create fully reproducible bespoke environments

```bash
pixi init example && cd example
pixi add coffea root sherpa mg5amcnlo
```

```
...
✔ Added coffea >=2025.10.2,<2026
✔ Added root >=6.36.4,<7
✔ Added sherpa >=2.2.16,<3
✔ Added mg5amcnlo >=3.5.7,<4
```

```console
$ pixi list | wc -l
477
```
