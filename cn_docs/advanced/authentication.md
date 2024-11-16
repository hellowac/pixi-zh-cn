---
part: pixi
title: 使用服务器验证 pixi
description: 验证 pixi 以访问私人频道
---

你可以使用像 prefix.dev、私有的 quetz 实例或 anaconda.org 这样的服务器来认证 pixi。不同的服务器使用不同的认证方法。在本页面中，我们详细说明了如何对不同的服务器进行认证以及认证信息存储的位置。

```shell
用法: pixi auth login [选项] <HOST>

参数:
  <HOST>  需要认证的主机（例如 repo.prefix.dev）

选项:
      --token <TOKEN>              使用的令牌（用于与 prefix.dev 认证）
      --username <USERNAME>        使用的用户名（用于基本 HTTP 认证）
      --password <PASSWORD>        使用的密码（用于基本 HTTP 认证）
      --conda-token <CONDA_TOKEN>  用于 anaconda.org / quetz 认证的令牌
  -v, --verbose...                 输出更多信息
  -q, --quiet...                   输出更少信息
  -h, --help                       打印帮助信息
```

不同的选项包括 "token"、"conda-token" 和 "username + password"。

- **token** 变体实现了标准的 "Bearer Token" 认证，这在 prefix.dev 平台上使用。Bearer Token 会在每次请求时作为额外的头信息发送，格式为 `Authentication: Bearer <TOKEN>`。
  
- **conda-token** 选项用于 anaconda.org，也可以与 quetz 服务器一起使用。此选项将令牌作为 URL 的一部分发送，格式为：`conda.anaconda.org/t/<TOKEN>/conda-forge/linux-64/...`。

- **用户名和密码** 选项用于 "基本 HTTP 认证"。这相当于将 `http://user:password@myserver.com/...` 添加到 URL 中。此认证方法可以通过反向 Nginx 或 Apache 服务器进行配置，因此在自托管系统中很常见。

## 示例

登录到 prefix.dev：

```shell
pixi auth login prefix.dev --token pfx_jj8WDzvnuTHEGdAhwRZMC1Ag8gSto8
```

登录到 anaconda.org：

```shell
pixi auth login anaconda.org --conda-token xy-72b914cc-c105-4ec7-a969-ab21d23480ed
```

登录到基本 HTTP 安全的服务器：

```shell
pixi auth login myserver.com --username user --password password
```

## pixi 如何存储认证信息？

认证信息的存储位置取决于系统。默认情况下，pixi 会尝试使用钥匙串在你的机器上安全存储这些敏感信息。

- 在 Windows 上，凭据存储在 "凭据管理器" 中。搜索 `rattler`（pixi 使用的底层库），你应该能找到任何由 pixi（或其他基于 rattler 的程序）存储的凭据。

- 在 macOS 上，密码存储在钥匙串中。你可以使用 macOS 自带的 `钥匙串访问` 程序访问密码。搜索 `rattler`（pixi 使用的底层库），你应该能找到任何由 pixi（或其他基于 rattler 的程序）存储的凭据。

- 在 Linux 上，可以使用 `GNOME Keyring`（或仅 Keyring）来访问由 `libsecret` 安全存储的凭据。搜索 `rattler` 应该会列出所有由 pixi 和其他基于 rattler 的程序存储的凭据。

## 后备存储

如果你的服务器没有提供上述任何钥匙串，pixi 会回退到将凭据存储在一个不安全的 JSON 文件中。
此 JSON 文件位于 `~/.rattler/credentials.json` 并包含认证信息。

## 覆盖认证存储

你可以使用 `RATTLER_AUTH_FILE` 环境变量来覆盖默认的凭据文件位置。
当设置此环境变量时，它将成为 pixi 使用的唯一认证数据来源。

例如：

```bash
export RATTLER_AUTH_FILE=$HOME/credentials.json
# 你也可以在命令行中指定文件
pixi global install --auth-file $HOME/credentials.json ...
```

JSON 文件应遵循以下格式：

```json
{
    "*.prefix.dev": {
        "BearerToken": "your_token"
    },
    "otherhost.com": {
        "BasicHTTP": {
            "username": "your_username",
            "password": "your_password"
        }
    },
    "conda.anaconda.org": {
        "CondaToken": "your_token"
    }
}
```

注意：如果使用了通配符，则任何子域都会匹配（例如 `*.prefix.dev` 也匹配 `repo.prefix.dev`）。

最后，你可以在 [全局配置文件](./../reference/pixi_configuration.md) 中设置认证覆盖文件。

## PyPI 认证

目前，我们支持以下几种 PyPI 认证方法：

1. [keyring](https://pypi.org/project/keyring/) 认证。
2. `.netrc` 文件认证。

我们希望将来能够支持更多的认证方法，因此，如果你有特定的方法希望看到支持，请告知我们。

### Keyring 认证

目前，pixi 通过 Python [keyring](https://pypi.org/project/keyring/) 库支持 uv 认证方法。

#### 安装 keyring

要安装 keyring，可以使用 `pixi global install`：

=== "基本认证"
    ```shell
    pixi global install keyring
    ```
=== "Google Artifact Registry"
    ```shell
    pixi global install keyring --with keyrings.google-artifactregistry-auth
    ```
=== "Azure DevOps Artifacts"
    ```shell
    pixi global install keyring --with keyring.artifacts
    ```

对于其他注册表，你需要根据这些说明调整并添加正确的 keyring 后端。

#### 配置项目以使用 keyring

=== "基本认证"
    使用 keyring 存储你的凭据，例如：

    ```shell
    keyring set https://my-index/simple your_username
    # 系统会提示输入密码
    ```

    在你的 pixi 清单中添加以下配置，确保在注册表的 URL 中包含 `your_username@`：

    ```toml
    [pypi-options]
    index-url = "https://your_username@custom-registry.com/simple"
    ```

=== "Google Artifact Registry"
    确保你已登录，例如通过运行 `gcloud auth login`，然后在你的 pixi 清单中添加以下配置：

    ```toml
    [pypi-options]
    extra-index-urls = ["https://oauth2accesstoken@<location>-python.pkg.dev/<project>/<repository>/simple"]
    ```

    !!!注意
        为了更容易找到此 URL，你可以使用 `gcloud` 命令：

        ```shell
        gcloud artifacts print-settings python --project=<project> --repository=<repository> --location=<location>
        ```

=== "Azure DevOps Artifacts"
    在遵循 [`keyring.artifacts` 说明](https://github.com/jslorrma/keyrings.artifacts?tab=readme-ov-file#usage) 并确保 keyring 正常工作后，在你的 pixi 清单中添加以下配置：

    ```toml
    [pypi-options]
    extra-index-urls = ["https://VssSessionToken@pkgs.dev.azure.com/{organization}/{project}/_packaging/{feed}/pypi/simple/"]
    ```

#### 安装你的环境

可以配置你的 [全局配置](../reference/pixi_configuration.md#pypi-config)，或者使用 `--pypi-keyring-provider` 选项，该选项可以设置为 `subprocess`（启用）或 `disabled`（禁用）：

```shell
# 从现有的 pixi 项目中安装
pixi install --pypi-keyring-provider subprocess
```

### `.netrc` 文件

`pixi` 允许你通过认证存储在 `.netrc` 文件中的凭据安全地访问私有注册表。

- `.netrc` 文件可以存储在你的主目录下（对于类 Unix 系统为 `$HOME/.netrc`）
- 或者在 Windows 的用户配置文件目录中（`%HOME%\_netrc`）。
- 你也可以使用 `NETRC` 变量设置 `.netrc` 文件的不同位置（`export NETRC=/my/custom/location/.netrc`）。
  例如：`export NETRC=/my/custom/location/.netrc pixi install`

在 `.netrc` 文件中，你可以像这样存储认证信息：

```sh
machine registry-name
login admin
password admin
```

有关更多详细信息，你可以访问 [.netrc 文档](https://www.ibm.com/docs/en/aix/7.2?topic=formats-netrc-file-format-tcpip)。
