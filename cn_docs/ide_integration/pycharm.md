---
part: pixi/ide_integration
title: PyCharm Integration
description: 将 PyCharm 与 pixi 环境结合使用
---

<!--
对本文件的修改与 https://github.com/pavelzw/pixi-pycharm 中的 README.md 相关，请通过在两个仓库中都提交 PR 来保持同步
-->

您可以通过使用 [pixi-pycharm](https://github.com/pavelzw/pixi-pycharm) 包提供的 `conda` Shim 来在 PyCharm 中使用 Pixi 环境。

## 使用方法

要开始使用，请将 `pixi-pycharm` 添加到您的 Pixi 项目中。

```bash
pixi add pixi-pycharm
```

这将确保 `conda` shim 安装到您项目的环境中。

安装了 `pixi-pycharm` 后，您可以配置 PyCharm 使用您的 Pixi 环境。
打开 PyCharm 的 _添加 Python 解释器_ 对话框（位于 PyCharm 窗口右下角），然后选择 _Conda 环境_。
将 _Conda 可执行文件_ 设置为 `conda` 文件的完整路径（在 Windows 上为 `conda.bat`），该文件位于 `.pixi/envs/default/libexec` 目录中。
您可以使用以下命令获取该路径：

=== "Linux & macOS"
    ```bash
    pixi run 'echo $CONDA_PREFIX/libexec/conda'
    ```

=== "Windows"
    ```bash
    pixi run 'echo $CONDA_PREFIX\\libexec\\conda.bat'
    ```

这是一个可执行文件，会使 PyCharm 认为它是正确的 `conda` 可执行文件。
在幕后，它会将所有调用重定向到相应的 `pixi` 等效项。

!!!warning "使用来自 Pixi 项目的 Conda Shim"
    请确保使用的是来自此 Pixi 项目的 `conda` shim，而不是其他项目的版本。
    如果您使用多个 Pixi 项目，可能需要相应调整路径，因为 PyCharm 会记住 `conda` 可执行文件的路径。

![添加 Python 解释器](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/add-conda-environment-light.png#only-light)
![添加 Python 解释器](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/add-conda-environment-dark.png#only-dark)

选择环境后，PyCharm 将使用来自您的 Pixi 环境的 Python 解释器。

PyCharm 现在应该能够显示您已安装的包。

![PyCharm 包列表](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/dependency-list-light.png#only-light)
![PyCharm 包列表](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/dependency-list-dark.png#only-dark)

现在，您可以像往常一样运行您的程序和测试。

![PyCharm 运行测试](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/tests-light.png#only-light)
![PyCharm 运行测试](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/tests-dark.png#only-dark)

!!! tip "将 `.pixi` 目录标记为排除"
    为了避免 PyCharm 因 `.pixi` 目录产生混淆，请将其标记为排除。

    ![标记目录为排除 1](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/mark-directory-as-excluded-1-light.png#only-light)
    ![标记目录为排除 1](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/mark-directory-as-excluded-1-dark.png#only-dark)
    ![标记目录为排除 2](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/mark-directory-as-excluded-2-light.png#only-light)
    ![标记目录为排除 2](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/mark-directory-as-excluded-2-dark.png#only-dark)

    此外，当使用远程解释器时，您应在远程机器上排除 `.pixi` 目录。
    代替此操作，您应该在远程机器上运行 `pixi install` 并从那里选择 `conda` shim。
    ![远程机器上排除部署](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/deployment-exclude-pixi-light.png#only-light)
    ![远程机器上排除部署](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/deployment-exclude-pixi-dark.png#only-dark)

### 多个环境

如果您的项目使用 [多个环境](../features/multi_environment.md) 来测试不同的 Python 版本或依赖项，您可以通过在 _添加 Python 解释器_ 对话框中选择 _使用现有环境_ 来将多个环境添加到 PyCharm。

![多个 Pixi 环境](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/python-interpreters-multi-env-light.png#only-light)
![多个 Pixi 环境](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/python-interpreters-multi-env-dark.png#only-dark)

然后，您可以在 PyCharm 窗口的右下角选择相应的环境。

![指定环境](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/specify-interpreter-light.png#only-light)
![指定环境](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/specify-interpreter-dark.png#only-dark)

### 多个 Pixi 项目

当使用多个 Pixi 项目时，请记住为每个项目选择正确的 _Conda 可执行文件_，如上所述。
如果遇到多个环境具有相同名称的情况，可以将环境重命名为唯一名称。

![多个默认环境](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/multiple-default-envs-light.png#only-light)
![多个默认环境](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/multiple-default-envs-dark.png#only-dark)

建议将环境重命名为独一无二的名称。

## 调试

日志写入 `~/.cache/pixi-pycharm.log`。
您可以使用这些日志来调试问题。
请在 [提交 bug 报告](https://github.com/pavelzw/pixi-pycharm/issues/new?template=bug-report.md) 时附上日志。

## 安装为可选依赖项

在某些情况下，您可能只想在本地开发机器上安装 `pixi-pycharm`，但不想在生产环境中安装。
为此，您可以使用 [多个环境](../features/multi_environment.md)。

```toml
[project]
name = "multi-env"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["numpy"]

[tool.pixi.project]
channels = ["conda-forge"]
platforms = ["linux-64"]

[tool.pixi.feature.lint.dependencies]
ruff =  "*"

[tool.pixi.feature.dev.dependencies]
pixi-pycharm = "*"

[tool.pixi.environments]
# 生产环境是默认功能集。
# 添加求解组以确保 `default` 和 `prod` 环境中使用相同的版本。
prod = { solve-group = "main" }

# 设置默认环境以包含开发功能。
# 通过使用 `default` 而不是 `dev`，您无需在运行 `pixi run` 时指定 `--environment` 标志。
default = { features = ["dev"], solve-group = "main" }

# lint 环境不需要默认功能集，只需要 `lint` 功能，
# 因此也可以从求解组中排除。
lint = { features = ["lint"], no-default-feature = true }
```

现在，您作为用户可以运行 `pixi shell`，这将启动默认环境。
在生产环境中，您只需运行 `pixi run -e prod COMMAND`，即可安装最小的生产环境。
