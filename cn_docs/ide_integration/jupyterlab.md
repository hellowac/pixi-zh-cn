---
part: pixi/ide_integration
title: JupyterLab Integration
description: 将 JupyterLab 与 pixi 环境结合使用
---

## 基本用法

使用 JupyterLab 和 Pixi 非常简单。
您只需创建一个新的 Pixi 项目并将 `jupyterlab` 包添加到其中即可。
完整的示例可以在以下 [Github 链接](https://github.com/prefix-dev/pixi/tree/main/examples/jupyterlab) 中查看。

```bash
pixi init
pixi add jupyterlab
```

这将创建一个新的 Pixi 项目并将 `jupyterlab` 包添加到其中。然后，您可以使用以下命令启动 JupyterLab：

```bash
pixi run jupyter lab
```

如果您想为 JupyterLab 添加更多“内核”，只需将它们添加到当前项目中，以及您可能需要的任何科学栈依赖。

```bash
pixi add bash_kernel ipywidgets matplotlib numpy pandas  # ...
```

### 可用的内核有哪些？

您可以轻松安装更多 JupyterLab 的“内核”。`conda-forge` 仓库中有许多有趣的附加内核——不仅仅是 Python！

- [**`bash_kernel`**](https://prefix.dev/channels/conda-forge/packages/bash_kernel) 适用于 bash 的内核
- [**`xeus-cpp`**](https://prefix.dev/channels/conda-forge/packages/xeus-cpp) 基于新的 clang-repl 的 C++ 内核
- [**`xeus-cling`**](https://prefix.dev/channels/conda-forge/packages/xeus-cling) 基于稍老的 Cling 的 C++ 内核
- [**`xeus-lua`**](https://prefix.dev/channels/conda-forge/packages/xeus-lua) Lua 内核
- [**`xeus-sql`**](https://prefix.dev/channels/conda-forge/packages/xeus-sql) SQL 内核
- [**`r-irkernel`**](https://prefix.dev/channels/conda-forge/packages/r-irkernel) R 内核

## 高级用法

<!--
以下部分的修改与 https://github.com/renan-r-santos/pixi-kernel 和 https://github.com/renan-r-santos/pixi-kernel-binder 中的 README.md 相关，请确保通过在两个仓库中都提交 PR 来保持同步
-->

如果您希望只运行一个 JupyterLab 实例，但仍希望为每个目录使用 Pixi 环境，可以使用 [**`pixi-kernel`**](https://prefix.dev/channels/conda-forge/packages/pixi-kernel) 包提供的内核之一。

### 配置 JupyterLab

要开始使用，创建一个 Pixi 项目，添加 `jupyterlab` 和 `pixi-kernel`，然后启动 JupyterLab：

```bash
pixi init
pixi add jupyterlab pixi-kernel
pixi run jupyter lab
```

这将启动 JupyterLab 并在您的浏览器中打开它。

![JupyterLab 启动器屏幕，显示 Pixi 内核](https://raw.githubusercontent.com/renan-r-santos/pixi-kernel/main/assets/launch-light.png#only-light)
![JupyterLab 启动器屏幕，显示 Pixi 内核](https://raw.githubusercontent.com/renan-r-santos/pixi-kernel/main/assets/launch-dark.png#only-dark)

`pixi-kernel` 会在您的笔记本或任何父目录中搜索一个清单文件，通常是 `pixi.toml` 或 `pyproject.toml`。当它找到一个文件时，它会使用该文件中指定的环境来启动内核并运行您的笔记本。

### 使用 Binder

如果您只想检查一个在云端运行的 JupyterLab 环境，可以使用 `pixi-kernel`，并访问 [Binder](https://mybinder.org/v2/gh/renan-r-santos/pixi-kernel-binder/main?labpath=example.ipynb)。
