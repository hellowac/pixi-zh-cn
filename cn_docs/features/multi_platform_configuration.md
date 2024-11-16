---
part: pixi/advanced
title: Multi platform config
description: Learn how to set up for different platforms/OS's
---

[Pixi 的愿景](../vision.md)包括支持所有主要平台。有时，这需要一些额外的配置才能正常工作。
在本页面中，您将了解可以配置哪些内容，以便更好地与您正在为其开发应用程序的平台对齐。

以下是一个示例清单文件，突出显示了一些功能：

=== "`pixi.toml`"

    ```toml title="pixi.toml"
    [project]
    # 默认项目信息....
    # 支持的平台列表。
    platforms = ["win-64", "linux-64", "osx-64", "osx-arm64"]

    [dependencies]
    python = ">=3.8"

    [target.win-64.dependencies]
    # 仅在 win-64 上覆盖所需的 Python 版本
    python = "3.7"


    [activation]
    scripts = ["setup.sh"]

    [target.win-64.activation]
    # 仅为 Windows 覆盖激活脚本
    scripts = ["setup.bat"]
    ```

=== "`pyproject.toml`"

    ```toml title="pyproject.toml"
    [tool.pixi.project]
    # 默认项目信息....
    # 支持的平台列表。
    platforms = ["win-64", "linux-64", "osx-64", "osx-arm64"]

    [tool.pixi.dependencies]
    python = ">=3.8"

    [tool.pixi.target.win-64.dependencies]
    # 仅在 win-64 上覆盖所需的 Python 版本
    python = "~=3.7.0"


    [tool.pixi.activation]
    scripts = ["setup.sh"]

    [tool.pixi.target.win-64.activation]
    # 仅为 Windows 覆盖激活脚本
    scripts = ["setup.bat"]
    ```

## 平台定义

`project.platforms` 定义了您的项目支持的平台。
当定义了多个平台时，pixi 会为每个平台分别确定要安装的依赖项。
所有这些都存储在一个锁文件中。

在未配置的平台上运行 `pixi install` 时，系统会警告用户该平台未配置：

```shell
❯ pixi install
  × the project is not configured for your current platform
   ╭─[pixi.toml:6:1]
 6 │ channels = ["conda-forge"]
 7 │ platforms = ["osx-64", "osx-arm64", "win-64"]
   ·             ────────────────┬────────────────
   ·                             ╰── add 'linux-64' here
 8 │
   ╰────
  help: The project needs to be configured to support your platform (linux-64).
```

## 目标指定符

使用目标指定符，您可以专门为单个平台覆盖原始配置。
如果您在目标指定符中指定了一个未在 `project.platforms` 中列出的特定平台，pixi 将抛出错误。

### 依赖项

有时您可能只希望在特定平台上安装某个依赖项，或者在不同平台上使用不同版本的依赖项。

```toml title="pixi.toml"
[dependencies]
python = ">=3.8"

[target.win-64.dependencies]
msmpi = "*"
python = "3.8"
```

在上面的示例中，我们指定了仅在 Windows 上依赖 `msmpi`。
我们还希望在 Windows 上安装时使用 `3.8` 版本的 `python`。
这将覆盖通用依赖项集中的依赖项。
它不会影响其他平台。

您可以使用 pixi 的命令行工具将这些依赖项添加到清单文件中。

```shell
pixi add --platform win-64 posix
```

这也适用于 `host` 和 `build` 依赖项。

```bash
pixi add --host --platform win-64 posix
pixi add --build --platform osx-64 clang
```

结果如下：

```toml title="pixi.toml"
[target.win-64.host-dependencies]
posix = "1.0.0.*"

[target.osx-64.build-dependencies]
clang = "16.0.6.*"
```

<span id="activation"></span>

### 激活

Pixi 的愿景是实现完全跨平台的项目，但您通常需要运行不是由您的项目构建的工具。
生成的激活脚本通常属于此类，Unix 系统上的默认脚本是 `bash`，而 Windows 系统上的默认脚本是 `bat`。

为了解决这个问题，您可以使用目标定义来定义您的激活脚本。

```toml title="pixi.toml"
[activation]
scripts = ["setup.sh", "local_setup.bash"]

[target.win-64.activation]
scripts = ["setup.bat", "local_setup.bat"]
```

当该项目在 `win-64` 上运行时，它将仅执行目标脚本，而不会执行默认 `activation.scripts` 中指定的脚本。
