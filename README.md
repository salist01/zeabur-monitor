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
# Zeabur Monitor — 发布版说明

轻量、生产就绪的 Zeabur 多账号监控面板，适合将多个 Zeabur 账号集中监控余额、项目用量和服务状态。

核心要点
- 运行环境：Node.js 18+
- 部署方式：Docker（推荐）或直接 Node 运行
- 镜像仓库：GitHub Container Registry (`ghcr.io/salist01/zeabur-monitor`)

快速开始（生产）

1) 使用镜像运行（生产示例）：
```bash
docker run -d --name zeabur-monitor \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -e ADMIN_PASSWORD="<your_admin_password>" \
  -v /opt/zeabur-monitor/data:/app/config \
  ghcr.io/salist01/zeabur-monitor:latest
```

2) 推荐使用 `docker-compose`（示例已包含在仓库）：
```bash
# 在项目根：
docker-compose up -d
```

配置说明（生产关注点）
- `ADMIN_PASSWORD`：管理员密码（优先于文件），在生产环境强烈建议使用平台 Secrets 注入。
- 持久化目录：建议挂载宿主目录 `./data` 到容器 `/app/config`（容器内文件：`/app/config/accounts.json`、`/app/config/password.json`）。
- `ACCOUNTS`：可选的预配置账号字符串，格式为 `name1:token1,name2:token2`（不建议将真实 token 放入版本控制或公开 Compose 文件）。

生产部署要点
- 请使用 TLS/HTTPS（反向代理如 Nginx/Traefik）与访问控制。
- 使用 Docker secrets / GitHub Secrets / 平台密钥管理存储敏感信息。
- 为镜像打标签并使用语义化版本：`v1.0.0`。推送 `v*` 标签将触发 CI 发布镜像（详见 GitHub Actions）。

发布流程（简要）
1. 更新代码并提交到 `main`。
2. 在本地或 CI 中运行测试并构建镜像。
3. 打 tag 并推送：
```bash
git tag v1.0.0
git push origin v1.0.0
```
4. GitHub Actions 将构建并推送镜像到 GHCR（工作流已配置）。

故障排查（快速）
- 资源 404（如 `/logo.png`）：确保 `public/logo.png` 在仓库中并且 `express.static` 正确指向 `public` 目录。
- 错误 `EISDIR`：通常因宿主挂载文件路径不存在导致 Docker 在宿主创建目录，请在宿主先创建 `./data/accounts.json` 与 `./data/password.json` 或使用命名卷挂载。
- 构建失败 `npm ci`：确保 `package-lock.json` 在构建上下文中，并且 `.dockerignore` 未排除此文件。

CI / 镜像发布（简述）
- 工作流会在推送 `main` 或 `v*` tag 时构建多平台镜像并推送到 `ghcr.io/salist01/zeabur-monitor`。
- 若要使用 Docker Hub，请在 Actions 中设置相应 Secrets（`DOCKERHUB_USERNAME` / `DOCKERHUB_TOKEN`），并调整工作流。

安全性建议
- 不要在仓库或公开配置中放置真实 API tokens。
- 使用只读或最小权限的 tokens。定期轮换 token。
- 容器运行在受限网络、并在反向代理/身份验证层保护前端接口。

项目结构（简要）
```
zeabur-monitor/
├── public/             # 前端静态资源
├── server.js           # Express 后端
├── Dockerfile          # 生产镜像构建
├── docker-compose.yml  # 示例部署（挂载 ./data:/app/config）
└── .github/workflows/  # CI 配置（镜像构建与发布）
```

联系与贡献
- 欢迎提交 Issue 或 PR；生产问题请附带日志与复现步骤。

许可证
- MIT

— Zeabur Monitor 团队
