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

## Fist example workspace

Let's begin by creating a Pixi
