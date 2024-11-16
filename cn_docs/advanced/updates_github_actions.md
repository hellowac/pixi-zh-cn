---
part: pixi/advanced
title: Update lockfiles with GitHub Actions
description: 了解如何使用 GitHub Actions 自动更新您的 pixi 锁文件。
---

你可以将 GitHub Actions 与 [pavelzw/pixi-diff-to-markdown](https://github.com/pavelzw/pixi-diff-to-markdown) 结合使用，  
自动更新你的 lockfile，类似于 dependabot 或其他生态系统中的 renovate。

![更新 lockfiles](../assets/update-lockfile-light.png#only-light)  
![更新 lockfiles](../assets/update-lockfile-dark.png#only-dark)

!!!note "Dependabot/Renovate 对 pixi 的支持"
    你可以在 [dependabot/dependabot-core #2227](https://github.com/dependabot/dependabot-core/issues/2227#issuecomment-1709069470) 中跟踪对 pixi 的原生 Dependabot 支持，  
    并在 [renovatebot/renovate #2213](https://github.com/renovatebot/renovate/issues/2213) 中跟踪 Renovate 的支持。

## 如何使用

要开始使用，请在你的仓库中创建一个新的 GitHub Actions 工作流文件。

```yaml title=".github/workflows/update-lockfiles.yml"
name: Update lockfiles

permissions: # (1)!
  contents: write
  pull-requests: write

on:
  workflow_dispatch:
  schedule:
    - cron: 0 5 1 * * # (2)!

jobs:
  pixi-update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up pixi
        uses: prefix-dev/setup-pixi@v0.8.1
        with:
          run-install: false
      - name: Update lockfiles
        run: |
          set -o pipefail
          pixi update --json | pixi exec pixi-diff-to-markdown >> diff.md
      - name: Create pull request
        uses: peter-evans/create-pull-request@v7
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          commit-message: Update pixi lockfile
          title: Update pixi lockfile
          body-path: diff.md
          branch: update-pixi
          base: main
          labels: pixi
          delete-branch: true
          add-paths: pixi.lock
```

1. `peter-evans/create-pull-request` 需要此权限
2. 该工作流将在每月的第一天的 05:00 运行

为了使该工作流生效，你需要在仓库设置中将 "Allow GitHub Actions to create and approve pull requests" 设置为 true（在 "Actions" -> "General" 中）。

!!! tip

    如果你没有任何 `pypi-dependencies`，可以使用 `pixi update --json --no-install` 来加速 diff 生成。

![允许 GitHub Actions PR](../assets/allow-github-actions-prs-light.png#only-light)  
![允许 GitHub Actions PR](../assets/allow-github-actions-prs-dark.png#only-dark)

## 在自动化 PR 中触发 CI

为了防止意外的递归 GitHub Workflow 运行，GitHub 决定在使用默认的 `GITHUB_TOKEN` 时不在自动化 PR 上触发任何工作流。  
有几种方法可以绕过这个限制，你可以在 `peter-evans/create-pull-request` 中找到相关的优秀文档，详见 [这里](https://github.com/peter-evans/create-pull-request/blob/main/docs/concepts-guidelines.md#triggering-further-workflow-runs)。

## 自定义摘要

你可以通过使用 `pixi-diff-to-markdown` 的命令行参数，或在 `pixi.toml` 文件中的 `[tool.pixi-diff-to-markdown]` 部分指定配置，来定制摘要。  
有关更多信息，请参见 [pixi-diff-to-markdown 文档](https://github.com/pavelzw/pixi-diff-to-markdown) 或运行 `pixi-diff-to-markdown --help`。

## 使用可重用工作流

如果你希望在 GitHub 组织中的多个仓库中使用相同的工作流，可以创建一个可重用的工作流。  
有关更多信息，请参见 [GitHub 文档](https://docs.github.com/en/actions/using-workflows/reusing-workflows)。
