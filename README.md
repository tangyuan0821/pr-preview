# PR Preview Action

这是一个GitHub Action，用于为上游仓库的Pull Request生成预览部署，并自动部署到GitHub Pages，并在PR上发布预览链接评论。

## 功能特性

- 自动收集上游仓库的开放PR（非草稿、非已合并）
- 下载PR的代码，构建预览网站
- 在预览页面添加免责声明横幅
- 验证预览页面可访问性
- 部署到GitHub Pages
- 在上游PR上发布或更新预览评论

## 使用方法

### 1. 创建预览仓库

创建一个新的GitHub仓库，用于存放预览文件和GitHub Pages。

### 2. 设置Secrets

在预览仓库中设置以下Secrets：

- `UPSTREAM_READ_PAT`: 具有上游仓库读取权限的Personal Access Token
- `PREVIEW_COMMENT_PAT`: 具有上游仓库PR评论写入权限的Personal Access Token
- `PREVIEW_COMMENT_USERNAME`: 可选，用于评论的机器人用户名（如果不设置，将自动从token解析）

### 3. 创建工作流

在预览仓库中创建`.github/workflows/pr-preview.yml`文件：

```yaml
name: PR Preview

on:
  schedule:
    - cron: "0 * * *"  # 每小时运行一次
  workflow_dispatch:
    inputs:
      pr_number:
        description: '仅处理指定的上游PR编号（可选）'
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
        uses: tangyuan0821/pr-preview@main  # 替换为你的仓库
        with:
          upstream_repo: 'owner/upstream-repo'  # 上游仓库
          preview_root: 'site'  # 预览根目录
          timezone: 'Asia/Shanghai'  # 时区
          gh_token_read: ${{ secrets.UPSTREAM_READ_PAT }}
          gh_token_write: ${{ secrets.PREVIEW_COMMENT_PAT }}
          bot_username: ${{ secrets.PREVIEW_COMMENT_USERNAME }}
          pr_number: ${{ github.event.inputs.pr_number }}
```

### 4. 配置GitHub Pages

在仓库设置中启用GitHub Pages，选择`gh-pages`分支作为源。

## 输入参数

| 参数 | 描述 | 必需 | 默认值 |
|------|------|------|-------|
| `upstream_repo` | 上游仓库，格式为`owner/repo` | 是 | - |
| `preview_root` | 预览文件存放目录 | 否 | `site` |
| `timezone` | 部署时间戳时区 | 否 | `Asia/Shanghai` |
| `gh_token_read` | 读取上游仓库的GitHub Token | 是 | - |
| `gh_token_write` | 写入上游仓库评论的GitHub Token | 是 | - |
| `bot_username` | 评论机器人用户名 | 否 | 自动解析 |
| `pr_number` | 指定处理的PR编号 | 否 | 处理所有开放PR |

## 许可证

本项目采用GPL 3.0许可证。详见[LICENSE](LICENSE)文件。

## 贡献

欢迎提交Issue和Pull Request！

## 致谢

本项目基于原有的PR预览工作流开发，感谢所有贡献者。