---
part: pixi
title: 基本用法
description: 使用 pixi 迈出第一步
---

确保已经正确设置了 `pixi`。如果运行 `pixi` 没有显示帮助信息，请参考 [入门指南](index.md) 进行配置。

```shell
pixi
```

### 初始化新项目并进入项目目录

```shell
pixi init pixi-hello-world
cd pixi-hello-world
```

### 添加你需要使用的依赖

```shell
pixi add python
```

### 创建一个名为 `hello_world.py` 的文件，并将以下代码粘贴到文件中。

```py title="hello_world.py"
def hello():
    print("Hello World, to the new revolution in package management.")

if __name__ == "__main__":
    hello()
```

### 在环境中运行代码

```shell
pixi run python hello_world.py
```

### 你也可以将此运行命令放入 **任务** 中。

```shell
pixi task add hello python hello_world.py
```

添加任务后，你可以通过任务名称来运行它。

```shell
pixi run hello
```

### 使用 `shell` 命令激活环境并启动新的 shell。

```shell
pixi shell
python
exit()
```

---

你刚刚学习了 `pixi` 的基本功能：

1. 初始化一个项目
2. 添加依赖
3. 添加任务并执行任务
4. 运行程序

现在可以自由地练习你刚学到的内容，比如添加更多的任务、依赖或代码。

祝编程愉快！

---

### 将 pixi 用作全局安装工具

你可以使用 pixi 来在机器上安装工具。以下是一些有用的示例：

```shell
# 强烈推荐，使用 pixi 时的跨 shell 提示
pixi global install starship

# 想尝试不同的 shell？
pixi global install fish

# 安装其他 prefix.dev 工具
pixi global install rattler-build

# 安装一个多包环境
pixi global install --environment data-science-env --expose python --expose jupyter python jupyter numpy pandas
```

### 在 GitHub Actions 中使用 pixi

你可以在 GitHub Actions 中使用 pixi 来安装依赖并运行命令，支持自动缓存你的环境。

```yml
- uses: prefix-dev/setup-pixi@v0.5.1
- run: pixi run cowpy "Thanks for using pixi"
```

有关更多详细信息，请参阅 [GitHub Actions](./advanced/github_actions.md)。
