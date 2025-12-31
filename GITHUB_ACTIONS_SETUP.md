# GitHub Actions 自动构建 Docker 镜像 - 完整设置指南

本指南将帮助你在 **ivanberry/audiobookshelf** 仓库配置 GitHub Actions，实现自动构建包含自定义功能（YouTube 下载等）的 Docker 镜像。

---

## 🎯 目标

- ✅ 推送到 `master` 分支时自动构建 Docker 镜像
- ✅ 支持手动触发构建
- ✅ 自动推送到 GitHub Container Registry (GHCR)
- ✅ 支持多架构 (amd64, arm64)
- ✅ 镜像名称: `ghcr.io/ivanberry/audiobookshelf`

---

## 📝 步骤 1: 启用 GitHub Actions

### 1.1 检查 Actions 是否启用

1. 访问你的 GitHub 仓库: `https://github.com/ivanberry/audiobookshelf`
2. 点击 **Settings** 标签
3. 在左侧菜单找到 **Actions** → **General**
4. 确保 **Actions permissions** 设置为:
   - ✅ **Allow all actions and reusable workflows** (推荐)
   - 或 **Allow select actions and reusable workflows** (需要添加信任的 actions)

### 1.2 配置 Workflow 权限

在同一页面向下滚动，找到 **Workflow permissions** 部分：

1. 选择: ✅ **Read and write permissions**
2. 勾选: ✅ **Allow GitHub Actions to create and approve pull requests**
3. 点击 **Save** 保存

> 💡 **为什么需要这个？** 这允许 GitHub Actions 推送镜像到 GitHub Container Registry。

---

## 📝 步骤 2: 设置 Package 权限（重要！）

GitHub Container Registry 需要特殊权限才能推送镜像。

### 2.1 创建 Personal Access Token (可选，推荐方法)

如果你想使用 PAT 而不是 `GITHUB_TOKEN`（更可控）：

1. 访问 GitHub Settings: `https://github.com/settings/tokens`
2. 点击 **Developer settings** → **Personal access tokens** → **Tokens (classic)**
3. 点击 **Generate new token** → **Generate new token (classic)**
4. 配置如下:
   - **Note**: `GHCR Push Token for audiobookshelf`
   - **Expiration**: 选择有效期（建议 1 年）
   - **Scopes**: 勾选以下权限
     - ✅ `write:packages` - 上传和发布包
     - ✅ `read:packages` - 下载包
     - ✅ `delete:packages` - 删除包版本（可选）
5. 点击 **Generate token**
6. **复制 token** (只会显示一次！)

### 2.2 添加 Secret 到仓库

1. 访问你的仓库: `https://github.com/ivanberry/audiobookshelf`
2. 点击 **Settings** → **Secrets and variables** → **Actions**
3. 点击 **New repository secret**
4. 添加如下 secret:
   - **Name**: `GHCR_TOKEN`
   - **Value**: 粘贴刚才复制的 token
5. 点击 **Add secret**

> 💡 **注意:** 如果不设置这个 secret，workflow 会使用自动提供的 `GITHUB_TOKEN`，在大多数情况下这也是可以的。

---

## 📝 步骤 3: 合并当前分支到 master

你的更改在 `claude/add-youtube-download-JYy2I` 分支，需要合并到 `master` 才能触发自动构建。

### 方法 1: 通过 Pull Request（推荐）

```bash
# 确保你在正确的分支
git checkout claude/add-youtube-download-JYy2I

# 推送到远程（已完成）
# git push origin claude/add-youtube-download-JYy2I

# 在 GitHub 上创建 PR
# 访问: https://github.com/ivanberry/audiobookshelf/pull/new/claude/add-youtube-download-JYy2I

# 然后合并 PR 到 master
```

**在 GitHub 网页操作：**
1. 访问: https://github.com/ivanberry/audiobookshelf/pulls
2. 点击 **New pull request**
3. Base: `master` ← Compare: `claude/add-youtube-download-JYy2I`
4. 点击 **Create pull request**
5. 填写标题: `Add YouTube download feature with yt-dlp`
6. 点击 **Create pull request**
7. 审查后点击 **Merge pull request**

### 方法 2: 直接合并（快速）

```bash
# 切换到 master 分支
git checkout master

# 拉取最新代码
git pull origin master

# 合并功能分支
git merge claude/add-youtube-download-JYy2I

# 推送到远程
git push origin master
```

> ⚡ **自动触发:** 推送到 master 后，GitHub Actions 会自动开始构建！

---

## 📝 步骤 4: 手动触发构建（可选）

如果你想立即测试构建，不等推送到 master：

1. 访问: https://github.com/ivanberry/audiobookshelf/actions
2. 在左侧选择 **Build and Push Docker Image**
3. 点击右上角 **Run workflow** 下拉按钮
4. 选择分支: `claude/add-youtube-download-JYy2I`（或 `master`）
5. 输入 Docker Tag (可选): `latest` 或 `test`
6. 点击绿色 **Run workflow** 按钮

---

## 📝 步骤 5: 监控构建过程

### 5.1 查看构建日志

1. 访问: https://github.com/ivanberry/audiobookshelf/actions
2. 点击最新的 **Build and Push Docker Image** workflow
3. 点击 **build** job 查看详细日志

### 5.2 预期输出

构建成功后，你会看到：

```
✅ Check out
✅ Docker meta
✅ Setup QEMU
✅ Set up Docker Buildx
✅ Cache Docker layers
✅ Login to GitHub Container Registry
✅ Build image
   - Building for linux/amd64, linux/arm64
   - Pushing to ghcr.io/ivanberry/audiobookshelf:latest
✅ Move cache
```

---

## 📝 步骤 6: 验证镜像已推送

### 6.1 在 GitHub Packages 查看

1. 访问你的仓库首页
2. 点击右侧 **Packages** 链接
3. 应该看到 `audiobookshelf` 包

或直接访问: https://github.com/ivanberry?tab=packages

### 6.2 设置 Package 可见性

如果你想公开镜像（让任何人都能拉取）：

1. 访问 Package 页面
2. 点击 **Package settings**
3. 向下滚动到 **Danger Zone**
4. 点击 **Change visibility** → 选择 **Public**
5. 确认更改

---

## 📝 步骤 7: 拉取和使用镜像

### 7.1 公开镜像（无需认证）

如果你设置为 public：

```bash
# 拉取镜像
docker pull ghcr.io/ivanberry/audiobookshelf:latest

# 运行容器
docker run -d \
  --name audiobookshelf \
  -p 13378:80 \
  -v $(pwd)/audiobooks:/audiobooks \
  -v $(pwd)/podcasts:/podcasts \
  -v $(pwd)/metadata:/metadata \
  -v $(pwd)/config:/config \
  -e TZ=Asia/Shanghai \
  ghcr.io/ivanberry/audiobookshelf:latest
```

### 7.2 私有镜像（需要认证）

如果镜像是私有的：

```bash
# 创建 GitHub PAT (如果还没有)
# https://github.com/settings/tokens
# 权限: read:packages

# 登录到 GHCR
echo YOUR_GITHUB_TOKEN | docker login ghcr.io -u ivanberry --password-stdin

# 拉取镜像
docker pull ghcr.io/ivanberry/audiobookshelf:latest

# 运行容器（同上）
```

---

## 🔄 自动触发规则

GitHub Actions 会在以下情况自动构建：

### ✅ 推送到 master 分支
```bash
git push origin master
```

### ✅ 创建版本标签
```bash
git tag v1.0.0
git push origin v1.0.0
```
这会构建并推送标签为 `v1.0.0` 和 `latest` 的镜像。

### ✅ 修改关键文件
只有当以下文件变更时才会触发构建：
- `client/**` - 前端代码
- `server/**` - 后端代码
- `index.js` - 主入口
- `package.json` - 依赖
- `Dockerfile` - Docker 配置
- `.github/workflows/docker-build.yml` - 工作流配置

### ⏸️ 跳过构建
如果你想推送代码但不触发构建：

```bash
git commit -m "Update README [skip ci]"
git push origin master
```

---

## 🐛 故障排查

### 问题 1: Actions 没有运行

**检查：**
1. GitHub Actions 是否启用？ (Settings → Actions → General)
2. 是否推送到正确的分支？ (master)
3. 文件路径是否匹配？ (查看 workflow paths 配置)

### 问题 2: 权限被拒绝

**错误：** `denied: permission_denied: write_package`

**解决：**
1. 检查 Workflow permissions: Settings → Actions → General → Workflow permissions
2. 确保选择了 "Read and write permissions"
3. 或者创建 GHCR_TOKEN secret

### 问题 3: 构建失败

**查看日志：**
1. Actions → 点击失败的 workflow
2. 点击 build job
3. 查看具体错误信息

**常见原因：**
- Dockerfile 语法错误
- 依赖安装失败
- 平台不支持

### 问题 4: 无法推送镜像

**错误：** `unauthorized: unauthenticated`

**解决：**
1. 确保 GITHUB_TOKEN 有正确权限
2. 或使用 GHCR_TOKEN secret
3. 检查 package visibility 设置

---

## 📊 完整的 Workflow 文件

你的最终 `.github/workflows/docker-build.yml` 应该是这样的：

```yaml
---
name: Build and Push Docker Image

on:
  workflow_dispatch:
    inputs:
      tags:
        description: 'Docker Tag'
        required: true
        default: 'latest'
  push:
    branches: [main, master]
    tags:
      - 'v*.*.*'
    paths:
      - client/**
      - server/**
      - index.js
      - package.json
      - Dockerfile
      - .github/workflows/docker-build.yml

jobs:
  build:
    if: ${{ !contains(github.event.head_commit.message, 'skip ci') }}
    runs-on: ubuntu-24.04

    permissions:
      contents: read
      packages: write

    steps:
      - name: Check out
        uses: actions/checkout@v4

      - name: Docker meta
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: |
            ghcr.io/${{ github.repository_owner }}/audiobookshelf
          tags: |
            type=edge,branch=master
            type=semver,pattern={{version}}
            type=ref,event=branch
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Setup QEMU
        uses: docker/setup-qemu-action@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Cache Docker layers
        uses: actions/cache@v4
        with:
          path: /tmp/.buildx-cache
          key: ${{ runner.os }}-buildx-${{ github.sha }}
          restore-keys: |
            ${{ runner.os }}-buildx-

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build image
        uses: docker/build-push-action@v6
        with:
          tags: ${{ github.event.inputs.tags || steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          context: .
          platforms: linux/amd64,linux/arm64
          push: true
          cache-from: type=local,src=/tmp/.buildx-cache
          cache-to: type=local,dest=/tmp/.buildx-cache-new,mode=max

      - name: Move cache
        run: |
          rm -rf /tmp/.buildx-cache
          mv /tmp/.buildx-cache-new /tmp/.buildx-cache
```

---

## ✅ 快速检查清单

在推送到 master 之前，确保：

- [ ] GitHub Actions 已启用
- [ ] Workflow permissions 设置为 "Read and write"
- [ ] 已修改 docker-build.yml (移除仓库限制)
- [ ] Dockerfile 包含 yt-dlp 安装
- [ ] 所有更改已提交
- [ ] 代码已推送到功能分支

准备好后：

- [ ] 创建 PR 或直接合并到 master
- [ ] 监控 Actions 构建过程
- [ ] 验证镜像已推送到 GHCR
- [ ] 测试拉取和运行镜像

---

## 🎉 完成！

设置完成后，每次推送到 master 分支，GitHub Actions 都会自动构建包含 YouTube 下载功能的 Docker 镜像！

**镜像地址：** `ghcr.io/ivanberry/audiobookshelf:latest`

---

## 💡 额外提示

### 定时构建（可选）

如果你想定期重新构建镜像（例如获取 yt-dlp 更新）：

在 `docker-build.yml` 的 `on:` 部分添加：

```yaml
on:
  schedule:
    - cron: '0 0 * * 0'  # 每周日午夜构建
  # ... 其他触发条件
```

### 构建通知

想在构建完成时收到通知？在 GitHub 仓库：

1. Settings → Notifications
2. 勾选 "Actions" 通知

或使用 Slack/Discord webhook 在 workflow 中发送通知。

---

**需要帮助？** 查看 [GitHub Actions 文档](https://docs.github.com/en/actions) 或在仓库创建 Issue。
