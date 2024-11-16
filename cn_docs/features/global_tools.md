# Pixi Global

<div style="text-align:center">
 <video autoplay muted loop>
  <source src="https://github.com/user-attachments/assets/e94dc06f-75ae-4aa0-8830-7cb190d3f367" type="video/webm" />
  <p>Pixi global demo</p>
 </video>
</div>

通过 `pixi global`，用户可以管理全局安装的工具，使它们可以从任何目录中访问。
这意味着 pixi 环境将被放置在一个全局位置，并且工具将暴露到系统的 `PATH` 中，使你可以从命令行运行它们。

## 基本用法

运行以下命令将 [`rattler-build`](https://prefix-dev.github.io/rattler-build/latest/) 安装到你的系统中。

```bash
pixi global install rattler-build
```

`pixi global` 的一个优点是，默认情况下，它会将每个包隔离在自己的环境中，仅暴露必要的入口点。
这意味着你不必担心删除某个包时意外破坏看似无关的包。
这种行为与 [`pipx`](https://pipx.pypa.io/latest/installation/) 非常相似。

但是，有时你可能希望将多个依赖项放在同一个环境中。
例如，`ipython` 本身非常有用，但在使用时如果能够同时提供 `numpy` 和 `matplotlib`，它将变得更有用。

执行以下命令：

```bash
pixi global install ipython --with numpy --with matplotlib
```

`numpy` 曝露了可执行文件，但由于它是通过 `--with` 添加的，它的可执行文件并不会被暴露。

现在，导入 `numpy` 和 `matplotlib` 将按预期工作。

```bash
ipython -c 'import numpy; import matplotlib'
```

有时你可能想在系统上安装多个版本的同一个包。
由于它们都会在系统的 `PATH` 中可用，因此需要使用不同的名称暴露它们。

看看以下命令：
```bash
pixi global install --expose py3=python "python=3.12"
```

通过指定 `--expose`，我们指定希望将可执行文件 `python` 暴露为 `py3`。
包 `python` 还包含更多可执行文件，但由于我们指定了 `--expose`，它们不会被自动暴露。

你可以运行 `py3` 来启动 Python 解释器。

```shell
py3 -c "print('Hello World')"
```

## 全局清单

自 `v0.33.0` 版本以来，pixi 引入了一个新的清单文件，它将被创建在全局目录中。
此文件将包含安装在全局环境中的环境列表、它们的依赖项和暴露的二进制文件。
该清单可以编辑、同步、提交到版本控制系统，并与其他人共享。

从前面部分执行的命令将生成如下的清单：

```toml
version = 1

[envs.rattler-build]
channels = ["conda-forge"]
dependencies = { rattler-build = "*" }
exposed = { rattler-build = "rattler-build" }

[envs.ipython]
channels = ["conda-forge"]
dependencies = { ipython = "*", numpy = "*", matplotlib = "*" }
exposed = { ipython = "ipython", ipython3 = "ipython3" }

[envs.python]
channels = ["conda-forge"]
dependencies = { python = "3.12.*" } # (1)!
exposed = { py3 = "python" } # (2)!
```

1. **依赖项**是将安装在环境中的包。你可以指定版本或使用通配符。
2. **暴露的二进制文件**是将在系统路径中可用的文件。在此示例中，`python` 被暴露为 `py3`。

### 清单位置

根据操作系统的不同，清单文件可以在以下位置找到：

=== "Linux"

    | **优先级** | **位置**                                                                 | **备注**                                                                                  |
    |------------|------------------------------------------------------------------------|-------------------------------------------------------------------------------------------|
    | 1          | `$HOME/.pixi/manifests/pixi-global.toml`                                | 用户特定的清单                                                                           |
    | 2          | `$PIXI_HOME/manifests/pixi-global.toml`                                 | 用户主目录中的全局清单。`PIXI_HOME` 默认为 `~/.pixi`                                      |

=== "macOS"

    | **优先级** | **位置**                                                                 | **备注**                                                                                  |
    |------------|------------------------------------------------------------------------|-------------------------------------------------------------------------------------------|
    | 1          | `$HOME/.pixi/manifests/pixi-global.toml`                                | 用户特定的清单                                                                           |
    | 2          | `$PIXI_HOME/manifests/pixi-global.toml`                                 | 用户主目录中的全局清单。`PIXI_HOME` 默认为 `~/.pixi`                                      |

=== "Windows"

    | **优先级** | **位置**                                                                 | **备注**                                                                                   |
    |------------|------------------------------------------------------------------------|--------------------------------------------------------------------------------------------|
    | 1          | `%USERPROFILE%\.pixi\manifests\pixi-global.toml`                       | 用户特定的清单                                                                            |
    | 2          | `$PIXI_HOME\manifests/pixi-global.toml`                                 | 用户主目录中的全局清单。`PIXI_HOME` 默认为 `%USERPROFILE%/.pixi`                         |

!!! note
    如果存在多个位置，将使用优先级最高的清单文件。


### Channels
通道是将用于搜索包的 conda 通道。
这些通道有优先级，因此第一个通道将具有最高优先级，如果在该通道中未找到包，则将使用下一个通道。
例如，运行：
```
pixi global install --channel conda-forge --channel bioconda snakemake
```
将生成以下清单条目：
```toml
[envs.snakemake]
channels = ["conda-forge", "bioconda"]
dependencies = { snakemake = "*" }
exposed = { snakemake = "snakemake" }
```

有关通道的更多信息，请参见 [这里](../advanced/channel_priority.md)。

### 自动暴露

有一些附加的自动行为，如果你安装的包与环境同名，它将以相同的名称暴露。
即使二进制名称仅通过包的依赖项暴露。
例如，运行：
```
pixi global install ansible
```
将创建以下清单条目：
```toml
[envs.ansible]
channels = ["conda-forge"]
dependencies = { ansible = "*" }
exposed = { ansible = "ansible" } # (1)!
```

1. `ansible` 二进制文件被暴露，尽管它是通过 `ansible` 的依赖包 `ansible-core` 安装的。

### 依赖项
依赖项是将安装到你的环境中的 **Conda** 包。例如，运行：
```
pixi global install "python<3.12"
```
将在清单中创建以下条目：
```toml
[envs.vim]
channels = ["conda-forge"]
dependencies = { python = "<3.12" }
# ...
```
通常，你只需指定要安装的工具，但如果需要，也可以添加更多包。
定义要安装的环境将允许你一次添加多个依赖项。
例如，运行：
```shell
pixi global install --environment my-env git vim python
```
将在清单中创建以下条目：
```toml
[envs.my-env]
channels = ["conda-forge"]
dependencies = { git = "*", vim = "*", python = "*" }
# ...
```

你可以通过运行以下命令 `add` 依赖项到现有环境：
```shell
pixi global install --environment my-env package-a package-b
```
这将作为依赖项添加到 `my-env` 环境中，但不会自动暴露新包的二进制文件。

你可以通过运行以下命令 `remove` 依赖项：
```shell
pixi global remove --environment my-env package-a package-b
```

### 跳板程序

为了提高效率，`pixi` 使用 *跳板程序*——小型、专用的二进制文件，在执行主二进制文件之前管理配置和环境设置。跳板程序方法允许跳过执行对性能影响较大的激活脚本。

当你执行全局安装的二进制文件时，跳板程序执行以下步骤：

* 每个跳板程序首先读取与正在执行的二进制文件同名的配置文件。该配置文件以 JSON 格式存储（例如，`python.json`），包含有关如何设置环境的关键信息。配置文件存储在 `.pixi/bin/trampoline_configuration` 中。
* 加载配置并设置好环境后，跳板程序执行原始二进制文件，并应用正确的环境设置。
* 安装新二进制文件时，新的跳板程序将放置在 `.pixi/bin` 目录中，并与 `.pixi/bin/trampoline_configuration/trampoline_bin` 进行硬链接。这优化了存储空间并避免了跳板程序的重复。

### 示例：一次添加一系列工具
如果不指定环境，你可以一次添加多个工具：
```shell
pixi global install pixi-pack rattler-build
```
该命令将在清单中生成以下条目：
```toml
[envs.pixi-pack]
channels = ["conda-forge"]
dependencies = { pixi-pack = "*" }
exposed = { pixi-pack = "pixi-pack" }

[envs.rattler-build]
channels = ["conda-forge"]
dependencies = { rattler-build = "*" }
exposed = { rattler-build = "rattler-build" }
```
创建两个独立且互不干扰的环境，同时仅暴露最低限度的必需二进制文件。

### 示例：创建数据科学沙盒环境
你可以使用以下命令创建一个包含多个工具的环境：
```shell
pixi global install --environment data-science --expose jupyter --expose ipython jupyter numpy pandas matplotlib ipython
```
该命令将在清单中生成以下条目：
```toml
[envs.data-science]
channels = ["conda-forge"]
dependencies = { jupyter = "*", ipython = "*" }
exposed = { jupyter = "jupyter", ipython = "ipython" }
```
在此设置中，`data-science` 环境中同时暴露了 `jupyter` 和 `ipython`，允许你运行：
```shell
> ipython
# 或
> jupyter lab
```
这些命令将在全局范围内可用，使你可以轻松访问你喜欢的工具，而无需切换环境。

### 示例：为不同平台安装包
你可以使用 `--platform` 标志为不同的平台安装包。
当你想在 `osx-arm64` 上安装 `osx-64` 的包时，这非常有用。
例如，在 `osx-arm64` 上运行以下命令：
```shell
pixi global install --platform osx-64 python
```
将会在清单中创建以下条目：
```toml
[envs.python]
channels = ["conda-forge"]
platforms = ["osx-64"]
dependencies = { python = "*" }
# ...
```

## 未来潜在功能

### PyPI 支持

我们可以通过以下命令支持来自 PyPI 的包：

```
pixi global install --pypi flask
```

### 锁定文件

锁定文件对于全局工具来说不太重要。
然而，需求仍然存在，并且不关心它的用户不应受到负面影响。

### 多个清单

我们可以使用一个默认的清单，但也可以解析同一目录中的其他清单。
被解析为清单的唯一要求是 `.toml` 扩展名。
为了通过 `CLI` 修改这些清单，可能需要添加 `--manifest` 选项来选择正确的清单。

- pixi-global.toml: 默认清单
- pixi-global-company-tools.toml
- pixi-global-from-my-dotfiles.toml

目前尚不清楚是否第一版实现已经需要支持这个功能。
至少我们应该将清单放在它自己的文件夹中，如 `~/.pixi/global/manifests/pixi-global.toml`。

### 无激活

当前 `pixi global install` 提供了 `--no-activation` 功能。
当设置此标志时，在运行暴露的可执行文件时，`CONDA_PREFIX` 和 `PATH` 不会被设置。
这在安装 Python 包管理器或 shell 时非常有用。

假设这需要按映射设置，一种暴露此功能的方式是允许如下设置：

```toml
[envs.pip.exposed]
pip = { executable = "pip", activation = false }
```