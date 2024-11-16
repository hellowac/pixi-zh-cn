# 将 pixi 带入生产环境

你可以通过使用 Docker 等工具将 pixi 项目容器化，或使用 [`quantco/pixi-pack`](https://github.com/quantco/pixi-pack) 将其带入生产环境。

!!!tip ""
    [@pavelzw](https://github.com/pavelzw) 来自 [QuantCo](https://quantco.com) 撰写了一篇关于将 pixi 带入生产环境的博客文章。你可以在 [这里](https://tech.quantco.com/blog/pixi-production) 阅读。

## Docker

<!-- 保持与 https://github.com/prefix-dev/pixi-docker/blob/main/README.md 同步 -->

我们提供了一个简单的 Docker 镜像 [`pixi-docker`](https://github.com/prefix-dev/pixi-docker)，该镜像包含了 pixi 可执行文件，并以不同的基础镜像为基础构建。

这些镜像可以在 [ghcr.io/prefix-dev/pixi](https://ghcr.io/prefix-dev/pixi) 上找到。

有不同的标签对应不同的基础镜像：

- `latest` - 基于 `ubuntu:jammy`
- `focal` - 基于 `ubuntu:focal`
- `bullseye` - 基于 `debian:bullseye`
- `jammy-cuda-12.2.2` - 基于 `nvidia/cuda:12.2.2-jammy`
- ... 等等

!!! tip "所有标签"
    查看所有标签，请查看 [构建脚本](https://github.com/prefix-dev/pixi-docker/blob/main/.github/workflows/build.yml)。

### 示例用法

以下示例使用 pixi Docker 镜像作为多阶段构建的基础镜像。  
它还利用了 `pixi shell-hook`，这样就不必在生产容器中安装 pixi。

!!!tip "更多示例"
    查看更多示例，请参考 [pavelzw/pixi-docker-example](https://github.com/pavelzw/pixi-docker-example)。

```Dockerfile
FROM ghcr.io/prefix-dev/pixi:0.36.0 AS build

# 将源代码、pixi.toml 和 pixi.lock 复制到容器中
WORKDIR /app
COPY . .
# 将依赖安装到 `/app/.pixi/envs/prod` 中
# 使用 `--locked` 来确保 lockfile 与 pixi.toml 保持同步
RUN pixi install --locked -e prod
# 创建 shell-hook bash 脚本来激活环境
RUN pixi shell-hook -e prod -s bash > /shell-hook
RUN echo "#!/bin/bash" > /app/entrypoint.sh
RUN cat /shell-hook >> /app/entrypoint.sh
# 扩展 shell-hook 脚本以运行传递给容器的命令
RUN echo 'exec "$@"' >> /app/entrypoint.sh

FROM ubuntu:24.04 AS production
WORKDIR /app
# 只将生产环境复制到生产容器中
# 请注意，“prefix”（路径）需要与构建容器中的保持一致
COPY --from=build /app/.pixi/envs/prod /app/.pixi/envs/prod
COPY --from=build --chmod=0755 /app/entrypoint.sh /app/entrypoint.sh
# 将你的项目代码也复制到容器中
COPY ./my_project /app/my_project

EXPOSE 8000
ENTRYPOINT [ "/app/entrypoint.sh" ]
# 在 pixi 环境中运行你的应用
CMD [ "uvicorn", "my_project:app", "--host", "0.0.0.0" ]
```

## pixi-pack

<!-- 保持与 https://github.com/quantco/pixi-pack/blob/main/README.md 同步 -->

[`pixi-pack`](https://github.com/quantco/pixi-pack) 是一个简单的工具，它将一个 pixi 环境打包成一个压缩档案，可以发送到目标机器。

你可以通过以下命令安装：

```bash
pixi global install pixi-pack
```

或者从 [发布页面](https://github.com/quantco/pixi-pack/releases) 下载我们预构建的二进制文件。

除了全局安装 pixi-pack，你还可以使用 pixi exec 在临时环境中运行 `pixi-pack`：

```bash
pixi exec pixi-pack pack
pixi exec pixi-pack unpack environment.tar
```

![pixi-pack 演示](https://raw.githubusercontent.com/quantco/pixi-pack/refs/heads/main/.github/assets/demo/demo-light.gif#only-light)
![pixi-pack 演示](https://raw.githubusercontent.com/quantco/pixi-pack/refs/heads/main/.github/assets/demo/demo-dark.gif#only-dark)

你可以使用以下命令来打包一个环境：

```bash
pixi-pack pack --manifest-file pixi.toml --environment prod --platform linux-64
```

这将创建一个 `environment.tar` 文件，里面包含了创建该环境所需的所有 conda 包。

```plain
# environment.tar
| pixi-pack.json
| environment.yml
| channel
|    ├── noarch
|    |    ├── tzdata-2024a-h0c530f3_0.conda
|    |    ├── ...
|    |    └── repodata.json
|    └── linux-64
|         ├── ca-certificates-2024.2.2-hbcca054_0.conda
|         ├── ...
|         └── repodata.json
```

### 解包环境

使用 `pixi-pack unpack environment.tar`，你可以在目标系统上解包环境。这将在 `./env` 中创建一个新的 conda 环境，里面包含了 `pixi.toml` 中指定的所有包。它还会创建一个 `activate.sh`（或在 Windows 上是 `activate.bat`）文件，让你无需安装 `conda` 或 `micromamba` 就能激活环境。

### 跨平台包

由于 `pixi-pack` 只是从 conda 仓库下载 `.conda` 和 `.tar.bz2` 文件，你可以轻松地为不同的平台创建包。

```bash
pixi-pack pack --platform win-64
```

!!!note ""
    你只能在与打包时使用的平台相同的平台上解包包。

### 注入额外的包

你可以通过使用 `--inject` 标志将未在 `pixi.lock` 中指定的额外包注入环境中：

```bash
pixi-pack pack --inject local-package-1.0.0-hbefa133_0.conda --manifest-pack pixi.toml
```

如果你自己构建了项目，并希望将构建的包包含在环境中，同时仍想使用项目中的 `pixi.lock`，这特别有用。

### 在没有 pixi-pack 的情况下解包

如果你在目标系统上没有 `pixi-pack`，你仍然可以安装环境，如果你有 `conda` 或 `micromamba` 的话。  
只需解压 `environment.tar`，然后你会在系统上找到一个本地通道，里面包含了所有必要的包。  
除此之外，你还会找到一个包含环境规格的 `environment.yml` 文件。  
然后你可以使用 `conda` 或 `micromamba` 安装环境：

```bash
tar -xvf environment.tar
micromamba create -p ./env --file environment.yml
# 或者
conda env create -p ./env --file environment.yml
```

!!!note ""
    `environment.yml` 和 `repodata.json` 文件仅用于此用例，`pixi-pack unpack` 不会使用它们。
