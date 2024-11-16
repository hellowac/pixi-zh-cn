### 教程：使用 `pixi` 开发 Rust 包

在本教程中，我们将向您展示如何使用 `pixi` 开发一个 Rust 包。
教程的内容将按步骤进行，如果缺少某些步骤可能会导致错误。

本教程的目标受众是熟悉 Rust 和 `cargo` 的开发人员，并且对尝试在开发工作流中使用 `pixi` 感兴趣。
该方法的好处是在 Rust 工作流中，您可以锁定项目可能使用的 Rust 和 C/System 依赖项。例如，tokio 用户几乎肯定会使用 `openssl`。

!!! note ""
    如果您是第一次接触 `pixi`，可以查看 [基础用法](../basic_usage.md) 指南。
    它将在 3 分钟内教您如何使用 `pixi` 项目。

## 前提条件

- 您需要安装 `pixi`。如果尚未安装，请按照 [安装指南](../index.md) 中的说明进行操作。
  本教程的关键是向您展示您只需要 `pixi`！

## 创建一个 `pixi` 项目

```shell
pixi init my_rust_project
cd my_rust_project
```

这将创建如下的目录结构：

```shell
my_rust_project
├── .gitattributes
├── .gitignore
└── pixi.toml
```

`pixi.toml` 文件是您项目的清单文件。它应该看起来像这样：

```toml title="pixi.toml"
[project]
name = "my_rust_project"
version = "0.1.0"
description = "Add a short description here"
authors = ["User Name <user.name@email.url>"]
channels = ["conda-forge"]
platforms = ["linux-64"] # (1)!

[tasks]

[dependencies]
```

1. `platforms` 默认为您的系统平台。您可以将其更改为任何您想要支持的平台，例如 `["linux-64", "osx-64", "osx-arm64", "win-64"]`。

### 教程：使用 `pixi` 开发 Rust 包

在本教程中，我们将向您展示如何使用 `pixi` 开发一个 Rust 包。
教程内容按步骤执行，缺少步骤可能会导致错误。

本教程面向熟悉 Rust 和 `cargo` 的开发人员，旨在展示如何使用 `pixi` 来管理和构建 Rust 项目。通过这种方法，您可以锁定项目中使用的 Rust 和 C/System 依赖项，例如，使用 tokio 的用户通常会使用 `openssl`。

## 前提条件

- 您需要安装 `pixi`。如果还没有安装，请按照 [安装指南](../index.md) 中的说明进行操作。
  本教程的重点是向您展示您只需要 `pixi`！

## 创建一个 `pixi` 项目

```shell
pixi init my_rust_project
cd my_rust_project
```

这将创建如下目录结构：

```shell
my_rust_project
├── .gitattributes
├── .gitignore
└── pixi.toml
```

`pixi.toml` 文件是您的项目清单文件。它应如下所示：

```toml title="pixi.toml"
[project]
name = "my_rust_project"
version = "0.1.0"
description = "Add a short description here"
authors = ["User Name <user.name@email.url>"]
channels = ["conda-forge"]
platforms = ["linux-64"] # (1)!

[tasks]

[dependencies]
```

1. `platforms` 默认为您的系统平台。您可以将其更改为您希望支持的任何平台，例如 `["linux-64", "osx-64", "osx-arm64", "win-64"]`。

## 添加 Rust 依赖项

为了使用 `pixi` 项目，您无需在系统上安装任何依赖项，所有必需的依赖项都应该通过 `pixi` 添加，这样其他用户也可以在没有问题的情况下使用您的项目。

```shell
pixi add rust
```

这将会把 `rust` 工具链和 `cargo` 包添加到 `pixi.toml` 文件中的 `[dependencies]` 部分。

## 添加 `cargo` 项目

现在您已经安装了 Rust，可以在您的 `pixi` 项目中创建一个 `cargo` 项目：

```shell
pixi run cargo init
```

`pixi run` 是 `pixi` 运行命令的一种方式，它会确保在运行命令之前正确设置环境。
它会运行自己的跨平台 shell。如果您想了解更多信息，请查看 [`tasks` 文档](../features/advanced_tasks.md)。
您还可以通过运行 `pixi shell` 来激活环境，这样就不需要每次都使用 `pixi run ...`。

现在我们可以使用 `pixi` 来构建 `cargo` 项目：

```shell
pixi run cargo build
```

为了简化构建过程，您可以使用以下命令将 `build` 任务添加到 `pixi.toml` 文件中：

```shell
pixi task add build "cargo build"
```

这将在 `pixi.toml` 文件中创建如下字段：

```toml title="pixi.toml"
[tasks]
build = "cargo build"
```

现在您可以通过以下命令来构建您的项目：

```shell
pixi run build
```

您也可以使用以下命令来运行您的项目：

```shell
pixi run cargo run
```

为了简化这个过程，您可以再次使用任务：

```shell
pixi task add start "cargo run"
```

这将使您能够通过以下命令运行项目：

```shell
pixi run start
Hello, world!
```

恭喜您，已经成功在机器上运行了 Rust 项目！

## 下一步，为什么 `rustup` 不足够？

`cargo` 是一个基于源代码的包管理器，而不是二进制包管理器。
这意味着您需要在系统上安装 Rust 编译器才能使用它，并且可能还需要安装 `cargo` 包管理器未包含的其他依赖项。
例如，您可能需要在系统上安装 `openssl` 或 `libssl-dev`，才能构建某些包。
这在 `pixi` 中也是如此，但 `pixi` 会在项目文件夹中安装这些依赖项，这样您就不必担心这些问题。

添加以下依赖项到您的 `cargo` 项目：

```shell
pixi run cargo add git2
```

如果您的系统没有预先配置好构建 C 语言项目并且没有安装 `libssl-dev` 包，您将无法构建该项目：

```shell
pixi run build
...
Could not find directory of OpenSSL installation, and this `-sys` crate cannot
proceed without this knowledge. If OpenSSL is installed and this crate had
trouble finding it, you can set the `OPENSSL_DIR` environment variable for the
compilation process.

Make sure you also have the development packages of openssl installed.
For example, `libssl-dev` on Ubuntu or `openssl-devel` on Fedora.

If you're in a situation where you think the directory *should* be found
automatically, please open a bug at https://github.com/sfackler/rust-openssl
and include information about your system as well as this message.

$HOST = x86_64-unknown-linux-gnu
$TARGET = x86_64-unknown-linux-gnu
openssl-sys = 0.9.102
```

您可以通过添加构建 `git2` 所需的依赖项来修复此问题：

```shell
pixi add openssl pkg-config compilers
```

现在，您应该能够再次构建项目：

```shell
pixi run build
...
   Compiling git2 v0.18.3
   Compiling my_rust_project v0.1.0 (/my_rust_project)
    Finished dev [unoptimized + debuginfo] target(s) in 7.44s
     Running `target/debug/my_rust_project`
```

## 扩展：添加更多任务

您可以将更多任务添加到 `pixi.toml` 文件中，以简化您的工作流程。

例如，您可以添加一个 `test` 任务来运行测试：

```shell
pixi task add test "cargo test"
```

您还可以添加一个 `clean` 任务来清理项目：

```shell
pixi task add clean "cargo clean"
```

您可以为项目添加格式化任务：

```shell
pixi task add fmt "cargo fmt"
```

您可以通过 `depends-on` 字段扩展这些任务，来运行多个命令：

```shell
pixi task add lint "cargo clippy" --depends-on fmt
```

## 结论

在本教程中，我们向您展示了如何使用 `pixi` 创建 Rust 项目。
我们还展示了如何通过 `pixi` **添加依赖项**，以确保您的项目在任何安装了 `pixi` 的系统上都是 **可重现的**。

## 展示您的作品！

项目完成了吗？
我们很期待看到您所创造的作品！
通过社交媒体使用标签 #pixi 并标记我们 @prefix_dev，分享您的作品吧！
让我们一起激励社区！
