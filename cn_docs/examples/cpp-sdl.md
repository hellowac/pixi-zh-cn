---
part: pixi/examples
title: SDL example
description: How to build and run an SDL application in C++
---

![](https://storage.googleapis.com/prefix-cms-images/docs/sdl_examle.png)  
`cpp-sdl` 示例位于 `pixi` 仓库中。

```shell
git clone https://github.com/prefix-dev/pixi.git
```

进入示例文件夹

```shell
cd pixi/examples/cpp-sdl
```

运行 `start` 命令

```shell
pixi run start
```

通过使用 [`depends-on`](../features/advanced_tasks.md#depends-on) 特性，您只需要运行 `start` 任务，但在后台它会运行以下任务：

```shell
# 配置 CMake 项目
pixi run configure

# 构建可执行文件
pixi run build

# 启动构建的可执行文件
pixi run start
```
