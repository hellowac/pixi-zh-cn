---
part: pixi
title: 入门
description: 让打包管理更简单
---

# 入门指南

![Pixi 与魔法棒](assets/pixi.webp)

Pixi 是一款面向开发者的包管理工具，能够以可复现的方式安装库和应用程序。它支持跨平台运行，可在 Windows、Mac 和 Linux 上使用。

---

## 安装

在终端中运行以下命令即可安装 `pixi`：

=== "Linux & macOS"

    ```bash
    curl -fsSL https://pixi.sh/install.sh | bash
    ```

    上述命令将自动下载 `pixi` 的最新版本，解压后将 `pixi` 可执行文件移动到 `~/.pixi/bin` 目录。如果目录不存在，脚本会自动创建它。

    脚本还会更新 `~/.bash_profile`，将 `~/.pixi/bin` 添加到 PATH 中，使得可以从任意位置调用 `pixi` 命令。

=== "Windows"

    - **PowerShell**:
    
        ```powershell
        iwr -useb https://pixi.sh/install.ps1 | iex
        ```

    - **winget**:
    
        ```
        winget install prefix-dev.pixi
        ```

    上述命令将自动下载 `pixi` 的最新版本，解压后将其移动到 `LocalAppData/pixi/bin` 目录。如果目录不存在，脚本会自动创建它。

    命令还会自动将 `LocalAppData/pixi/bin` 添加到 PATH 中，使得可以从任意位置调用 `pixi`。

!!! tip

    您可能需要重启终端或重新加载 shell 配置文件以使更改生效。

安装脚本的更多选项可以参考 [这里](#installer-script-options)。

---

## 自动补全

按照以下说明为您的 shell 启用命令补全功能。完成后，请重启终端或重新加载 shell 配置文件。

### Bash（大多数 Linux 系统的默认 shell）

```bash
echo 'eval "$(pixi completion --shell bash)"' >> ~/.bashrc
```

### Zsh（macOS 的默认 shell）

```zsh
echo 'eval "$(pixi completion --shell zsh)"' >> ~/.zshrc
```

### PowerShell（所有 Windows 系统的预装 shell）

```pwsh
Add-Content -Path $PROFILE -Value '(& pixi completion --shell powershell) | Out-String | Invoke-Expression'
```

!!! tip "如果提示“未找到配置文件”"
    
    请确保配置文件已存在，否则使用以下命令创建它：

    ```PowerShell
    New-Item -Path $PROFILE -ItemType File -Force
    ```


### Fish

```fish
echo 'pixi completion --shell fish | source' > ~/.config/fish/completions/pixi.fish
```

### Nushell

在 Nushell 中，向环境文件的末尾添加以下内容（可通过运行 `$nu.env-path` 查找路径）：

```nushell
mkdir ~/.cache/pixi
pixi completion --shell nushell | save -f ~/.cache/pixi/completions.nu
```

然后，将以下内容添加到 Nushell 配置文件的末尾（可通过运行 `$nu.config-path` 查找路径）：

```nushell
use ~/.cache/pixi/completions.nu *
```

---

### Elvish

在 Elvish 中，运行以下命令将自动补全脚本添加到配置文件中：

```elv
echo 'eval (pixi completion --shell elvish | slurp)' >> ~/.elvish/rc.elv
```

---

## 其他安装方法

尽管推荐使用上述安装方法，但我们也提供其他安装方式以满足不同需求。

### Homebrew

Pixi 可通过 Homebrew 安装。运行以下命令：

```shell
brew install pixi
```

---

### Windows 安装程序

我们在 [GitHub Releases 页面](https://github.com/prefix-dev/pixi/releases/latest) 提供了一个 `msi` 安装程序。安装程序会下载 pixi 并将其添加到 PATH。

---

### 从源码安装

Pixi 完全使用 Rust 编写，因此可以通过 `cargo` 安装、构建和测试。

运行以下命令从源码构建并安装 Pixi：

```shell
cargo install --locked --git https://github.com/prefix-dev/pixi.git pixi
```

我们已停止将 Pixi 发布到 `crates.io`，需从官方仓库安装。原因是 Pixi 依赖于一些未发布的 crates，因此无法在 `crates.io` 上发布。

若需进行修改，可以使用以下命令：

```shell
cargo build
cargo test
```

如果因依赖 `rattler` 导致构建失败，请参考其[编译步骤](https://github.com/mamba-org/rattler/tree/main#give-it-a-try)。

<span id="installer-script-options"></span>

### 安装脚本选项

#### Linux 和 macOS

安装脚本支持通过环境变量调整多个选项：

| 变量名                 | 描述                                                                                     | 默认值              |
|------------------------|-----------------------------------------------------------------------------------------|---------------------|
| `PIXI_VERSION`         | 要安装的 pixi 版本，可用于升级或降级。                                                   | `latest`            |
| `PIXI_HOME`            | 二进制文件的存储目录。                                                                   | `$HOME/.pixi`       |
| `PIXI_ARCH`            | 指定要安装的 pixi 构建所适用的架构。                                                     | `uname -m`          |
| `PIXI_NO_PATH_UPDATE`  | 如果设置，则不会将 `pixi` 路径加入 `$PATH`。                                              |                     |
| `TMP_DIR`              | 脚本用于下载和解压二进制文件的临时目录。                                                 | `/tmp`              |

例如，在 Apple Silicon 上强制安装 x86 架构版本：

```shell
curl -fsSL https://pixi.sh/install.sh | PIXI_ARCH=x86_64 bash
```

或指定安装版本：
```shell
curl -fsSL https://pixi.sh/install.sh | PIXI_VERSION=v0.18.0 bash
```

---

#### Windows

Windows 安装脚本支持以下选项，通过环境变量调整：

| 选项            | 环境变量名         | 描述                                                                                     | 默认值                      |
|------------------|--------------------|-----------------------------------------------------------------------------------------|-----------------------------|
| `PixiVersion`    | `PIXI_VERSION`    | 要安装的 pixi 版本，可用于升级或降级。                                                   | `latest`                    |
| `PixiHome`       | `PIXI_HOME`       | 指定安装位置。                                                                           | `$Env:USERPROFILE\.pixi`    |
| `NoPathUpdate`   |                   | 如果设置，则不会将 `pixi` 路径加入 `$PATH`。                                              |                             |

例如，通过以下方式指定安装版本：

```powershell
iwr -useb https://pixi.sh/install.ps1 | iex -Args "-PixiVersion v0.18.0"
```

---

### 更新

更新 Pixi 与安装方式类似，重新运行安装脚本即可获取最新版本：

```shell
pixi self-update
```

也可以获取特定版本：
```shell
pixi self-update --version x.y.z
```

!!! 注意
    如果使用 `brew`、`mamba`、`conda` 或 `paru` 等包管理工具安装了 pixi，请使用相应工具更新，例如：
    ```shell
    brew upgrade pixi
    ```

---

### 卸载

要卸载 Pixi，只需删除二进制文件。

#### Linux 和 macOS
```shell
rm ~/.pixi/bin/pixi
```

#### Windows
```powershell
$PIXI_BIN = "$Env:LocalAppData\pixi\bin\pixi"; Remove-Item -Path $PIXI_BIN
```

卸载后，仍可使用通过 pixi 安装的工具。如果希望一并删除这些工具，请移除整个 `~/.pixi` 目录并从路径中移除相关目录。
