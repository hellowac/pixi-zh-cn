# 从 `poetry` 迁移到 `pixi`

欢迎使用本指南，帮助你从 `poetry` 迁移到 `pixi`。
本文将比较这两个工具之间的关键命令和概念，重点介绍 `pixi` 在管理环境和包方面的独特方法。
通过 `pixi`，你将体验到类似 `poetry` 的项目工作流，同时能够使用 `conda` 生态系统，并轻松分享你的工作。

## 为什么选择 Pixi？
在 Python 生态系统中，`poetry` 是最接近 `pixi` 的项目管理工具。
除了支持 PyPI 生态系统，`pixi` 还加入了 `conda` 生态系统，使环境管理更灵活、更强大。

## 简要比较
| 任务                        | Poetry                                                              | Pixi                                                                                                                                              |
|-----------------------------|---------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| 创建环境                    | `poetry new myenv`                                                  | `pixi init myenv`                                                                                                                                 |
| 运行任务                    | `poetry run which python`                                           | `pixi run which python`<br/>`pixi` 使用内置的跨平台 shell 来运行，而 `poetry` 使用你的 shell。                                         |
| 安装包                      | `poetry add numpy`                                                  | `pixi add numpy` 会安装 conda 版本，`pixi add --pypi numpy` 安装 PyPI 版本。                                                                      |
| 卸载包                      | `poetry remove numpy`                                               | `pixi remove numpy` 会卸载 conda 版本，`pixi remove --pypi numpy` 会卸载 PyPI 版本。                                                               |
| 构建包                      | `poetry build`                                                      | `pixi` 目前尚未实现包的构建与发布功能                                                                                                            |
| 发布包                      | `poetry publish`                                                    | `pixi` 目前尚未实现包的构建与发布功能                                                                                                            |
| 读取 `pyproject.toml` 文件 | `[tool.poetry]`                                                     | `[tool.pixi]`                                                                                                                                     |
| 定义依赖                    | `[tool.poetry.dependencies]`                                        | `[tool.pixi.dependencies]` 用于 conda，`[tool.pixi.pypi-dependencies]` 或 `[project.dependencies]` 用于 PyPI 依赖                                   |
| 依赖定义                    | - `numpy = "^1.2.3"`<br/>- `numpy = "~1.2.3"`<br/>- `numpy = "*"`   | - `numpy = ">=1.2.3 <2.0.0"`<br/>- `numpy = ">=1.2.3 <1.3.0"`<br/>- `numpy = "*"`                                                                 |
| 锁文件                      | `poetry.lock`                                                       | `pixi.lock`                                                                                                                                       |
| 环境目录                    | `~/.cache/pypoetry/virtualenvs/myenv`                                | `./.pixi` 默认在项目文件夹中，可以通过 [`detached-environments`](../reference/pixi_configuration.md#detached-environments) 配置移动它。 |

## 在我的项目中同时支持 `poetry` 和 `pixi`
你可以允许用户在同一个项目中同时使用 `poetry` 和 `pixi`，它们不会互相干扰。
最好是复制依赖，将 `tool.poetry.dependencies` 中的内容精确地复制到 `tool.pixi.pypi-dependencies` 中。
确保 `python` 只在 `tool.pixi.dependencies` 中定义，而不是在 `tool.pixi.pypi-dependencies` 中。

!!! Danger "混用 `pixi` 和 `poetry`"

    虽然在 `pixi` 环境中使用 `poetry` 是可行的，但不推荐这么做。
    `pixi` 和 `poetry` 处理 PyPI 依赖的方式不同，混用它们可能导致意外行为。
    因为每次只能使用一个包管理工具，最好坚持使用一个工具。

    如果在 `pixi` 项目上使用 `poetry` ，你始终需要在 `pixi` 环境之后安装 `poetry` 环境。
    并让 `pixi` 处理 `python` 和 `poetry` 的安装。