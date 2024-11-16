---
part: pixi
title: Packaging pixi
description: 如何使用另一个包管理器打包 pixi 以便分发？
---

这是为希望将 pixi 打包到不同包管理器的发行版维护者提供的指南。  
pixi 的用户可以忽略此页面。

## 构建

Pixi 是用 Rust 编写的，并通过 Cargo 编译，这些是编译时所需的依赖。  
在运行时，pixi 除了它所编译时使用的运行时环境（如 `libc` 等）外，不需要其他依赖。

要构建 pixi，请运行：
```shell
cargo build --locked --profile dist
```
除了使用预定义的 `dist` 配置文件（该配置文件针对二进制文件大小进行了优化），你还可以传递其他选项，让 Cargo 根据其他指标优化二进制文件。

### 构建时选项

Pixi 提供了一些构建时选项，这些选项会影响构建过程。

#### TLS

默认情况下，pixi 使用 Rustls TLS 实现进行构建。你可以通过在构建命令中添加 `--no-default-features --feature native-tls` 来使用平台本地的 TLS 实现进行编译。请注意，这可能会增加额外的运行时依赖，如 Linux 上的 OpenSSL。

#### 自我更新

Pixi 具有自我更新功能。当 pixi 是通过其他包管理器安装时，通常不希望 pixi 尝试自我更新，而是让包管理器更新它。  
因此，默认情况下自我更新功能是禁用的。可以通过在构建命令中添加 `--feature self_update` 来启用此功能。

当禁用自我更新功能时，如果用户尝试运行 `pixi self-update`，将显示一条错误消息。  
这条消息可以通过在构建时设置 `PIXI_SELF_UPDATE_DISABLED_MESSAGE` 环境变量来定制，指示用户使用哪个包管理器来更新 pixi。
```shell
PIXI_SELF_UPDATE_DISABLED_MESSAGE="`self-update` has been disabled for this build. Run `brew upgrade pixi` instead" cargo build --locked --profile dist
```

#### 自定义版本

你可以通过在构建时设置 `PIXI_VERSION` 环境变量来指定自定义版本字符串，用于 `--version` 输出。

```shell
PIXI_VERSION="HEAD-123456" cargo build --locked --profile dist
```

## Shell 自动补全

构建完 pixi 后，你可以通过运行以下命令生成 shell 自动补全脚本：
```shell
pixi completion --shell <SHELL>
```
并将输出保存到文件中。  
目前支持的 shell 有 `bash`、`elvish`、`fish`、`nushell`、`powershell` 和 `zsh`。
