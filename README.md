# PRPeek
这是一个GitHub Action，用于为上游仓库的Pull Request生成预览部署，并自动部署到GitHub Pages，并在PR上发布预览链接评论。<br>This is a GitHub Action for generating preview deployments for Pull Requests from upstream repositories, automatically deploying to GitHub Pages, and posting preview link comments on PRs.

## 功能特性 / Features

- 自动收集上游仓库的开放PR（非草稿、非已合并）<br>Automatically collect open PRs from upstream repository (non-draft, non-merged)
- 下载PR的代码，构建预览网站<br>Download PR code and build preview website
- 在预览页面添加免责声明横幅<br>Add disclaimer banner to preview pages
- 验证预览页面可访问性<br>Validate preview page accessibility
- 部署到GitHub Pages<br>Deploy to GitHub Pages
- 在上游PR上发布或更新预览评论<br>Post or update preview comments on upstream PRs

## 使用方法 / Usage

### 1. 创建预览仓库 / Create Preview Repository

创建一个新的GitHub仓库，用于存放预览文件和GitHub Pages。<br>Create a new GitHub repository for storing preview files and GitHub Pages.

### 2. 设置Secrets / Set up Secrets

在预览仓库中设置以下Secrets:<br>Set the following Secrets in the preview repository:

- `UPSTREAM_READ_PAT`: 具有上游仓库读取权限的Personal Access Token / Personal Access Token with read access to upstream repository
- `PREVIEW_COMMENT_PAT`: 具有上游仓库PR评论写入权限的Personal Access Token / Personal Access Token with write access to upstream repository PR comments
- `PREVIEW_COMMENT_USERNAME`: 可选，用于评论的机器人用户名（如果不设置，将自动从token解析） /  Optional, bot username for comments (auto-detected if not set)

### 3. 创建工作流 / Create Workflow

在预览仓库中创建`.github/workflows/pr-preview.yml`文件：<br>Create `.github/workflows/pr-preview.yml` file in the preview repository:

```yaml
name: PR Preview

on:
  schedule:
    - cron: "0 * * *"  # 每小时运行一次 / Run every hour
  workflow_dispatch:
    inputs:
      pr_number:
        description: '仅处理指定的上游PR编号（可选） / Process only specified upstream PR number (optional)'
        required: false
  repository_dispatch:
    types:
      - upstream-pr-opened
      - upstream-pr-synchronize
      - upstream-pr-reopened

permissions:
  contents: write

concurrency:
  group: upstream-pr-preview
  cancel-in-progress: true

jobs:
  preview:
    runs-on: ubuntu-latest
    steps:
      - name: Generate PR Previews
        uses: tangyuan0821/pr-preview@main  # 替换为你的仓库 / Replace with your repository
        with:
          upstream_repo: 'owner/upstream-repo'  # 上游仓库 / Upstream repository
          preview_root: 'site'  # 预览根目录 / Preview root directory
          timezone: 'Asia/Shanghai'  # 时区 / Timezone
          gh_token_read: ${{ secrets.UPSTREAM_READ_PAT }}
          gh_token_write: ${{ secrets.PREVIEW_COMMENT_PAT }}
          bot_username: ${{ secrets.PREVIEW_COMMENT_USERNAME }}
          pr_number: ${{ github.event.inputs.pr_number }}
```

### 4. 配置GitHub Pages / Configure GitHub Pages

在仓库设置中启用GitHub Pages，选择`gh-pages`分支作为源。<br>Enable GitHub Pages in repository settings, select `gh-pages` branch as source.

## 输入参数 / Inputs

| 参数 | 描述 | 必需 | 默认值 |
|------|------|------|-------|
| `upstream_repo` | 上游仓库，格式为`owner/repo` | 是 | - |
| `preview_root` | 预览文件存放目录 | 否 | `site` |
| `timezone` | 部署时间戳时区 | 否 | `Asia/Shanghai` |
| `gh_token_read` | 读取上游仓库的GitHub Token | 是 | - |
| `gh_token_write` | 写入上游仓库评论的GitHub Token | 是 | - |
| `bot_username` | 评论机器人用户名 | 否 | 自动解析 |
| `pr_number` | 指定处理的PR编号 | 否 | 处理所有开放PR |


| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `upstream_repo` | Upstream repository in owner/repo format | Yes | - |
| `preview_root` | Directory to store preview files | No | `site` |
| `timezone` | Timezone for deployment timestamps | No | `Asia/Shanghai` |
| `gh_token_read` | GitHub Token for reading upstream repository | Yes | - |
| `gh_token_write` | GitHub Token for writing comments to upstream repository | Yes | - |
| `bot_username` | Bot username for comments | No | Auto-detected |
| `pr_number` | Specific PR number to process | No | Process all open PRs |

## 许可证 / License

本项目采用GPL 3.0许可证。详见[LICENSE](https://github.com/tangyuan0821/pr-preview/blob/main/LICENSE)文件。<br>This project uses GPL 3.0 license. See [LICENSE](https://github.com/tangyuan0821/pr-preview/blob/main/LICENSE) file.

## 贡献 / Contributing

欢迎提交Issue和Pull Request！
<br>Welcome to submit Issues and Pull Requests!

## 最佳实践 / Best Practices

- [Win12 pr preview](https://github.com/tangyuan0821/win12-pr-preview) - 生产环境实现案例 / Production implementation example
