---
part: pixi/advanced
title: GitHub Action
description: 了解如何通过 GitHub Actions 使用 pixi
---

<!--
对这个文件的修改与 [prefix-dev/setup-pixi](https://github.com/prefix-dev/setup-pixi) 中的 README.md 相关，
请通过在两个文件中都提交 PR 来保持同步
-->

我们创建了 [prefix-dev/setup-pixi](https://github.com/prefix-dev/setup-pixi) 来便于在 CI 中使用 pixi。

## 使用方法

```yaml
- uses: prefix-dev/setup-pixi@v0.8.0
  with:
    pixi-version: v0.36.0
    cache: true
    auth-host: prefix.dev
    auth-token: ${{ secrets.PREFIX_DEV_TOKEN }}
- run: pixi run test
```

!!!warning "固定你的 Action 版本"
    由于 pixi 还不稳定，此 Action 的 API 可能会在小版本之间发生变化。  
    请固定此 Action 的版本到一个特定版本（例如，`prefix-dev/setup-pixi@v0.8.0`），以避免破坏性更改。  
    你可以通过使用 [Dependabot](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/keeping-your-actions-up-to-date-with-dependabot) 来自动更新此 Action 的版本。

    将以下内容添加到 `.github/dependabot.yml` 文件中，以启用 Dependabot 来更新 GitHub Actions：

    ```yaml title=".github/dependabot.yml"
    version: 2
    updates:
      - package-ecosystem: github-actions
        directory: /
        schedule:
          interval: monthly # (1)!
        groups:
          dependencies:
            patterns:
              - "*"
    ```

    1. 或者使用 `daily`、`weekly` 等频率。

## 功能

要查看所有可用的输入参数，请查看 [`action.yml`](https://github.com/prefix-dev/setup-pixi/blob/main/action.yml) 文件中的 `setup-pixi`。  
最重要的特性如下所述。

### 缓存

此 Action 支持缓存 pixi 环境。  
默认情况下，如果存在 `pixi.lock` 文件，则启用缓存。  
然后它将使用 `pixi.lock` 文件生成环境的哈希并进行缓存。  
如果缓存命中，Action 将跳过安装并使用缓存的环境。  
你可以通过设置 `cache` 输入参数来指定行为。

!!!tip "自定义缓存键"
    如果需要自定义缓存键，可以使用 `cache-key` 输入参数。  
    这将作为缓存键的前缀。完整的缓存键将是 `<cache-key><conda-arch>-<hash>`。

!!!tip "仅在 `main` 分支上保存缓存"
    为了避免过快超过 [10 GB 缓存大小限制](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#usage-limits-and-eviction-policy)，你可能希望限制缓存保存的时机。  
    这可以通过设置 `cache-write` 参数来实现。

    ```yaml
    - uses: prefix-dev/setup-pixi@v0.8.0
      with:
        cache: true
        cache-write: ${{ github.event_name == 'push' && github.ref_name == 'main' }}
    ```

### 多环境支持

使用 pixi，你可以为不同的需求创建多个环境。  
你还可以通过设置 `environments` 输入参数指定要安装的环境。  
这将安装所有指定的环境并缓存它们。

```toml
[project]
name = "my-package"
channels = ["conda-forge"]
platforms = ["linux-64"]

[dependencies]
python = ">=3.11"
pip = "*"
polars = ">=0.14.24,<0.21"

[feature.py311.dependencies]
python = "3.11.*"
[feature.py312.dependencies]
python = "3.12.*"

[environments]
py311 = ["py311"]
py312 = ["py312"]
```

#### 使用矩阵安装多个环境

以下示例将在不同的作业中安装 `py311` 和 `py312` 环境。

```yaml
test:
  runs-on: ubuntu-latest
  strategy:
    matrix:
      environment: [py311, py312]
  steps:
  - uses: actions/checkout@v4
  - uses: prefix-dev/setup-pixi@v0.8.0
    with:
      environments: ${{ matrix.environment }}
```

#### 在一个作业中安装多个环境

以下示例将在运行器上安装 `py311` 和 `py312` 环境。

```yaml
- uses: prefix-dev/setup-pixi@v0.8.0
  with:
    environments: >- # (1)!
      py311
      py312
- run: |
  pixi run -e py311 test
  pixi run -e py312 test
```

1. 通过空格分隔，相当于

   ```yaml
   environments: py311 py312
   ```

!!!warning "如果未指定环境，缓存行为"
    如果未指定任何环境，将会安装并缓存 `default` 环境，即使你使用了其他环境。

### 身份验证

目前有三种方式可以通过 pixi 进行身份验证：

- 使用令牌
- 使用用户名和密码
- 使用 conda-token

更多信息请参见 [身份验证](./authentication.md)。

!!!warning "谨慎处理机密"
    请仅通过 [GitHub secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) 存储敏感信息，切勿将其存储在仓库中。  
    当敏感信息存储在 GitHub secret 中时，可以通过 `${{ secrets.SECRET_NAME }}` 语法访问。  
    这些机密信息在日志中会始终被掩码处理。

#### 令牌

通过 `auth-token` 输入参数指定令牌。  
这种身份验证形式（请求头中的 bearer 令牌）主要用于 [prefix.dev](https://prefix.dev)。

```yaml
- uses: prefix-dev/setup-pixi@v0.8.0
  with:
    auth-host: prefix.dev
    auth-token: ${{ secrets.PREFIX_DEV_TOKEN }}
```

#### 用户名和密码

通过 `auth-username` 和 `auth-password` 输入参数指定用户名和密码。  
这种身份验证形式（HTTP 基本身份验证）在一些企业环境中使用，例如与 [artifactory](https://jfrog.com/artifactory) 配合使用。

```yaml
- uses: prefix-dev/setup-pixi@v0.8.0
  with:
    auth-host: custom-artifactory.com
    auth-username: ${{ secrets.PIXI_USERNAME }}
    auth-password: ${{ secrets.PIXI_PASSWORD }}
```

#### Conda-token

通过 `conda-token` 输入参数指定 conda-token。  
这种身份验证形式（令牌编码在 URL 中：`https://my-quetz-instance.com/t/<token>/get/custom-channel`）用于 [anaconda.org](https://anaconda.org) 或与 [quetz 实例](https://github.com/mamba-org/quetz) 配合使用。

```yaml
- uses: prefix-dev/setup-pixi@v0.8.0
  with:
    auth-host: anaconda.org # (1)!
    conda-token: ${{ secrets.CONDA_TOKEN }}
```

1. 或 `my-quetz-instance.com`

### 自定义 shell 包装器

`setup-pixi` 允许通过指定自定义 shell 包装器 `shell: pixi run bash -e {0}` 在 pixi 环境中运行命令。  
如果你希望在 pixi 环境中运行命令，但又不想对每个命令都使用 `pixi run`，这种方式会非常有用。

```yaml
- run: | # (1)!
    python --version
    pip install --no-deps -e .
  shell: pixi run bash -e {0}
```

1. 这里的所有命令都将在 pixi 环境中运行

你甚至可以像这样运行 Python 脚本：

```yaml
- run: | # (1)!
    import my_package
    print("Hello world!")
  shell: pixi run python {0}
```

1. 这里的所有命令都将在 pixi 环境中运行

如果你想使用 PowerShell，你需要同时指定 `-Command`。

```yaml
- run: | # (1)!
    python --version | Select-String "3.11"
  shell: pixi run pwsh -Command {0} # pwsh 在所有平台上均可用
```

1. 这里的所有命令都将在 pixi 环境中运行

!!!note "底层是如何工作的？"
    底层实现中，`shell: xyz {0}` 选项通过创建一个临时脚本文件并将该脚本文件作为参数传递给 `xyz` 来实现。  
    这个文件没有设置可执行位，因此不能直接使用 `shell: pixi run {0}`，而需要使用 `shell: pixi run bash {0}`。  
    GitHub 提供了一些自定义 shell，它们的行为稍有不同，详见文档中的 [`jobs.<job_id>.steps[*].shell`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsshell)。  
    更多关于 `shell:` 输入在 GitHub Actions 中的工作原理，请参阅 [官方文档](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#custom-shell) 和 [ADR 0277](https://github.com/actions/runner/blob/main/docs/adrs/0277-run-action-shell-options.md)。

#### 使用 `pixi exec` 的一次性 shell 包装器

通过 `pixi exec`，你也可以在临时的 pixi 环境中运行一次性命令。

```yaml
- run: | # (1)!
    zstd --version
  shell: pixi exec --spec zstd -- bash -e {0}
```

1. 这里的所有命令都将在临时的 pixi 环境中运行

```yaml
- run: | # (1)!
    import ruamel.yaml
    # ...
  shell: pixi exec --spec python=3.11.* --spec ruamel.yaml -- python {0}
```

1. 这里的所有命令都将在临时的 pixi 环境中运行

有关 `pixi exec` 的更多信息，请参见 [此处](../reference/cli.md#exec)。

### 环境激活

除了使用自定义 shell 包装器外，你还可以通过在当前作业中“激活”安装的环境，将所有 pixi 安装的二进制文件提供给后续步骤。  
为此，`setup-pixi` 会将执行 `pixi run` 时设置的所有环境变量添加到 `$GITHUB_ENV`，并将所有路径修改添加到 `$GITHUB_PATH`。  
因此，所有已安装的二进制文件都可以访问，而无需调用 `pixi run`。

```yaml
- uses: prefix-dev/setup-pixi@v0.8.0
  with:
    activate-environment: true
```

如果你安装了多个环境，你需要指定要激活的环境名称。

```yaml
- uses: prefix-dev/setup-pixi@v0.8.0
  with:
    environments: >-
      py311
      py312
    activate-environment: py311
```

激活一个环境可能比使用自定义 shell 包装器更有用，因为它允许非 shell 的步骤访问路径中的二进制文件。  
但是，请注意，这个选项会增加你作业的环境设置。

### `--frozen` 和 `--locked`

你可以根据 `frozen` 或 `locked` 输入参数来指定 `setup-pixi` 是否应该运行 `pixi install --frozen` 或 `pixi install --locked`。  
有关 `--frozen` 和 `--locked` 标志的更多信息，请参见 [官方文档](https://prefix.dev/docs/pixi/cli#install)。

```yaml
- uses: prefix-dev/setup-pixi@v0.8.0
  with:
    locked: true
    # 或者
    frozen: true
```

如果未指定任何参数，默认行为是在存在 `pixi.lock` 文件时运行 `pixi install --locked`，否则运行 `pixi install`。

### 调试

你可以启用两种类型的调试日志记录。



#### Action 的调试日志

第一种是记录 `setup-pixi` 操作本身的调试日志。  
你可以通过以调试模式重新运行操作来启用此日志：

![以调试模式重新运行](https://raw.githubusercontent.com/prefix-dev/setup-pixi/main/.github/assets/enable-debug-logging-light.png#only-light)
![以调试模式重新运行](https://raw.githubusercontent.com/prefix-dev/setup-pixi/main/.github/assets/enable-debug-logging-dark.png#only-dark)

!!!tip "调试日志文档"
    有关 GitHub Actions 中调试日志的更多信息，请参见 [官方文档](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/enabling-debug-logging)。

#### pixi 的调试日志

第二种是记录 pixi 可执行文件的调试日志。  
你可以通过设置 `log-level` 输入来指定此日志级别。

```yaml
- uses: prefix-dev/setup-pixi@v0.8.0
  with:
    log-level: vvv # (1)!
```

1. 可以是 `q`、`default`、`v`、`vv` 或 `vvv` 中的一个。

如果没有指定，`log-level` 默认设置为 `default` 或 `vv`，具体取决于是否已为操作启用 [调试日志](#action)。

### 自托管运行器

在自托管运行器上，可能会发生一些文件在作业之间持久化。  
这可能导致问题或机密在作业运行之间泄漏。  
为避免此问题，你可以使用 `post-cleanup` 输入来指定操作的后清理行为（即所有命令执行完成后的操作）。

如果将 `post-cleanup` 设置为 `true`，操作将删除以下文件：

- `.pixi` 环境
- pixi 二进制文件
- rattler 缓存
- `~/.rattler` 中的其他 rattler 文件

如果未指定，`post-cleanup` 默认设置为 `true`。

在自托管运行器上，你可能还希望将默认的 pixi 安装位置更改为临时位置。你可以使用 `pixi-bin-path: ${{ runner.temp }}/bin/pixi` 来实现这一点。

```yaml
- uses: prefix-dev/setup-pixi@v0.8.0
  with:
    post-cleanup: true
    pixi-bin-path: ${{ runner.temp }}/bin/pixi # (1)!
```

1. 在 Windows 上为 `${{ runner.temp }}\Scripts\pixi.exe`

你也可以通过不设置任何 `pixi-version`、`pixi-url` 或 `pixi-bin-path` 输入，使用运行器上预安装的本地版本的 pixi。这样，操作会尝试在运行器的 PATH 中查找本地版本的 pixi。

### 使用 `pyproject.toml` 作为 pixi 的清单文件

如果 `pyproject.toml` 文件包含 `[tool.pixi.project]` 部分且没有 `pixi.toml` 文件，`setup-pixi` 将自动使用它。  
你也可以通过设置 `manifest-path` 输入参数来覆盖此行为。

```yaml
- uses: prefix-dev/setup-pixi@v0.8.0
  with:
    manifest-path: pyproject.toml
```

## 更多示例

如果你想查看更多示例，可以查看 [`setup-pixi` 仓库的 GitHub 工作流](https://github.com/prefix-dev/setup-pixi/blob/main/.github/workflows/test.yml)。
