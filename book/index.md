# [Reproducible reuse by default: Use of Pixi for software in (Py)HEP](https://indico.cern.ch/event/1566263/contributions/6733144/)

Presented at PyHEP 2025 workshop on Monday October 27th, 2025.

## Abstract

While advancements in software development practices across particle physics and adoption of Linux container technology have made substantial impact in the ease of replicability and reuse of analysis software stacks, the underlying software environments are still primarily bespoke builds that lack a full manifest to ensure reproducibility across time.
Pixi is a new technology supporting the conda and Python packaging communities that allows for the declarative specification of dependencies across multi-platforms and automatic creation of fully specified and portable scientific computing environments.
This applies to the Python ecosystem, hardware accelerated software, and the broader HEP and scientific open source ecosystems as well.

This talk will be structured as a practical hands on tutorial that will explore relevant use cases for the PyHEP community as well as provide participants with their own example repository for future reference.

## Rough Outline (50 minutes: 45 talk + 5 questions)

**00:00 &ndash; 00:05 (5 min):**
* Install Pixi on your machine and get a repository setup.

**00:05 &ndash; 00:15 (10 min):**
* Walk through Pixi manifest
   - Explain `workspace` table
   - Show `features`
   - Show `environments`

**00:15 &ndash; 00:20 (5 min):**
* Explain lock files

**00:20 &ndash; 00:25 (5 min):**
* Explain adding Python packages
   - Show adding from PyPI
   - Show adding local
   - Show adding from remote

**00:25 &ndash; 00:30 (5 min):**
* Show adding to `pyproject.toml` with minimal example project

**00:30 &ndash; 00:40 (10 min):**
* Show introduction to CUDA packages with

**00:40 &ndash; 00:45 (5 min):**
* Show Pixi build
