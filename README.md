# Zeabur Monitor

前提
- 已安装 Docker（和 docker-compose 可选）
- Node.js 18+（仅在本地直接运行时需要）

快速运行（使用镜像）

Linux / macOS：
```bash
docker run -d --name zeabur-monitor \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -e ADMIN_PASSWORD="<your_admin_password>" \
  -v /opt/zeabur-monitor/data:/app/config \
  ghcr.io/salist01/zeabur-monitor:latest
```

使用 `docker ps` 检查容器是否启动，使用 `docker logs zeabur-monitor -f` 查看启动日志。

Docker Compose（推荐用于持久化管理）

```yaml
version: '3.8'
services:
  zeabur-monitor:
    image: ghcr.io/salist01/zeabur-monitor:latest
    container_name: zeabur-monitor
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - PORT=3000
      # - ACCOUNTS=account1:token1,account2:token2   # 可选，不推荐明文存放
      # - ADMIN_PASSWORD=${ADMIN_PASSWORD}            # 推荐通过 env 或 secrets 提供
    volumes:
      - ./data:/app/config
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000" ]
      interval: 30s
      timeout: 10s
      retries: 3

```

启动/管理：
```powershell
# 启动
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止并移除容器
docker-compose down
```

重要：如果挂载单个宿主文件到容器（例如 `./data/password.json:/app/config/password.json`），请确保宿主文件存在，否则 Docker 可能在宿主创建目录导致容器内出现 `EISDIR` 错误。推荐挂载目录（`./data:/app/config`）。

本地直接运行（开发或调试）

仅在本地调试或开发时使用（生产请使用 Docker）：

```powershell
# 安装依赖
npm ci

# 启动（开发模式）
npm start

# 访问 http://localhost:3000
```

主要环境变量
- `PORT`：应用监听端口，默认 `3000`。
- `ADMIN_PASSWORD`：管理员密码（优先于 `password.json`）。生产中请通过 Secrets 提供。
- `ACCOUNTS`：预配置账号字符串，格式 `name1:token1,name2:token2`（建议仅用于临时或不可公开的环境）。
- `CONFIG_DIR`：应用读取配置的目录（容器内默认 `/app/config`）。

构建与发布
- 仓库已配置 GitHub Actions 自动构建并将镜像推送到 `ghcr.io/salist01/zeabur-monitor`。推送 `v*` tag 时会产生带版本的镜像。

常见问题（快速排查）
- 容器启动但无法访问：确认防火墙、端口映射和容器日志（`docker logs`）。
- 启动中出现 `EISDIR`：通常是宿主挂载导致目录/文件冲突，请按“持久化目录与初始文件”步骤预建文件或改用目录挂载。
- 构建失败 `npm ci`：确保 `package-lock.json` 在构建上下文中，且 `.dockerignore` 未把它排除。

安全提醒
- 请勿在仓库或公开 Compose 文件中存放真实 API Token 或管理员密码。使用平台 Secret/环境变量或 Docker secrets 管理敏感信息。