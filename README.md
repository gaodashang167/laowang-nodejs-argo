# Node.js 代理服务器

优化版的 Node.js 代理服务器，支持 Xray、Nezha 监控和 Cloudflare Argo 隧道。

## ✨ 主要特性

- ✅ **进程管理优化**：使用 spawn 管理子进程，避免僵尸进程
- ✅ **自动健康检查**：每 5 分钟检查进程状态，自动重启挂掉的服务
- ✅ **内存监控**：实时监控内存使用，防止内存泄漏
- ✅ **优雅关闭**：正确清理所有子进程和资源
- ✅ **智能重试**：启动失败时自动重试，最多 3 次
- ✅ **健康检查接口**：提供 `/health` API 用于外部监控

## 📦 安装依赖

```bash
npm install express axios dotenv
```

## 🚀 快速开始

### 方法 1: 直接运行

```bash
node server.js
```

### 方法 2: 使用环境变量

```bash
UUID=your-uuid-here \
NEZHA_KEY=your-key-here \
ARGO_DOMAIN=your-domain.com \
PORT=8080 \
node server.js
```

### 方法 3: 使用 .env 文件（推荐）

1. 创建 `.env` 文件：

```env
# 必需配置
UUID=your-uuid-here

# Nezha 监控配置（可选）
NEZHA_SERVER=your-server.com:53100
NEZHA_KEY=your-key-here
NEZHA_PORT=

# Cloudflare Argo 配置（可选）
ARGO_DOMAIN=your-domain.com
ARGO_AUTH=your-auth-token
ARGO_PORT=8001

# Cloudflare 配置
CFIP=cdns.doon.eu.org
CFPORT=443

# 服务器配置
PORT=3000
NAME=MyNode

# 路径配置
FILE_PATH=./tmp
SUB_PATH=sb

# 上传配置（可选）
UPLOAD_URL=https://your-api.com
PROJECT_URL=https://your-project.com
AUTO_ACCESS=true
```

2. 在代码开头添加（如果使用 .env 文件）：

```javascript
require('dotenv').config();
```

3. 运行：

```bash
node server.js
```

### 方法 4: 启用垃圾回收（推荐）

```bash
node --expose-gc server.js
```

## 📋 环境变量说明

| 变量名 | 说明 | 默认值 | 必需 |
|--------|------|--------|------|
| `UUID` | 用户唯一标识符 | `650ad2cd-0933-46ee-a044-812aa7a6ae46` | ✅ |
| `NEZHA_SERVER` | Nezha 监控服务器地址 | `mbb.svip888.us.kg:53100` | ❌ |
| `NEZHA_PORT` | Nezha 服务器端口 | `` | ❌ |
| `NEZHA_KEY` | Nezha 密钥 | `iz2q6GK7gAFmQljm54fuePp3K98AqB0D` | ❌ |
| `ARGO_DOMAIN` | Cloudflare Argo 域名 | `pella.wuge.nyc.mn` | ❌ |
| `ARGO_AUTH` | Argo 认证 token | `eyJh...` | ❌ |
| `ARGO_PORT` | Argo 本地端口 | `8001` | ❌ |
| `CFIP` | Cloudflare IP 地址 | `cdns.doon.eu.org` | ❌ |
| `CFPORT` | Cloudflare 端口 | `443` | ❌ |
| `PORT` / `SERVER_PORT` | HTTP 服务器监听端口 | `3000` | ❌ |
| `NAME` | 节点名称前缀 | `` | ❌ |
| `FILE_PATH` | 文件存储路径 | `./tmp` | ❌ |
| `SUB_PATH` | 订阅路径 | `sb` | ❌ |
| `UPLOAD_URL` | 上传节点的 API 地址 | `` | ❌ |
| `PROJECT_URL` | 项目访问地址 | `` | ❌ |
| `AUTO_ACCESS` | 是否启用自动保活 | `false` | ❌ |

## 🔍 API 接口

### 1. 主页

```
GET /
```

返回: `Hello world!`

### 2. 订阅地址

```
GET /{SUB_PATH}
```

默认: `GET /sb`

返回: Base64 编码的订阅内容（vless、vmess、trojan 节点）

### 3. 健康检查

```
GET /health
```

返回示例:

```json
{
  "status": "ok",
  "uptime": 3600,
  "memory": {
    "rss": 55664640,
    "heapTotal": 9437184,
    "heapUsed": 8421376,
    "external": 1089536
  },
  "processes": [
    {
      "name": "xray",
      "pid": 12345,
      "alive": true
    },
    {
      "name": "cloudflare",
      "pid": 12346,
      "alive": true
    }
  ]
}
```

## 📊 监控说明

### 内存监控

服务器每分钟输出一次内存使用情况：

```
[INFO] Memory - Heap: 8MB / 9MB, RSS: 53MB
```

**正常情况**: 内存应该保持稳定，波动在 ±20MB 范围内

**异常情况**: 如果内存持续增长（如 53MB → 60MB → 70MB → ...），说明存在内存泄漏

### 健康检查

每 5 分钟自动检查一次所有子进程状态：

```
[INFO] Running health check...
[INFO] Health check passed - all processes alive
```

如果检测到进程挂掉，会自动重启：

```
[WARN] Process xray (PID: 12345) is dead
[WARN] Detected dead process, restarting all services...
[INFO] xray started with PID: 12347
```

## 🐳 Docker 部署

### Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "--expose-gc", "server.js"]
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  proxy-server:
    build: .
    container_name: proxy-server
    environment:
      - UUID=your-uuid-here
      - NEZHA_SERVER=your-server.com:53100
      - NEZHA_KEY=your-key-here
      - ARGO_DOMAIN=your-domain.com
      - ARGO_AUTH=your-auth-token
      - PORT=3000
      - NAME=MyNode
      - AUTO_ACCESS=true
    ports:
      - "3000:3000"
    restart: unless-stopped
    volumes:
      - ./tmp:/app/tmp
```

运行：

```bash
docker-compose up -d
```

## 🛠️ 故障排查

### 1. 查看日志

```bash
# 查看启动日志
[INFO] Server initialization started
[INFO] Step 1: Setting up Argo tunnel
[INFO] Step 2: Deleting historical nodes
[INFO] Step 3: Cleaning up old files
[INFO] Step 4: Generating Xray config
[INFO] Step 5: Downloading files
[INFO] Step 6: Starting processes
[INFO] Step 7: Extracting domain
[INFO] Step 8: Generating subscription
[INFO] Step 9: Adding auto-access task
[INFO] Step 10: Starting health check
[INFO] Step 11: Starting memory monitor
[INFO] Step 12: Scheduling file cleanup
[INFO] Server initialization completed successfully
```

### 2. 检查健康状态

```bash
curl http://localhost:3000/health
```

### 3. 常见问题

#### 问题: 进程频繁重启

**原因**: 可能是下载的文件损坏或权限不足

**解决**:
```bash
# 清理临时文件
rm -rf ./tmp/*

# 重新启动
node server.js
```

#### 问题: 内存持续增长

**原因**: 可能存在内存泄漏

**解决**:
```bash
# 使用 --expose-gc 启动，允许手动触发垃圾回收
node --expose-gc server.js
```

#### 问题: 订阅地址无法访问

**原因**: SUB_PATH 配置错误或服务未完全启动

**解决**:
```bash
# 检查配置
echo $SUB_PATH

# 等待服务完全启动（约 30 秒）
curl http://localhost:3000/sb
```

## 📝 日志说明

### 正常运行日志

```
[INFO] HTTP server listening on port 3000
[INFO] Health check endpoint: http://localhost:3000/health
[INFO] Server initialization started
[INFO] Downloaded web
[INFO] Downloaded bot
[INFO] xray started with PID: 12345
[INFO] cloudflare started with PID: 12346
[INFO] Subscription generated and saved
[INFO] Server initialization completed successfully
[INFO] Memory - Heap: 8MB / 9MB, RSS: 53MB
[INFO] Running health check...
[INFO] Health check passed - all processes alive
```

### 异常日志

```
[ERROR] Download error: timeout of 30000ms exceeded
[WARN] Process xray (PID: 12345) is dead
[ERROR] Startup failed: Domain extraction failed after max retries
```

## 🔒 安全建议

1. **不要在代码中硬编码敏感信息**，使用环境变量
2. **定期更新 UUID 和密钥**
3. **限制 `/health` 接口访问**（如使用 nginx 反向代理）
4. **使用 HTTPS** 访问订阅地址
5. **定期检查日志**，及时发现异常

## 📄 许可证

MIT License

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## 📮 联系方式

如有问题，请通过 Issue 反馈。
