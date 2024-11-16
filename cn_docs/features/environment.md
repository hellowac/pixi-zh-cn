---
part: pixi
title: Environment
description: The resulting environment of a pixi installation.
---

# 环境

Pixi 是一个用于管理虚拟环境的工具。
本文档解释了环境的结构以及如何使用它。

## 结构

默认情况下，pixi 环境位于项目的 `.pixi/envs` 目录中。
这将使您的机器和项目彼此保持干净和隔离，并且在项目完成后易于清理。
虽然这种结构通常是推荐的，但通过启用 [分离环境](../reference/pixi_configuration.md#detached-environments)，环境也可以存储在项目目录之外。

如果您查看 `.pixi/envs` 目录，您将看到每个环境都有一个目录，`default` 是通常使用的环境，如果指定了自定义环境，则会使用您指定的名称。

```shell
.pixi
└── envs
    ├── cuda
    │   ├── bin
    │   ├── conda-meta
    │   ├── etc
    │   ├── include
    │   ├── lib
    │   ...
    └── default
        ├── bin
        ├── conda-meta
        ├── etc
        ├── include
        ├── lib
        ...
```

这些目录是 conda 环境，您可以像使用 conda 环境一样使用它们，但不能手动编辑它们，所有操作应该通过 `pixi.toml` 文件进行。
Pixi 会始终确保环境与 `pixi.lock` 文件同步。
如果不是这种情况，所有使用该环境的命令将自动更新环境，例如 `pixi run`、`pixi shell`。

<span id="environment-installation-metadata"></span>

### 环境安装元数据


在安装环境时，pixi 会在环境中写入一个小文件，包含有关安装的元数据。
该文件名为 `pixi`，位于环境的 `conda-meta` 文件夹中。
该文件包含以下信息：

- `manifest_path`: 描述用于创建此环境的项目的 manifest 文件路径
- `environment_name`: 环境的名称
- `pixi_version`: 用于创建此环境的 pixi 版本
- `environment_lock_file_hash`: 用于创建此环境的 `pixi.lock` 文件的哈希值

```json
{
  "manifest_path": "/home/user/dev/pixi/pixi.toml",
  "environment_name": "default",
  "pixi_version": "0.34.0",
  "environment_lock_file_hash": "4f36ee620f10329d"
}
```

`environment_lock_file_hash` 用于检查环境是否与 `pixi.lock` 文件同步。
如果 `pixi.lock` 文件的哈希值与 `pixi` 文件中的哈希值不同，pixi 会更新环境。

这用于加速激活，若要触发完全重新验证，可以在 `pixi run` 或 `pixi shell` 命令中加上 `--revalidate` 标志。
通过哈希比较，通常无法发现环境的损坏，但重新验证会重新安装环境。
默认情况下，所有修改锁文件的命令都会使用重新验证，`pixi install` 始终会重新验证。

### 清理

如果您想清理环境，您可以简单地删除 `.pixi/envs` 目录，pixi 会在需要时重新创建环境。

```shell
# 或者：
rm -rf .pixi/envs

# 或者按环境删除：
rm -rf .pixi/envs/default
rm -rf .pixi/envs/cuda
```

## 激活

环境不过是安装到某个位置的一组文件，某种程度上类似于全局系统安装。
您需要激活环境才能使用它。
最简单的方式是将环境的 `bin` 目录添加到 `PATH` 变量中。
但是，在 conda 环境中，激活还有更多内容，因为它还设置了一些环境变量。

我们有多种方式来进行激活：

- 使用 `pixi shell` 命令打开一个激活了环境的 shell。
- 使用 `pixi shell-hook` 命令打印激活当前 shell 中环境的命令。
- 使用 `pixi run` 命令在环境中运行命令。

其中，`run` 命令非常特殊，因为它运行自己的跨平台 shell，并且能够运行任务。
有关任务的更多信息，请参见 [任务文档](advanced_tasks.md)。

使用 `pixi shell-hook` 时，您将看到以下输出：

```shell
export PATH="/home/user/development/pixi/.pixi/envs/default/bin:/home/user/.local/bin:/home/user/bin:/usr/local/bin:/usr/local/sbin:/usr/bin:/home/user/.pixi/bin"
export CONDA_PREFIX="/home/user/development/pixi/.pixi/envs/default"
export PIXI_PROJECT_NAME="pixi"
export PIXI_PROJECT_ROOT="/home/user/development/pixi"
export PIXI_PROJECT_VERSION="0.12.0"
export PIXI_PROJECT_MANIFEST="/home/user/development/pixi/pixi.toml"
export CONDA_DEFAULT_ENV="pixi"
export PIXI_ENVIRONMENT_PLATFORMS="osx-64,linux-64,win-64,osx-arm64"
export PIXI_ENVIRONMENT_NAME="default"
export PIXI_PROMPT="(pixi) "
. "/home/user/development/pixi/.pixi/envs/default/etc/conda/activate.d/activate-binutils_linux-64.sh"
. "/home/user/development/pixi/.pixi/envs/default/etc/conda/activate.d/activate-gcc_linux-64.sh"
. "/home/user/development/pixi/.pixi/envs/default/etc/conda/activate.d/activate-gfortran_linux-64.sh"
. "/home/user/development/pixi/.pixi/envs/default/etc/conda/activate.d/activate-gxx_linux-64.sh"
. "/home/user/development/pixi/.pixi/envs/default/etc/conda/activate.d/libglib_activate.sh"
. "/home/user/development/pixi/.pixi/envs/default/etc/conda/activate.d/rust.sh"
```

它设置了 `PATH` 和其他一些环境变量。更重要的是，它还运行了由已安装的软件包提供的激活脚本。
例如 [`libglib_activate.sh`](https://github.com/conda-forge/glib-feedstock/blob/52ba1944dffdb2d882d824d6548325155b58819b/recipe/scripts/activate.sh) 脚本。
因此，单纯将 `bin` 目录添加到 `PATH` 是不够的。

## 传统的 `conda activate` 类似激活方式

如果你更倾向于使用传统的 `conda activate` 类似的激活方式，可以使用 `pixi shell-hook` 命令。

```shell
$ which python
python not found
$ eval "$(pixi shell-hook)"
$ (default) which python
/path/to/project/.pixi/envs/default/bin/python
```

!!! warning
    不建议使用传统的 `conda activate` 类似激活方式，因为无法真正停用环境。请改用 `pixi shell`。

### 使用 `pixi` 配合 `direnv`

??? note "安装 direnv"

    当然，你可以使用 `pixi` 来全局安装 `direnv`。我们推荐运行

    ```
    pixi global install direnv
    ```

    来在你的计算机上安装最新版本的 `direnv`。

这使你可以将 `pixi` 和 `direnv` 结合使用。
在你的 `.envrc` 文件中输入以下内容：

```shell title=".envrc"
watch_file pixi.lock # (1)!
eval "$(pixi shell-hook)" # (2)!
```

1. 这确保每次 `pixi.lock` 文件发生变化时，`direnv` 会再次调用 shell-hook。
2. 这会在需要时安装并激活环境。`direnv` 确保当你离开目录时，环境会被停用。

```shell
$ cd my-project
direnv: error /my-project/.envrc is blocked. Run `direnv allow` to approve its content
$ direnv allow
direnv: loading /my-project/.envrc
✔ Project in /my-project is ready to use!
direnv: export +CONDA_DEFAULT_ENV +CONDA_PREFIX +PIXI_ENVIRONMENT_NAME +PIXI_ENVIRONMENT_PLATFORMS +PIXI_PROJECT_MANIFEST +PIXI_PROJECT_NAME +PIXI_PROJECT_ROOT +PIXI_PROJECT_VERSION +PIXI_PROMPT ~PATH
$ which python
/my-project/.pixi/envs/default/bin/python
$ cd ..
direnv: unloading
$ which python
python not found
```

## 环境变量

在使用 `pixi run`、`pixi shell` 或 `pixi shell-hook` 命令时，pixi 会设置以下环境变量：

- `PIXI_PROJECT_ROOT`: 项目的根目录。
- `PIXI_PROJECT_NAME`: 项目的名称。
- `PIXI_PROJECT_MANIFEST`: 清单文件 (`pixi.toml`) 的路径。
- `PIXI_PROJECT_VERSION`: 项目的版本。
- `PIXI_PROMPT`: 在 shell 中使用的提示符，也由 `pixi shell` 使用。
- `PIXI_ENVIRONMENT_NAME`: 环境的名称，默认为 `default`。
- `PIXI_ENVIRONMENT_PLATFORMS`: 项目支持的多平台列表，以逗号分隔。
- `CONDA_PREFIX`: 环境的路径。（多个已经支持 conda 环境的工具会使用该路径）
- `CONDA_DEFAULT_ENV`: 环境的名称。（多个已经支持 conda 环境的工具会使用该名称）
- `PATH`: 我们将环境的 `bin` 目录添加到 `PATH` 变量中，以便你可以直接使用环境中安装的工具。
- `INIT_CWD`: 仅在 `pixi run` 中：命令运行时所在的目录。

!!! note
    尽管这些是环境变量，但无法覆盖它们。例如，你无法通过设置 `PIXI_PROJECT_ROOT` 来更改项目的根目录。

## 求解环境

当你运行一个使用环境的命令时，pixi 会检查环境是否与 `pixi.lock` 文件同步。
如果不同步，pixi 会求解该环境并更新它。
这意味着 pixi 会根据你在 `pixi.toml` 中指定的依赖要求，检索最佳的包集，并将求解步骤的输出放入 `pixi.lock` 文件中。
求解是一个数学问题，可能需要一些时间，但我们对求解环境的方式非常自豪，并且有信心在合理的时间内解决你的环境问题。
如果你想了解更多关于求解过程的信息，可以阅读以下内容：

- [Rattler(conda) 求解器博客](https://prefix.dev/blog/the_new_rattler_resolver)
- [UV(PyPI) 求解器博客](https://astral.sh/blog/uv-unified-python-packaging)

Pixi 同时求解 `conda` 和 `PyPI` 依赖，其中 `PyPI` 依赖以 conda 包为基础，因此你可以确保包之间是兼容的。
这些求解器被分为 [`rattler`](https://github.com/mamba-org/rattler) 和 [`uv`](https://github.com/astral-sh/uv) 库，这些库负责求解过程中的繁重工作，实际求解由我们的自定义 SAT 求解器 [`resolvo`](https://github.com/mamba-org/resolvo) 执行。
`resolve` 能够解决多个生态系统，如 `conda` 和 `PyPI`。它实现了 `PyPI` 包的懒加载求解过程，这意味着它只会下载解决环境所需的包的元数据。
它还支持 `conda` 的求解方式，即一次性下载所有包的元数据，然后一次性完成求解。

对于 `[pypi-dependencies]`，`uv` 实现了 `sdist` 构建以检索包的元数据，并通过 `wheel` 构建安装这些包。
对于这个构建步骤，`pixi` 需要首先在 `pixi.toml` 文件的 `(conda)` `[dependencies]` 部分安装 `python`。
这将总比纯粹的 conda 求解慢。因此，为了获得最佳的 pixi 使用体验，建议你保持在 `pixi.toml` 文件的 `[dependencies]` 部分内。

<span id="caching-packages"> </span>

## 缓存包

Pixi 会将所有之前下载的包缓存到一个缓存文件夹中。
这个缓存文件夹在所有 pixi 项目和全局安装的工具之间共享。

通常，缓存文件夹的位置会是以下平台特定的默认缓存文件夹：

- Linux: `$XDG_CACHE_HOME/rattler` 或 `$HOME/.cache/rattler`
- macOS: `$HOME/Library/Caches/rattler`
- Windows: `%LOCALAPPDATA%\rattler`

你可以通过设置 `PIXI_CACHE_DIR` 或 `RATTLER_CACHE_DIR` 环境变量来配置此位置。

当你想清理缓存时，可以简单地删除缓存目录，pixi 会在需要时重新创建缓存。

缓存包含多个文件夹，涉及 pixi 内部的不同缓存。

- `pkgs`: 包含下载/解包后的 `conda` 包。
- `repodata`: 包含 `conda` 的 repodata 缓存。
- `uv-cache`: 包含 `uv` 缓存。包括多个缓存，例如 `built-wheels`、`wheels`、`archives` 等。
- `http-cache`: 包含 `conda-pypi` 映射缓存。
