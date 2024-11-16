<h1>
  <a href="https://github.com/prefix-dev/pixi/">
    <picture>
      <source srcset="https://github.com/prefix-dev/pixi/assets/4995967/a3f9ff01-c9fb-4893-83c0-2a3f924df63e" type="image/webp">
      <source srcset="https://github.com/prefix-dev/pixi/assets/4995967/e42739c4-4cd9-49bb-9d0a-45f8088494b5" type="image/png">
      <img src="https://github.com/prefix-dev/pixi/assets/4995967/e42739c4-4cd9-49bb-9d0a-45f8088494b5" alt="banner">
    </picture>
  </a>
</h1>

<h1 align="center">

![License][license-badge]
[![Project Chat][chat-badge]][chat-url]
[![Pixi Badge][pixi-badge]][pixi-url]


[license-badge]: https://img.shields.io/badge/license-BSD--3--Clause-blue?style=flat-square
[chat-badge]: https://img.shields.io/discord/1082332781146800168.svg?label=&logo=discord&logoColor=ffffff&color=7389D8&labelColor=6A7EC2&style=flat-square
[chat-url]: https://discord.gg/kKV8ZxyzY4
[pixi-badge]:https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/prefix-dev/pixi/main/assets/badge/v0.json&style=flat-square
[pixi-url]: https://pixi.sh

</h1>

# pixi: 让打包管理更简单

## 概览

`pixi` 是一个跨平台、多语言的包管理和工作流工具，基于 conda 生态系统构建。它为开发者提供了一种卓越的使用体验，类似于流行的包管理工具如 `cargo` 或 `yarn`，但适用于任何语言。

由 [prefix.dev](https://prefix.dev) ❤️ 开发。  
[![实时 pixi_demo](https://github.com/prefix-dev/pixi/assets/12893423/0fc8f8c8-ac13-4c14-891b-dc613f25475b)](https://asciinema.org/a/636482)

## 高亮显示

- 支持 **多种语言**，包括 Python、C++ 和 R，通过 Conda 包进行管理。可在 [prefix.dev](https://prefix.dev) 上查找可用包。  
- 兼容所有主流操作系统：Linux、Windows、macOS（包括 Apple Silicon）。  
- 始终包含最新的 **锁定文件（lock file）**。  
- 提供简洁、类似 Cargo 的 **命令行界面**。  
- 支持 **按项目** 或 **系统范围** 安装工具。  
- 完全用 **Rust** 编写，基于 **[rattler](https://github.com/mamba-org/rattler)** 库构建。

## 入门

- ⚡ [安装指南](#installation)  
- ⚙️ [示例](/examples)  
- 📚 [文档](https://pixi.sh/)  
- 😍 [贡献指南](#contributing)  
- 🔨 [基于 Pixi 构建的项目](#built-using-pixi)  
- 🚀 [GitHub Action](https://github.com/prefix-dev/setup-pixi)  

## 状态

Pixi 已经准备好投入生产环境使用！  
我们正在努力保持文件格式的向后兼容性，以确保您可以放心地依赖 Pixi。

以下是我们计划在即将发布的版本中实现的一些重要功能：

- **构建和发布**您的项目为 Conda 包。  
- 支持从**源码构建依赖项**。  
- 提供更强大的“全局安装”功能，以实现多个设备上全局包的确定性设置。  

## 安装指南

`pixi` 支持在 macOS、Linux 和 Windows 上安装。提供的脚本会自动下载 `pixi` 的最新版本，解压并将 `pixi` 二进制文件移动到 `~/.pixi/bin` 目录。如果该目录不存在，脚本会自动创建它。

### macOS 和 Linux

在 macOS 和 Linux 上安装 Pixi，请打开终端并运行以下命令：

```bash
curl -fsSL https://pixi.sh/install.sh | bash
# 或使用 brew
brew install pixi
```

安装脚本还会更新您的 `~/.bash_profile` 文件，将 `~/.pixi/bin` 添加到 PATH 中，这样您可以在任意位置调用 `pixi` 命令。
安装完成后，可能需要重启终端或执行 `source` 命令以使更改生效。

从 macOS Catalina 开始，[zsh 是默认的登录和交互式 shell](https://support.apple.com/en-us/102360)。因此，您可能需要在安装命令中使用 `zsh` 替代 `bash`：

```zsh
curl -fsSL https://pixi.sh/install.sh | zsh
```

该脚本还会更新您的 `~/.zshrc` 文件，将 `~/.pixi/bin` 添加到 PATH 中，确保可以从任意位置调用 `pixi` 命令。

### Windows

在 Windows 上安装 Pixi，请打开 PowerShell 终端（可能需要以管理员身份运行）并运行以下命令：

```powershell
iwr -useb https://pixi.sh/install.ps1 | iex
```

脚本完成后会提示安装成功，并将 `~/.pixi/bin` 目录添加到 PATH 中，这样您可以从任意位置运行 `pixi` 命令。
或者使用 `winget`：

```shell
winget install prefix-dev.pixi
```

### 自动补全

要启用自动补全功能，请按照以下说明为您的 shell 配置。完成后，请重新启动 shell 或手动加载 shell 配置文件。

#### Bash（大多数 Linux 系统默认的 shell）

```bash
echo 'eval "$(pixi completion --shell bash)"' >> ~/.bashrc
```

#### Zsh（macOS 默认的 shell）

```zsh
echo 'eval "$(pixi completion --shell zsh)"' >> ~/.zshrc
```

#### PowerShell（所有 Windows 系统预装）

```pwsh
Add-Content -Path $PROFILE -Value '(& pixi completion --shell powershell) | Out-String | Invoke-Expression'
```

如果提示 “由于配置文件不存在而失败”，请确保配置文件存在。如果不存在，请使用以下命令创建：

```PowerShell
New-Item -Path $PROFILE -ItemType File -Force
```

#### Fish

```fish
echo 'pixi completion --shell fish | source' >> ~/.config/fish/config.fish
```

#### Nushell

在 Nushell 中，添加以下内容到您的环境文件末尾（通过运行 `$nu.env-path` 找到文件路径）：

```nushell
mkdir ~/.cache/pixi
pixi completion --shell nushell | save -f ~/.cache/pixi/completions.nu
```

然后，在您的 Nushell 配置文件末尾添加以下内容（通过运行 `$nu.config-path` 找到文件路径）：

```nushell
use ~/.cache/pixi/completions.nu *
```

#### Elvish

在 Elvish 中，运行以下命令将自动补全配置添加到启动脚本：

```elv
echo 'eval (pixi completion --shell elvish | slurp)' >> ~/.elvish/rc.elv
```

---

### 使用 Distro 包安装

[![Packaging status](https://repology.org/badge/vertical-allrepos/pixi.svg)](https://repology.org/project/pixi/versions)

#### Arch Linux

您可以通过 [extra 仓库](https://archlinux.org/packages/extra/x86_64/pixi/) 使用 [pacman](https://wiki.archlinux.org/title/Pacman) 安装 `pixi`：

```shell
pacman -S pixi
```

#### Alpine Linux

`pixi` 已可用于 [Alpine Edge](https://pkgs.alpinelinux.org/packages?name=pixi&branch=edge)。启用 [testing 仓库](https://wiki.alpinelinux.org/wiki/Repositories) 后，可通过 [apk](https://wiki.alpinelinux.org/wiki/Alpine_Package_Keeper) 安装：

```shell
apk add pixi
```

---

### 从源码构建/安装

由于 `pixi` 是完全用 Rust 编写的，您可以使用 `cargo` 进行安装、构建和测试。

运行以下命令从源码构建并开始使用 `pixi`：

```shell
cargo install --locked --git https://github.com/prefix-dev/pixi.git pixi
```

注意，`pixi` 不再发布到 `crates.io`，需要直接从存储库安装。原因是我们依赖了一些未发布的 crates，无法在 `crates.io` 发布。

如果您计划修改代码，请使用以下命令：

```shell
cargo build
cargo test
```

如果因依赖 `rattler` 而导致构建问题，请查看其[编译步骤](https://github.com/mamba-org/rattler/tree/main#give-it-a-try)。

---

### 卸载

要卸载 `pixi`，只需删除安装的二进制文件。在默认路径 `~/.pixi/bin/pixi` 中找到 `pixi` 并删除它。

在 Linux 上运行以下命令：

```shell
rm ~/.pixi/bin/pixi
```

在 Windows 上运行：

```shell
$PIXI_BIN = "$Env:LocalAppData\pixi\bin\pixi"; Remove-Item -Path $PIXI_BIN
```

卸载后，您仍然可以使用之前通过 `pixi` 安装的工具。如果想删除这些工具，请同时删除整个 `~/.pixi` 目录，并从 PATH 中移除该路径。

## 使用说明

`pixi` 的命令行界面如下所示：

```bash
➜ pixi
A package management and workflow tool

Usage: pixi [OPTIONS] <COMMAND>

Commands:
  completion  Generates a completion script for a shell
  init        Creates a new project
  add         Adds a dependency to the project
  run         Runs task in project
  shell       Start a shell in the pixi environment of the project
  global      Global is the main entry point for the part of pixi that executes on the global(system) level
  auth        Login to prefix.dev or anaconda.org servers to access private channels
  install     Install all dependencies
  task        Command management in project
  info        Information about the system and project
  upload      Upload a package to a prefix.dev channel
  search      Search a package, output will list the latest version of package
  project     
  help        Print this message or the help of the given subcommand(s)

Options:
  -v, --verbose...     More output per occurrence
  -q, --quiet...       Less output per occurrence
      --color <COLOR>  Whether the log needs to be colored [default: auto] [possible values: always, never, auto]
  -h, --help           Print help
  -V, --version        Print version
```

---

### 创建一个 Pixi 项目

初始化一个新项目并进入项目目录：

```bash
pixi init myproject
cd myproject
```

添加所需的依赖项：

```bash
pixi add cowpy
```

在项目环境中运行已安装的包：

```bash
pixi run cowpy "Thanks for using pixi"
```

激活项目环境中的 shell：

```bash
pixi shell
cowpy "Thanks for using pixi"
exit
```

---

### 全局安装 Conda 包

您也可以将 Conda 包全局安装到单独的环境中。此行为类似于 [`pipx`](https://github.com/pypa/pipx) 或 [`condax`](https://github.com/mariusvniekerk/condax)。

```bash
pixi global install cowpy
```

---

### 在 GitHub Actions 中使用

可以在 GitHub Actions 中使用 `pixi` 安装依赖项并运行命令，支持环境的自动缓存。

示例：

```yml
- uses: prefix-dev/setup-pixi@v0.8.1
- run: pixi exec cowpy "Thanks for using pixi"
```

有关更多详情，请参阅 [文档](https://pixi.sh/latest/advanced/github_actions)。

---

## 贡献 😍

我们热忱欢迎您为 `pixi` 项目做出贡献！无论是提交问题、修复错误，还是提出改进建议，每一个贡献都值得珍惜。

如果您是初次接触本项目或 Rust 生态系统，请放心，我们会为您提供帮助！建议从标记为 `good first issue` 的问题入手，这些任务相对简单，适合熟悉项目和流程。

如有疑问、想法或希望交流，欢迎加入我们的 Discord 社区，与大家一起探讨。我们期待您的加入！[立即加入我们的 Discord 服务器][chat-url]。

---

## 使用 Pixi 构建的项目

查看 [社区页面](/docs/Community.md) 来了解更多使用 `pixi` 构建的项目。


## 部署中文文档

```bash
mkdocs gh-deploy -f mkdocs_cn.yml
```