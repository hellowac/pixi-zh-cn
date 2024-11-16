# 多环境支持

### 激励示例

在多个场景中，使用多个环境非常有用。

- **测试多个包版本**，例如 `py39` 和 `py310` 或者 polars `0.12` 和 `0.13`。
- **较小的单工具环境**，例如 `lint` 或 `docs`。
- **大型开发者环境**，将所有小型环境组合在一起，例如 `dev`。
- **严格的环境超集**，例如 `prod` 和 `test-prod`，其中 `test-prod` 是 `prod` 的严格超集。
- **一个项目下的多个机器环境**，例如 `cuda` 环境和 `cpu` 环境。
- **还有更多**。 （欢迎在我们的 GitHub 上编辑此文档并添加您的用例。）

这为 `pixi` 在大型项目中准备了多种使用场景，包括多个开发人员和不同的 CI 需求。

## 设计考虑

在设计时，我们考虑了以下几点：

1. **用户友好性**：Pixi 是一个以用户为中心的工具，超越了开发者的范畴。该功能应该从一开始就具备良好的错误报告和有帮助的文档。
2. **保持简单**：不理解多环境功能不应限制用户使用 pixi。对于非多环境的用例，该功能应该是“不可见”的。
3. **没有自动组合**：为了确保依赖关系解析过程可管理，解决方案应避免依赖集的组合爆炸。通过让环境由用户定义，而不是通过测试特性矩阵自动推断来实现这一点。
4. **单一环境激活**：设计应允许在任何给定时刻只有一个环境处于活动状态，从而简化解析过程并防止冲突。
5. **固定锁文件**：为了保持一致性和可预测性，确保锁文件的可靠性至关重要。解决方案不仅要确保作者的可靠性，还要确保最终用户的可靠性，尤其是在锁文件创建时。

### 特性与环境集定义

在 `pixi.toml` 中引入环境集，这些环境基于 `feature` 来描述。为 `pixi.toml` 引入特性（features），可以描述环境的一部分内容。
由于环境不仅仅包括 `dependencies`，因此特性应包括以下字段：

- `dependencies`：conda 包依赖
- `pypi-dependencies`：pypi 包依赖
- `system-requirements`：环境的系统要求
- `activation`：环境的激活信息
- `platforms`：环境可以运行的平台
- `channels`：用于创建环境的渠道。为允许连接渠道而不是覆盖，需要为渠道添加 `priority` 字段。
- `target`：所有上述特性，但也按目标进行分隔。
- `tasks`：特定于特性的任务，某个环境中的任务可作为该环境的默认任务。

```toml title="默认特性"
[dependencies] # 即 [feature.default.dependencies] 的简写
python = "*"
numpy = "==2.3"

[pypi-dependencies] # 即 [feature.default.pypi-dependencies] 的简写
pandas = "*"

[system-requirements] # 即 [feature.default.system-requirements] 的简写
libc = "2.33"

[activation] # 即 [feature.default.activation] 的简写
scripts = ["activate.sh"]
```

```toml title="每个特性不同的依赖"
[feature.py39.dependencies]
python = "~=3.9.0"
[feature.py310.dependencies]
python = "~=3.10.0"
[feature.test.dependencies]
pytest = "*"
```

```toml title="在一个特性中完整的环境修改"
[feature.cuda]
dependencies = {cuda = "x.y.z", cudnn = "12.0"}
pypi-dependencies = {torch = "1.9.0"}
platforms = ["linux-64", "osx-arm64"]
activation = {scripts = ["cuda_activation.sh"]}
system-requirements = {cuda = "12"}
# 渠道通过优先级连接，而不是覆盖，因此默认渠道仍然会使用。
# 使用优先级来控制连接顺序，默认值为 0，默认渠道排在最后。
# 最高优先级的渠道排在前面。
channels = ["nvidia", {channel = "pytorch", priority = -1}] # 结果为：["nvidia", "conda-forge", "pytorch"]，如果默认渠道为 `conda-forge`
tasks = { warmup = "python warmup.py" }
target.osx-arm64 = {dependencies = {mlx = "x.y.z"}}
```

```toml title="定义环境的默认任务"
[feature.test.tasks]
test = "pytest"

[environments]
test = ["test"]

# `pixi run test` == `pixi run --environment test test`
```

环境定义应包含以下字段：

- `features: Vec<Feature>`：环境集中包含的特性，这也是环境的默认字段。
- `solve-group: String`：解决组用于在求解阶段将环境组合在一起。
  这对于那些需要具有相同依赖关系但可能通过附加依赖项进行扩展的环境非常有用。
  例如，在测试生产环境时附加测试依赖项。

```toml title="从特性创建环境"
[environments]
# 隐式: default = ["default"]
default = ["py39"] # 隐式: default = ["py39", "default"]
py310 = ["py310"] # 隐式: py310 = ["py310", "default"]
test = ["test"] # 隐式: test = ["test", "default"]
test39 = ["test", "py39"] # 隐式: test39 = ["test", "py39", "default"]
```

```toml title="使用附加依赖项测试生产环境"
[environments]
# 创建 `prod` 环境，它是用于生产的最小依赖集。
prod = {features = ["py39"], solve-group = "prod"}
# 创建 `test_prod` 环境，它是 `prod` 环境加上 `test` 特性。
test_prod = {features = ["py39", "test"], solve-group = "prod"}
# 使用 `solve-group` 一起求解 `prod` 和 `test_prod` 环境
# 这确保测试环境具有与生产环境相同版本的依赖项。
```

```toml title="创建没有包含默认特性的环境"
[dependencies]
python = "*"
numpy = "*"

[feature.lint.dependencies]
pre-commit = "*"

[environments]
# 创建一个自定义环境，仅具有 `lint` 特性（numpy 不属于该环境）。
lint = {features = ["lint"], no-default-feature = true}
```

### 锁文件结构

在 `pixi.lock` 文件中，包现在可以包括一个额外的 `environments` 字段，指定其所属的环境。
为了避免重复，包的 `environments` 字段可以包含多个环境，从而使锁文件保持最小大小。

```yaml
- platform: linux-64
  name: pre-commit
  version: 3.3.3
  category: main
  environments:
    - dev
    - test
    - lint
  ...:
- platform: linux-64
  name: python
  version: 3.9.3
  category: main
  environments:
    - dev
    - test
    - lint
    - py39
    - default
  ...:
```

### 用户界面环境激活

用户可以通过命令行或配置手动激活所需的环境。
这种方法通过允许一次只激活一个特性集，确保环境无冲突。
对于用户而言，命令行界面如下所示：

```shell title="默认行为"
➜ pixi run python
# 在 `default` 环境中运行 python
```

```shell title="激活特定环境"
➜ pixi run -e test pytest
➜ pixi run --environment test pytest
# 在 `test` 环境中运行 `pytest`
```

```shell title="在环境中激活 shell"
➜ pixi shell -e cuda
pixi shell --environment cuda
# 在 `cuda` 环境中启动 shell
```

```shell title="在环境中运行任何命令"
➜ pixi run -e test any_command
# 在 `test` 环境中运行 any_command，这不需要预定义为任务。
```

### 模糊的环境选择
可以在多个环境中定义任务，在这种情况下，用户应被提示选择环境。

这是一个只有任务的简单示例：

```toml title="pixi.toml"
[project]
name = "test_ambiguous_env"
channels = []
platforms = ["linux-64", "win-64", "osx-64", "osx-arm64"]

[tasks]
default = "echo Default"
ambi = "echo Ambi::Default"
[feature.test.tasks]
test = "echo Test"
ambi = "echo Ambi::Test"

[feature.dev.tasks]
dev = "echo Dev"
ambi = "echo Ambi::Dev"

[environments]
default = ["test", "dev"]
test = ["test"]
dev = ["dev"]
```

尝试运行 `abmi` 任务时，系统会提示用户选择环境。
因为它在所有环境中都有定义。

```shell title="如果任务在多个环境中定义，则交互式选择环境"
➜ pixi run ambi
? The task 'ambi' can be run in multiple environments.

Please select an environment to run the task in: ›
❯ default # 选择 default
  test
  dev

✨ Pixi task (ambi in default): echo Ambi::Test
Ambi::Test
```

如您所见，它在 `default` 环境中运行了在 `feature.task` 中定义的任务。
这是因为 `ambi` 任务在 `test` 特性中定义，并且它在默认环境中被覆盖了。
因此，`tasks.default` 现在在任何环境中都无法访问。

在此示例中运行的其他结果：
```shell
➜ pixi run --environment test ambi
✨ Pixi task (ambi in test): echo Ambi::Test
Ambi::Test

➜ pixi run --environment dev ambi
✨ Pixi task (ambi in dev): echo Ambi::Dev
Ambi::Dev

# dev 在默认环境中运行
➜ pixi run dev
✨ Pixi task (dev in default): echo Dev
Dev

# dev 在 dev 环境中运行
➜ pixi run -e dev dev
✨ Pixi task (dev in dev): echo Dev
Dev
```


## 重要链接

- 提案的初步写作： [GitHub Gist by 0xbe7a](https://gist.github.com/0xbe7a/bbf8a323409be466fe1ad77aa6dd5428)
- GitHub 项目： [#10](https://github.com/orgs/prefix-dev/projects/10)

## 真实世界示例用例

??? tip "Polarify 测试设置"

    在 `polarify` 中，他们想要测试多个版本与多个版本的 polars 组合。
    当前是通过 GitHub actions 中使用矩阵来完成的。
    这可以通过使用多个环境来替代。

    ```toml title="pixi.toml"
    [project]
    name = "polarify"
    # ...
    channels = ["conda-forge"]
    platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]

    [tasks]
    postinstall = "pip install --no-build-isolation --no-deps --disable-pip-version-check -e ."

    [dependencies]
    python = ">=3.9"
    pip = "*"
    polars = ">=0.14.24,<0.21"

    [feature.py39.dependencies]
    python = "3.9.*"
    [feature.py310.dependencies]
    python = "3.10.*"
    [feature.py311.dependencies]
    python = "3.11.*"
    [feature.py312.dependencies]
    python = "3.12.*"
    [feature.pl017.dependencies]
    polars = "0.17.*"
    [feature.pl018.dependencies]
    polars = "0.18.*"
    [feature.pl019.dependencies]
    polars = "0.19.*"
    [feature.pl020.dependencies]
    polars = "0.20.*"

    [feature.test.dependencies]
    pytest = "*"
    pytest-md = "*"
    pytest-emoji = "*"
    hypothesis = "*"
    [feature.test.tasks]
    test = "pytest"

    [feature.lint.dependencies]
    pre-commit = "*"
    [feature.lint.tasks]
    lint = "pre-commit run --all"

    [environments]
    pl017 = ["pl017", "py39", "test"]
    pl018 = ["pl018", "py39", "test"]
    pl019 = ["pl019", "py39", "test"]
    pl020 = ["pl020", "py39", "test"]
    py39 = ["py39", "test"]
    py310 = ["py310", "test"]
    py311 = ["py311", "test"]
    py312 = ["py312", "test"]
    ```

    ```yaml title=".github/workflows/test.yml"
    jobs:
      tests-per-env:
        runs-on: ubuntu-latest
        strategy:
          matrix:
            environment: [py311, py312]
        steps:
        - uses: actions/checkout@v4
          - uses: prefix-dev/setup-pixi@v0.5.1
            with:
              environments: ${{ matrix.environment }}
          - name: Run tasks
            run: |
              pixi run --environment ${{ matrix.environment }} test
      tests-with-multiple-envs:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v4
        - uses: prefix-dev/setup-pixi@v0.5.1
          with:
           environments: pl017 pl018
        - run: |
            pixi run -e pl017 test
            pixi run -e pl018 test
    ```

??? tip "测试与生产示例"

    这是一个具有 `test` 特性和 `prod` 环境的项目示例。
    `prod` 环境是包含运行时依赖项的生产环境。
    `test` 特性是我们希望在先前解决的 `prod` 环境上添加的依赖项和任务。
    这是一个常见的用例，我们希望在生产环境中测试附加的依赖项。

    ```toml title="pixi.toml"
    [project]
    name = "my-app"
    # ...
    channels = ["conda-forge"]
    platforms = ["osx-arm64", "linux-64"]

    [tasks]
    postinstall-e = "pip install --no-build-isolation --no-deps --disable-pip-version-check -e ."
    postinstall = "pip install --no-build-isolation --no-deps --disable-pip-version-check ."
    dev = "uvicorn my_app.app:main --reload"
    serve = "uvicorn my_app.app:main"

    [dependencies]
    python = ">=3.12"
    pip = "*"
    pydantic = ">=2"
    fastapi = ">=0.105.0"
    sqlalchemy = ">=2,<3"
    uvicorn = "*"
    aiofiles = "*"

    [feature.test.dependencies]
    pytest = "*"
    pytest-md = "*"
    pytest-asyncio = "*"
    [feature.test.tasks]
    test = "pytest --md=report.md"

    [environments]
    # both default and prod will have exactly the same dependency versions when they share a dependency
    default = {features = ["test"], solve-group = "prod-group"}
    prod = {features = [], solve-group = "prod-group"}
    ```
    在 CI 中，您可以运行以下命令：
    ```shell
    pixi run postinstall-e && pixi run test
    ```
    本地运行时，您可以运行以下命令：
    ```shell
    pixi run postinstall-e && pixi run dev
    ```

    然后在 Dockerfile 中运行以下命令：
    ```dockerfile title="Dockerfile"
    FROM ghcr.io/prefix-dev/pixi:latest # 此版本尚不存在
    WORKDIR /app
    COPY . .
    RUN pixi run --environment prod postinstall
    EXPOSE 8080
    CMD ["/usr/local/bin/pixi", "run", "--environment", "prod", "serve"]
    ```

??? tip "从一个项目运行多个机器"
    
    这是一个适用于机器支持 `cuda` 和 `mlx` 的 ML 项目的示例。它也应该可以在不支持 `cuda` 或 `mlx` 的机器上执行，我们为此使用了 `cpu` 特性。

    ```toml title="pixi.toml"
    [project]
    name = "my-ml-project"
    description = "A project that does ML stuff"
    authors = ["Your Name <your.name@gmail.com>"]
    channels = ["conda-forge", "pytorch"]
    # 项目支持的所有平台，因为特性将采用此处定义平台的交集。
    platforms = ["win-64", "linux-64", "osx-64", "osx-arm64"]

    [tasks]
    train-model = "python train.py"
    evaluate-model = "python test.py"

    [dependencies]
    python = "3.11.*"
    pytorch = {version = ">=2.0.1", channel = "pytorch"}
    torchvision = {version = ">=0.15", channel = "pytorch"}
    polars = ">=0.20,<0.21"
    matplotlib-base = ">=3.8.2,<3.9"
    ipykernel = ">=6.28.0,<6.29"

    [feature.cuda]
    platforms = ["win-64", "linux-64"]
    channels = ["nvidia", {channel = "pytorch", priority = -1}]
    system-requirements = {cuda = "12.1"}

    [feature.cuda.tasks]
    train-model = "python train.py --cuda"
    evaluate-model = "python test.py --cuda"

    [feature.cuda.dependencies]
    pytorch-cuda = {version = "12.1.*", channel = "pytorch"}

    [feature.mlx]
    platforms = ["osx-arm64"]
    # MLX 仅适用于 macOS >=13.5（推荐使用 >14.0）
    system-requirements = {macos = "13.5"}

    [feature.mlx.tasks]
    train-model = "python train.py --mlx"
    evaluate-model = "python test.py --mlx"

    [feature.mlx.dependencies]
    mlx = ">=0.16.0,<0.17.0"

    [feature.cpu]
    platforms = ["win-64", "linux-64", "osx-64", "osx-arm64"]

    [environments]
    cuda = ["cuda"]
    mlx = ["mlx"]
    default = ["cpu"]
    ```

    ```shell title="在支持 cuda 的机器上运行项目"
    pixi run train-model --environment cuda
    # 将执行 `python train.py --cuda`
    # 如果不是 linux-64 或 win-64 并且不支持 cuda 12.1，将失败
    ```

    ```shell title="在支持 mlx 的机器上运行项目"
    pixi run train-model --environment mlx
    # 将执行 `python train.py --mlx`
    # 如果不是 osx-arm64，将失败
    ```

    ```shell title="在没有 cuda 或 mlx 的机器上运行项目"
    pixi run train-model
    ```
