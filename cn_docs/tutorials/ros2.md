### 教程：使用 `pixi` 开发 ROS 2 包

在本教程中，我们将向您展示如何使用 `pixi` 开发一个 ROS 2 包。
教程将按顺序执行，缺少步骤可能会导致错误。

本教程的目标读者是已经熟悉 ROS 2 并希望尝试 `pixi` 开发工作流的开发者。

## 前提条件

- 您需要安装 `pixi`。如果尚未安装，可以按照[安装指南](../index.md)中的说明进行操作。
  本教程的核心目的是向您展示，您只需要 `pixi` 即可完成开发！
- 在 Windows 上，建议启用开发者模式。请进入设置 -> 更新与安全 -> 开发者选项 -> 开发者模式。

!!! note ""
    如果您是 `pixi` 的新手，可以先查看 [基本使用](../basic_usage.md)指南。
    该指南会在 3 分钟内向您介绍如何创建一个 `pixi` 项目。

## 创建一个 pixi 项目

```shell
pixi init my_ros2_project -c robostack-staging -c conda-forge
cd my_ros2_project
```

执行后，应该会创建如下的目录结构：

```shell
my_ros2_project
├── .gitattributes
├── .gitignore
└── pixi.toml
```

`pixi.toml` 文件是您的项目的清单文件，内容应该类似于：

```toml title="pixi.toml"
[project]
name = "my_ros2_project"
version = "0.1.0"
description = "Add a short description here"
authors = ["User Name <user.name@email.url>"]
channels = ["robostack-staging", "conda-forge"]
# Your project can support multiple platforms, the current platform will be automatically added.
platforms = ["linux-64"]

[tasks]

[dependencies]
```

在 `init` 命令中添加的 `channels` 是软件包的仓库，您可以通过我们的网站 [prefix.dev](https://prefix.dev/channels) 搜索这些仓库中的包。
`platforms` 定义了您希望支持的操作系统，`pixi` 可以支持多个平台，但需要您定义这些平台，以便 `pixi` 检查您的依赖项是否支持这些平台。
其余字段可以根据需要填写。

## 添加 ROS 2 依赖项

使用 `pixi` 开发项目时，您不需要在系统上安装任何依赖项，所有需要的依赖项都应该通过 `pixi` 添加，这样其他用户就可以无障碍地使用您的项目。

首先，我们以 `turtlesim` 示例开始：

```shell
pixi add ros-humble-desktop ros-humble-turtlesim
```

这将会把 `ros-humble-desktop` 和 `ros-humble-turtlesim` 包添加到您的清单文件中。
根据您的网络速度，这可能需要一会儿，因为 `pixi` 还将 ROS 安装到您的项目文件夹（`.pixi`）中。

现在，运行 `turtlesim` 示例：

```shell
pixi run ros2 run turtlesim turtlesim_node
```

**或者** 使用 `shell` 命令在终端中启动一个已激活的环境。

```shell
pixi shell
ros2 run turtlesim turtlesim_node
```

恭喜您，您已经在机器上通过 `pixi` 成功运行了 ROS 2！

??? example "更多与小乌龟的互动"
    要控制小乌龟，您可以在新的终端窗口中运行以下命令：
    ```shell
    cd my_ros2_project
    pixi run ros2 run turtlesim turtle_teleop_key
    ```
    现在，您可以使用键盘上的箭头键控制小乌龟的运动。

![Turtlesim 控制](https://github.com/user-attachments/assets/9424c44b-b7c0-48f4-8e7d-501131e9e9e5)

### 添加自定义 Python 节点

由于 ROS 使用自定义节点，我们将向您的项目中添加一个自定义节点。

```shell
pixi run ros2 pkg create --build-type ament_python --destination-directory src --node-name my_node my_package
```

为了构建包，我们还需要一些额外的依赖项：

```shell
pixi add colcon-common-extensions "setuptools<=58.2.0"
```

将创建的 ROS 工作区初始化脚本添加到您的清单文件中。

然后运行构建命令：

```shell
pixi run colcon build
```

这将创建一个可源化的脚本，位于 `install` 文件夹中，您可以通过激活脚本使用您的自定义节点。
通常，这是您会添加到 `.bashrc` 中的脚本，但在这里，您可以通过向 `pixi.toml` 文件添加以下内容来告诉 `pixi` 使用它：

=== "Linux & macOS"
    ```toml title="pixi.toml"
    [activation]
    scripts = ["install/setup.sh"]
    ```

=== "Windows"
    ```toml title="pixi.toml"
    [activation]
    scripts = ["install/setup.bat"]
    ```

??? tip "多平台支持"
    您可以为不同的平台添加多个激活脚本，这样一个项目就可以支持多个平台。
    使用以下示例，您可以支持 Linux 和 Windows，并使用 [target](../features/multi_platform_configuration.md#activation) 语法进行配置：

    ```toml
    [project]
    platforms = ["linux-64", "win-64"]

    [activation]
    scripts = ["install/setup.sh"]
    [target.win-64.activation]
    scripts = ["install/setup.bat"]
    ```

现在，您可以使用以下命令运行您的自定义节点：

```shell
pixi run ros2 run my_package my_node
```

## 简化用户体验

在 `pixi` 中，我们有一个名为 `tasks` 的功能，允许您在清单文件中定义任务，并通过简单的命令来运行它。
让我们添加一个任务来运行 `turtlesim` 示例和自定义节点。

```shell
pixi task add sim "ros2 run turtlesim turtlesim_node"
pixi task add build "colcon build --symlink-install"
pixi task add hello "ros2 run my_package my_node"
```

现在，您只需运行以下命令即可执行这些任务：

```shell
pixi run sim
pixi run build
pixi run hello
```

???+ tip "高级任务使用"
    任务是 `pixi` 中的强大功能。

    - 您可以向任务中添加 [`depends-on`](../features/advanced_tasks.md#depends-on) 来创建任务链。
    - 您可以向任务中添加 [`cwd`](../features/advanced_tasks.md#working-directory) 来在与项目根目录不同的目录中运行任务。
    - 您可以向任务中添加 [`inputs` 和 `outputs`](../features/advanced_tasks.md#caching) 来创建一个仅在输入更改时才运行的任务。
    - 您还可以使用 [`target`](../reference/project_configuration.md#target) 语法在特定的机器上运行特定的任务。

```toml
[tasks]
sim = "ros2 run turtlesim turtlesim_node"
build = {cmd = "colcon build --symlink-install", inputs = ["src"]}
hello = { cmd = "ros2 run my_package my_node", depends-on = ["build"] }
```

## 构建 C++ 节点

要构建 C++ 节点，您需要将 `ament_cmake` 和一些其他构建依赖项添加到清单文件中。

```shell
pixi add ros-humble-ament-cmake-auto compilers pkg-config cmake ninja
```

现在，您可以使用以下命令创建一个 C++ 节点：

```shell
pixi run ros2 pkg create --build-type ament_cmake --destination-directory src --node-name my_cpp_node my_cpp_package
```

然后，您可以使用以下命令再次构建并运行它：

```shell
# 传递参数给构建命令，以便使用 Ninja 构建，将这些参数添加到清单文件中以便默认使用 ninja。
pixi run build --cmake-args -G Ninja
pixi run ros2 run my_cpp_package my_cpp_node
```

??? tip
    将 C++ 任务添加到清单文件中，以简化用户体验：

    ```shell
    pixi task add hello-cpp "ros2 run my_cpp_package my_cpp_node"
    ```

## 结论

在本教程中，我们向您展示了如何使用 `pixi` 创建一个 Python 和 CMake 的 ROS 2 项目。
我们还向您展示了如何使用 `pixi` 向项目中 **添加依赖项**，并如何使用 `pixi run` **运行项目**。
通过这种方式，您可以确保您的项目在所有安装了 `pixi` 的机器上都是 **可重复构建** 的。

## 展示您的作品！

完成您的项目了吗？
我们很想看看您做了什么！
通过社交媒体使用标签 #pixi 并标记我们 @prefix_dev，分享您的作品。
让我们一起激励社区！

## 常见问题

### `rosdep` 怎么办？

目前，我们不支持在 `pixi` 环境中使用 `rosdep`，因此您需要使用 `pixi add` 来添加包。
`rosdep` 会调用 `conda install`，而这在 `pixi` 环境中是不可用的。
