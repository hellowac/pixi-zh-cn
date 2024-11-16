---
part: pixi
title: Commands
description: 所有 pixi cli 子命令
---

## 全局选项

- `--verbose (-v|vv|vvv)` 增加输出消息的详细程度，-v|vv|vvv 会分别增加详细程度。
- `--help (-h)` 显示帮助信息，使用 `-h` 获取简短版帮助。
- `--version (-V)`: 显示当前使用的 pixi 版本。
- `--quiet (-q)`: 减少输出信息。
- `--color <COLOR>`: 是否需要为日志着色 [env: `PIXI_COLOR=`] [默认: `auto`] [可选值: `always`, `never`, `auto`]。
  Pixi 还会尊重 `FORCE_COLOR` 和 `NO_COLOR` 环境变量。
  这两个变量优先于 `--color` 和 `PIXI_COLOR`。
- `--no-progress`: 禁用进度条。[env: `PIXI_NO_PROGRESS`] [默认: `false`]

## `init`

此命令用于创建一个新项目。
它会初始化一个 `pixi.toml` 文件，并准备一个 `.gitignore` 文件，以防止将环境添加到 `git` 中。

如果目录中存在 `pyproject.toml` 文件，`pixi init` 会将 pixi 数据追加到该文件，而不是创建一个新的 `pixi.toml` 文件。

##### 参数

1. `[PATH]`: 项目的存放位置（默认为当前路径） [默认: `.`]

##### 选项

- `--channel <CHANNEL> (-c)`: 指定项目使用的渠道。默认为 `conda-forge`。 (允许多次使用)
- `--platform <PLATFORM> (-p)`: 指定项目支持的平台。 (允许多次使用)
- `--import <ENV_FILE> (-i)`: 导入现有的 conda 环境文件，例如 `environment.yml`。
- `--format <FORMAT>`: 指定项目文件的格式，可以是 `pyproject` 或 `pixi`。[默认: `pixi`]
- `--scm <SCM>`: 指定用于管理项目的 SCM。可选值：github, gitlab, codeberg。[默认: `github`]

!!! info "导入 environment.yml"
  导入环境时，`pixi.toml` 将根据环境文件中的依赖项创建。
  安装环境时将创建 `pixi.lock` 文件。
  我们不支持将 `git+` URL 作为 pip 包的依赖，对于 `defaults` 渠道，我们使用 `main`、`r` 和 `msys2` 作为默认渠道。

```shell
pixi init myproject
pixi init ~/myproject
pixi init  # 直接在当前目录初始化。
pixi init --channel conda-forge --channel bioconda myproject
pixi init --platform osx-64 --platform linux-64 myproject
pixi init --import environment.yml
pixi init --format pyproject
pixi init --format pixi --scm gitlab
```

## `add`

向 [清单文件](project_configuration.md) 添加依赖项。
只有当包及其版本约束能与项目中的其他依赖项兼容时，才会被添加。
[更多信息](../features/multi_platform_configuration.md) 关于多平台配置。

如果项目清单是 `pyproject.toml`，添加一个 pypi 依赖时，会将其添加到原生 pyproject 的 `project.dependencies` 数组中，或者如果指定了功能，则添加到原生的 `project.optional-dependencies` 表中：

- `pixi add --pypi boto3` 会将 `boto3` 添加到 `project.dependencies` 数组中
- `pixi add --pypi boto3 --feature aws` 会将 `boto3` 添加到 `project.dependencies.aws` 数组中

这些依赖将被 pixi 作为已添加到默认或命名功能的 pixi `pypi-dependencies` 表中读取。

##### 参数

1. `[SPECS]`: 要添加的包（空格分隔）。版本约束是可选的。

##### 选项

- `--manifest-path <MANIFEST_PATH>`: [清单文件](project_configuration.md)的路径，默认情况下会在父目录中查找该文件。
- `--host`: 指定主机依赖项，重要用于构建包。
- `--build`: 指定构建依赖项，重要用于构建包。
- `--pypi`: 指定 PyPI 依赖项，而非 conda 包。
  按照 [PEP508](https://peps.python.org/pep-0508/) 要求解析依赖项，支持额外的功能和版本。
  详见 [配置](project_configuration.md)。
- `--no-install`: 不安装包到环境中，只将包添加到锁文件中。
- `--no-lockfile-update`: 不更新锁文件，意味着会使用 `--no-install` 标志。
- `--platform <PLATFORM> (-p)`: 要为其添加依赖项的平台。 (允许多次使用)
- `--feature <FEATURE> (-f)`: 要为其添加依赖项的功能。
- `--editable`: 指定可编辑的依赖项，仅在与 `--pypi` 一起使用时有效。

```shell
pixi add numpy # (1)!
pixi add numpy pandas "pytorch>=1.8" # (2)!
pixi add "numpy>=1.22,<1.24" # (3)!
pixi add --manifest-path ~/myproject/pixi.toml numpy # (4)!
pixi add --host "python>=3.9.0" # (5)!
pixi add --build cmake # (6)!
pixi add --platform osx-64 clang # (7)!
pixi add --no-install numpy # (8)!
pixi add --no-lockfile-update numpy # (9)!
pixi add --feature featurex numpy # (10)!

# 添加一个 pypi 依赖
pixi add --pypi requests[security] # (11)!
pixi add --pypi Django==5.1rc1 # (12)!
pixi add --pypi "boltons>=24.0.0" --feature lint # (13)!
pixi add --pypi "boltons @ https://files.pythonhosted.org/packages/46/35/e50d4a115f93e2a3fbf52438435bb2efcf14c11d4fcd6bdcd77a6fc399c9/boltons-24.0.0-py3-none-any.whl" # (14)!
pixi add --pypi "exchangelib @ git+https://github.com/ecederstrand/exchangelib" # (15)!
pixi add --pypi "project @ file:///absolute/path/to/project" # (16)!
pixi add --pypi "project@file:///absolute/path/to/project" --editable # (17)!
```

1. 这会将 `numpy` 包添加到项目中，使用解决后的环境中的最新版本。
2. 这会将多个包一起添加到项目中，并一起解决它们。
3. 这会将 `numpy` 包添加到项目中，并指定版本约束。
4. 这会将 `numpy` 包添加到给定路径的项目的清单文件中。
5. 这会将 `python` 包作为主机依赖项添加。目前主机依赖项没有不同的行为。
6. 这会将 `cmake` 包作为构建依赖项添加。目前构建依赖项没有不同的行为。
7. 这会将 `clang` 包仅为 `osx-64` 平台添加。
8. 这会将 `numpy` 包添加到清单和锁文件中，但不安装到环境中。
9. 这会将 `numpy` 包添加到清单中，但不更新锁文件，也不安装到环境中。
10. 这会将 `numpy` 包添加到 `featurex` 功能中。
11. 这会将 `requests` 包作为 `pypi` 依赖项，附带 `security` 额外功能。
12. 这会将 `Django` 的预发布版本添加到项目中作为 `pypi` 依赖项。
13. 这会将 `boltons` 包作为 `pypi` 依赖项，在 `lint` 功能中。
14. 这会将通过给定 URL 添加 `boltons` 包作为 `pypi` 依赖项。
15. 这会将通过给定 Git URL 添加 `exchangelib` 包作为 `pypi` 依赖项。
16. 这会将通过给定文件 URL 添加 `project` 包作为 `pypi` 依赖项。
17. 这会将通过给定文件 URL 添加 `project` 包作为可编辑的 `pypi` 依赖项。

!!! tip
    如果你想使用非默认的固定策略，可以通过 [pixi 配置](./pixi_configuration.md#pinning-strategy) 设置：
    ```
    pixi config set pinning-strategy no-pin --global
    ```
    默认是 `semver`，这将把依赖项固定到最新的

主版本或次版本（对于 `v0` 版本）。
    !!! note
        当添加一个我们定义为非 `semver` 的包时，会使用 `minor` 策略，除非该包是以下非 `semver` 包：
        Python、Rust、Julia、GCC、GXX、GFortran、NodeJS、Deno、R、R-Base、Perl


## `install`

根据 [清单文件](project_configuration.md) 安装环境。
如果没有 `pixi.lock` 文件，或者它没有与 [清单文件](project_configuration.md) 保持同步，它将重新生成锁定文件。

`pixi install` 每次只安装一个环境，如果有多个环境，可以使用 `--environment` 标志选择正确的环境。
如果没有提供环境，默认环境将被安装。

运行 `pixi install` 并不是其他命令执行前的必需操作。
所有与环境交互的命令在环境未准备好时会首先运行 `install` 命令，以确保始终在正确的状态下运行。
例如：`pixi run`、`pixi shell`、`pixi shell-hook`、`pixi add`、`pixi remove` 等。

##### 选项
- `--manifest-path <MANIFEST_PATH>`: [清单文件](project_configuration.md) 的路径，默认情况下会在父目录中查找该文件。
- `--frozen`: 按照锁定文件中的定义安装环境，如果锁定文件与 [清单文件](project_configuration.md) 不同步，则不会更新 `pixi.lock`。也可以通过 `PIXI_FROZEN` 环境变量来控制（示例：`PIXI_FROZEN=true`）。
- `--locked`: 只有在 `pixi.lock` 与 [清单文件](project_configuration.md) 保持同步时才安装【^1】。也可以通过 `PIXI_LOCKED` 环境变量来控制（示例：`PIXI_LOCKED=true`）。与 `--frozen` 参数冲突。
- `--environment <ENVIRONMENT> (-e)`: 要安装的环境，如果没有提供，默认环境将被使用。

```shell
pixi install
pixi install --manifest-path ~/myproject/pixi.toml
pixi install --frozen
pixi install --locked
pixi install --environment lint
pixi install -e lint
```

## `update`

`update` 命令检查是否有更新版本的依赖项，并相应地更新 `pixi.lock` 文件和环境。
只有在 [清单文件](project_configuration.md) 中的依赖项与新版本兼容时，才会更新锁定文件。

##### 参数

1. `[PACKAGES]...` 要更新的包，以空格分隔。如果没有提供包，所有包将会被更新。

##### 选项
- `--manifest-path <MANIFEST_PATH>`: [清单文件](project_configuration.md) 的路径，默认情况下会在父目录中查找该文件。
- `--environment <ENVIRONMENT> (-e)`: 要更新的环境，如果没有提供，则所有环境都会更新。
- `--platform <PLATFORM> (-p)`: 要更新依赖项的平台。
- `--dry-run (-n)`: 仅显示将要进行的更改，而不实际更新锁定文件或环境。
- `--no-install`: 不安装解决 pypi-dependencies 所需的环境。
- `--json`: 以 JSON 格式输出更改。

```shell
pixi update numpy
pixi update numpy pandas
pixi update --manifest-path ~/myproject/pixi.toml numpy
pixi update --environment lint python
pixi update -e lint -e schema -e docs pre-commit
pixi update --platform osx-arm64 mlx
pixi update -p linux-64 -p osx-64 numpy
pixi update --dry-run
pixi update --no-install boto3
```

## `upgrade`

`upgrade` 命令检查是否有更新版本的依赖项，并在 [清单文件](project_configuration.md) 中将其升级。
`update` 会更新锁定文件中的依赖项，同时仍然满足清单中设置的版本要求。
`upgrade` 会放宽给定包的要求，更新锁定文件并相应地调整清单。

##### 参数

1. `[PACKAGES]...` 要升级的包，以空格分隔。如果没有提供包，所有包都会被升级。

##### 选项
- `--manifest-path <MANIFEST_PATH>`: [清单文件](project_configuration.md) 的路径，默认情况下会在父目录中查找该文件。
- `--feature <FEATURE> (-e)`: 要升级的功能，如果没有提供，则使用默认功能。
- `--no-install`: 不安装解决 pypi-dependencies 所需的环境。
- `--json`: 以 JSON 格式输出更改。
- `--dry-run (-n)`: 仅显示将要进行的更改，而不实际更新清单、锁定文件或环境。

```shell
pixi upgrade
pixi upgrade numpy
pixi upgrade numpy pandas
pixi upgrade --manifest-path ~/myproject/pixi.toml numpy
pixi upgrade --feature lint python
pixi upgrade --json
pixi upgrade --dry-run
```

!!! note
    `pixi upgrade` 命令只会更新 `version`，除非指定了确切的包名（例如：`pixi upgrade numpy`）。

    在这种情况下，它将删除所有字段，除了：
    - `build` 字段，包含通配符 `*`
    - `channel`
    - `file_name`
    - `url`
    - `subdir`

## `run`

`run` 命令首先检查环境是否已准备好使用。
如果没有运行 `pixi install`，`run` 命令会为你执行此操作。
在 [清单文件](project_configuration.md) 中定义的自定义任务也可以通过 `run` 命令执行。

你不能运行 `pixi run source setup.bash`，因为 `source` 在 `deno_task_shell` 命令中不可用，并且不是一个可执行文件。

##### 参数

1. `[TASK]...` 你希望在项目环境中运行的任务，这也可以是一个普通命令。所有任务后的参数都会传递给该任务。

##### 选项

- `--manifest-path <MANIFEST_PATH>`: [清单文件](project_configuration.md) 的路径，默认情况下会在父目录中查找该文件。
- `--frozen`: 按照锁定文件中的定义安装环境，如果锁定文件与 [清单文件](project_configuration.md) 不同步，则不会更新 `pixi.lock`。也可以通过 `PIXI_FROZEN` 环境变量来控制（示例：`PIXI_FROZEN=true`）。
- `--locked`: 只有在 `pixi.lock` 与 [清单文件](project_configuration.md) 保持同步时才安装【^1】。也可以通过 `PIXI_LOCKED` 环境变量来控制（示例：`PIXI_LOCKED=true`）。与 `--frozen` 参数冲突。
- `--environment <ENVIRONMENT> (-e)`: 要运行任务的环境，如果没有提供，默认环境将被使用，或者会提供选择器来选择正确的环境。
- `--clean-env`: 在干净的环境中运行任务，这将删除所有除 `pixi` 设置的环境变量外的 shell 环境变量。此功能在 `Windows` 上不可用。
- `--revalidate`: 重新验证完整环境，而不是检查锁定文件的哈希值。[更多信息](../features/environment.md#environment-installation-metadata)

```shell
pixi run python
pixi run cowpy "Hey pixi user"
pixi run --manifest-path ~/myproject/pixi.toml python
pixi run --frozen python
pixi run --locked python
# 如果你在 pixi.toml 中指定了自定义任务，也可以使用 run 执行
pixi run build
# 额外的参数将传递给任务命令。
pixi run task argument1 argument2

# 如果有多个环境，可以使用 --environment 标志选择正确的环境。
pixi run --environment cuda python

# 此功能在 Windows 上不可用
# 如果想在干净的环境中运行命令，可以使用 --clean-env 标志。
# PATH 应仅包含 pixi 环境。
pixi run --clean-env "echo \$PATH"
```

!!! info
    在 `pixi` 中， [`deno_task_shell`](https://deno.land/manual@v1.35.0/tools/task_runner#task-runner) 是 `run` 命令的底层运行器。
    查看他们的 [文档](https://deno.land/manual@v1.35.0/tools/task_runner#task-runner) 以了解语法和可用的命令。
    这样做是为了确保 `run` 命令可以在所有平台上运行。

!!! tip "跨环境任务"
    如果你使用了 `tasks` 的 `depends-on` 特性，任务将按照你指定的顺序执行。
    `depends-on` 可以跨环境使用，例如，你有如下的 `pixi.toml`：
    ??? "pixi.toml"
        ```toml
        [tasks]
        start = { cmd = "python start.py", depends-on = ["build"] }

        [feature.build.tasks]
        build = "cargo build"
        [feature.build.dependencies]
        rust = ">=1.74"

        [environments]
        build = ["build"]
        ```

        然后你可以从 `build` 环境运行 `build`，并从默认环境运行 `start`。
        只需调用：
        ```shell
        pixi run start
        ```

## `exec`

在一个临时环境中运行命令，该环境与任何项目都断开连接。
这对于快速测试某个包或版本非常有用。

临时环境会被缓存。如果再次运行相同的命令，将会重用相同的环境。

??? note "清理临时环境"
    目前，临时环境只能手动清理。
    `pixi exec` 的环境存储在缓存目录下的 `cached-envs-v0/` 中。
    运行 `pixi info` 以查找缓存目录。

##### 参数

1. `<COMMAND>`: 要运行的命令。

#### 选项：
* `--spec <SPECS> (-s)`: 要安装的包的匹配规格。如果未提供，则从命令中推测包。
* `--channel <CHANNELS> (-c)`: 要安装包的频道。如果未指定，将使用默认频道。
* `--force-reinstall`: 如果指定，始终创建一个新环境，即使已经存在。

```shell
pixi exec python

# 为 Python 版本添加约束
pixi exec -s python=3.9 python

# 运行 ipython 并在环境中包含 py-rattler 包
pixi exec -s ipython -s py-rattler ipython

# 强制重新安装以重建环境并获取最新的包版本
pixi exec --force-reinstall -s ipython -s py-rattler ipython
```

## `remove`

从 [清单文件](project_configuration.md) 中移除依赖项。

如果项目清单是 `pyproject.toml`，使用 `--pypi` 标志移除一个 PyPI 依赖项时，它将从以下位置之一移除：

- 原生 pyproject 的 `project.dependencies` 数组或原生的 `project.optional-dependencies` 表（如果指定了特性）
- pixi 的默认或指定特性的 `pypi-dependencies` 表（如果指定了特性）

##### 参数

1. `<DEPS>...`: 要从项目中移除的依赖项列表。

##### 选项

- `--manifest-path <MANIFEST_PATH>`: [清单文件](project_configuration.md) 的路径，默认情况下会在父目录中查找该文件。
- `--host`: 指定主机依赖项，对于构建包很重要。
- `--build`: 指定构建依赖项，对于构建包很重要。
- `--pypi`: 指定一个 PyPI 依赖项，而非 conda 包。
- `--platform <PLATFORM> (-p)`: 要移除依赖项的平台。
- `--feature <FEATURE> (-f)`: 要移除依赖项的特性。
- `--no-install`: 不安装环境，仅从锁定文件和清单文件中移除包。
- `--no-lockfile-update`: 不更新锁定文件，意味着同时启用 `--no-install` 标志。

```shell
pixi remove numpy
pixi remove numpy pandas pytorch
pixi remove --manifest-path ~/myproject/pixi.toml numpy
pixi remove --host python
pixi remove --build cmake
pixi remove --pypi requests
pixi remove --platform osx-64 --build clang
pixi remove --feature featurex clang
pixi remove --feature featurex --platform osx-64 clang
pixi remove --feature featurex --platform osx-64 --build clang
pixi remove --no-install numpy
```

## `task`

如果你希望为特定命令创建简短别名，可以为其添加任务。

##### 选项

- `--manifest-path <MANIFEST_PATH>`: [清单文件](project_configuration.md) 的路径，默认情况下会在父目录中查找该文件。

### `task add`

向 [清单文件](project_configuration.md) 中添加任务，使用 `--depends-on` 来添加你希望在此任务之前运行的任务，例如在执行任务之前先构建。

##### 参数

1. `<NAME>`: 任务的名称。
2. `<COMMAND>`: 要运行的命令。这可以是多个单词。
!!! info
    如果你在任务中使用 `$` 来表示环境变量，环境变量会在添加任务之前被解析。
    如果你希望在任务中使用 `$`，需要用 `\` 进行转义，例如 `echo \$HOME`。

##### 选项

- `--platform <PLATFORM> (-p)`: 为哪个平台添加此任务。
- `--feature <FEATURE> (-f)`: 为哪个特性添加任务，如果没有提供，则会添加默认任务。
- `--depends-on <DEPENDS_ON>`: 该任务所依赖的任务，依赖的任务会在添加的任务之前运行。
- `--cwd <CWD>`: 任务的工作目录，相对于项目根目录。
- `--env <ENV>`: 任务的环境变量，格式为 `key=value` 对，可以多次使用，例如 `--env "VAR1=VALUE1" --env "VAR2=VALUE2"`。
- `--description <DESCRIPTION>`: 任务的描述。

```shell
pixi task add cow cowpy "Hello User"
pixi task add tls ls --cwd tests
pixi task add test cargo t --depends-on build
pixi task add build-osx "METAL=1 cargo build" --platform osx-64
pixi task add train python train.py --feature cuda
pixi task add publish-pypi "hatch publish --yes --repo main" --feature build --env HATCH_CONFIG=config/hatch.toml --description "Publish the package to pypi"
```

这将添加以下内容到 [清单文件](project_configuration.md)：

```toml
[tasks]
cow = "cowpy \"Hello User\""
tls = { cmd = "ls", cwd = "tests" }
test = { cmd = "cargo t", depends-on = ["build"] }

[target.osx-64.tasks]
build-osx = "METAL=1 cargo build"

[feature.cuda.tasks]
train = "python train.py"

[feature.build.tasks]
publish-pypi = { cmd = "hatch publish --yes --repo main", env = { HATCH_CONFIG = "config/hatch.toml" }, description = "Publish the package to pypi" }
```

然后你可以使用 `run` 命令运行它：

```shell
pixi run cow
# 额外的参数将传递给任务命令。
pixi run test --test test1
```

### `task remove`

从 [清单文件](project_configuration.md) 中移除任务。

##### 参数

- `<NAMES>`: 任务名称，空格分隔。

##### 选项

- `--platform <PLATFORM> (-p)`: 为哪个平台移除此任务。
- `--feature <FEATURE> (-f)`: 为哪个特性移除此任务。

```shell
pixi task remove cow
pixi task remove --platform linux-64 test
pixi task remove --feature cuda task
```

### `task alias`

为任务创建别名。

##### 参数

1. `<ALIAS>`: 别名名称
2. `<DEPENDS_ON>`: 要在此别名下执行的任务名称，顺序重要，第一个执行的任务排在最前面。

##### 选项

- `--platform <PLATFORM> (-p)`: 为哪个平台创建此别名。

```shell
pixi task alias test-all test-py test-cpp test-rust
pixi task alias --platform linux-64 test test-linux
pixi task alias moo cow
```

### `task list`

列出项目中的所有任务。

##### 选项

- `--environment`(`-e`): 环境的任务列表，如果未提供，则列出默认任务。
- `--summary`(`-s`): 按环境列出任务。

```shell
pixi task list
pixi task list --environment cuda
pixi task list --summary
```

## `list`

列出项目的包。高亮显示的包是显式依赖项。

##### 参数

1. `[REGEX]`: 仅列出匹配正则表达式的包（可选）。

##### 选项

- `--platform <PLATFORM> (-p)`: 要列出包的平台。默认是当前平台。
- `--json`: 是否以 JSON 格式输出。
- `--json-pretty`: 是否以美化的 JSON 格式输出。
- `--sort-by <SORT_BY>`: 排序策略 [默认: name] [可能的值: size, name, type]。
- `--explicit (-x)`: 仅列出显式添加到 [清单文件](project_configuration.md) 的包。
- `--manifest-path <MANIFEST_PATH>`: [清单文件](project_configuration.md) 的路径，默认情况下会在父目录中查找该文件。
- `--environment (-e)`: 要列出包的环境，如果未提供，则列出默认环境的包。
- `--frozen`: 按照锁定文件中定义的环境安装，不更新 `pixi.lock`，如果它与 [清单文件](project_configuration.md) 不匹配。也可以通过 `PIXI_FROZEN` 环境变量控制（例如：`PIXI_FROZEN=true`）。
- `--locked`: 仅在 `pixi.lock` 文件与 [清单文件](project_configuration.md) 保持一致时安装[^1]。也可以通过 `PIXI_LOCKED` 环境变量控制（例如：`PIXI_LOCKED=true`）。与 `--frozen` 冲突。
- `--no-install`: 不安装环境，仅在无需安装时更新锁定文件（由 `--frozen` 和 `--locked` 隐式要求）。

```shell
pixi list
pixi list py
pixi list --json-pretty
pixi list --explicit
pixi list --sort-by size
pixi list --platform win-64
pixi list --environment cuda
pixi list --frozen
pixi list --locked
pixi list --no-install
```

输出将如下所示，其中 `python` 会以绿色显示，因为它是显式添加到 [清单文件](project_configuration.md) 中的包：

```shell
➜ pixi list
 Package           Version     Build               Size       Kind   Source
 _libgcc_mutex     0.1         conda_forge         2.5 KiB    conda  _libgcc_mutex-0.1-conda_forge.tar.bz2
 _openmp_mutex     4.5         2_gnu               23.1 KiB   conda  _openmp_mutex-4.5-2_gnu.tar.bz2
 bzip2             1.0.8       hd590300_5          248.3 KiB  conda  bzip2-1.0.8-hd590300_5.conda
 ca-certificates   2023.11.17  hbcca054_0          150.5 KiB  conda  ca-certificates-2023.11.17-hbcca054_0.conda
 ld_impl_linux-64  2.40        h41732ed_0          688.2 KiB  conda  ld_impl_linux-64-2.40-h41732ed_0.conda
 libexpat          2.5.0       hcb278e6_1          76.2 KiB   conda  libexpat-2.5.0-hcb278e6_1.conda
 libffi            3.4.2       h7f98852_5          56.9 KiB   conda  libffi-3.4.2-h7f98852_5.tar.bz2
 libgcc-ng         13.2.0      h807b86a_4          755.7 KiB  conda  libgcc-ng-13.2.0-h807b86a_4.conda
 libgomp           13.2.0      h807b86a_4          412.2 KiB  conda  libgomp-13.2.0-h807b86a_4.conda
 libnsl            2.0.1       hd590300_0          32.6 KiB   conda  libnsl-2.0.1-hd590300_0.conda
 libsqlite         3.44.2      h2797004_0          826 KiB    conda  libsqlite-3.44.2-h2797004_0.conda
 libuuid           2.38.1      h0b41bf4_0          32.8 KiB   conda  libuuid-2.38.1-h0b41bf4_0.conda
 libxcrypt         4.4.36      hd590300_1          98 KiB     conda  libxcrypt-4.4.36-hd590300_1.conda
 libzlib           1.2.13      hd590300_5          60.1 KiB   conda  libzlib-1.2.13-hd590300_5.conda
 ncurses           6.4         h59595ed_2          863.7 KiB  conda  ncurses-6.4-h59595ed_2.conda
 openssl           3.2.0       hd590300_1          2.7 MiB    conda  openssl-3.2.0-hd590300_1.conda
 python            3.12.1      hab00c5b_1_cpython  30.8 MiB   conda  python-3.12.1-hab00c5b_1_cpython.conda
 readline          8.2         h8228510_1          274.9 KiB  conda  readline-8.2-h8228510_1.conda
 tk                8.6.13      noxft_h4845f30_101  3.2 MiB    conda  tk-8.6.13-noxft_h4845f30_101.conda
 tzdata            2023d       h0c530f3_0          116.8 KiB  conda  tzdata-2023d-h0c530f3_0.conda
 xz                5.2.6       h166bdaf_0          408.6 KiB  conda  xz-5.2.6-h166bdaf_0.tar.bz2
```

## `tree`

以树状图显示项目的包。高亮显示的包是那些在清单中指定的。

包的树状结构也可以反转（`-i`），查看哪些包依赖于特定的依赖项。

##### 参数

- `REGEX`：可选的正则表达式，用于过滤树中显示的依赖项，或者在反转树时，确定从哪些依赖项开始显示。

##### 选项

- `--invert (-i)`：反转依赖树，给定一个匹配某些包的 `REGEX` 模式，显示所有依赖于这些包的包。
- `--platform <PLATFORM> (-p)`：列出特定平台的包。默认为当前平台。
- `--manifest-path <MANIFEST_PATH>`：清单文件的路径，默认会在父目录中搜索。
- `--environment (-e)`：列出特定环境的包，如果没有提供，则列出默认环境的包。
- `--frozen`：按照锁定文件中的定义安装环境，如果 `pixi.lock` 没有与清单文件同步，安装时不会更新 `pixi.lock`。也可以通过 `PIXI_FROZEN` 环境变量来控制（例如：`PIXI_FROZEN=true`）。
- `--locked`：仅在 `pixi.lock` 与清单文件同步时安装。也可以通过 `PIXI_LOCKED` 环境变量来控制（例如：`PIXI_LOCKED=true`）。与 `--frozen` 选项冲突。
- `--no-install`：不为 pypi 解决方案安装环境，仅在无需安装的情况下更新锁定文件。（`--frozen` 和 `--locked` 隐式启用）

```shell
pixi tree
pixi tree pre-commit
pixi tree -i yaml
pixi tree --environment docs
pixi tree --platform win-64
```

!!! warning
    使用 `-v` 显示哪些 `pypi` 包尚未正确解析。`extras` 和 `markers` 的解析仍在开发中。

输出将如下所示，其中直接在 [清单文件](project_configuration.md) 中指定的包将以绿色显示。
一旦包被显示，树状图将不再递归显示其依赖项（请比较第一次显示 `python` 时和其后的显示），而是会用星号 `(*)` 标记。

版本号根据包的类型进行着色，黄色表示 Conda 包，蓝色表示 PyPI 包。

```shell
➜ pixi tree
├── pre-commit v3.3.3
│   ├── cfgv v3.3.1
│   │   └── python v3.12.2
│   │       ├── bzip2 v1.0.8
│   │       ├── libexpat v2.6.2
│   │       ├── libffi v3.4.2
│   │       ├── libsqlite v3.45.2
│   │       │   └── libzlib v1.2.13
│   │       ├── libzlib v1.2.13 (*)
│   │       ├── ncurses v6.4.20240210
│   │       ├── openssl v3.2.1
│   │       ├── readline v8.2
│   │       │   └── ncurses v6.4.20240210 (*)
│   │       ├── tk v8.6.13
│   │       │   └── libzlib v1.2.13 (*)
│   │       └── xz v5.2.6
│   ├── identify v2.5.35
│   │   └── python v3.12.2 (*)
...
└── tbump v6.9.0
...
    └── tomlkit v0.12.4
        └── python v3.12.2 (*)
```

可以指定正则表达式模式，过滤树只显示特定的直接或传递依赖项：

```shell
➜ pixi tree pre-commit
└── pre-commit v3.3.3
    ├── virtualenv v20.25.1
    │   ├── filelock v3.13.1
    │   │   └── python v3.12.2
    │   │       ├── libexpat v2.6.2
    │   │       ├── readline v8.2
    │   │       │   └── ncurses v6.4.20240210
    │   │       ├── libsqlite v3.45.2
    │   │       │   └── libzlib v1.2.13
    │   │       ├── bzip2 v1.0.8
    │   │       ├── libzlib v1.2.13 (*)
    │   │       ├── libffi v3.4.2
    │   │       ├── tk v8.6.13
    │   │       │   └── libzlib v1.2.13 (*)
    │   │       ├── xz v5.2.6
    │   │       ├── ncurses v6.4.20240210 (*)
    │   │       └── openssl v3.2.1
    │   ├── platformdirs v4.2.0
    │   │   └── python v3.12.2 (*)
    │   ├── distlib v0.3.8
    │   │   └── python v3.12.2 (*)
    │   └── python v3.12.2 (*)
    ├── pyyaml v6.0.1
...
```

此外，树形结构还可以反转，并显示哪些包依赖于正则表达式模式。
在这种情况下，清单中指定的包也会高亮显示（此例中为 `cffconvert` 和 `pre-commit`）。

```shell
➜ pixi tree -i yaml

ruamel.yaml v0.18.6
├── pykwalify v1.8.0
│   └── cffconvert v2.0.0
└── cffconvert v2.0.0

pyyaml v6.0.1
└── pre-commit v3.3.3

ruamel.yaml.clib v0.2.8
└── ruamel.yaml v0.18.6
    ├── pykwalify v1.8.0
    │   └── cffconvert v2.0.0
    └── cffconvert v2.0.0

yaml v0.2.5
└── pyyaml v6.0.1
    └── pre-commit v3.3.3
```

## `shell`

此命令启动一个新的 shell，并进入项目的环境中。
要退出 pixi shell，只需运行 `exit`。

##### 选项

- `--change-ps1 <true 或 false>`：如果设置为 false，将移除 shell 提示符中的 `(pixi)` 前缀（默认：`true`）。默认行为可以在 [全局配置](pixi_configuration.md#change-ps1) 中进行配置。
- `--manifest-path <MANIFEST_PATH>`：清单文件的路径，默认会在父目录中搜索。
- `--frozen`：按照锁定文件中的定义安装环境，如果 `pixi.lock` 与清单文件不一致，则不会更新 `pixi.lock`。也可以通过 `PIXI_FROZEN` 环境变量来控制（例如：`PIXI_FROZEN=true`）。
- `--locked`：仅当 `pixi.lock` 与清单文件同步时才安装。也可以通过 `PIXI_LOCKED` 环境变量来控制（例如：`PIXI_LOCKED=true`）。与 `--frozen` 冲突。
- `--environment <ENVIRONMENT> (-e)`：激活 shell 的环境，如果未提供，则使用默认环境，或者提供选择器以选择正确的环境。
- `--revalidate`：重新验证整个环境，而不是检查锁定文件的哈希值。[更多信息](../features/environment.md#environment-installation-metadata)

```shell
pixi shell
exit
pixi shell --manifest-path ~/myproject/pixi.toml
exit
pixi shell --frozen
exit
pixi shell --locked
exit
pixi shell --environment cuda
exit
```

## `shell-hook`

此命令打印环境的激活脚本。

##### 选项

- `--shell <SHELL> (-s)`：指定应打印的激活脚本的 shell 类型。默认为当前 shell。
  当前支持的变体：[ `bash`, `zsh`, `xonsh`, `cmd`, `powershell`, `fish`, `nushell` ]
- `--manifest-path`：清单文件的路径，默认会在父目录中搜索。
- `--frozen`：按照锁定文件中的定义安装环境，如果 `pixi.lock` 与清单文件不一致，则不会更新 `pixi.lock`。也可以通过 `PIXI_FROZEN` 环境变量来控制（例如：`PIXI_FROZEN=true`）。
- `--locked`：仅当 `pixi.lock` 与清单文件同步时才安装。也可以通过 `PIXI_LOCKED` 环境变量来控制（例如：`PIXI_LOCKED=true`）。与 `--frozen` 冲突。
- `--environment <ENVIRONMENT> (-e)`：激活的环境，如果未提供，则使用默认环境，或者提供选择器以选择正确的环境。
- `--json`：以 JSON 格式打印运行激活脚本时导出的所有环境变量。指定此选项时，`--shell` 会被忽略。
- `--revalidate`：重新验证整个环境，而不是检查锁定文件的哈希值。[更多信息](../features/environment.md#environment-installation-metadata)

```shell
pixi shell-hook
pixi shell-hook --shell bash
pixi shell-hook --shell zsh
pixi shell-hook -s powershell
pixi shell-hook --manifest-path ~/myproject/pixi.toml
pixi shell-hook --frozen
pixi shell-hook --locked
pixi shell-hook --environment cuda
pixi shell-hook --json
```

使用示例：当你希望在 Docker 容器中不使用 `pixi` 可执行文件时。

```shell
pixi shell-hook --shell bash > /etc/profile.d/pixi.sh
rm ~/.pixi/bin/pixi # 现在环境将在没有 pixi 可执行文件的情况下激活。
```

## `search`

搜索包，输出将列出包的最新版本。

##### 参数

1. `<PACKAGE>`：要搜索的包名称，支持使用通配符（`*`）。

###### 选项

- `--manifest-path <MANIFEST_PATH>`：清单文件的路径，默认会在父目录中搜索。
- `--channel <CHANNEL> (-c)`：指定项目使用的频道。默认为 `conda-forge`。（可以多次使用）
- `--limit <LIMIT> (-l)`：可选的，限制搜索结果的数量。
- `--platform <PLATFORM> (-p)`：指定要搜索的特定平台。（默认：当前平台）

```zsh
pixi search pixi
pixi search --limit 30 "py*"
# 在不同的频道和特定平台上搜索
pixi search -c robostack --platform linux-64 "plotjuggler*"
```

## `self-update`

更新 pixi 到最新版本或指定版本。如果 pixi 是通过其他包管理器安装的，则此功能可能不可用，应该使用安装时所用的包管理器更新 pixi。

##### 选项

- `--version <VERSION>`：所需的版本（用于降级或升级）。如果未指定，则更新到最新版本。

```shell
pixi self-update
pixi self-update --version 0.13.0
```

## `info`

显示关于 pixi 安装、缓存目录、磁盘使用情况等的有用信息。
更多信息 [请参见此处](../advanced/explain_info_command.md)。

##### 选项

- `--manifest-path <MANIFEST_PATH>`：清单文件的路径，默认会在父目录中搜索。
- `--extended`：扩展信息，进行更多较慢的系统查询，如目录大小。
- `--json`：以机器可读的格式输出信息。

```shell
pixi info
pixi info --json --extended
```

## `clean`

清理系统中由 pixi 操作的部分。
默认会清理环境和任务缓存。
使用 `cache` 子命令清理缓存。

##### 选项
- `--manifest-path <MANIFEST_PATH>`：清单文件的路径，默认会在父目录中搜索。
- `--environment <ENVIRONMENT> (-e)`：要清理的环境，如果未提供，则会删除所有环境。

```shell
pixi clean
```

### `clean cache`

清理系统上的 pixi 缓存。

##### 选项
- `--pypi`：清理 pypi 缓存。
- `--conda`：清理 conda 缓存。
- `--yes`：跳过确认提示。

```shell
pixi clean cache # 清理所有 pixi 缓存
pixi clean cache --pypi # 仅清理 pypi 缓存
pixi clean cache --conda # 仅清理 conda 缓存
pixi clean cache --yes # 跳过确认提示
```

## `upload`

将包上传到 prefix.dev 渠道。

##### 参数

1. `<HOST>`：要上传的主机 + 渠道。
2. `<PACKAGE_FILE>`：要上传的包文件。

```shell
pixi upload https://prefix.dev/api/v1/upload/my_channel my_package.conda
```

## `auth`

此命令用于验证用户访问远程主机的权限，例如 `prefix.dev` 或 `anaconda.org` 用于私有频道。

### `auth login`

存储给定主机的认证信息。

!!! 提示
    主机是实际的主机名，而不是频道。

##### 参数

1. `<HOST>`：要认证的主机。

##### 选项

- `--token <TOKEN>`：用于认证的 prefix.dev 令牌。
- `--username <USERNAME>`：用于基本 HTTP 认证的用户名。
- `--password <PASSWORD>`：用于基本 HTTP 认证的密码。
- `--conda-token <CONDA_TOKEN>`：用于 `anaconda.org` / `quetz` 认证的令牌。

```shell
pixi auth login repo.prefix.dev --token pfx_JQEV-m_2bdz-D8NSyRSaAndHANx0qHjq7f2iD
pixi auth login anaconda.org --conda-token ABCDEFGHIJKLMNOP
pixi auth login https://myquetz.server --username john --password xxxxxx
```

### `auth logout`

移除给定主机的认证信息。

##### 参数

1. `<HOST>`：要认证的主机。

```shell
pixi auth logout <HOST>
pixi auth logout repo.prefix.dev
pixi auth logout anaconda.org
```

## `config`

使用此命令管理配置。

##### 选项

- `--system (-s)`：指定管理范围为系统配置。
- `--global (-g)`：指定管理范围为全局配置。
- `--local (-l)`：指定管理范围为本地配置。

有关位置的更多信息，请查看 [pixi 配置](./pixi_configuration.md)。

### `config edit`

在默认编辑器中编辑配置文件。

##### 参数

1. `[EDITOR]`：要使用的编辑器，默认为 `EDITOR` 环境变量，Unix 系统上默认为 `nano`，Windows 系统上默认为 `notepad`。

```shell
pixi config edit --system
pixi config edit --local
pixi config edit -g
pixi config edit --global code
pixi config edit --system vim
```

### `config list`

列出配置项。

##### 参数

1. `[KEY]`：要列出其值的键。（如果未提供，则列出所有）

##### 选项

- `--json`：以 JSON 格式输出配置。

```shell
pixi config list default-channels
pixi config list --json
pixi config list --system
pixi config list -g
```

### `config prepend`

将值添加到列表配置项的前面。

##### 参数

1. `<KEY>`：要将值添加到的键。
2. `<VALUE>`：要添加的值。

```shell
pixi config prepend default-channels conda-forge
```

### `config append`

将值追加到列表配置项的末尾。

##### 参数

1. `<KEY>`：要将值追加到的键。
2. `<VALUE>`：要追加的值。

```shell
pixi config append default-channels robostack
pixi config append default-channels bioconda --global
```

### `config set`

将配置项的值设置为指定的值。

##### 参数

1. `<KEY>`：要设置值的键。
2. `[VALUE]`：要设置的值。（如果未提供，将移除该键）

```shell
pixi config set default-channels '["conda-forge", "bioconda"]'
pixi config set --global mirrors '{"https://conda.anaconda.org/": ["https://prefix.dev/conda-forge"]}'
pixi config set repodata-config.disable-zstd true --system
pixi config set --global detached-environments "/opt/pixi/envs"
pixi config set detached-environments false
```

### `config unset`

取消配置项的设置。

##### 参数

1. `<KEY>`：要取消设置的键。

```shell
pixi config unset default-channels
pixi config unset --global mirrors
pixi config unset repodata-config.disable-zstd --system
```

## `global`

Global 是 pixi 中用于全局（系统级）执行部分的主要入口点。
此部分的所有命令用于通过全局清单管理全局包和环境的安装。
有关全局清单的更多信息，请查看 [这里](../features/global_tools.md)。

!!! 提示
    全局安装的二进制文件和环境默认存储在 `~/.pixi` 中，
    可以通过设置 `PIXI_HOME` 环境变量更改此路径。

### `global add`

将依赖项添加到全局环境中。
默认情况下，不会将该包的二进制文件暴露到系统中。

##### 参数
1. `[PACKAGE]`：要添加的包，支持 matchspec 格式。（例如 `python=3.9.*`，`python [version='3.11.0', build_number=1]`）

##### 选项
- `--environment <ENVIRONMENT> (-e)`：要安装包的环境。
- `--expose <EXPOSE>`：将包的二进制文件映射为系统可用的名称。

```shell
pixi global add python=3.9.* --environment my-env
pixi global add python=3.9.* --expose py39=python3.9 --environment my-env
pixi global add numpy matplotlib --environment my-env
pixi global add numpy matplotlib --expose np=python3.9 --environment my-env
```

### `global edit`
在默认编辑器中编辑全局清单文件。

将尝试使用 `EDITOR` 环境变量，如果未设置，将在 Unix 系统上使用 `nano`，在 Windows 系统上使用 `notepad`。

##### 参数
1. `<EDITOR>`：要使用的编辑器。（可选）

```shell
pixi global edit
pixi global edit code
pixi global edit vim
```

### `global install`

此命令将包安装到其自己的环境中，并将二进制文件添加到 `PATH`。
允许你在系统的任何地方访问该包，而无需激活该环境。

##### 参数

1. `[PACKAGE]`：要安装的包，可以是版本约束。

##### 选项

- `--channel <CHANNEL> (-c)`：指定项目使用的通道。默认为 `conda-forge`。（可以多次使用）
- `--platform <PLATFORM> (-p)`：指定要为其安装包的目标平台。（默认：当前平台）
- `--environment <ENVIRONMENT> (-e)`：要安装包的环境。（默认：工具的名称）
- `--expose <EXPOSE>`：将包的二进制文件映射为系统可用的名称。（默认：工具的名称）
- `--with <WITH>`：向环境中添加额外的依赖项，它们的可执行文件不会被暴露。

```shell
pixi global install ruff
# 一次安装多个包
pixi global install starship rattler-build
# 指定通道
pixi global install --channel conda-forge --channel bioconda trackplot
# 或更简洁的方式
pixi global install -c conda-forge -c bioconda trackplot

# 支持完整的 conda matchspec
pixi global install python=3.9.*
pixi global install "python [version='3.11.0', build_number=1]"
pixi global install "python [version='3.11.0', build=he550d4f_1_cpython]"
pixi global install python=3.11.0=h10a6764_1_cpython

# 为特定平台安装，仅对 osx-arm64 有用
pixi global install --platform osx-64 ruff

# 安装包及其所有可执行文件暴露，同时安装不暴露任何可执行文件的其他包
pixi global install ipython --with numpy --with scipy

# 安装到特定环境名称并暴露所有可执行文件
pixi global install --environment data-science ipython jupyterlab numpy matplotlib

# 将二进制文件暴露为不同的名称
pixi global install --expose "py39=python3.9" "python=3.9.*"
```

!!! 提示
    在 Apple Silicon 上运行 `osx-64` 将安装 Intel 二进制文件，但使用 [Rosetta](https://developer.apple.com/documentation/apple-silicon/about-the-rosetta-translation-environment) 运行它
    ```
    pixi global install --platform osx-64 ruff
    ```

使用 `global install` 后，你可以在系统的任何地方使用已安装的包。

### `global uninstall`
从全局环境中卸载环境。
这将从全局环境中移除该环境及其所有依赖项。
它还将从系统中移除相关的二进制文件。

##### 参数
1. `[ENVIRONMENT]`：要卸载的环境。

```shell
pixi global uninstall my-env
pixi global uninstall pixi-pack rattler-build
```

### `global remove`

从全局环境中移除包。

##### 参数

1. `[PACKAGE]`：要移除的包。

##### 选项

- `--environment <ENVIRONMENT> (-e)`：要从中移除包的环境。

```shell
pixi global remove -e my-env package1 package2
```


### `global list`

此命令显示当前已安装的全局环境，包括随环境一起安装的二进制文件。
全局安装的包/环境可能包含多个暴露的二进制文件，它们将会在命令输出中列出。

##### 选项
- `--environment <ENVIRONMENT> (-e)`：要安装包的环境。（默认：工具名称）

如果环境的依赖项和暴露的二进制文件与环境名称不同，我们只会显示它们。
以下是一些已安装包的示例：

```
pixi global list
```
结果如下：
```
Global environments at /home/user/.pixi:
├── gh: 2.57.0
├── pixi-pack: 0.1.8
├── python: 3.11.0
│   └─ exposes: 2to3, 2to3-3.11, idle3, idle3.11, pydoc, pydoc3, pydoc3.11, python, python3, python3-config, python3.1, python3.11, python3.11-config
├── rattler-build: 0.22.0
├── ripgrep: 14.1.0
│   └─ exposes: rg
├── vim: 9.1.0611
│   └─ exposes: ex, rview, rvim, view, vim, vimdiff, vimtutor, xxd
└── zoxide: 0.9.6
```

以下是列出单个环境的示例：
```
pixi g list -e pixi-pack
```
结果如下：
```
The 'pixi-pack' environment has 8 packages:
Package          Version    Build        Size
_libgcc_mutex    0.1        conda_forge  2.5 KiB
_openmp_mutex    4.5        2_gnu        23.1 KiB
ca-certificates  2024.8.30  hbcca054_0   155.3 KiB
libgcc           14.1.0     h77fa898_1   826.5 KiB
libgcc-ng        14.1.0     h69a702a_1   50.9 KiB
libgomp          14.1.0     h77fa898_1   449.4 KiB
openssl          3.3.2      hb9d3cd8_0   2.8 MiB
pixi-pack        0.1.8      hc762bcd_0   4.3 MiB
Package          Version    Build        Size

Exposes:
pixi-pack
Channels:
conda-forge
Platform: linux-64
```

### `global sync`
由于全局清单可以手动编辑，此命令将全局清单与当前全局环境的状态同步。
您可以在 `$HOME/manifests/pixi_global.toml` 中修改清单。

```shell
pixi global sync
```

### `global expose`
修改全局环境中的暴露二进制文件。

#### `global expose add`
将环境中的暴露二进制文件添加到全局环境中。

##### 参数
1. `[MAPPING]`：要暴露的二进制文件（例如 `python`），或提供映射以将二进制文件暴露为不同的名称（例如 `py310=python3.10`）。
映射格式为 `exposed_name=binary_name`，其中暴露的名称是你在终端中可以使用的名称，二进制名称是环境中的二进制文件名称。

##### 选项
- `--environment <ENVIRONMENT> (-e)`：要暴露二进制文件的环境。

```shell
pixi global expose add python --environment my-env
pixi global expose add py310=python3.10 --environment python
```

#### `global expose remove`
从全局环境中移除暴露的二进制文件。

##### 参数
1. `[EXPOSED_NAME]`：要从全局环境中移除的二进制文件。

```shell
pixi global expose remove python
pixi global expose remove py310 python3
```

### `global update`

更新所有环境或指定环境的版本。

##### 参数

1. `[ENVIRONMENT]`：要更新的环境。

```shell
pixi global update
pixi global update pixi-pack
pixi global update bat rattler-build
```

## `project`

此子命令允许您通过命令行界面修改项目配置。

##### 选项

- `--manifest-path <MANIFEST_PATH>`：指定 [manifest 文件](project_configuration.md) 的路径，默认情况下，它会在父目录中查找该文件。

### `project channel add`

将渠道添加到项目配置中的渠道列表。
当您添加渠道时，系统会检查该渠道是否存在，并将其添加到锁文件中，然后重新安装环境。

##### 参数

1. `<CHANNEL>`：要添加的渠道，可以是名称或 URL。

##### 选项

- `--no-install`：不更新环境，只将更改的包添加到锁文件中。
- `--feature <FEATURE> (-f)`：为其添加渠道的功能。
- `--prepend`：将渠道添加到渠道列表的前面。

```
pixi project channel add robostack
pixi project channel add bioconda conda-forge robostack
pixi project channel add file:///home/user/local_channel
pixi project channel add https://repo.prefix.dev/conda-forge
pixi project channel add --no-install robostack
pixi project channel add --feature cuda nvidia
pixi project channel add --prepend pytorch
```

### `project channel list`

列出 manifest 文件中的渠道。

##### 选项

- `urls`：显示渠道的 URL，而不是名称。

```sh
$ pixi project channel list
Environment: default
- conda-forge

$ pixi project channel list --urls
Environment: default
- https://conda.anaconda.org/conda-forge/
```

### `project channel remove`

从 manifest 文件中移除渠道。

##### 参数

1. `<CHANNEL>...`：要移除的渠道，可以是名称或 URL。

##### 选项

- `--no-install`：不更新环境，只将更改的包添加到锁文件中。
- `--feature <FEATURE> (-f)`：为其移除渠道的功能。

```sh
pixi project channel remove conda-forge
pixi project channel remove https://conda.anaconda.org/conda-forge/
pixi project channel remove --no-install conda-forge
pixi project channel remove --feature cuda nvidia
```

### `project description get`

获取项目描述。

```sh
$ pixi project description get
Package management made easy!
```

### `project description set`

设置项目描述。

##### 参数

1. `<DESCRIPTION>`：要设置的描述。

```sh
pixi project description set "my new description"
```

### `project environment add`

将环境添加到 manifest 文件中。

##### 参数

1. `<NAME>`：要添加的环境名称。

##### 选项

- `-f, --feature <FEATURES>`：要添加到环境中的功能。
- `--solve-group <SOLVE_GROUP>`：要将环境添加到的解算组。
- `--no-default-feature`：不在环境中包含默认功能。
- `--force`：即使环境已存在，也强制更新 manifest 文件。

```sh
pixi project environment add env1 --feature feature1 --feature feature2
pixi project environment add env2 -f feature1 --solve-group test
pixi project environment add env3 -f feature1 --no-default-feature
pixi project environment add env3 -f feature1 --force
```

### `project environment remove`

从 manifest 文件中移除环境。

##### 参数

1. `<NAME>`：要移除的环境名称。

```shell
pixi project environment remove env1
```

### `project environment list`

列出 manifest 文件中的环境。

```shell
pixi project environment list
```

### `project export conda_environment`

导出 conda [`environment.yml` 文件](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-from-an-environment-yml-file)。该文件可用于使用 conda/mamba 创建 conda 环境：

```shell
pixi project export conda-environment environment.yml
mamba create --name <env> --file environment.yml
```

##### 参数

1. `<OUTPUT_PATH>`：可选路径，用于渲染 environment.yml 文件。否则会输出到标准输出。

##### 选项

- `--environment <ENVIRONMENT> (-e)`：要渲染的环境。
- `--platform <PLATFORM> (-p)`：要渲染的平台。

```sh
pixi project export conda-environment --environment lint
pixi project export conda-environment --platform linux-64 environment.linux-64.yml
```

### `project export conda_explicit_spec`

渲染一个平台特定的 conda [显式规范文件](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#building-identical-conda-environments)，用于某个环境。然后可以使用 conda/mamba 创建该 conda 环境：

```shell
mamba create --name <env> --file <explicit spec file>
```

由于显式规范文件格式不支持 pypi 依赖项，使用 `--ignore-pypi-errors` 选项可以忽略这些依赖项。

##### 参数

1. `<OUTPUT_DIR>`：渲染的显式环境规范文件的输出目录。

##### 选项

- `--environment <ENVIRONMENT> (-e)`：要渲染的环境。可以重复使用以处理多个环境。默认情况下，渲染所有环境。
- `--platform <PLATFORM> (-p)`：要渲染的平台。可以重复使用以处理多个平台。默认情况下，渲染所有可用平台的环境。
- `--ignore-pypi-errors`：显式规范文件不支持 PyPI 依赖项。此标志允许即使存在 PyPI 依赖项时也能创建规范文件。

```sh
pixi project export conda_explicit_spec output
pixi project export conda_explicit_spec -e default -e test -p linux-64 output
```

### `project platform add`

将平台添加到 manifest 文件中并更新锁文件。

##### 参数

1. `<PLATFORM>...`：要添加的平台。

##### 选项

- `--no-install`：不更新环境，只将更改的包添加到锁文件中。
- `--feature <FEATURE> (-f)`：要为其添加平台的功能。

```sh
pixi project platform add win-64
pixi project platform add --feature test win-64
```

### `project platform list`

列出 manifest 文件中的平台。

```sh
$ pixi project platform list
osx-64
linux-64
win-64
osx-arm64
```

### `project platform remove`

从 manifest 文件中移除平台，并更新锁文件。

##### 参数

1. `<PLATFORM>...`：要移除的平台。

##### 选项

- `--no-install`：不更新环境，只将更改的包添加到锁文件中。
- `--feature <FEATURE> (-f)`：要移除平台的功能。

```sh
pixi project platform remove win-64
pixi project platform remove --feature test win-64
```

### `project version get`

获取项目版本。

```sh
$ pixi project version get
0.11.0
```

### `project version set`

设置项目版本。

##### 参数

1. `<VERSION>`：要设置的版本。

```sh
pixi project version set "0.13.0"
```

### `project version {major|minor|patch}`

将项目版本提升为 {MAJOR|MINOR|PATCH}。

```sh
pixi project version major
pixi project version minor
pixi project version patch
```

[^1]:
    **最新的** 锁文件意味着锁文件中的依赖项是被 manifest 文件中的依赖项允许的。
    例如：

    - 一个包含 `python = ">= 3.11"` 的 manifest 与锁文件中的 `name: python, version: 3.11.0` 是最新的。
    - 一个包含 `python = ">= 3.12"` 的 manifest 与锁文件中的 `name: python, version: 3.11.0` **不是** 最新的。

    “最新的”并不意味着锁文件中包含的依赖项版本是该通道中给定依赖项的最新版本。
