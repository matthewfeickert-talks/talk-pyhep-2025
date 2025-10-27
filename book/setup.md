# Setup

## Software

This tutorial requires minimal software to be installed in advance:

* A computer running an 64 bit version of Linux, macOS, or Windows.
* [Pixi](https://pixi.sh/)
* [Git](https://git-scm.com/)
   - If you are not already familiar with using `git` and GitHub, please install [`gh`](https://cli.github.com/) as well to simplify the workflow.
* A text editor or IDE of your choice.

### Web Platforms (optional, but encouraged)

* [GitHub](https://github.com/)

For this tutorial we would like you to create your own Git repository where you add the results of your work as you move through the tutorial so that you have a sharable form of what you have learned by the end.
It doesn't _need_ to be GitHub (GitLab.com or some other alternative exist) but for the sake of consistency, the instructions will assume you are using GitHub.

::: {attention} GitHub mandatory two-factor authentication

As [GitHub requires two-factor authentication](https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/about-mandatory-two-factor-authentication), it is highly recommended that you [generate an SSH key pair](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) specifically for GitHub, [add the generated SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account), and then use your SSH keys to connect with GitHub.

:::

## Installation

(install-pixi)=
### Pixi

To install Pixi follow [the installation instructions](https://pixi.sh/latest/#installation) for your particular machine and then restart your shell.

::::{tab-set}

:::{tab-item} Unix (Linux and macOS) and Windows Terminal
:sync: tab1

```bash
curl -fsSL https://pixi.sh/install.sh | sh
```

:::

:::{tab-item} Windows PowerShell
:sync: tab2

```powershell
powershell -ExecutionPolicy ByPass -c "irm -useb https://pixi.sh/install.ps1 | iex"
```

:::

::::

:::{caution} Don't use Homebrew

It is strongly recommended to **not** use Homebrew to install Pixi, as this breaks all of Pixi's self-update features.
Please use one of the provided solutions above.

:::

#### Pixi Shell completions

Additionally, install the [Pixi shell completions](https://pixi.sh/latest/advanced/installation/#autocompletion) for your particular shell choice.

### Git

You probably already have Git installed on your machine.
You can check with

```bash
command -v git
```

If the command doesn't return a filepath to the `git` executable, first make sure you have [Pixi installed](#install-pixi), as described above, and then run

```bash
pixi global install git
```

You can now use the Git anywhere on your machine.

### GitHub CLI (`gh`)
If you are not already familiar with `git` and GitHub, we recommend installing the [GitHub CLI](https://cli.github.com/) to simplify the workflow.
To install the GitHub CLI, first make sure you have [Pixi installed](#install-pixi), as described above, and then run

```bash
pixi global install gh
```

Then log in to your GitHub account with

```bash
gh auth login
```
You can now use the GitHub CLI anywhere on your machine.

## Setup a Personal GitHub Repository

To share your work from this tutorial, we will create a GitHub repository to store your code and results.
This will allow you to easily share your work with others and keep track of your progress, or share it with the Brev instance.

1. Create a personal [GitHub account](https://github.com/) _if you don’t have one yet_.
1. Add a new repository to your account through this link: [Create a new repository](https://github.com/new).
1. Name the new repository `pyhep-2025-pixi-tutorial`, make it public, and give it a README and an [open source license](https://docs.github.com/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/licensing-a-repository) (e.g. MIT License).

To streamline this you can optionally use the [GitHub CLI](https://cli.github.com/) to create the repository.

Feel free to change any of the options below to suit your needs, but the following command will create a new public repository with a README, a Python [`.gitignore`](https://docs.github.com/en/get-started/git-basics/ignoring-files), and an MIT [license](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/licensing-a-repository):

```bash
gh repo create pyhep-2025-pixi-tutorial \
   --public \
   --description "PyHEP 2025 Pixi tutorial" \
   --add-readme \
   --gitignore Python \
   --license MIT \
   --clone
```

Because of the `--clone` option, this will also clone the newly created repository to your local machine.
Now you can navigate to the newly created repository directory:

```bash
cd pyhep-2025-pixi-tutorial
```

Now you have a GitHub repository set up to store your work from this tutorial.
