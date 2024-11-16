# 教程：使用 Pixi 进行 Python 开发

在本教程中，我们将展示如何使用 Pixi 创建一个简单的 Python 项目。我们还会展示一些 Pixi 提供的功能，这些功能目前不包含在 `pdm`、`poetry` 等工具中。

## 为什么这很有用？

Pixi 基于 conda 生态系统，允许您创建一个包含所有所需依赖项的 Python 环境。当您需要使用多个 Python 解释器，并且绑定到 C 和 C++ 库时，这尤其有用。例如，PyPI 上的 GDAL 并不包含二进制 C 依赖项，但 conda 包包含这些依赖项。另一方面，一些包只通过 PyPI 提供，`pixi` 也可以为您安装这些包。结合两者的优势，赶紧试试吧！

## `pixi.toml` 和 `pyproject.toml`

我们支持两种清单格式：`pyproject.toml` 和 `pixi.toml`。在本教程中，我们将使用 `pyproject.toml` 格式，因为它是 Python 项目的最常见格式。

## 开始吧

我们从创建一个使用 `pyproject.toml` 文件的新项目开始。

```shell
pixi init pixi-py --format pyproject
```

这将创建一个具有以下结构的项目：

```shell
├── src
│   └── pixi_py
│       └── __init__.py
└── pyproject.toml
```

项目的 `pyproject.toml` 文件如下所示：

```toml
[project]
name = "pixi-py"
version = "0.1.0"
description = "Add a short description here"
authors = [{name = "Tim de Jager", email = "tim@prefix.dev"}]
requires-python = ">= 3.11"
dependencies = []

[build-system]
build-backend = "hatchling.build"
requires = ["hatchling"]

[tool.pixi.project]
channels = ["conda-forge"]
platforms = ["osx-arm64"]

[tool.pixi.pypi-dependencies]
pixi-py = { path = ".", editable = true }

[tool.pixi.tasks]
```

该项目使用了 `src` 布局，但 Pixi 支持 [flat 和 src 布局](https://hellowac.github.io/pypug-zh-cn/discussions/src-layout-vs-flat-layout.html)。

### `pyproject.toml` 中的内容

现在，让我们看看 `pyproject.toml` 中添加了哪些部分，以及我们如何修改它。

以下条目已添加到 `pyproject.toml` 文件中：

```toml
# 主 Pixi 入口
[tool.pixi.project]
channels = ["conda-forge"]
# 默认使用您的机器平台
platforms = ["osx-arm64"]
```

`channels` 和 `platforms` 被添加到了 `[tool.pixi.project]` 部分。
像 `conda-forge` 这样的频道管理类似 PyPI 的包，但允许跨语言的不同包。
`platforms` 关键字决定了该项目支持的操作平台。

`pixi_py` 包本身被添加为一个可编辑依赖项。  
这意味着包将以可编辑模式安装，您可以对包进行修改，并立即在环境中看到变化，而无需重新安装环境。

```toml
# 可编辑安装
[tool.pixi.pypi-dependencies]
pixi-py = { path = ".", editable = true }
```

在 Pixi 中，与其他包管理器不同，这在 `pyproject.toml` 文件中显式声明。
这样做的主要原因是为了让您选择该包应包含在哪个环境中。

### 在 Pixi 中管理 Conda 和 PyPI 依赖

我们的项目通常依赖其他包。

```shell
$ pixi add black
Added black
```

这将导致以下内容被添加到 `pyproject.toml` 中：

```toml
# 依赖
[tool.pixi.dependencies]
black = ">=24.4.2,<24.5"
```

但是我们也可以通过 `pixi add black=24` 来严格指定要使用的版本，结果如下：

```toml
[tool.pixi.dependencies]
black = "24.*"
```

现在，让我们添加一些可选依赖：

```shell
pixi add --pypi --feature test pytest
```

这将导致以下字段被添加到 `pyproject.toml`：

```toml
[project.optional-dependencies]
test = ["pytest"]
```

在将可选依赖添加到 `pyproject.toml` 后，Pixi 会自动创建一个 [`feature`](../reference/project_configuration.md/#the-feature-and-environments-tables)，其中可以包含一组 `dependencies`、`tasks`、`channels` 等。

有时，有些包在 conda 渠道中不可用，但在 PyPI 上发布。我们也可以将这些包添加进去，Pixi 会与默认依赖一起解决它们。

```shell
$ pixi add black --pypi
Added black
Added these as pypi-dependencies.
```

这将导致以下内容添加到 `pyproject.toml` 的 `dependencies` 键中：

```toml
dependencies = ["black"]
```

使用 `pypi-dependencies` 时，您可以利用其他包提供的 `optional-dependencies`。
例如，`black` 提供了 `cli` 依赖选项，可以通过 `--pypi` 关键字添加：

```shell
$ pixi add black[cli] --pypi
Added black[cli]
Added these as pypi-dependencies.
```

这将更新 `dependencies` 条目为：

```toml
dependencies = ["black[cli]"]
```

??? note "Pixi.toml 中的可选依赖"
    本教程侧重于使用 `pyproject.toml`，但如果您感兴趣的话，安装带有可选依赖项的 PyPI 包后，`pixi.toml` 中会包含以下条目：
    ```toml
    [pypi-dependencies]
    black = { version = "*", extras = ["cli"] }
    ```

### 安装：`pixi install`

现在让我们使用 `pixi install` 来安装项目：

```shell
$ pixi install
✔ Project in /path/to/pixi-py is ready to use!
```

我们现在在项目根目录下有一个新的目录 `.pixi`。  
该目录包含了我们运行 `pixi install` 时创建的环境。  
该环境是一个包含我们在 `pyproject.toml` 文件中指定的依赖项的 conda 环境。  
我们还可以通过 `pixi install -e test` 安装测试环境。  
我们可以使用这些环境来执行代码。

我们还在项目根目录下得到了一个新的文件 `pixi.lock`。  
该文件包含了在不同平台上安装的依赖项的确切版本。

## 环境中的内容

使用 `pixi list`，您可以查看环境中的内容，这实际上是对锁定文件的一种更友好的查看方式：

```shell
$ pixi list
Package          Version       Build               Size       Kind   Source
bzip2            1.0.8         h93a5062_5          119.5 KiB  conda  bzip2-1.0.8-h93a5062_5.conda
black            24.4.2                            3.8 MiB    pypi   black-24.4.2-cp312-cp312-win_amd64.http.whl
ca-certificates  2024.2.2      hf0a4a13_0          152.1 KiB  conda  ca-certificates-2024.2.2-hf0a4a13_0.conda
libexpat         2.6.2         hebf3989_0          62.2 KiB   conda  libexpat-2.6.2-hebf3989_0.conda
libffi           3.4.2         h3422bc3_5          38.1 KiB   conda  libffi-3.4.2-h3422bc3_5.tar.bz2
libsqlite        3.45.2        h091b4b1_0          806 KiB    conda  libsqlite-3.45.2-h091b4b1_0.conda
libzlib          1.2.13        h53f4e23_5          47 KiB     conda  libzlib-1.2.13-h53f4e23_5.conda
ncurses          6.4.20240210  h078ce10_0          801 KiB    conda  ncurses-6.4.20240210-h078ce10_0.conda
openssl          3.2.1         h0d3ecfb_1          2.7 MiB    conda  openssl-3.2.1-h0d3ecfb_1.conda
python           3.12.3        h4a7b5fc_0_cpython  12.6 MiB   conda  python-3.12.3-h4a7b5fc_0_cpython.conda
readline         8.2           h92ec313_1          244.5 KiB  conda  readline-8.2-h92ec313_1.conda
tk               8.6.13        h5083fa2_1          3 MiB      conda  tk-8.6.13-h5083fa2_1.conda
tzdata           2024a         h0c530f3_0          117 KiB    conda  tzdata-2024a-h0c530f3_0.conda
pixi-py          0.1.0                                        pypi   . (editable)
xz               5.2.6         h57fd34a_0          230.2 KiB  conda  xz-5.2.6-h57fd34a_0.tar.bz2
```


!!! Python 解释器

    Python 解释器也会安装在环境中。这是因为 Python 解释器的版本是从 `pyproject.toml` 文件中的 `requires-python` 字段读取的。  
    这用于确定要在环境中安装的 Python 版本。  
    这样，Pixi 会自动为您管理/引导 Python 解释器，不再需要使用 `brew`、`apt` 或其他系统安装步骤。

在这里，您可以看到列出的不同 conda 和 PyPI 包。  
正如您所看到的，我们正在使用的 `pixi-py` 包是以可编辑模式安装的。  
Pixi 中的每个环境都是隔离的，但会重用从中央缓存目录硬链接的文件。  
这意味着您可以拥有多个包含相同包的环境，但只需在磁盘上存储一次单独的文件。

我们可以根据自己的 `test` 特性从 `optional-dependency` 创建 `default` 和 `test` 环境：

```shell
pixi project environment add default --solve-group default
pixi project environment add test --feature test --solve-group default
```

这会导致如下变化：

```toml
# 环境
[tool.pixi.environments]
default = { solve-group = "default" }
test = { features = ["test"], solve-group = "default" }
```

??? note "求解组"
    求解组是将依赖项分组在一起的方式。当您有多个共享相同依赖项的环境时，这非常有用。例如，也许 `pytest` 是一个依赖项，影响 `default` 环境的依赖项。  
    通过将这些放在同一个求解组中，您可以确保 `test` 和 `default` 中的版本完全相同。

`default` 环境是在运行 `pixi install` 时创建的。  
`test` 环境是从 `pyproject.toml` 文件中的可选依赖项创建的。  
您可以在此环境中执行命令，例如使用 `pixi run -e test python`。

### 让代码运行起来

接下来，我们将向 `pixi-py` 包中添加一些代码。
我们将向 `pixi_py/__init__.py` 文件添加一个新函数：

```python
from rich import print

def hello():
    return "Hello, [bold magenta]World[/bold magenta]!", ":vampire:"

def say_hello():
    print(*hello())
```

现在，通过以下命令从 PyPI 添加 `rich` 依赖项：

```shell
pixi add --pypi rich
```

然后，使用以下命令来测试它是否工作：

```shell
pixi r python -c "import pixi_py; pixi_py.say_hello()"
Hello, World! 🧛
```

??? note "慢吗？"
    由于 Pixi 会首次安装项目，可能会有些慢（大约 2 分钟），但第二次运行时几乎是即时的。

Pixi 运行自安装的 Python 解释器。
接着，我们导入了 `pixi_py` 包，该包以可编辑模式安装。
代码调用了我们刚才添加的 `say_hello` 函数。
它成功运行了！很酷！

### 测试代码

接下来，让我们为这个函数添加一个测试。
我们将在项目根目录下添加一个 `tests/test_me.py` 文件。

这时，项目的结构如下：

```shell
.
├── pixi.lock
├── pixi_py
│   └── __init__.py
├── pyproject.toml
└── tests/test_me.py
```

```python
from pixi_py import hello

def test_pixi_py():
    assert hello() == ("Hello, [bold magenta]World[/bold magenta]!", ":vampire:")
```

然后，我们为运行测试添加一个简单的任务。

```shell
$ pixi task add --feature test test "pytest"
✔ Added task `test`: pytest .
```

Pixi 有一个任务系统，可以方便地运行命令。
类似于 `npm` 脚本，或者你在 `Justfile` 中指定的命令。

??? tip "Pixi 任务"
    任务是 Pixi 的一个很酷的功能，跨平台的 shell 可以运行这些任务。
    你可以进行缓存、依赖处理等操作。
    更多关于任务的信息，请阅读 [任务](../features/advanced_tasks.md) 部分。

```shell
$ pixi r test
✨ Pixi task (test): pytest .
================================================================================================= test session starts =================================================================================================
platform darwin -- Python 3.12.2, pytest-8.1.1, pluggy-1.4.0
rootdir: /private/tmp/pixi-py
configfile: pyproject.toml
collected 1 item

test_me.py .                                                                                                                                                                                                    [100%]

================================================================================================== 1 passed in 0.00s =================================================================================================
```

太棒了！测试通过了！

### 测试环境与默认环境

接下来，比较测试环境和默认环境的输出…

```shell
pixi list -e test
# 与默认环境比较
pixi list
```

我们可以看到，测试环境包含以下内容：

```shell
package          version       build               size       kind   source
...
pytest           8.1.1                             1.1 mib    pypi   pytest-8.1.1-py3-none-any.whl
...
```

而默认环境则没有这个包。
这样，你就可以根据环境需求来调整包的安装。
例如，你可以为 `dev` 环境安装 `pytest` 和 `ruff`，但不在 `prod` 环境中安装它们。
这里有一个 [docker](https://github.com/prefix-dev/pixi/tree/main/examples/docker) 示例，展示如何设置一个最小的 `prod` 环境，并从中复制设置。

## 将 PyPI 包替换为 conda 包

最后，Pixi 还支持让 `pypi` 包依赖于 `conda` 包。
我们可以用 `pixi list` 来确认这一点：

```shell
$ pixi list
Package          Version       Build               Size       Kind   Source
...
pygments         2.17.2                            4.1 MiB    pypi   pygments-2.17.2-py3-none-any.http.whl
...
```

我们显式地将 `pygments` 添加到 `pyproject.toml` 文件中。
这是 `rich` 包的一个依赖项。

```shell
pixi add pygments
```

这会将以下内容添加到 `pyproject.toml` 文件：

```toml
[tool.pixi.dependencies]
pygments = ">=2.17.2,<2.18"
```

现在我们可以看到，`pygments` 包已作为 conda 包安装。

```shell
$ pixi list
Package          Version       Build               Size       Kind   Source
...
pygments         2.17.2        pyhd8ed1ab_0        840.3 KiB  conda  pygments-2.17.2-pyhd8ed1ab_0.conda
```

这样，PyPI 依赖项和 conda 依赖项可以混合使用，顺畅地进行互操作。

```shell
$  pixi r python -c "import pixi_py; pixi_py.say_hello()"
Hello, World! 🧛
```

它仍然工作正常！

## 总结

在本教程中，你了解了如何使用 `pyproject.toml` 来管理 Pixi 依赖项和环境。
我们还探讨了如何在同一个项目中无缝结合 PyPI 和 conda 依赖项，并安装可选依赖项来管理 Python 包。

希望这个教程能为你提供灵活强大的方法来管理你的 Python 项目，并为你在 Pixi 的进一步探索奠定坚实基础。

感谢阅读！祝编码愉快 🚀

如果有任何问题，欢迎联系或分享此教程到 [X](https://twitter.com/prefix_dev)，[加入我们的 Discord](https://discord.gg/kKV8ZxyzY4)，发送邮件至 [hi@prefix.dev](mailto:hi@prefix.dev)，或关注我们的 [GitHub](https://github.com/prefix-dev)。
