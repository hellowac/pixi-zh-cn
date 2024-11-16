# pixi 本身的配置

除了 [项目特定配置](../reference/project_configuration.md) 外，pixi 还支持一些不是项目运行所必需的，但与机器本地相关的配置选项。配置加载顺序如下：

=== "Linux"

    | **优先级** | **位置**                                                                 | **备注**                                                                              |
    |------------|--------------------------------------------------------------------------|---------------------------------------------------------------------------------------|
    | 1          | `/etc/pixi/config.toml`                                                  | 系统范围的配置                                                                        |
    | 2          | `$XDG_CONFIG_HOME/pixi/config.toml`                                      | XDG 兼容的用户特定配置                                                                |
    | 3          | `$HOME/.config/pixi/config.toml`                                         | 用户特定配置                                                                          |
    | 4          | `$PIXI_HOME/config.toml`                                                 | 用户主目录下的全局配置。`PIXI_HOME` 默认值为 `~/.pixi`                              |
    | 5          | `your_project/.pixi/config.toml`                                         | 项目特定配置                                                                          |
    | 6          | 命令行参数 (`--tls-no-verify`, `--change-ps1=false` 等)                  | 通过命令行参数配置                                                                    |

=== "macOS"

    | **优先级** | **位置**                                                                 | **备注**                                                                              |
    |------------|--------------------------------------------------------------------------|---------------------------------------------------------------------------------------|
    | 1          | `/etc/pixi/config.toml`                                                  | 系统范围的配置                                                                        |
    | 2          | `$XDG_CONFIG_HOME/pixi/config.toml`                                      | XDG 兼容的用户特定配置                                                                |
    | 3          | `$HOME/Library/Application Support/pixi/config.toml`                     | 用户特定配置                                                                          |
    | 4          | `$PIXI_HOME/config.toml`                                                 | 用户主目录下的全局配置。`PIXI_HOME` 默认值为 `~/.pixi`                              |
    | 5          | `your_project/.pixi/config.toml`                                         | 项目特定配置                                                                          |
    | 6          | 命令行参数 (`--tls-no-verify`, `--change-ps1=false` 等)                  | 通过命令行参数配置                                                                    |

=== "Windows"

    | **优先级** | **位置**                                                                 | **备注**                                                                                   |
    |------------|--------------------------------------------------------------------------|--------------------------------------------------------------------------------------------|
    | 1          | `C:\ProgramData\pixi\config.toml`                                        | 系统范围的配置                                                                              |
    | 2          | `%APPDATA%\pixi\config.toml`                                             | 用户特定配置                                                                                |
    | 3          | `$PIXI_HOME\config.toml`                                                 | 用户主目录下的全局配置。`PIXI_HOME` 默认值为 `%USERPROFILE%/.pixi`                        |
    | 4          | `your_project\.pixi\config.toml`                                         | 项目特定配置                                                                                |
    | 5          | 命令行参数 (`--tls-no-verify`, `--change-ps1=false` 等)                  | 通过命令行参数配置                                                                          |

!!! note
    优先级最高的配置生效。如果在优先级更高的位置找到了配置文件，则会覆盖来自优先级较低位置的配置值。

!!! note
    要查看 `pixi` 查找配置文件的路径，请使用 `-vv` 选项运行 `pixi`。

## 参考

??? info "配置中的大小写"
    在 `pixi 0.20.1` 及更早版本中，全球配置使用的是 `snake_case`，为了与其他配置保持一致，我们已经改为使用 `kebab-case`。但我们仍然支持旧的 `snake_case` 配置选项，用于旧的配置选项。以下选项仍使用旧的格式：

    - `default_channels`
    - `change_ps1`
    - `tls_no_verify`
    - `authentication_override_file`
    - `mirrors` 及其子选项
    - `repodata-config` 及其子选项

以下参考文档描述了所有可用的配置选项。

### `default-channels`

运行 `pixi init` 或 `pixi global install` 时选择的默认渠道。默认情况下仅使用 conda-forge。
```toml title="config.toml"
default-channels = ["conda-forge"]
```
!!! note
    `default-channels` 仅在初始化新项目时使用。一旦项目初始化完成，`channels` 将从项目清单中使用。

### `change-ps1`

当设置为 `false` 时，Shell 提示符中的 `(pixi)` 前缀将被移除。
此设置适用于 `pixi shell` 子命令。
可以通过 CLI 使用 `--change-ps1` 来覆盖此设置。

```toml title="config.toml"
change-ps1 = true
```

### `tls-no-verify`
当设置为 `true` 时，不会验证 TLS 证书。

!!! warning
    这是一种安全风险，仅应在测试或内部网络中使用。

可以通过 CLI 使用 `--tls-no-verify` 来覆盖此设置。

```toml title="config.toml"
tls-no-verify = false
```

### `authentication-override-file`
覆盖加载身份验证信息的位置。
通常，我们尝试使用密钥环来加载身份验证数据，只有在无法找到时才使用 JSON 文件作为回退。此选项允许你强制使用 JSON 文件。
更多信息请参见身份验证部分。
```toml title="config.toml"
authentication-override-file = "/path/to/your/override.json"
```

### `detached-environments`
pixi 存储项目环境的目录，这些环境通常会被放置在项目根目录的 `.pixi/envs` 文件夹中。
它不影响为 `pixi global` 构建的环境。
可以使用 `PIXI_HOME` 环境变量控制为 `pixi global` 安装创建的环境位置。
!!! warning
    我们不推荐使用此选项，因为为项目创建的任何环境将不再与项目放在同一文件夹中。
    这会导致项目与其环境之间脱节，并且在删除项目时需要手动清理环境。

    然而，在某些情况下，此选项仍然非常有用，例如：

    - 强制在特定的文件系统或驱动器上安装。
    - 在本地安装环境，但将项目保留在网络驱动器上。
    - 让系统管理员更好地控制系统上的所有环境。

此字段可以是两种类型的输入。

- 布尔值 `true` 或 `false`，分别启用或禁用此功能。（不是 `"true"` 或 `"false"`，这会被读取为 `false`）
- 字符串值，即环境将被存储的目录的绝对路径。

```toml title="config.toml"
detached-environments = true
```

或：

```toml title="config.toml"
detached-environments = "/opt/pixi/envs"
```

当此选项设置为 `true` 时，环境将存储在 [缓存目录](../features/environment.md#caching) 中。
当你指定自定义路径时，环境将存储在该目录中。

结果的目录结构如下所示：
```toml title="config.toml"
detached-environments = "/opt/pixi/envs"
```
```shell
/opt/pixi/envs
├── pixi-6837172896226367631
│   └── envs
└── NAME_OF_PROJECT-HASH_OF_ORIGINAL_PATH
    ├── envs # 可运行的环境
    └── solve-group-envs # 如果有求解组
```

### `pinning-strategy`
运行 `pixi add` 时用于固定依赖项的策略。默认值为 `semver`，但你可以设置以下选项：

- `no-pin`: 不进行固定，导致依赖项没有约束。`*`
- `semver`: 固定到满足 semver 约束的最新版本。对于大多数版本来说，固定到主版本号，对于 `v0` 版本来说，固定到次版本号。
- `exact-version`: 固定到确切的版本，`1.2.3` -> `==1.2.3`。
- `major`: 固定到主版本，`1.2.3` -> `>=1.2.3, <2`。
- `minor`: 固定到次版本，`1.2.3` -> `>=1.2.3, <1.3`。
- `latest-up`: 固定到最新版本，`1.2.3` -> `>=1.2.3`。

```toml title="config.toml"
pinning-strategy = "no-pin"
```

### `mirrors`
用于配置 conda 渠道镜像，更多信息请参见 [下文](#mirror-configuration)。

```toml title="config.toml"
[mirrors]
# 将所有针对 conda-forge 的请求重定向到 prefix.dev 镜像
"https://conda.anaconda.org/conda-forge" = [
    "https://prefix.dev/conda-forge"
]

# 将所有针对 bioconda 的请求重定向到三个列出的镜像中的一个
# 注意：对于 repodata，我们首先尝试第一个镜像。
"https://conda.anaconda.org/bioconda" = [
    "https://conda.anaconda.org/bioconda",
    # 也支持 OCI 注册表
    "oci://ghcr.io/channel-mirrors/bioconda",
    "https://prefix.dev/bioconda",
]
```

### `repodata-config`
用于配置 repodata 获取。

```toml title="config.toml"
[repodata-config]
# 禁用获取 jlap、bz2 或 zstd 格式的 repodata 文件。
# 仅应在特定的旧版本 artifactory 和其他不兼容的服务器上使用此设置。
disable-jlap = true  # 不尝试下载 repodata.jlap
disable-bzip2 = true # 不尝试下载 repodata.json.bz2
disable-zstd = true  # 不尝试下载 repodata.json.zst
disable-sharded = true  # 不尝试下载分片的 repodata
```

上述设置可以通过在配置中指定渠道前缀来逐个渠道覆盖。

```toml title="config.toml"
[repodata-config."https://prefix.dev"]
disable-sharded = false
```

### `pypi-config`
用于设置 PyPI 注册表的若干默认配置。你可以使用以下配置选项：

- `index-url`: 用于 PyPI 包的默认索引 URL。它将在 `pixi init` 时添加到清单文件中。
- `extra-index-urls`: 用于 PyPI 包的额外 URL 列表。它将在 `pixi init` 时添加到清单文件中。
- `keyring-provider`: 允许使用 [keyring](https://pypi.org/project/keyring/) Python 包来存储和检索凭证。

```toml title="config.toml"
[pypi-config]
# 主索引 URL
index-url = "https://pypi.org/simple"
# 额外的 URL 列表
extra-index-urls = ["https://pypi.org/simple2"]
# 可以是 "subprocess" 或 "disabled"
keyring-provider = "subprocess"
```

!!! Note "`index-url` 和 `extra-index-urls` 不是全局的"
    与 pip 不同，这些设置（除了 `keyring-provider`）只会修改 `pixi.toml`/`pyproject.toml` 文件，在清单中不存在时不会全局解释。
    这是因为我们希望保持清单文件尽可能完整和可复现。

<span id="mirror-configuration"> </span>

## 镜像配置

你可以为 conda 渠道配置镜像。我们期望镜像是原始渠道的完整副本。实现方式是，在配置文件的 `mirrors` 部分查找镜像键（即 URL），并将原始 URL 替换为镜像 URL。

为了同时包含原始 URL，你必须将其重复添加到镜像列表中。

镜像会根据列表中的顺序进行优先级排序。我们尝试从列表中的第一个镜像获取 repodata（最重要的文件）。repodata 包含所有软件包的 SHA256 哈希，因此从受信任的来源获取此文件至关重要。

你还可以为整个“主机”指定镜像，例如：

```toml title="config.toml"
[mirrors]
"https://conda.anaconda.org" = [
    "https://prefix.dev/"
]
```

这将把所有针对 anaconda.org 上渠道的请求转发到 prefix.dev。如果某个渠道目前没有在 prefix.dev 上镜像，以上示例将失败。

### OCI 镜像

你还可以指定 OCI 注册表上的镜像。GitHub 容器注册表（ghcr.io）上有一个由 conda-forge 团队维护的公共镜像。你可以这样使用它：

```toml title="config.toml"
[mirrors]
"https://conda.anaconda.org/conda-forge" = [
    "oci://ghcr.io/channel-mirrors/conda-forge"
]
```

GHCR 镜像还包含 `bioconda` 包。你可以搜索 [GitHub 上可用的包](https://github.com/orgs/channel-mirrors/packages)。
