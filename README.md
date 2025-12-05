docker build -t zeabur-monitor:latest .
docker run -d \
docker logs zeabur-monitor
docker stop zeabur-monitor
docker run -d \
docker-compose up -d
docker-compose logs -f
docker-compose down
docker volume create zeabur-monitor-data
docker run -d \
docker logs zeabur-monitor
docker logs zeabur-monitor | grep -E "密码|账号|已加载"
docker exec -it zeabur-monitor sh
env | grep -E "ADMIN_PASSWORD|ACCOUNTS|PORT"
docker-compose config | grep -A 10 "environment:"
# Zeabur Monitor — Docker 与本地部署指南

本文件仅包含生产/本地部署所需的 Docker 与本地启动步骤，适合把本仓库快速部署到服务器或在本地测试。

前提
- 已安装 Docker（和 docker-compose 可选）
- Node.js 18+（仅在本地直接运行时需要）

快速运行（使用镜像）

推荐在生产主机上以非交互方式运行容器并挂载持久化目录：

PowerShell（示例）：
```powershell
# 在 Windows PowerShell 上（示例目录: C:\opt\zeabur-monitor\data）
docker run -d --name zeabur-monitor `
  -p 3000:3000 `
  -e NODE_ENV=production `
  -e ADMIN_PASSWORD="<your_admin_password>" `
  -v C:\opt\zeabur-monitor\data:/app/config `
  ghcr.io/salist01/zeabur-monitor:latest
```

Linux / macOS（示例）：
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

在项目根（仓库）放置或使用仓库内的 `docker-compose.yml`，示例：

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

持久化目录与初始文件（避免 EISDIR 问题）

建议在宿主机上创建一个 `data` 目录并预置必要文件。容器会在 `/app/config` 下读取/写入这些文件。

PowerShell 创建示例：
```powershell
mkdir .\data
Set-Content .\data\accounts.json '[]' -NoNewline
Set-Content .\data\password.json '{"password":""}' -NoNewline
```

Linux / macOS：
```bash
mkdir -p ./data
echo '[]' > ./data/accounts.json
echo '{"password":""}' > ./data/password.json
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

主要环境变量（简要）
- `PORT`：应用监听端口，默认 `3000`。
- `ADMIN_PASSWORD`：管理员密码（优先于 `password.json`）。生产中请通过 Secrets 提供。
- `ACCOUNTS`：预配置账号字符串，格式 `name1:token1,name2:token2`（建议仅用于临时或不可公开的环境）。
- `CONFIG_DIR`：应用读取配置的目录（容器内默认 `/app/config`）。

构建与发布（简要）
- 仓库已配置 GitHub Actions 自动构建并将镜像推送到 `ghcr.io/salist01/zeabur-monitor`。推送 `v*` tag 时会产生带版本的镜像。

常见问题（快速排查）
- 容器启动但无法访问：确认防火墙、端口映射和容器日志（`docker logs`）。
- 启动中出现 `EISDIR`：通常是宿主挂载导致目录/文件冲突，请按“持久化目录与初始文件”步骤预建文件或改用目录挂载。
- 构建失败 `npm ci`：确保 `package-lock.json` 在构建上下文中，且 `.dockerignore` 未把它排除。

安全提醒
- 请勿在仓库或公开 Compose 文件中存放真实 API Token 或管理员密码。使用平台 Secret/环境变量或 Docker secrets 管理敏感信息。

如果你想，我可以：
- 把 `docker-compose.yml` 示例转换为使用命名卷和 Docker secrets；
- 添加一个自动化 `entrypoint` 用于容器首次启动时初始化 `./data` 文件（可选）。

---

（仅包含 Docker 与本地部署教程。如需把 README 恢复成完整文档或加入更多运维细节，请告知。）
