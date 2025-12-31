# Docker 构建指南 - 自定义功能版本

本指南介绍如何构建和运行包含自定义功能的 Audiobookshelf Docker 镜像。

## 📋 当前功能

✅ 支持从 YouTube 下载音频（使用 yt-dlp）
✅ 支持 YouTube 播放列表批量下载
✅ 自动提取元数据和封面
✅ MP3 格式输出，多种音质选择
✅ 管理员权限控制
✅ 扩展性强，方便添加更多功能

---

## 🚀 方案 1: 本地构建（快速测试）

### 1. 构建镜像

```bash
# 使用 docker-compose 构建
docker-compose -f docker-compose.build.yml build

# 或者直接使用 docker build
docker build -t audiobookshelf:latest .
```

### 2. 启动容器

```bash
# 使用 docker-compose 启动
docker-compose -f docker-compose.build.yml up -d

# 查看日志
docker-compose -f docker-compose.build.yml logs -f
```

### 3. 访问应用

打开浏览器访问: `http://localhost:13378`

### 4. 停止容器

```bash
docker-compose -f docker-compose.build.yml down
```

---

## 🌐 方案 2: GitHub Actions 自动构建

### 1. 推送代码触发构建

GitHub Actions 会在以下情况自动构建：
- 推送到 `claude/add-youtube-download-*` 分支
- 推送到 `main` 或 `master` 分支
- 创建标签（如 `v1.0.0`）

构建完成后，镜像会自动推送到 GitHub Container Registry。

### 2. 手动触发构建

在 GitHub 仓库页面：
1. 点击 **Actions** 标签
2. 选择 **Build Docker Image with YouTube Support**
3. 点击 **Run workflow**
4. 输入自定义标签（可选）
5. 点击 **Run workflow** 按钮

### 3. 拉取并使用构建的镜像

```bash
# 登录到 GitHub Container Registry
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# 拉取镜像
docker pull ghcr.io/ivanberry/audiobookshelf:latest

# 使用拉取的镜像
docker run -d \
  --name audiobookshelf \
  -p 13378:80 \
  -v $(pwd)/audio/audiobooks:/audiobooks \
  -v $(pwd)/audio/podcasts:/podcasts \
  -v $(pwd)/audio/metadata:/metadata \
  -v $(pwd)/audio/config:/config \
  -e TZ=Asia/Shanghai \
  ghcr.io/ivanberry/audiobookshelf:latest
```

---

## 📝 docker-compose.build.yml 配置说明

```yaml
services:
  audiobookshelf:
    build:
      context: .              # 使用当前目录的 Dockerfile
      dockerfile: Dockerfile  # Dockerfile 路径
      tags:
        - audiobookshelf:latest  # 镜像标签
        - audiobookshelf:dev     # 开发版标签

    ports:
      - "13378:80"  # 外部端口:容器端口

    volumes:
      # 根据实际情况修改路径
      - ./audio/audiobooks:/audiobooks
      - ./audio/podcasts:/podcasts
      - ./audio/metadata:/metadata
      - ./audio/config:/config

    environment:
      - TZ=Asia/Shanghai  # 时区设置
```

### 自定义配置

**修改端口:**
```yaml
ports:
  - "8080:80"  # 改为 8080 端口
```

**修改存储路径:**
```yaml
volumes:
  - /path/to/your/audiobooks:/audiobooks
  - /path/to/your/podcasts:/podcasts
  - /path/to/your/metadata:/metadata
  - /path/to/your/config:/config
```

**以特定用户运行:**
```yaml
user: "1000:1000"  # UID:GID
```

**添加资源限制:**
```yaml
deploy:
  resources:
    limits:
      cpus: '2'
      memory: 2G
    reservations:
      memory: 512M
```

---

## 🔧 验证 yt-dlp 安装

进入容器检查 yt-dlp 是否正确安装：

```bash
# 进入容器
docker exec -it audiobookshelf sh

# 检查 yt-dlp 版本
yt-dlp --version

# 测试下载（不实际下载）
yt-dlp --simulate https://www.youtube.com/watch?v=dQw4w9WgXcQ

# 退出容器
exit
```

---

## 🐛 故障排查

### 问题 1: yt-dlp 未找到

**症状:** 下载时提示找不到 yt-dlp

**解决:**
```bash
# 进入容器
docker exec -it audiobookshelf sh

# 手动安装 yt-dlp
pip3 install --upgrade yt-dlp

# 或者重新构建镜像
docker-compose -f docker-compose.build.yml build --no-cache
```

### 问题 2: 权限问题

**症状:** 无法创建目录或写入文件

**解决:**
```bash
# 检查目录权限
ls -la audio/

# 修改目录权限
sudo chown -R 1000:1000 audio/

# 或在 docker-compose.yml 中设置用户
user: "1000:1000"
```

### 问题 3: 下载速度慢

**解决:**
在容器中配置 yt-dlp 代理（如果需要）：
```bash
# 进入容器
docker exec -it audiobookshelf sh

# 编辑 yt-dlp 配置
mkdir -p /config/.config/yt-dlp
cat > /config/.config/yt-dlp/config << EOF
--proxy http://your-proxy:port
--socket-timeout 30
EOF
```

---

## 📊 多平台构建

构建支持 AMD64 和 ARM64 架构的镜像：

```bash
# 创建多平台构建器
docker buildx create --name multiarch --use

# 构建并推送多平台镜像
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t audiobookshelf:latest \
  --push \
  .
```

---

## 🔄 更新镜像

### 本地构建版本

```bash
# 停止容器
docker-compose -f docker-compose.build.yml down

# 拉取最新代码
git pull

# 重新构建
docker-compose -f docker-compose.build.yml build --no-cache

# 启动新容器
docker-compose -f docker-compose.build.yml up -d
```

### GitHub Actions 版本

```bash
# 拉取最新镜像
docker pull ghcr.io/ivanberry/audiobookshelf:latest

# 停止旧容器
docker stop audiobookshelf
docker rm audiobookshelf

# 启动新容器（使用新镜像）
docker run -d --name audiobookshelf ...
```

---

## 📖 使用 YouTube 下载功能

1. 以**管理员身份**登录 Audiobookshelf
2. 点击顶部工具栏的**下载图标**（📥）
3. 输入 YouTube 视频或播放列表 URL
4. 选择目标库和文件夹
5. 选择音质
6. 点击"Download"

下载进度会通过通知实时显示。

---

## 💡 提示

- yt-dlp 需要稳定的网络连接
- 首次下载会花费较长时间（提取元数据）
- 播放列表会批量加入队列
- 下载的文件自动按 `上传者/视频标题/` 组织

---

## 🆘 获取帮助

如有问题，请查看：
- 容器日志: `docker-compose -f docker-compose.build.yml logs -f`
- GitHub Issues: 项目仓库的 Issues 页面
