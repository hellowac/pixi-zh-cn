---
part: pixi/advanced
title: Info command
description: Learn what the info command reports
---

`pixi info` 打印出有用的信息，以帮助调试问题或了解你的机器/项目的概况。你还可以使用 `--json` 标志以 `json` 格式检索这些信息，这对于程序化读取非常有用。

```title="在 pixi 仓库中运行 pixi info"
➜ pixi info
      Pixi version: 0.13.0
          Platform: linux-64
  Virtual packages: __unix=0=0
                  : __linux=6.5.12=0
                  : __glibc=2.36=0
                  : __cuda=12.3=0
                  : __archspec=1=x86_64
         Cache dir: /home/user/.cache/rattler/cache
      Auth storage: /home/user/.rattler/credentials.json

Project
------------
           Version: 0.13.0
     Manifest file: /home/user/development/pixi/pixi.toml
      Last updated: 25-01-2024 10:29:08

Environments
------------
default
          Features: default
          Channels: conda-forge
  Dependency count: 10
      Dependencies: pre-commit, rust, openssl, pkg-config, git, mkdocs, mkdocs-material, pillow, cairosvg, compilers
  Target platforms: linux-64, osx-arm64, win-64, osx-64
             Tasks: docs, test-all, test, build, lint, install, build-docs
```

## 全局信息

信息输出的第一部分是 pixi 能够读取的、关于你机器的始终可用的信息。

### 平台

这定义了你当前所在的平台（根据 pixi 的定义）。  
如果这是不正确的，请在 [pixi 仓库](https://github.com/prefix-dev/pixi) 提交问题。

### 虚拟包

pixi 能够在你的机器上找到的虚拟包。

在 Conda 生态系统中，你可以依赖虚拟包。  
这些包并不是实际安装的依赖，而是在解决步骤中用来判断一个包是否能在机器上安装的工具。  
一个简单的例子是：当一个包依赖于主机上存在 Cuda 驱动时，它可以通过依赖 `__cuda` 虚拟包来检查。  
如果 pixi 无法在你的机器上找到 `__cuda` 虚拟包，则安装将会失败。

### 缓存目录

pixi 存储其缓存的目录。  
查看更多关于 [缓存的文档](../features/environment.md#caching-packages)。

### 认证存储

查看 [认证文档](authentication.md)。

### 缓存大小

[需要 `--extended`]

上述提到的“缓存目录”的大小，以 Mebibytes 为单位。

## 项目信息

`项目` 以下的信息是关于你当前所在项目的信息。  
只有当你的路径中有 [清单文件](../reference/project_configuration.md) 时，这些信息才会显示。

### 清单文件

描述项目的 [清单文件](../reference/project_configuration.md) 的路径。

### 最后更新

锁文件最后一次更新的时间，无论是手动更新还是由 pixi 本身更新。

## 环境信息

每个环境定义的环境信息。如果你没有定义任何环境，这部分只会显示 `default` 环境。

### 特性

列出在该环境中启用的特性。  
对于默认环境，仅列出 `default`。

### 通道

该环境中使用的通道列表。

### 依赖包数量

该环境中定义的依赖包数量（不是安装的依赖包数量）。

### 依赖包

该环境中定义的依赖包列表。

### 目标平台

项目定义的目标平台。
