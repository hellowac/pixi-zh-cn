---
part: pixi/advanced
title: Advanced tasks
description: Learn how to interact with pixi tasks
---

在构建一个包时，通常不仅仅是运行代码。
格式化、代码检查、编译、测试、基准测试等步骤常常是项目的一部分。
使用 pixi 任务，这些操作应该变得更加容易。

以下是一些快速示例：

```toml title="pixi.toml"
[tasks]
# 作为列表的命令，以便你可以在其中添加文档。
configure = { cmd = [
    "cmake",
    # 使用跨平台的 Ninja 生成器
    "-G",
    "Ninja",
    # 源代码在根目录
    "-S",
    ".",
    # 我们希望在 .build 目录中构建
    "-B",
    ".build",
] }

# 依赖于其他任务
build = { cmd = ["ninja", "-C", ".build"], depends-on = ["configure"] }

# 使用环境变量
run = "python main.py $PIXI_PROJECT_ROOT"
set = "export VAR=hello && echo $VAR"

# 跨平台文件操作
copy = "cp pixi.toml pixi_backup.toml"
clean = "rm pixi_backup.toml"
move = "mv pixi.toml backup.toml"
```

<span id="depends-on"></span>

## 依赖于

就像包可以依赖于其他包一样，我们的任务也可以依赖于其他任务。
这允许通过一个命令运行完整的流水线。

一个明显的例子是 **编译** 在 **运行** 应用程序之前。

查看我们的 [`cpp_sdl` 示例](https://github.com/prefix-dev/pixi/tree/main/examples/cpp-sdl) 了解实际的例子。
在这个包中，我们有一些任务相互依赖，这样可以确保当你运行 `pixi run start` 时，一切都会按预期设置。

```fish
pixi task add configure "cmake -G Ninja -S . -B .build"
pixi task add build "ninja -C .build" --depends-on configure
pixi task add start ".build/bin/sdl_example" --depends-on build
```

这将导致以下行被添加到 `pixi.toml` 文件中：

```toml title="pixi.toml"
[tasks]
# 配置 CMake
configure = "cmake -G Ninja -S . -B .build"
# 构建可执行文件，但确保首先配置了 CMake。
build = { cmd = "ninja -C .build", depends-on = ["configure"] }
# 启动构建的可执行文件
start = { cmd = ".build/bin/sdl_example", depends-on = ["build"] }
```

```shell
pixi run start
```

任务将依次执行：

- 首先执行 `configure`，因为它没有依赖项。
- 然后执行 `build`，因为它只依赖于 `configure`。
- 最后执行 `start`，因为它的所有依赖项都已执行。

如果其中一个命令失败（退出时返回非零代码），它将停止，并且下一个命令将不会被执行。

使用这种逻辑，你还可以创建别名，因为你不必在任务中指定任何命令。

```shell
pixi task add fmt ruff
pixi task add lint pylint
```

```shell
pixi task alias style fmt lint
```

这将导致以下内容添加到 `pixi.toml` 中：

```toml title="pixi.toml"
fmt = "ruff"
lint = "pylint"
style = { depends-on = ["fmt", "lint"] }
```

现在，可以通过一个命令运行两个工具。

```shell
pixi run style
```

<span id="working-directory"></span>

## 工作目录

Pixi 任务支持定义工作目录。

`cwd` 代表当前工作目录（Current Working Directory）。
该目录是相对于 pixi 包根目录（即 `pixi.toml` 文件所在的目录）。

假设一个 pixi 项目结构如下：

```shell
├── pixi.toml
└── scripts
    └── bar.py
```

要添加一个任务来运行 `bar.py` 文件，可以使用：

```shell
pixi task add bar "python bar.py" --cwd scripts
```

这将添加以下行到 [清单文件](../reference/project_configuration.md) 中：

```toml title="pixi.toml"
[tasks]
bar = { cmd = "python bar.py", cwd = "scripts" }
```

<span id="caching"></span>

## 缓存

当你为任务指定了 `inputs` 和/或 `outputs` 时，pixi 会重用该任务的结果。

对于缓存，pixi 会检查以下条件是否成立：

- 环境中的包没有发生变化。
- 选定的输入和输出与上次运行任务时相同。我们会计算所有由 glob 表达式选中的文件的指纹，并与上次运行任务时的指纹进行比较。
- 命令与上次运行任务时相同。

如果满足所有这些条件，pixi 将不会重新运行任务，而是使用现有的结果。

输入和输出可以通过 glob 表达式来指定，这样它们会扩展为所有匹配的文件。

```toml title="pixi.toml"
[tasks]
# 只有当 `main.py` 文件发生变化时，这个任务才会运行。
run = { cmd = "python main.py", inputs = ["main.py"] }

# 这个任务会记住 `curl` 命令的结果，如果文件 `data.csv` 已经存在，则不会再次运行。
download_data = { cmd = "curl -o data.csv https://example.com/data.csv", outputs = ["data.csv"] }

# 只有当 `src` 目录发生变化时，这个任务才会运行，并且会记住 `make` 命令的结果。
build = { cmd = "make", inputs = ["src/*.cpp", "include/*.hpp"], outputs = ["build/app.exe"] }
```

注意：如果你想调试 glob 表达式，可以使用 `--verbose` 标志查看选中的文件。

```shell
# 显示所有被 glob 表达式选中的文件的日志信息
pixi run -v start
```

## 环境变量

你可以为任务设置环境变量。
这些变量被视为该任务的“默认”值，你可以从 shell 中覆盖它们。

```toml title="pixi.toml"
[tasks]
echo = { cmd = "echo $ARGUMENT", env = { ARGUMENT = "hello" } }
```
如果你运行 `pixi run echo`，它会输出 `hello`。
当你在运行任务之前设置了环境变量 `ARGUMENT` 时，它将使用该值。

```shell
ARGUMENT=world pixi run echo
✨ Pixi task (echo in default): echo $ARGUMENT
world
```

这些变量不会在任务之间共享，因此你需要为每个任务单独定义你希望使用的变量。

!!! note "扩展而非覆盖"
    如果你在映射的值和键中使用相同的环境变量，它将覆盖该变量。
    例如，覆盖 `PATH`：

    ```toml title="pixi.toml"
    [tasks]
    echo = { cmd = "echo $PATH", env = { PATH = "/tmp/path:$PATH" } }
    ```

    这将输出 `/tmp/path:/usr/bin:/bin`，而不是原始的 `/usr/bin:/bin`。

## 清洁环境
你可以确保任务的环境是“仅 Pixi”环境。
在这种情况下，pixi 只会包含运行命令所需的最小环境变量，这些变量取决于你的平台。
环境将包含由 conda 环境设置的所有变量，如 `"CONDA_PREFIX"`。
然而，它还会包含一些来自 shell 的默认值，如：
`"DISPLAY"`、`"LC_ALL"`、`"LC_TIME"`、`"LC_NUMERIC"`、`"LC_MEASUREMENT"`、`"SHELL"`、`"USER"`、`"USERNAME"`、`"LOGNAME"`、`"HOME"`、`"HOSTNAME"`、`"TMPDIR"`、`"XPC_SERVICE_NAME"`、`"XPC_FLAGS"`

```toml
[tasks]
clean_command = { cmd = "python run_in_isolated_env.py", clean-env = true}
```

此设置也可以通过命令行中的 `pixi run --clean-env TASK_NAME` 来设置。

!!! warning "`clean-env` 在 Windows 上不支持"
    在 Windows 上创建一个“干净的环境”比较困难，因为 `conda-forge` 没有提供 Windows 编译器，且 Windows 需要大量的基础变量。
    由于边缘情况太多，这使得该功能不值得实现，可能会导致不可用。

## 我们的任务执行器：deno_task_shell

为了支持不同的操作系统（Windows、OSX 和 Linux），pixi 集成了一个可以在所有这些系统上运行的 shell。
这个 shell 是 [`deno_task_shell`](https://deno.land/manual@v1.35.0/tools/task_runner#built-in-commands)。
任务 shell 是一个有限实现的 Bourne-shell 接口。

### 内置命令

除了运行实际的可执行文件如 `./myprogram`、`cmake` 或 `python`，该 shell 还提供了一些内置命令。

- `cp`：复制文件。
- `mv`：移动文件。
- `rm`：删除文件或目录。
  示例：`rm -rf [FILE]...` - 常用来递归删除文件或目录。
- `mkdir`：创建目录。
  示例：`mkdir -p DIRECTORY...` - 常用来创建目录及其所有父目录，且若目录已存在则不报错。
- `pwd`：打印当前/工作目录的名称。
- `sleep`：延迟指定的时间。
  示例：`sleep 1` 睡眠 1 秒，`sleep 0.5` 睡眠半秒，或 `sleep 1m` 睡眠 1 分钟。
- `echo`：显示一行文本。
- `cat`：连接文件并输出到标准输出。当没有提供参数时，它读取并输出标准输入。
- `exit`：导致 shell 退出。
- `unset`：取消环境变量。
- `xargs`：从标准输入构建参数并执行命令。

### 语法

- **布尔列表：** 使用 `&&` 或 `||` 来分隔两个命令。
  - `&&`：如果 `&&` 前的命令成功，则继续执行下一个命令。
  - `||`：如果 `||` 前的命令失败，则继续执行下一个命令。
- **顺序列表：** 使用 `;` 来运行两个命令，而不检查第一个命令是否成功或失败。
- **环境变量：**
  - 设置环境变量：`export ENV_VAR=value`
  - 使用环境变量：`$ENV_VAR`
  - 取消设置环境变量：`unset ENV_VAR`
- **Shell 变量：** Shell 变量类似于环境变量，但不会被导出到派生的命令中。
  - 设置变量：`VAR=value`
  - 使用变量：`VAR=value && echo $VAR`
- **管道：** 将一个命令的标准输出传递给后续命令的标准输入。
  - `|`：`echo Hello | python receiving_app.py`
  - `|&`：使用这个符号可以将标准错误输出也作为输入。
- **命令替换：** 使用 `$()` 将一个命令的输出作为另一个命令的输入。
  - `python main.py $(git rev-parse HEAD)`
- **否定退出码：** 在任何命令前加上 `!` 将否定其退出码，1变为0，反之亦然。
- **重定向：** 使用 `>` 将标准输出重定向到文件。
  - `echo hello > file.txt` 会将 `hello` 放入 `file.txt` 并覆盖现有内容。
  - `python main.py 2> file.txt` 会将标准错误输出放入 `file.txt`。
  - `python main.py &> file.txt` 会将标准错误和标准输出都放入 `file.txt`。
  - `echo hello >> file.txt` 会将 `hello` 追加到现有的 `file.txt` 文件中。
- **Glob 扩展：** 使用 `*` 来扩展所有选项。
  - `echo *.py` 会显示所有以 `.py` 结尾的文件名。
  - `echo **/*.py` 会显示当前目录及所有子目录中以 `.py` 结尾的文件名。
  - `echo data[0-9].csv` 会显示所有在 `data` 后面跟着一个数字且以 `.csv` 结尾的文件名。

更多信息请参阅 [`deno_task_shell` 文档](https://deno.land/manual@v1.35.0/tools/task_runner#task-runner)。
