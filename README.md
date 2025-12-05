# 🚀 Zeabur 多账号监控面板

一个美观、强大的 Zeabur 多账号监控工具，实时显示免费额度使用情况、项目费用和服务状态。

![](https://img.shields.io/badge/Node.js-18+-green.svg)
![](https://img.shields.io/badge/License-MIT-blue.svg)
![](https://img.shields.io/badge/Zeabur-Ready-blueviolet.svg)

## ✨ 功能特性

- 🎨 **现代化 UI** - 粉色主题 + 玻璃拟态效果 + 动漫背景
- 💰 **实时余额监控** - 显示每月免费额度剩余（$X.XX / $5.00）
- ***项目费用追踪** - 每个项目的实时费用统计
- ✏️ **项目快速改名** - 点击铅笔图标即可重命名项目
- 🌐 **域名显示** - 显示项目的所有域名，点击直接访问
- 🐳 **服务状态监控** - 显示所有服务的运行状态和资源配置
-  ***多账号支持** - 同时管理多个 Zeabur 账号
-  ***自动刷新** - 每 30 秒自动更新数据
- 🎚️ **透明度调节** - 可调节卡片透明度（0-100%）
- 📱 **响应式设计** - 完美适配各种屏幕尺寸
- ***密码保护** - 管理员密码验证，保护账号安全
- 💾 **服务器存储** - 账号数据存储在服务器，多设备自动同步
- ⏸️ **服务控制** - 暂停、启动、重启服务
- 📋 **查看日志** - 实时查看服务运行日志

## 📦 快速开始

### 前置要求

- Node.js 18+
- Zeabur 账号和 API Token

### 获取 Zeabur API Token

1. 登录 [Zeabur 控制台](https://zeabur.com)
2. 点击右上角头像 → **Settings**
3. 找到 **Developer** 或 **API Keys** 选项
4. 点击 **Create Token**
5. 复制生成的 Token（格式：`sk-xxxxxxxxxxxxxxxx`）

### 本地部署

```bash
# 1. 克隆项目
git clone https://github.com/jiujiu532/zeabur-monitor.git
cd zeabur-monitor

# 2. 安装依赖
npm install

# 3. 启动服务
npm start

# 4. 访问应用
# 打开浏览器访问：http://localhost:3000
```

### Zeabur 部署（推荐）

详细部署步骤请查看 [DEPLOY.md](./DEPLOY.md)

## 📖 使用说明

### 首次使用

1. 访问应用后，首次使用需要设置管理员密码（至少 6 位）
2. 设置完成后，使用密码登录
3. 点击 **"⚙️ 管理账号"** 添加 Zeabur 账号

### 添加账号

#### 单个添加
1. 点击 **"⚙️ 管理账号"**
2. 输入账号名称和 API Token
3. 点击 **"➕ 添加到列表"**

#### 批量添加
支持以下格式（每行一个账号）：
- `账号名称:API_Token`
- `账号名称：API_Token`
- `账号名称(API_Token)`
- `账号名称（API_Token）`

### 项目改名

1. 找到项目卡片
2. 点击项目名称右侧的 **✏️** 铅笔图标
3. 输入新名称，按 `Enter` 保存或 `Esc` 取消

### 服务控制

- **暂停服务**：点击 **⏸️ 暂停** 按钮
- **启动服务**：点击 **▶️ 启动** 按钮
- **重启服务**：点击 **🔄 重启** 按钮
- **查看日志**：点击 **📋 日志** 按钮

## 🔧 技术栈

- **后端**：Node.js + Express
- **前端**：Vue.js 3 (CDN)
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
