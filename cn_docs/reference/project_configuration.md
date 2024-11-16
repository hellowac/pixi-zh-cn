---
part: pixi
title: Configuration
description: 了解您可以在 pixi.toml 配置中做什么。
---

`pixi.toml` 是 pixi 项目的配置文件，也称为项目清单文件。

`toml` 文件的结构由不同的表格组成。  
本文将解释不同表格的用法。  
有关更详细的技术文档，请查看 [crates.io](https://docs.rs/pixi/latest/pixi/project/manifest/struct.ProjectManifest.html) 上的 pixi 文档。

!!! tip
    我们也支持 `pyproject.toml` 文件。它与 `pixi.toml` 文件具有相同的结构，唯一的不同是，你需要在表格名前加上 `tool.pixi`，而不是仅使用表格名称。例如，`[project]` 表格变为 `[tool.pixi.project]`。  
    在 `pyproject.toml` 文件中还有一些额外的功能，详细信息请参阅 [pyproject.toml](../advanced/pyproject_toml.md) 文档。

## `project` 表格

在 `project` 表格中，最基本的必需信息是：

```toml
--8<-- "docs/source_files/pixi_tomls/simple_pixi.toml:project"
```

### `name`

项目的名称。

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_name"
```

### `channels`

这是一个定义用来获取包的渠道的列表。  
如果你想使用托管在 `anaconda.org` 上的渠道，只需要直接使用该渠道的名称。

```toml
--8<-- "docs/source_files/pixi_tomls/lots_of_channels.toml:project_channels_long"
```

文件系统上的渠道也可以使用 **绝对路径** 来支持：

```toml
--8<-- "docs/source_files/pixi_tomls/lots_of_channels.toml:project_channels_path"
```

要访问 [prefix.dev](https://prefix.dev/channels) 或 [Quetz](https://github.com/mamba-org/quetz) 上的私有或公共渠道，可以使用包括主机名的 URL：

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_channels"
```

### `platforms`

定义项目支持的平台列表。  
Pixi 会为所有这些平台解决依赖关系，并将其放入锁文件 (`pixi.lock`) 中。

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_platforms"
```

可用的平台列表请参见此处：[链接](https://docs.rs/rattler_conda_types/latest/rattler_conda_types/enum.Platform.html)

!!! tip "macOS 特殊行为"
    macOS 有两个平台：`osx-64` 用于 Intel Mac 和 `osx-arm64` 用于 Apple Silicon Mac。  
    要同时支持两者，请将两者都包括在你的平台列表中。  
    回退：如果 `osx-arm64` 无法解析，使用 `osx-64`。  
    在 Apple Silicon 上运行 `osx-64` 时，会使用 [Rosetta](https://developer.apple.com/documentation/apple-silicon/about-the-rosetta-translation-environment) 来运行 Intel 二进制文件。

### `version`（可选）

项目的版本。  
该版本应为基于 conda 版本规范的有效版本。  
有关版本规范允许的内容，请参阅 [版本文档](https://docs.rs/rattler_conda_types/latest/rattler_conda_types/struct.Version.html)。

```toml
-8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_version"
```

### `authors`（可选）

这是一个列出项目作者的列表。

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_authors"
```

### `description`（可选）

应包含项目的简短描述。

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_description"
```

### `license`（可选）

项目的许可证，作为有效的 [SPDX](https://spdx.org/licenses/) 字符串（例如 MIT AND Apache-2.0）

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_license"
```

### `license-file`（可选）

许可证文件的相对路径。

```toml
license-file = "LICENSE.md"
```

### `readme`（可选）

README 文件的相对路径。

```toml
readme = "README.md"
```

### `homepage`（可选）

项目主页的 URL。

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_homepage"
```

### `repository`（可选）

项目源代码仓库的 URL。

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_repository"
```

### `documentation`（可选）

项目文档的 URL。

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_documentation"
```

### `conda-pypi-map`（可选）

将频道名称或 URL 映射到可以是 URL 或路径的映射位置。  
映射应采用 `json` 格式，格式为 `{conda_name}: {pypi_package_name}`。  
示例如下：

```json title="local/robostack_mapping.json"
{
  "jupyter-ros": "my-name-from-mapping",
  "boltons": "boltons-pypi"
}
```

如果 `conda-forge` 不在 `conda-pypi-map` 中，`pixi` 将使用 `prefix.dev` 的映射。

```toml
conda-pypi-map = { "conda-forge" = "https://example.com/mapping", "https://repo.prefix.dev/robostack" = "local/robostack_mapping.json"}
```

### `channel-priority`（可选）

这是设置求解步骤中频道优先级的选项。

选项：

- `strict`：**默认**，频道将按 `channels` 列表中定义的顺序使用。  
   只有第一个包含该包的频道中的包会被使用。  
   这确保了不会混合来自不同频道的同一包的不同变体。  
   使用来自不同不兼容频道（如 `conda-forge` 和 `main`）的包可能会导致难以调试的 ABI 不兼容问题。

   我们强烈建议不要更改默认设置。

- `disabled`：没有优先级，所有频道的包变体将按包名设置并作为一个解决方案。  
   使用此选项时应小心。  
   由于包变体可以来自 _任何_ 渠道，因此使用此模式时，包可能不兼容。  
   这可能导致难以调试的 ABI 不兼容问题。

   我们强烈不建议使用此选项。

```toml
channel-priority = "disabled"
```
!!! warning "`channel-priority = "disabled"` 是一个安全风险"
    禁用频道优先级可能会导致不可预测的依赖关系解析结果。  
    这可能是一个安全风险，因为它可能导致从意外的频道安装包。  
    建议保持默认的严格设置，并谨慎排序频道。  
    如有必要，直接为依赖项指定一个频道。  
    ```toml
    [project]
    # 将 conda-forge 放在首位解决大多数问题
    channels = ["conda-forge", "channel-name"]
    [dependencies]
    package = {version = "*", channel = "channel-name"}
    ```

## `tasks` 表

任务是一种在项目中自动化某些自定义命令的方式。  
例如，一个 `lint` 或 `format` 步骤。  
Pixi 项目中的任务本质上是跨平台的 shell 命令，在不同平台上有统一的语法。  
有关更深入的信息，请查看 [高级任务文档](../features/advanced_tasks.md)。  
Pixi 的任务在 pixi 环境中使用 `pixi run` 运行，并通过 [`deno_task_shell`](../features/advanced_tasks.md#deno_task_shell) 执行。

```toml
[tasks]
simple = "echo This is a simple task"
cmd = { cmd="echo Same as a simple task but now more verbose"}
depending = { cmd="echo run after simple", depends-on="simple"}
alias = { depends-on=["depending"]}
download = { cmd="curl -o file.txt https://example.com/file.txt" , outputs=["file.txt"]}
build = { cmd="npm build", cwd="frontend", inputs=["frontend/package.json", "frontend/*.js"]}
run = { cmd="python run.py $ARGUMENT", env={ ARGUMENT="value" }}
format = { cmd="black $INIT_CWD" } # 在运行 pixi run format 时执行 black
clean-env = { cmd = "python isolated.py", clean-env = true} # 仅限 Unix！
```

您可以使用 [`pixi task`](cli.md#task) 修改此表。

!!! note  
    使用 [target](#target) 表为不同平台指定不同的任务

!!! info  
    如果您希望隐藏某个任务，使其不出现在 `pixi task list` 或 `pixi info` 中，您可以在任务名称前加上 `_`。  
    例如，如果您想隐藏 `depending`，可以将其重命名为 `_depending`。

## `system-requirements` 表

系统要求用于定义在依赖解析过程中使用的最低系统规格。

例如，我们可以定义一个具有特定最低 libc 版本的 Unix 系统。
```toml
[system-requirements]
libc = "2.28"
```
或使项目依赖于特定版本的 `cuda`：
```toml
[system-requirements]
cuda = "12"
```

可以定义的选项有：

- `linux`：最低版本的 Linux 内核。
- `libc`：最低版本的 libc 库。还允许指定 libc 库的家族。  
  例如：`libc = { family="glibc", version="2.28" }`
- `macos`：最低版本的 macOS 操作系统。
- `cuda`：最低版本的 CUDA 库。

有关更多信息，请参阅 [系统要求文档](../features/system_requirements.md)。

## `pypi-options` 表

`pypi-options` 表用于定义特定于 PyPI 注册表的选项。  
这些选项可以在根级别指定，这将添加到默认选项中，或者在功能级别指定，当功能被包含到环境中时，会创建这些选项的联合。

可以定义的选项有：

- `index-url`：替换主索引 URL。
- `extra-index-urls`：添加额外的索引 URL。
- `find-links`：类似于 `pip` 中的 `--find-links` 选项。
- `no-build-isolation`：禁用构建隔离，仅能针对每个包设置。
- `index-strategy`：允许指定使用的索引策略。

这些选项在下面的章节中有详细说明。大部分选项直接来自或略有修改自 [uv 设置](https://docs.astral.sh/uv/reference/settings/)。如果您需要的选项没有列出，请随时创建问题 [请求](https://github.com/prefix-dev/pixi/issues)。

### 替代注册表

!!! info "严格的索引优先级"  
    与 pip 不同，由于我们使用 uv，我们具有严格的索引优先级。这意味着将使用第一个可以找到包的索引。  
    索引的顺序由 toml 文件中的定义顺序决定。`extra-index-urls` 优先于 `index-url`。更多信息请参阅 [uv 文档](https://docs.astral.sh/uv/pip/compatibility/#packages-that-exist-on-multiple-indexes)

通常，您可能希望为项目使用替代或额外的索引。这可以通过将 `pypi-options` 表添加到 `pixi.toml` 文件中来完成，以下选项是可用的：

- `index-url`：替换主索引 URL。如果未设置，默认索引为 `https://pypi.org/simple`。  
   每个环境只能定义 **一个** `index-url`。
- `extra-index-urls`：添加额外的索引 URL。索引 URL 按定义顺序使用，并且优先于 `index-url`。这些 URL 会在功能间合并到环境中。
- `find-links`：可以是路径 `{path = './links'}` 或 URL `{url = 'https://example.com/links'}`。  
   这类似于 `pip` 中的 `--find-links` 选项。这些 URL 会在功能间合并到环境中。

示例：

```toml
[pypi-options]
index-url = "https://pypi.org/simple"
extra-index-urls = ["https://example.com/simple"]
find-links = [{path = './links'}]
```

Pixi 仓库中有一些 [示例](https://github.com/prefix-dev/pixi/tree/main/examples/pypi-custom-registry)，展示了如何使用此功能。

!!! tip "认证方法"  
    要了解有关私有注册表的现有认证方法，请查看 [PyPI 认证](../advanced/authentication.md#pypi) 部分。


### 无构建隔离

尽管构建隔离是一个很好的默认设置，  
但可以选择**不**对某个包名称进行隔离，这允许构建访问 `pixi` 环境。  
如果您希望在构建过程中使用 `torch` 或类似的库，这将非常方便。

```toml
[dependencies]
pytorch = "2.4.0"

[pypi-options]
no-build-isolation = ["detectron2"]

[pypi-dependencies]
detectron2 = { git = "https://github.com/facebookresearch/detectron2.git", rev = "5b72c27ae39f99db75d43f18fd1312e1ea934e60"}
```

!!! tip "Conda 依赖定义构建环境"  
    要有效使用 `no-build-isolation`，请使用 conda 依赖来定义构建环境。这些依赖会在解析 PyPI 依赖之前安装，这样它们在构建过程中就可以使用。在上面的示例中，将 `torch` 作为 PyPI 依赖添加将不起作用，因为它在 PyPI 解析阶段尚未安装。

### 索引策略

用于解析多个索引 URL 时的策略。描述修改自 [uv](https://docs.astral.sh/uv/reference/settings/#index-strategy) 文档：

默认情况下，`uv` 以及 `pixi` 会在找到给定包的第一个索引时停止，并将解析限制在该第一个索引中存在的包（即首个匹配）。  
这可以防止 *依赖混淆* 攻击，在这种攻击中，攻击者可能会将恶意包上传到一个次级索引，并使用相同的名称。

!!! warning "每个环境只能有一个索引策略"  
    每个环境或求解组只能定义一个 `index-strategy`，否则会显示错误。

#### 可选值：

- **"first-index"**：仅使用返回匹配给定包名的第一个索引的结果
- **"unsafe-first-match"**：在所有索引中搜索每个包名，先在第一个索引中消耗完版本，再转到下一个索引。这意味着如果包 `a` 在索引 `x` 和 `y` 中都有，除非您请求的包版本**仅**在 `y` 上可用，否则会优先选择 `x` 中的版本。
- **"unsafe-best-match"**：在所有索引中搜索每个包名，优先选择找到的*最佳*版本。如果包版本在多个索引中都有，最多只查看第一个索引中的条目。所以，对于包含包 `a` 的索引 `x` 和 `y`，它会从 `x` 或 `y` 中选择*最佳*版本，但如果该版本在两个索引中都有，它会优先选择 `x`。

!!! info "仅限 PyPI"  
    `index-strategy` 只会影响 PyPI 包的解析，不会影响 conda 包的解析。

## `dependencies` 表

此部分定义您希望为项目使用的依赖。

有多个依赖表。  
默认的是 `[dependencies]`，它是跨平台共享的依赖。

依赖项使用 [VersionSpec](https://docs.rs/rattler_conda_types/latest/rattler_conda_types/version_spec/enum.VersionSpec.html) 定义。  
`VersionSpec` 将 [Version](https://docs.rs/rattler_conda_types/latest/rattler_conda_types/struct.Version.html) 与可选的操作符结合。

一些示例如下：

```toml
# 使用此确切包版本
package0 = "1.2.3"
# 使用 1.2.3 到 1.3.0 的版本
package1 = "~=1.2.3"
# 使用大于 1.2 且小于等于 1.4 的版本
package2 = ">1.2,<=1.4"
# 大于等于 1.2.3 或小于 1.0.0 的版本
package3 = ">=1.2.3|<1.0.0"
```

依赖项也可以作为映射进行定义，使用 [matchspec](https://docs.rs/rattler_conda_types/latest/rattler_conda_types/struct.NamelessMatchSpec.html)：

```toml
package0 = { version = ">=1.2.3", channel="conda-forge" }
package1 = { version = ">=1.2.3", build="py34_0" }
```

!!! tip  
    可以使用 `pixi add` 命令轻松添加依赖项。  
    对现有依赖运行 `add` 命令将其替换为可用的最新版本。

!!! note  
    要为不同平台指定不同的依赖项，请使用 [target](#target) 表。

### `dependencies`

添加您希望安装到环境中的任何 conda 包依赖项。  
如果使用的不是 `conda-forge`，请不要忘记将该通道添加到项目表中。  
即使依赖项定义了通道，也应将该通道添加到 `project.channels` 列表中。

```toml
[dependencies]
python = ">3.9,<=3.11"
rust = "1.72"
pytorch-cpu = { version = "~=1.1", channel = "pytorch" }
```

### `pypi-dependencies`

??? info "有关 PyPI 集成的详细信息"  
    我们使用 [`uv`](https://github.com/astral-sh/uv)，这是一个用 Rust 编写的新型快速 pip 替代品。

    我们将 uv 作为库集成，因此我们使用 uv 解析器，并将 conda 包作为“锁定”传递给它。  
    这禁止 uv 自行安装这些依赖，并确保它在解析时使用这些包的确切版本。  
    这在基于 conda 的包管理器中是独特的，因为通常它们只是从子进程调用 pip。

    uv 解析直接包含在锁文件中。

Pixi 直接支持依赖 PyPI 包，PyPA 将分发包称为“分发”。  
Pixi 支持 [源](https://packaging.python.org/en/latest/specifications/source-distribution-format/) 和 [二进制](https://packaging.python.org/en/latest/specifications/binary-distribution-format/) 分发，  
这些都可以在 Pixi 中使用。  
这些分发包在 conda 环境解析并安装后安装到环境中。  
PyPI 包没有在 [prefix.dev](https://prefix.dev/channels) 上索引，但可以在 [pypi.org](https://pypi.org/) 查看。

!!! warning "重要事项"  
    - **稳定性**：PyPI 包可能不如 conda 包稳定。在可能的情况下，优先使用 `dependencies` 表中的 conda 包。

#### 版本规范：

这些依赖项不遵循 conda 的 matchspec 规范。  
`version` 是根据 [PEP404/PyPA](https://packaging.python.org/en/latest/specifications/version-specifiers/) 定义的版本字符串规范。  
此外，可以包含一个额外的列表，这些是可选的依赖项。  
请注意，这里的 `version` 与 conda 的 MatchSpec 类型不同。  
请参见以下示例，了解如何在实践中使用：

```toml
[dependencies]
# 使用 pypi-dependencies 时，需要 python 来解析 pypi 依赖
# 请确保包含此项
python = ">=3.6"

[pypi-dependencies]
fastapi = "*"  # 这表示任何版本（通配符 `*` 是 pixi 的扩展，而不是规范的一部分）
pre-commit = "~=3.5.0" # 这是一个单一版本的规范
# 使用 toml 映射可以让用户添加 `extras`
pandas = { version = ">=1.0.0", extras = ["dataframe", "sql"]}

# git 依赖
# 使用 ssh
flask = { git = "ssh://git@github.com/pallets/flask" }
# 使用 https 和特定的修订版
requests = { git = "https://github.com/psf/requests.git", rev = "0106aced5faa299e6ede89d1230bd6784f2c3660" }
# TODO: 后续会支持 -> branch = '' 或 tag = '' 用于指定分支或标签

# 也可以直接从路径添加源依赖，建议将路径保持为相对于项目根目录的路径
minimal-project = { path = "./minimal-project", editable = true}

# 还可以使用直接的 url，指向 `.tar.gz` 或 `.zip` 文件，或 `.whl` 文件
click = { url = "https://github.com/pallets/click/releases/download/8.1.7/click-8.1.7-py3-none-any.whl" }

# 还可以直接使用默认的 git 仓库，它将检出默认分支
pytest = { git = "https://github.com/pytest-dev/pytest.git"}
```

#### 完整规范

Pixi 支持的 PyPI 依赖的完整规范可以分为以下几个字段：

##### `extras`

要与包一起安装的额外依赖列表，例如 `["dataframe", "sql"]`。  
`extras` 字段与所有其他版本规范兼容，因为它是版本规范的附加部分。

```toml
pandas = { version = ">=1.0.0", extras = ["dataframe", "sql"]}
pytest = { git = "URL", extras = ["dev"]}
black = { url = "URL", extras = ["cli"]}
minimal-project = { path = "./minimal-project", editable = true, extras = ["dev"]}
```

##### `version`

要安装的包的版本，例如 `">=1.0.0"` 或 `*`，代表任何版本，这是 Pixi 特有的。  
`version` 是我们的默认字段，因此如果没有使用内联表（`{}`），将默认使用此字段。

```toml
py-rattler = "*"
ruff = "~=1.0.0"
pytest = {version = "*", extras = ["dev"]}
```

##### `git`

从 git 仓库安装。  
支持 `https://` 和 `ssh://` 两种 URL。

使用 `git` 时可以结合 `rev` 或 `subdirectory`：

- `rev`：指定要安装的修订版，例如 `rev = "0106aced5faa299e6ede89d1230bd6784f2c3660"`
- `subdirectory`：指定要安装的子目录，例如 `subdirectory = "src"` 或 `subdirectory = "src/packagex"`

```toml
# 注意不要忘记 `ssh://` 或 `https://` 前缀！
pytest = { git = "https://github.com/pytest-dev/pytest.git"}
requests = { git = "https://github.com/psf/requests.git", rev = "0106aced5faa299e6ede89d1230bd6784f2c3660" }
py-rattler = { git = "ssh://git@github.com/mamba-org/rattler.git", subdirectory = "py-rattler" }
```

##### `path`

从本地路径安装，例如 `path = "./path/to/package"`。  
我们建议将路径项目保存在项目内部，并使用相对路径。

将 `editable` 设置为 `true` 以启用可编辑模式安装，强烈建议使用此模式，因为如果不使用可编辑模式，重新安装将变得困难。例如，`editable = true`

```toml
minimal-project = { path = "./minimal-project", editable = true}
```

##### `url`

从 URL 直接安装 wheel 或 sdist 文件。

```toml
pandas = {url = "https://files.pythonhosted.org/packages/3d/59/2afa81b9fb300c90531803c0fd43ff4548074fa3e8d0f747ef63b3b5e77a/pandas-2.2.1.tar.gz"}
```

??? tip "你知道可以使用：`add --pypi` 吗？"  
    使用 `add` 命令时加上 `--pypi` 标志，可以快速从 CLI 添加 PyPI 包。  
    例如 `pixi add --pypi flask`  

    _这还不支持 `pypi-dependencies` 表的所有功能。_

#### 源依赖（`sdist`）

[源分发格式](https://packaging.python.org/en/latest/specifications/source-distribution-format/) 是一种基于源代码的格式（简称 sdist），包可以与二进制 wheel 格式一起包含。  
因为这些分发包需要构建，所以需要一个 Python 可执行文件来完成此操作。  
这就是为什么在 conda 环境中需要存在 python。  
源分发通常依赖于系统包来构建，特别是在编译 C/C++ 基于 Python 的绑定时。  
例如，Python SDL2 绑定依赖 C 库 SDL2。  
为了帮助构建这些依赖项，我们会在解析之前激活包含这些 PyPI 依赖项的 conda 环境。  
这样，当源分发依赖于例如 `gcc` 时，它将使用来自 conda 环境中的版本，而不是系统版本。

### `host-dependencies`

此表包含构建项目所需的依赖项，但这些依赖项在项目作为其他项目的一部分安装时不应被包含。  
换句话说，这些依赖项在构建过程中可用，但在项目安装后将不再可用。  
列在此表中的依赖项将根据目标机器的架构进行安装。

```toml
[host-dependencies]
python = "~=3.10.3"
```

主机依赖项的典型示例如下：

- 基础解释器：Python 包会在此列出 `python`，R 包会列出 `mro-base` 或 `r-base`。
- 在编译过程中链接的库，例如 `openssl`、`rapidjson` 或 `xtensor`。

### `build-dependencies`

此表包含构建项目所需的依赖项。
与 `dependencies` 和 `host-dependencies` 不同，这些包是针对 _构建_ 机器架构安装的。
这使得可以从一个机器架构交叉编译到另一个机器架构。

```toml
[build-dependencies]
cmake = "~=3.24"
```

构建依赖项的典型示例包括：

- 编译器在构建机器上调用，但它们为目标机器生成代码。
  如果项目是交叉编译的，构建机器和目标机器的架构可能不同。
- 在构建机器上调用 `cmake` 以生成附加的代码或项目文件，这些文件随后会被包含在编译过程中。

!!! info
    _构建_ 目标指的是执行构建的机器。
    这些依赖项安装的程序和库将在构建机器上执行。

    例如，如果你在带有 Apple Silicon 芯片的 MacBook 上编译，但目标平台是 Linux x86_64，那么你的 *build* 平台是 `osx-arm64`，而你的 *host* 平台是 `linux-64`。

## `activation` 表

`activation` 表用于在激活环境时执行的专门激活操作。

用户可以修改以下两种类型的激活操作：

- `scripts`：激活环境时运行的脚本列表。
- `env`：激活环境时设置的环境变量映射。

这些激活操作将在 `pixi run` 和 `pixi shell` 命令之前执行。

!!! note
    激活操作由系统的 shell 解释器运行，因为它们在环境可用之前运行。
    这意味着它们会在 Windows 上作为 `cmd.exe` 运行，在 Linux 和 osx（Unix）上作为 `bash` 运行。
    仅支持 `.sh`、`.bash` 和 `.bat` 文件。

    环境变量将在运行激活脚本的 shell 中设置，因此在使用例如 `$` 或 `%` 时需要注意。

    如果你有特定平台的脚本或环境变量，请使用 [target](#target) 表。

```toml
[activation]
scripts = ["env_setup.sh"]
env = { ENV_VAR = "value" }

# 若要同时支持 Windows 平台，请添加以下内容
[target.win-64.activation]
scripts = ["env_setup.bat"]

[target.linux-64.activation.env]
ENV_VAR = "linux-value"

# 你还可以引用现有的环境变量，但这必须
# 分别针对类 Unix 操作系统和 Windows 执行
[target.unix.activation.env]
ENV_VAR = "$OTHER_ENV_VAR/unix-value"

[target.win.activation.env]
ENV_VAR = "%OTHER_ENV_VAR%\\windows-value"
```

## `target` 表

`target` 表是一个允许进行平台特定配置的表。
它使你能够为每个平台设置不同的任务或依赖项。

`target` 表目前已经实现了以下子表：

- [`activation`](#activation)
- [`dependencies`](#dependencies)
- [`tasks`](#tasks)

`target` 表是使用 `[target.PLATFORM.SUB-TABLE]` 定义的。
例如，`[target.linux-64.dependencies]`

平台可以是以下之一：

- `win`、`osx`、`linux` 或 `unix`（`unix` 匹配 `linux` 和 `osx`）
- 或任何更具体的 [目标平台](#platforms)，例如 `linux-64`、`osx-arm64`

子表可以是上述任意之一。

为了更清楚，下面是一个例子。
目前，pixi 将顶级表（如 `dependencies`）与特定于目标的平台表合并为一个集合。
在依赖项的情况下，它们可以同时添加或覆盖依赖项。
在下面的例子中，我们使用 `cmake` 为所有目标，但在 `osx-64` 或 `osx-arm64` 上将选择不同版本的 Python。

```toml
[dependencies]
cmake = "3.26.4"
python = "3.10"

[target.osx.dependencies]
python = "3.11"
```

以下是更多的示例：

```toml
[target.win-64.activation]
scripts = ["setup.bat"]

[target.win-64.dependencies]
msmpi = "~=10.1.1"

[target.win-64.build-dependencies]
vs2022_win-64 = "19.36.32532"

[target.win-64.tasks]
tmp = "echo $TEMP"

[target.osx-64.dependencies]
clang = ">=16.0.6"
```

## `feature` 和 `environments` 表

`feature` 表允许你定义可以用来创建不同的 `[environments]` 的功能。
`[environments]` 表允许你定义不同的环境。其设计在 [此设计文档](../features/multi_environment.md) 中有详细解释。

```toml title="最简单的示例"
[feature.test.dependencies]
pytest = "*"

[environments]
test = ["test"]
```

这将创建一个名为 `test` 的环境，其中安装了 `pytest`。

### `feature` 表

`feature` 表允许你为每个功能定义以下字段：

- `dependencies`：与 [dependencies](#dependencies) 相同。
- `pypi-dependencies`：与 [pypi-dependencies](#pypi-dependencies) 相同。
- `pypi-options`：与 [pypi-options](#the-pypi-options-table) 相同。
- `system-requirements`：与 [system-requirements](#system-requirements) 相同。
- `activation`：与 [activation](#activation) 相同。
- `platforms`：与 [platforms](#platforms) 相同。如果没有覆盖，功能的 `platforms` 将与项目级别定义的相同。
- `channels`：与 [channels](#channels) 相同。如果没有覆盖，功能的 `channels` 将与项目级别定义的相同。
- `channel-priority`：与 [channel-priority](#channel-priority) 相同。
- `target`：与 [target](#target) 相同。
- `tasks`：与 [tasks](#tasks) 相同。

这些表也可以不带 `feature` 前缀使用。
当它们被使用时，我们称之为 `default` 功能。`default` 是一个受保护的名称，你不能为自己的功能使用此名称。

```toml title="Cuda 特性表示例"
[feature.cuda]
activation = {scripts = ["cuda_activation.sh"]}
# 默认情况下，结果是: ["nvidia", "conda-forge"]，当默认为 `conda-forge` 时
channels = ["nvidia"]
dependencies = {cuda = "x.y.z", cudnn = "12.0"}
pypi-dependencies = {torch = "==1.9.0"}
platforms = ["linux-64", "osx-arm64"]
system-requirements = {cuda = "12"}
tasks = { warmup = "python warmup.py" }
target.osx-arm64 = {dependencies = {mlx = "x.y.z"}}
```

```toml title="Cuda 特性表示例，但写作分开的表"
[feature.cuda.activation]
scripts = ["cuda_activation.sh"]

[feature.cuda.dependencies]
cuda = "x.y.z"
cudnn = "12.0"

[feature.cuda.pypi-dependencies]
torch = "==1.9.0"

[feature.cuda.system-requirements]
cuda = "12"

[feature.cuda.tasks]
warmup = "python warmup.py"

[feature.cuda.target.osx-arm64.dependencies]
mlx = "x.y.z"

# Channels 和 Platforms 不能作为单独的表格存在，因为它们是作为列表实现的
[feature.cuda]
channels = ["nvidia"]
platforms = ["linux-64", "osx-arm64"]
```

### `environments` 表

`[environments]` 表允许你定义使用 `[feature]` 表中定义的功能创建的环境。

`environments` 表使用以下字段进行定义：

- `features`：包含在环境中的功能。除非 `no-default-feature` 设置为 `true`，否则默认功能会隐式包含在环境中。
- `solve-group`：`solve-group` 用于在求解阶段将环境分组在一起。
  这对于需要具有相同依赖项但可能通过附加依赖项扩展它们的环境非常有用。
  例如，在测试生产环境时添加额外的测试依赖项。
  这些依赖项将会在所有具有相同求解组的环境中保持相同版本。
  但不同的环境包含不同子集的求解组依赖项集。
- `no-default-feature`：是否在该环境中包含默认功能。默认值为 `false`，表示包括默认功能。

```toml title="完整的 environments 表规范"
[environments]
test = {features = ["test"], solve-group = "test"}
prod = {features = ["prod"], solve-group = "test"}
lint = {features = ["lint"], no-default-feature = true}
```

如上例所示，在最简单的情况下，可以仅通过列出其功能来定义一个环境：

```toml title="最简单的示例"
[environments]
test = ["test"]
```

等同于

```toml title="最简单的示例扩展"
[environments]
test = {features = ["test"]}
```

当一个环境包含多个功能（包括默认功能）时：

- 环境的 `activation` 和 `tasks` 是其所有功能的 `activation` 和 `tasks` 的并集。
- 环境的 `dependencies` 和 `pypi-dependencies` 是其所有功能的 `dependencies` 和 `pypi-dependencies` 的并集。这意味着如果多个功能定义了对相同包的需求，这些需求将被合并。注意避免在添加到同一环境的功能之间出现冲突的需求。
- 环境的 `system-requirements` 是其所有功能的 `system-requirements` 的并集。如果多个功能指定了相同系统包的需求，则选择最高版本。
- 环境的 `channels` 是其所有功能的 `channels` 的并集。可以在每个功能中指定通道优先级，以确保在环境中按正确的顺序考虑通道。
- 环境的 `platforms` 是其所有功能的 `platforms` 的交集。请注意，功能支持的平台（包括默认功能）将被视为在项目级别定义的 `platforms`（除非在功能中被覆盖）。因此，通常最好将项目的 `platforms` 设置为它可以跨其环境支持的所有平台。

## 全局配置

全局配置选项在 [全局配置](../reference/pixi_configuration.md) 部分有文档说明。
