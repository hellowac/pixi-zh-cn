所有关于决定哪些依赖可以从哪个通道安装的逻辑，都是由我们给求解器提供的指令来决定的。

实际处理这部分的代码位于 [`rattler_solve`](https://github.com/mamba-org/rattler/blob/02e68c9539c6009cc1370fbf46dc69ca5361d12d/crates/rattler_solve/src/resolvo/mod.rs) crate 中。  
然而，这部分代码可能比较难以阅读。  
因此，本文将继续使用简化的流程图来解释。

## 通道特定依赖
当用户为每个依赖定义一个通道时，求解器需要知道其他通道对该依赖是不可用的。

```toml
[project]
channels = ["conda-forge", "my-channel"]

[dependencies]
packgex = { version = "*", channel = "my-channel" }
```
在 `packagex` 示例中，求解器将理解该包仅在 `my-channel` 中可用，因此不会在 `conda-forge` 中查找它。

以下是排除所有其他通道的逻辑流程图：

``` mermaid
flowchart TD
    A[开始] --> B[给定一个依赖]
    B --> C{通道特定依赖?}
    C -->|是| D[为该包排除所有其他通道]
    C -->|否| E{还有其他依赖?}
    E -->|是| B
    E -->|否| F[结束]
    D --> E
```

## 通道优先级
通道优先级由 `project.channels` 数组中的顺序决定，其中第一个通道具有最高优先级。  
例如：

```toml
[project]
channels = ["conda-forge", "my-channel", "your-channel"]
```
如果在 `conda-forge` 中找到包，求解器将不会在 `my-channel` 和 `your-channel` 中查找，因为这告诉求解器这些通道被排除了。  
如果在 `conda-forge` 中找不到包，求解器将会在 `my-channel` 中查找，如果在那里找到，它会告诉求解器排除 `your-channel` 对该包的作用。  
以下图解了这一逻辑：

``` mermaid
flowchart TD
    A[开始] --> B[给定一个依赖]
    B --> C{遍历通道}
    C --> D{包在该通道中吗?}
    D -->|否| C
    D -->|是| E{"这是该包的第一个通道吗?"}
    E -->|是| F[将包加入候选包]
    E -->|否| G[将包从候选包中排除]
    F --> H{还有其他通道吗?}
    G --> H
    H -->|是| C
    H -->|否| I{还有其他依赖吗?}
    I -->|否| J[结束]
    I -->|是| B
```

此方法确保求解器仅在包在最高优先级的通道中找到时，才将该包添加到候选列表中。  
如果你有 10 个通道，并且包在第 5 个通道中找到，求解器将排除接下来的 5 个通道（如果它们也包含该包）。

## 使用案例：pytorch 和 nvidia 与 conda-forge
一个常见的使用场景是同时使用 `pytorch` 和 `nvidia` 驱动，同时需要 `conda-forge` 通道来获取主要依赖。

```toml
[project]
channels = ["nvidia/label/cuda-11.8.0", "nvidia", "conda-forge", "pytorch"]
platforms = ["linux-64"]

[dependencies]
cuda = {version = "*", channel="nvidia/label/cuda-11.8.0"}
pytorch = {version = "2.0.1.*", channel="pytorch"}
torchvision = {version = "0.15.2.*", channel="pytorch"}
pytorch-cuda = {version = "11.8.*", channel="pytorch"}
python = "3.10.*"
```
这将从 `nvidia/label/cuda-11.8.0` 通道获取尽可能多的包，实际上只有 `cuda` 包。

然后，它会从 `nvidia` 通道获取所有包，其中一些包与 `nvidia` 和 `conda-forge` 通道重叠。  
例如 `cuda-cudart` 包，现在只会从 `nvidia` 通道获取，因为优先级逻辑的作用。

接着，它会从 `conda-forge` 通道获取包，这个通道是依赖的主要通道。

但是，用户只希望从 `pytorch` 通道获取 `pytorch` 包，这就是为什么 `pytorch` 被最后添加，并且依赖被作为通道特定依赖添加的原因。

我们没有在 `conda-forge` 之前定义 `pytorch` 通道，因为我们希望尽可能从 `conda-forge` 获取包，因为 `pytorch` 通道并不总是发布所有包的最佳版本。

例如，它也发布了 `ffmpeg` 包，但仅发布了旧版本，与较新的 `pytorch` 版本不兼容。  
如果我们跳过 `conda-forge` 通道来获取 `ffmpeg`，就会破坏安装过程。

### 强制特定通道优先级
如果你希望强制特定通道的优先级，可以在通道定义中使用 `priority`（整数）键。  
数字越大，优先级越高。  
未指定的优先级默认为 0，但数组中的索引仍然算作优先级，列表中的第一个具有最高优先级。

这种优先级定义主要在 [多环境](../features/multi_environment.md) 使用中重要，因为默认情况下，特性通道会被添加到项目通道之前。

```toml
[project]
name = "test_channel_priority"
platforms = ["linux-64", "osx-64", "win-64", "osx-arm64"]
channels = ["conda-forge"]

[feature.a]
channels = ["nvidia"]

[feature.b]
channels = [ "pytorch", {channel = "nvidia", priority = 1}]

[feature.c]
channels = [ "pytorch", {channel = "nvidia", priority = -1}]

[environments]
a = ["a"]
b = ["b"]
c = ["c"]
```
此示例创建了 4 个环境，`a`、`b`、`c` 和默认环境。  
它们的通道顺序如下所示：

| 环境        | 结果通道顺序                        |
|-------------|-----------------------------------|
| 默认环境    | `conda-forge`                      |
| a 环境      | `nvidia`, `conda-forge`            |
| b 环境      | `nvidia`, `pytorch`, `conda-forge` |
| c 环境      | `pytorch`, `conda-forge`, `nvidia` |

??? note "检查优先级结果使用 `pixi info`"
    使用 `pixi info` 可以检查环境中通道的优先级。  
    ```bash
    pixi info
    Environments
    ------------
           Environment: default
              Features: default
              Channels: conda-forge
    Dependency count: 0
    Target platforms: linux-64

           Environment: a
              Features: a, default
              Channels: nvidia, conda-forge
    Dependency count: 0
    Target platforms: linux-64

           Environment: b
              Features: b, default
              Channels: nvidia, pytorch, conda-forge
    Dependency count: 0
    Target platforms: linux-64

           Environment: c
              Features: c, default
              Channels: pytorch, conda-forge, nvidia
    Dependency count: 0
    Target platforms: linux-64
    ```