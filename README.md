# Zeabur 多账号监控（简洁说明）

轻量的 Zeabur 多账号仪表盘，用于查看余额、项目费用、服务状态与日志。

快速概览
- Node.js 18+、Express 后端，前端为单页应用（`public/index.html`）
- 支持通过环境变量或宿主挂载文件导入账号（详见下文）

快速启动（Docker 推荐）

1) 使用预构建镜像：
```powershell
docker run -d -p 3000:3000 ghcr.io/salist01/zeabur-monitor:latest
```

2) 使用 `docker-compose`（推荐持久化配置）：
```powershell
# 项目根目录运行
docker-compose up -d
```

配置说明（精简）
- `ADMIN_PASSWORD`：管理员密码（可选，优先于文件）
- `ACCOUNTS`：预配置账号，格式 `name1:token1,name2:token2`
- `CONFIG_DIR`：配置目录（容器中建议挂载 `./data:/app/config`），默认使用仓库内 `config` 目录

数据持久化（推荐）
- 在项目根创建 `./data`，并挂载到容器 `/app/config`：
  - `./data/accounts.json`（数组）
  - `./data/password.json`（{ "password": "..." }）

示例（创建文件）：
```powershell
mkdir .\data
Set-Content .\data\accounts.json '[]' -NoNewline
Set-Content .\data\password.json '{ "password": "" }' -NoNewline
```

常见问题（快速排查）
- 报错 `EISDIR`: 请确认宿主挂载的是文件而不是不存在时被创建的目录；建议挂载 `./data:/app/config` 并提前创建 `./data` 下的文件。
- 构建失败 `npm ci`: 确保 `package-lock.json` 在上下文中（不要在 `.dockerignore` 中排除它）。

如需完整文档、开发指南或希望我把 `docker-compose.yml` 改为命名卷方案，请告诉我。
- **API**：Zeabur GraphQL API
- **样式**：原生 CSS（玻璃拟态效果）

## 📁 项目结构

```
zeabur-monitor/
├── public/
│   ├── index.html      # 前端页面
│   ├── bg.png          # 背景图片
│   └── favicon.png     # 网站图标
├── server.js           # 后端服务
├── package.json        # 项目配置
├── .env.example        # 环境变量示例
├── .gitignore          # Git 忽略规则
├── .dockerignore       # Docker 忽略规则
├── Dockerfile          # Docker 镜像构建配置
├── zbpack.json         # Zeabur 配置
├── README.md           # 项目说明
└── DEPLOY.md           # 部署指南
```

## 🐳 Docker 部署

### 使用 Docker 运行

#### 快速启动（从预构建镜像）

```bash
# 使用 GitHub Container Registry 镜像运行
docker run -d \
  --name zeabur-monitor \
  -p 3000:3000 \
  ghcr.io/salist01/zeabur-monitor:latest
```

然后访问 `http://localhost:3000`

#### 本地构建并运行

```bash
# 1. 构建镜像
docker build -t zeabur-monitor:latest .

# 2. 运行容器
docker run -d \
  --name zeabur-monitor \
  -p 3000:3000 \
  zeabur-monitor:latest

# 3. 查看日志
docker logs zeabur-monitor

# 4. 停止容器
docker stop zeabur-monitor
```

### Docker 环境变量

运行 Docker 容器时可通过 `-e` 传递以下环境变量：

| 环境变量 | 说明 | 默认值 | 示例 |
|---------|------|-------|------|
| `PORT` | 应用监听端口 | `3000` | `-e PORT=8080` |
| `NODE_ENV` | Node 环境 | `production` | `-e NODE_ENV=production` |
| `ACCOUNTS` | 预配置账号列表 | 无 | `-e ACCOUNTS="alice:token1,bob:token2"` |
| `ADMIN_PASSWORD` | 管理员密码（可选） | 无 | `-e ADMIN_PASSWORD="mypassword123"` |

**ACCOUNTS 格式说明：**
```
"账号名1:API_Token1,账号名2:API_Token2"
```

完整示例：
```bash
docker run -d \
  --name zeabur-monitor \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -e PORT=3000 \
  -e ACCOUNTS="my-account:sk-xxxxxxxxxxxxxxxx,backup-account:sk-yyyyyyyyyyyyyyy" \
  -e ADMIN_PASSWORD="secure_password_123" \
  ghcr.io/salist01/zeabur-monitor:latest
```

### 使用 Docker Compose（推荐）

创建 `docker-compose.yml` 文件：

```yaml
version: '3.8'

services:
  zeabur-monitor:
    image: ghcr.io/salist01/zeabur-monitor:latest
    container_name: zeabur-monitor
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      PORT: "3000"
      # 如需预配置账号，取消注释并填入（不推荐在此放入敏感信息）
      # ACCOUNTS: "account1:token1,account2:token2"
    volumes:
      # 持久化账号和密码文件（可选）
      - ./data/accounts.json:/app/accounts.json
      - ./data/password.json:/app/password.json
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
```

启动服务：

```bash
# 启动
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止
docker-compose down
```

### 持久化数据

如需保留账号信息和密码，需要挂载数据卷：

#### 方法 1：使用 Docker 卷（推荐）
```bash
docker volume create zeabur-monitor-data

docker run -d \
  --name zeabur-monitor \
  -p 3000:3000 \
  -v zeabur-monitor-data:/app \
  ghcr.io/salist01/zeabur-monitor:latest
```

#### 方法 2：挂载宿主机目录
```bash
# 创建数据目录
mkdir -p ./data

# 预置密码文件（可选）
echo '{"password":"your_password"}' > ./data/password.json

# 运行容器
docker run -d \
  --name zeabur-monitor \
  -p 3000:3000 \
  -v $(pwd)/data:/app \
  ghcr.io/salist01/zeabur-monitor:latest
```

### 支持的平台

当前 GitHub Actions 工作流自动构建以下平台的镜像：
- `linux/amd64` - 64 位 Intel/AMD 处理器
- `linux/arm64` - ARM 64 位处理器（如树莓派 4、Apple Silicon Mac 用户）

### GitHub Container Registry（GHCR）

镜像自动发布到 GitHub Container Registry：
- **最新版本**：`ghcr.io/salist01/zeabur-monitor:latest`
- **版本标签**：`ghcr.io/salist01/zeabur-monitor:COMMIT_SHA`
- **发布标签**（tag）：`ghcr.io/salist01/zeabur-monitor:v1.0.0`（当推送 v* tag 时）

### 故障排查

**问题：容器启动后立即退出**
```bash
# 查看日志
docker logs zeabur-monitor

# 常见原因：port 已被占用或依赖安装失败
```

**问题：无法访问应用**
- 检查 port 映射：`docker port zeabur-monitor`
- 检查防火墙规则
- 在容器内测试：`docker exec zeabur-monitor curl http://localhost:3000`

**问题：构建失败（npm ci 出错）**
- 确保 `package-lock.json` 存在
- 确保 `.dockerignore` 未排除 `package-lock.json`
- 本地清理缓存后重试：`docker build --no-cache -t zeabur-monitor:latest .`

### 环境变量调试

**验证环境变量是否生效**

1. 进入容器查看启动日志：
```bash
docker logs zeabur-monitor | grep -E "密码|账号|已加载"
```

2. 检查环境变量是否被正确读取：
```bash
# 进入容器终端
docker exec -it zeabur-monitor sh

# 查看 Node 进程的环境变量
env | grep -E "ADMIN_PASSWORD|ACCOUNTS|PORT"
```

3. 如果使用 docker-compose，检查配置：
```bash
docker-compose config | grep -A 10 "environment:"
```

**常见问题与解决方案**

| 问题 | 原因 | 解决方案 |
|------|------|--------|
| 密码环境变量未生效 | 环境变量未被正确传递 | 检查 docker run/docker-compose 的 `-e` 或 `environment:` 配置 |
| ACCOUNTS 账号未显示 | 解析错误或格式不正确 | 检查格式是否为 `name1:token1,name2:token2`，token 中不能包含逗号 |
| 前端显示"密码已设置，无法重复设置" | 文件或环境变量中已存在密码 | 如果无法进入，尝试删除 `password.json` 文件或清空环境变量 |
| Docker 构建时依赖缺失 | `package-lock.json` 被忽略 | 确保 `.dockerignore` 中没有 `package-lock.json` |

## 🔒 安全说明

### 密码保护
- 首次使用需要设置管理员密码（至少 6 位）
- 密码存储在服务器的 `password.json` 文件中
- 登录后 4 天内自动保持登录状态

### API Token 安全
- Token 存储在服务器的 `accounts.json` 文件中
- 输入时自动打码显示（`●●●●●●`）
- 不会暴露在前端代码或浏览器中

### 重要提示
⚠️ **请勿将以下文件提交到 Git：**
- `.env` - 环境变量
- `accounts.json` - 账号数据
- `password.json` - 管理员密码

这些文件已在 `.gitignore` 中配置。

## 🎨 自定义

### 更换背景图片
替换 `public/bg.png` 为你喜欢的图片

### 调整透明度
使用页面上的透明度滑块调节

### 修改主题色
在 `public/index.html` 中搜索 `#f696c6` 并替换为你喜欢的颜色

## 🔄 多设备同步

账号信息存储在服务器上，所有设备自动同步！

- 在电脑上添加账号 → 手机、平板立即可见
- 在手机上删除账号 → 所有设备同步删除
- 无需任何配置，开箱即用

## 🛠️ 开发

### 环境变量（可选）

创建 `.env` 文件：
```env
PORT=3000
ACCOUNTS=账号1:token1,账号2:token2
```

### API 端点

- `GET /` - 前端页面
- `POST /api/check-password` - 检查是否已设置密码
- `POST /api/set-password` - 设置管理员密码
- `POST /api/verify-password` - 验证密码
- `POST /api/temp-accounts` - 获取账号信息
- `POST /api/temp-projects` - 获取项目信息
- `POST /api/validate-account` - 验证账号
- `GET /api/server-accounts` - 获取服务器存储的账号
- `POST /api/server-accounts` - 保存账号到服务器
- `DELETE /api/server-accounts/:index` - 删除账号
- `POST /api/project/rename` - 重命名项目
- `POST /api/service/pause` - 暂停服务
- `POST /api/service/restart` - 重启服务
- `POST /api/service/logs` - 获取服务日志

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## 📄 许可证

MIT License - 自由使用和修改

## ⭐ Star History

如果这个项目对你有帮助，请给个 Star ⭐

## 🙏 致谢

- [Zeabur](https://zeabur.com) - 提供优秀的云服务平台
- [Vue.js](https://vuejs.org) - 渐进式 JavaScript 框架
- [Express](https://expressjs.com) - 快速、开放、极简的 Web 框架

---

Made with ❤️ by [jiujiu532](https://github.com/jiujiu532)
