# 从 `conda` 或 `mamba` 迁移到 `pixi`

欢迎使用本指南，帮助你从 `conda` 或 `mamba` 迁移到 `pixi`。
本文将比较这两个工具之间的关键命令和概念，重点介绍 `pixi` 在管理环境和包方面的独特方法。
通过 `pixi`，你将体验到基于项目的工作流，提升开发效率，并轻松分享你的工作。

## 为什么选择 Pixi？

`Pixi` 基于 `conda` 生态系统，提出了一种以项目为中心的方法，而不仅仅是专注于环境。
这种转变使得管理依赖和运行代码变得更加有序和高效，符合现代开发实践。

## 关键区别一览

| 任务                        | Conda/Mamba                                         | Pixi                                                                 |
|-----------------------------|-----------------------------------------------------|----------------------------------------------------------------------|
| 安装                         | 需要安装程序                                         | 下载并添加到 PATH 中（见 [安装指南](../index.md)）                   |
| 创建环境                     | `conda create -n myenv -c conda-forge python=3.8`   | `pixi init myenv` 然后 `pixi add python=3.8`                         |
| 激活环境                     | `conda activate myenv`                              | 在项目目录中使用 `pixi shell`                                         |
| 停用环境                     | `conda deactivate`                                  | 在 `pixi shell` 中使用 `exit`                                        |
| 运行任务                     | `conda run -n myenv python my_program.py`           | `pixi run python my_program.py`（见 [run](../reference/cli.md#run)）   |
| 安装包                       | `conda install numpy`                               | `pixi add numpy`                                                     |
| 卸载包                       | `conda remove numpy`                                | `pixi remove numpy`                                                  |

!!! warn "没有 `base` 环境"
    `Conda` 有一个 `base` 环境，这是你启动新 shell 时的默认环境。
    **Pixi 不使用 `base` 环境**。它要求你在项目或全局安装所需的工具。
    使用 `pixi global install bat` 将会在全局环境中安装 `bat`，这与 `conda` 中的 `base` 环境不同。

??? tip "在当前 shell 中激活 pixi 环境"
    对于一些高级用例，你可以在当前 shell 中激活环境。
    这使用了 `pixi shell-hook`，它会打印激活脚本，你可以用它在当前 shell 中激活环境，而无需使用 `pixi` 本身。
    ```shell
    ~/myenv > eval "$(pixi shell-hook)"
    ```

## 环境与项目

`Conda` 和 `mamba` 专注于环境管理，而 `pixi` 强调项目管理。
在 `pixi` 中，项目是一个包含 [manifest](../reference/project_configuration.md)（`pixi.toml` / `pyproject.toml`）文件的文件夹，描述了项目内容、一个 `pixi.lock` 锁文件，描述了具体的依赖，还有一个 `.pixi` 文件夹，包含了环境。

这种基于项目的方法使得共享和协作变得更加容易，因为项目文件夹包含了所有必要的信息来重建环境。
它支持在一个项目中管理多个环境，适应不同平台，并允许轻松切换。（见 [多个环境](../features/multi_environment.md)）

## 全局环境

`Conda` 将所有环境安装在一个全局位置。
如果你需要这种文件系统结构，可以使用 `pixi` 的 [detached-environments](../reference/pixi_configuration.md#detached-environments) 功能：
```shell
pixi config set detached-environments true
# 或者指定一个位置
pixi config set detached-environments /path/to/envs
```
这将使环境安装在相同的文件夹中，但不会允许你使用 `pixi shell -n` 来激活环境。

`pixi` 有 `pixi global` 命令来安装工具到你的机器上。（见 [global](../reference/cli.md#global)）
这并不是 `conda` 的替代品，但它的工作方式类似于 [`pipx`](https://pipx.pypa.io/stable/) 和 [`condax`](https://mariusvniekerk.github.io/condax/)。
它为给定的需求创建一个单独的隔离环境，并将二进制文件安装到全局路径中。
```shell
pixi global install bat
bat pixi.toml
```

!!! warn "永远不要用 `pixi global` 安装 `pip`"
    使用 `pixi global` 安装的工具将创建一个单独的隔离环境。
    用 `pixi global` 安装 `pip` 会在该隔离环境中创建一个新的 `pip` 二进制文件。
    使用这个 `pip` 二进制文件会在该环境中安装包，导致这些包无法在其他地方访问，因为你无法激活它。

## 自动切换

通过 `pixi`，你可以将 `environment.yml` 文件导入到 `pixi` 项目中。（见 [import](../reference/cli.md#init)）
```shell
pixi init --import environment.yml
```
这将基于 `environment.yml` 文件中的依赖创建一个新项目。

??? tip "导出你的环境"

    如果你与 Conda 用户或系统一起工作，可以将环境 [导出为 `environment.yml`](../reference/cli.md#project-export-conda_environment) 文件来分享：
    
    ```shell
    pixi project export conda
    ```

    你也可以导出 [conda 显式规范](../reference/cli.md#project-export-conda_explicit_spec)。

## 故障排除

在迁移到 `pixi` 时，可能会遇到一些问题，以下是一些常见问题的解决方案：

- 依赖 `is excluded because due to strict channel priority not using this option from: 'https://conda.anaconda.org/conda-forge/'`
  这个错误发生在包存在于多个频道时。`pixi` 使用严格的频道优先级。了解更多信息，见 [频道优先级](../advanced/channel_priority.md)。
- `pixi global install pip`，`pip` 无法工作。
  `pip` 安装在全局隔离环境中。请在项目中使用 `pixi add pip` 安装 `pip`，并使用该项目环境。
- `pixi global install <任何库>` -> `import <任何库>` -> `ModuleNotFoundError: No module named '<任何库>'`
  该库安装在全局隔离环境中。请在项目中使用 `pixi add <任何库>` 安装该库，并使用该项目环境。