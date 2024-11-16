# 在 RStudio 中开发 R 脚本

您可以使用 `pixi` 来管理您的 R 依赖项。conda-forge 通道包含了许多 R 包，可以通过 `pixi` 安装。

## 安装 R 包

R 包通常在 conda-forge 通道中以 `r-` 为前缀。要安装 R 包，您可以使用以下命令：

```bash
pixi add r-<package-name>
# 例如
pixi add r-ggplot2
```

## 在 RStudio 中使用 R 包

要在 RStudio 中使用通过 `pixi` 安装的 R 包，您需要从已激活的环境中运行 RStudio。这可以通过从 `pixi shell` 中运行 RStudio 或在 `pixi.toml` 文件中的任务中运行 RStudio 来实现。

## 完整示例

完整示例请参见：[RStudio 示例](https://github.com/prefix-dev/pixi/tree/main/examples/r)。
以下是一个设置 RStudio 任务的 `pixi.toml` 文件示例：

```toml
[project]
name = "r"
channels = ["conda-forge"]
platforms = ["linux-64", "osx-64", "osx-arm64"]

[target.linux.tasks]
rstudio = "rstudio"

[target.osx.tasks]
rstudio = "open -a rstudio"
# 或者使用完整路径：
# rstudio = "/Applications/RStudio.app/Contents/MacOS/RStudio"

[dependencies]
r = ">=4.3,<5"
r-ggplot2 = ">=3.5.0,<3.6"
```

一旦 RStudio 加载完成，您可以执行以下使用 `ggplot2` 包的 R 代码：

```R
# 加载 ggplot2 包
library(ggplot2)

# 加载内置的 'mtcars' 数据集
data <- mtcars

# 创建 'mpg' 与 'wt' 的散点图
ggplot(data, aes(x = wt, y = mpg)) +
  geom_point() +
  labs(x = "Weight (1000 lbs)", y = "Miles per Gallon") +
  ggtitle("Fuel Efficiency vs. Weight")
```

!!! Note
    该示例假设您已经全局安装了 RStudio。
    我们正在努力更新 RStudio 以及 Windows 上的 R 解释器构建，以确保与 `pixi` 具有最大的兼容性。
