# `pyproject.toml` 在 pixi 中

我们支持在 pixi 中使用 `pyproject.toml` 作为清单文件。  
这允许用户将所有配置保存在一个文件中。  
`pyproject.toml` 文件是 Python 项目的标准。  
我们不建议将 `pyproject.toml` 文件用于 Python 以外的项目，`pixi.toml` 更适合其他类型的项目。

## `pyproject.toml` 文件的初始设置

当你在项目中已经有一个 `pyproject.toml` 文件时，可以在该文件所在的文件夹中运行 `pixi init`。 Pixi 会自动执行以下操作：

- 向文件中添加 `[tool.pixi.project]` 部分，包含 pixi 所需的平台和频道信息；
- 将当前项目添加为可编辑的 pypi 依赖；
- 向 `.gitignore` 和 `.gitattributes` 文件中添加一些默认值。

如果你没有现有的 `pyproject.toml` 文件，可以在项目文件夹中运行 `pixi init --format pyproject`。在这种情况下，pixi 会从头开始创建一个具有合理默认值的 `pyproject.toml` 清单。

## Python 依赖

`pyproject.toml` 文件支持 `requires_python` 字段。  
Pixi 能够理解该字段，并自动将版本添加到依赖项中。

以下是一个包含 `requires_python` 字段的 `pyproject.toml` 文件示例，它将作为 Python 依赖使用：

```toml title="pyproject.toml"
[project]
name = "my_project"
requires-python = ">=3.9"

[tool.pixi.project]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]
```

其等效于：

```toml title="equivalent pixi.toml"
[project]
name = "my_project"
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]

[dependencies]
python = ">=3.9"
```

## 依赖部分

`pyproject.toml` 文件支持 `dependencies` 字段。  
Pixi 能够理解该字段，并自动将依赖项添加到项目中作为 `[pypi-dependencies]`。

以下是一个包含 `dependencies` 字段的 `pyproject.toml` 文件示例：

```toml title="pyproject.toml"
[project]
name = "my_project"
requires-python = ">=3.9"
dependencies = [
    "numpy",
    "pandas",
    "matplotlib",
]

[tool.pixi.project]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]
```

其等效于：

```toml title="equivalent pixi.toml"
[project]
name = "my_project"
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]

[pypi-dependencies]
numpy = "*"
pandas = "*"
matplotlib = "*"

[dependencies]
python = ">=3.9"
```

你可以通过将它们添加到 `dependencies` 字段来使用 conda 依赖覆盖这些依赖：

```toml title="pyproject.toml"
[project]
name = "my_project"
requires-python = ">=3.9"
dependencies = [
    "numpy",
    "pandas",
    "matplotlib",
]

[tool.pixi.project]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]

[tool.pixi.dependencies]
numpy = "*"
pandas = "*"
matplotlib = "*"
```

这将导致安装 conda 依赖，并忽略 pypi 依赖。  
因为 pixi 会优先使用 conda 依赖，而不是 pypi 依赖。

## 可选依赖

如果你的 Python 项目包含一组可选依赖，pixi 将自动将它们解释为同名的 [pixi 特性](../reference/project_configuration.md#feature-environments)，并与关联的 `pypi-dependencies` 一起使用。

你可以手动将它们添加到 pixi 环境中，或者使用 `pixi init` 来设置项目，这样将为每个特性创建一个环境。对于其他可选依赖组的自引用也会得到处理。

例如，假设你的项目文件夹中有一个类似以下内容的 `pyproject.toml` 文件：

```toml
[project]
name = "my_project"
dependencies = ["package1"]

[project.optional-dependencies]
test = ["pytest"]
all = ["package2","my_project[test]"]
```

在该项目文件夹中运行 `pixi init` 将会将 `pyproject.toml` 文件转换为：

```toml
[project]
name = "my_project"
dependencies = ["package1"]

[project.optional-dependencies]
test = ["pytest"]
all = ["package2","my_project[test]"]

[tool.pixi.project]
channels = ["conda-forge"]
platforms = ["linux-64"] # 如果在 linux 上执行

[tool.pixi.environments]
default = {features = [], solve-group = "default"}
test = {features = ["test"], solve-group = "default"}
all = {features = ["all", "test"], solve-group = "default"}
```

在这个例子中，pixi 将创建三个环境：

- **default**：包含 `package1` 作为 pypi 依赖
- **test**：包含 `package1` 和 `pytest` 作为 pypi 依赖
- **all**：包含 `package1`、`package2` 和 `pytest` 作为 pypi 依赖

所有环境将一起求解，正如公共的 `solve-group` 所指示的那样，并添加到锁文件中。你可以手动编辑 `[tool.pixi.environments]` 部分，以适应你的用例（例如，如果你不需要某个特定环境）。

## 示例

由于 `pyproject.toml` 文件支持完整的 pixi 规范，并且会预加上 `[tool.pixi]`，以下是一个示例：

```toml title="pyproject.toml"
[project]
name = "my_project"
requires-python = ">=3.9"
dependencies = [
    "numpy",
    "pandas",
    "matplotlib",
    "ruff",
]

[tool.pixi.project]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]

[tool.pixi.dependencies]
compilers = "*"
cmake = "*"

[tool.pixi.tasks]
start = "python my_project/main.py"
lint = "ruff lint"

[tool.pixi.system-requirements]
cuda = "11.0"

[tool.pixi.feature.test.dependencies]
pytest = "*"

[tool.pixi.feature.test.tasks]
test = "pytest"

[tool.pixi.environments]
test = ["test"]
```

## 构建系统部分

`pyproject.toml` 文件通常包含一个 `[build-system]` 部分。如果该部分被添加为 pypi 路径依赖，pixi 将使用该部分来构建和安装项目。

如果 `pyproject.toml` 文件没有包含任何 `[build-system]` 部分，pixi 将回退到 [uv](https://github.com/astral-sh/uv) 的默认设置，等同于以下内容：

```toml title="pyproject.toml"
[build-system]
requires = ["setuptools >= 40.8.0"]
build-backend = "setuptools.build_meta:__legacy__"
```

强烈建议包括 `[build-system]` 部分。如果你不确定要使用的 [build-backend](https://packaging.python.org/en/latest/tutorials/packaging-projects/#choosing-build-backend)，可以在 `pyproject.toml` 中包含下面的 `[build-system]` 部分作为起点。  
`pixi init --format pyproject` 默认使用 `hatchling`。  
`hatchling` 相较于 `setuptools` 的优势可在其 [官网](https://hatch.pypa.io/latest/why/#build-backend) 中查看。

```toml title="pyproject.toml"
[build-system]
build-backend = "hatchling.build"
requires = ["hatchling"]
```
