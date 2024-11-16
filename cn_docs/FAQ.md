---
part: pixi
title: Frequently asked questions
description: 我们最常遇到哪些问题？
---

## `conda`、`mamba`、`poetry`、`pip` 之间的区别

| 工具    | 安装 Python | 构建包 | 执行预定义任务 | 内建锁文件 | 快速 | 无需 Python 即可使用 |
|---------|-------------|--------|----------------|-----------|------|---------------------|
| Conda   | ✅          | ❌     | ❌             | ❌        | ❌   | ❌                   |
| Mamba   | ✅          | ❌     | ❌             | ❌        | ✅   | [✅](https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html) |
| Pip     | ❌          | ✅     | ❌             | ❌        | ❌   | ❌                   |
| Pixi    | ✅          | 🚧     | ✅             | ✅        | ✅   | ✅                   |
| Poetry  | ❌          | ✅     | ❌             | ✅        | ❌   | ❌                   |

## 为什么叫 `pixi`
从名字 `prefix` 开始，我们进行了多次迭代，直到找到了一个既容易发音、拼写又容易记住的名字。
而且目前还没有任何 CLI 工具使用这个名字。
不同于 `px`、`pex`、`pax` 等。
我们认为这个名字能激发好奇心和趣味，如果你不同意，抱歉，但你可以随时将它别名为你喜欢的名字。

=== "Linux & macOS"
    ```shell
    alias not_pixi="pixi"
    ```
=== "Windows"
    PowerShell:
    ```powershell
    New-Alias -Name not_pixi -Value pixi
    ```

## `pixi build` 在哪里
**简短回答**: 它马上就来，我们保证！

`pixi build` 将是一个子命令，可以从 pixi 项目生成一个 conda 包。
这需要一个强大的构建工具，我们正在使用 [`rattler-build`](https://github.com/prefix-dev/rattler-build) 创建它，并将作为库在 pixi 中使用。
