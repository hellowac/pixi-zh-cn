# pixi 中的系统要求

**系统要求**定义了在项目的依赖解析过程中所需的最小系统规格。
例如，指定一个特定版本的 `libc` 的 Unix 系统可以确保依赖项与项目的环境兼容。

系统规格与 [虚拟包](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-virtual.html) 密切相关，允许灵活和准确的依赖管理。

## 默认系统要求

以下配置列出了不同操作系统的默认最小系统要求：

=== "Linux"
    ```toml
    # Linux 的默认系统要求
    [system-requirements]
    linux = "4.18"
    libc = { family = "glibc", version = "2.28" }
    ```
=== "Windows"
    目前 Windows 没有定义最小系统要求。如果你的项目需要特定的 Windows 配置，你应根据需要进行定义。
=== "osx-64"
    ```toml
    # macOS 的默认系统要求
    [system-requirements]
    macos = "13.0"
    ```
=== "osx-arm64"
    ```toml
    # macOS ARM64 的默认系统要求
    [system-requirements]
    macos = "13.0"
    ```

## 自定义系统要求

你只需要在项目需要与默认要求不同的系统要求时进行定义。
这通常发生在将环境安装到较旧或较新的操作系统版本时。

### 针对旧系统的调整
如果遇到如下错误：

```bash
× 当前系统有不匹配的虚拟包。该项目要求'__linux'至少为版本'4.18'，但系统的版本为'4.12.14'
```

这表示项目的系统要求高于当前系统的规格。
为了解决这个问题，你可以在项目配置中降低系统要求：

```toml
[system-requirements]
linux = "4.12.14"
```

此调整将通知依赖解析器以适应较旧的系统版本。

### 在 pixi 中使用 CUDA

要在项目中使用 CUDA，你必须在 `system-requirements` 表中指定所需的 CUDA 版本。
这将确保 CUDA 被识别，并在必要时被适当地锁定到锁文件中。

示例配置：

```toml
[system-requirements]
cuda = "12"  # 用你打算使用的具体 CUDA 版本替换 "12"
```

### 设置特定环境的系统要求
这可以在清单文件中为 `feature` 设置：

```toml
[feature.cuda.system-requirements]
cuda = "12"

[environments]
cuda = ["cuda"]
```

### 可用的覆盖选项
在某些情况下，你可能需要覆盖在机器上检测到的系统要求。
这在处理不符合项目默认要求的系统时特别有用。

你可以通过设置以下环境变量来覆盖虚拟包：

- `CONDA_OVERRIDE_CUDA`
  - 描述：设置 CUDA 版本。
  - 使用示例：`CONDA_OVERRIDE_CUDA=11`
- `CONDA_OVERRIDE_GLIBC`
  - 描述：设置 glibc 版本。
  - 使用示例：`CONDA_OVERRIDE_GLIBC=2.28`
- `CONDA_OVERRIDE_OSX`
  - 描述：设置 macOS 版本。
  - 使用示例：`CONDA_OVERRIDE_OSX=13.0`

## 其他资源

有关管理 `虚拟包` 和覆盖系统要求的更多详细信息，请参阅 [Conda 文档](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-virtual.html)。
