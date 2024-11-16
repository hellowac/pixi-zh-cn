# 在 devcontainer 中使用 pixi

[VSCode Devcontainers](https://code.visualstudio.com/docs/devcontainers/containers) 是一种流行的工具，用于在具有一致环境的项目中进行开发。它们也用于 [GitHub Codespaces](https://github.com/features/codespaces)，这是一个在不需要在本地机器上安装任何东西的情况下开发项目的绝佳方式。

要在 devcontainer 中使用 pixi，请按照以下步骤操作：

1. 在项目的根目录下创建一个新的 `.devcontainer` 目录。
2. 然后，在 `.devcontainer` 目录中创建以下两个文件：

```dockerfile title=".devcontainer/Dockerfile"
FROM mcr.microsoft.com/devcontainers/base:jammy

ARG PIXI_VERSION=v0.36.0

RUN curl -L -o /usr/local/bin/pixi -fsSL --compressed "https://github.com/prefix-dev/pixi/releases/download/${PIXI_VERSION}/pixi-$(uname -m)-unknown-linux-musl" \
    && chmod +x /usr/local/bin/pixi \
    && pixi info

# 设置一些用户和工作目录设置，使其与 vscode 良好配合
USER vscode
WORKDIR /home/vscode

RUN echo 'eval "$(pixi completion -s bash)"' >> /home/vscode/.bashrc
```

```json title=".devcontainer/devcontainer.json"
{
    "name": "my-project",
    "build": {
      "dockerfile": "Dockerfile",
      "context": ".."
    },
    "customizations": {
      "vscode": {
        "settings": {},
        "extensions": ["ms-python.python", "charliermarsh.ruff", "GitHub.copilot"]
      }
    },
    "features": {
      "ghcr.io/devcontainers/features/docker-in-docker:2": {}
    },
    "mounts": ["source=${localWorkspaceFolderBasename}-pixi,target=${containerWorkspaceFolder}/.pixi,type=volume"],
    "postCreateCommand": "sudo chown vscode .pixi && pixi install"
}
```

### 提示 "将 `.pixi` 放入挂载中"

在上面的示例中，我们将 `.pixi` 目录挂载到一个卷中。这是必要的，因为 `.pixi` 目录不应位于大小写不敏感的文件系统上（如 macOS、Windows 的默认文件系统），而应位于其自己的卷中。某些 conda 包（例如 [ncurses-feedstock#73](https://github.com/conda-forge/ncurses-feedstock/issues/73)）包含仅在大小写上有所不同的文件，这会导致在大小写不敏感的文件系统上出现错误。

## 秘密管理

如果您想要在私有 conda 通道中进行身份验证，可以将秘密信息添加到您的 devcontainer 中。

```json title=".devcontainer/devcontainer.json"
{
    "build": "Dockerfile",
    "context": "..",
    "options": [
        "--secret",
        "id=prefix_dev_token,env=PREFIX_DEV_TOKEN"
    ]
}
```

```dockerfile title=".devcontainer/Dockerfile"
# ...
RUN --mount=type=secret,id=prefix_dev_token,uid=1000 \
    test -s /run/secrets/prefix_dev_token \
    && pixi auth login --token "$(cat /run/secrets/prefix_dev_token)" https://repo.prefix.dev
```

这些秘密需要在本地启动 devcontainer 时作为环境变量提供，或者在您的 [GitHub Codespaces 设置](https://github.com/settings/codespaces) 中的 `Secrets` 下提供。
