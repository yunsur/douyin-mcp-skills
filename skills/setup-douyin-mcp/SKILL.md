---
name: setup-douyin-mcp
description: |
  安装部署 douyin-mcp 服务并配置 MCP 连接，引导用户完成从零到可用的全流程。
  当用户第一次使用抖音功能、提到安装/部署/配置抖音 MCP、服务连不上、或 check_login_status 等 MCP 工具不可用时使用。
---

项目仓库：https://github.com/yunsur/douyin-mcp（Go 版，HTTP API + MCP 双协议）

## 执行流程

### 1. 检测服务状态

```bash
curl -sf http://127.0.0.1:18080/health && echo "running" || echo "not running"
```

注意：只有 `/health` 可以用 GET 探测（MCP 端点 `/mcp` 只接受 POST，GET 会返回 405），不要用 `/mcp` 判断存活。

- 已运行 → 记录地址 `http://127.0.0.1:18080/mcp`，跳到步骤 3
- 未运行 → 询问用户：服务是否部署在其他地址/端口？
  - 用户提供地址 → 验证可达后跳到步骤 3
  - 未部署 → 进入步骤 2

### 2. 部署服务

确认操作系统（macOS / Linux / Windows）和是否已安装 Docker。

#### 方式一：Docker Compose（推荐）

镜像内置 Node.js（仅在命中 `__ac_signature` 挑战页时用到），**不含无头浏览器**，体积小、启动快。

```bash
# 下载 docker-compose.yml
wget https://raw.githubusercontent.com/yunsur/douyin-mcp/main/docker/docker-compose.yml

# 启动服务
docker compose up -d

# 查看日志
docker compose logs -f
```

镜像源：`yunsur/douyin-mcp`（自行构建：在项目根目录 `docker build -t yunsur/douyin-mcp .`）。

数据持久化：`./data`（cookie 与运行态，容器内 `/app/data`）。登录时在容器内执行一次：

```bash
docker exec -it douyin-mcp sh -c '/app/douyin-login -phone 138xxxxxxxx'
# 扫码登录：-qr /app/data/login_qrcode.png 后把图片 docker cp 出来
```

#### 方式二：下载二进制

从 GitHub Releases 下载：https://github.com/yunsur/douyin-mcp/releases/latest

```bash
curl -s https://api.github.com/repos/yunsur/douyin-mcp/releases/latest | grep browser_download_url
```

- 主程序：`douyin-mcp-{darwin-arm64,darwin-amd64,linux-amd64,linux-arm64,windows-amd64.exe}`
- 登录工具：`douyin-login-{...}`（运行后把 cookie 写入当前目录 `cookies.txt`）

注意：二进制方式不需要浏览器；仅当抖音返回挑战页时需要本机有 Node.js（可选）。

#### 方式三：源码编译

适合 Go 开发者。

```bash
git clone https://github.com/yunsur/douyin-mcp
cd douyin-mcp
go mod download
go build -o douyin-mcp .
go build -o douyin-login ./cmd/login

# 前台启动
./douyin-mcp -port 127.0.0.1:18080
```

#### 常驻运行（macOS LaunchAgent，可选）

服务需常驻才能被 MCP 客户端调用。写入 `~/Library/LaunchAgents/sh.omp.douyin-mcp.plist`：

```xml
<key>ProgramArguments</key>
<array>
  <string>/绝对路径/douyin-mcp</string>
  <string>-port</string><string>127.0.0.1:18080</string>
</array>
<key>WorkingDirectory</key><string>/绝对路径/douyin-mcp</string>
<key>EnvironmentVariables</key>
<dict>
  <key>DOUYIN_COOKIES_FILE</key><string>/绝对路径/douyin-mcp/cookies.txt</string>
</dict>
<key>RunAtLoad</key><true/>
<key>KeepAlive</key><true/>
```

```bash
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/sh.omp.douyin-mcp.plist
curl -sf http://127.0.0.1:18080/health
```

改代码后重建并重启：`go build -o douyin-mcp . && launchctl kickstart -k gui/$(id -u)/sh.omp.douyin-mcp`

### 3. 检测 MCP 连接配置

检查客户端是否已配置 douyin MCP 连接。

- **omp**：`~/.omp/agent/mcp.json` 的 `mcpServers.douyin`
- **Claude Code**：`~/.claude/settings.json`、项目级 `.claude/settings.json`，或 `claude mcp list`
- **Cursor**：`.cursor/mcp.json`

- 已配置且地址正确 → 跳到步骤 5
- 已配置但地址不匹配 → 修正地址
- 未配置 → 进入步骤 4

### 4. 配置 MCP 连接

**omp**（`~/.omp/agent/mcp.json`）：

```json
{
  "mcpServers": {
    "douyin": { "type": "http", "url": "http://127.0.0.1:18080/mcp" }
  }
}
```

**Claude Code**：

```bash
claude mcp add douyin --transport http http://127.0.0.1:18080/mcp
```

配置文件中对应写法：

```json
{ "mcpServers": { "douyin": { "url": "http://127.0.0.1:18080/mcp" } } }
```

**其他客户端**：告知用户 MCP 地址，让用户按客户端文档自行配置（HTTP / Streamable HTTP 传输）。

### 5. 验证与提示

1. **提示用户重启当前会话** — MCP 配置变更后需重启客户端才会加载新工具
2. 重启后调用 `check_login_status` 验证连接
3. 未登录 → 引导用户使用 `/douyin-login`

## 环境变量（可选）

| 变量 | 说明 |
| --- | --- |
| `DOUYIN_COOKIES` / `DOUYIN_COOKIES_FILE` | Cookie 字符串 / cookie 文件路径（默认 `./cookies.txt`） |
| `DOUYIN_PORT` | 监听端口，默认 `:18080` |
| `AUTH_TOKEN` | 静态 Bearer Token 鉴权，留空关闭（仅监听 127.0.0.1 时可留空） |
| `DOUYIN_TICKET` / `DOUYIN_TS_SIGN` / `DOUYIN_CLIENT_CERT` / `DOUYIN_PRIVATE_KEY` | bd-ticket-guard 凭据（写接口需要） |
| `DOUYIN_DTRAIT_BLOB` | `x-tt-session-dtrait` 设备特征（**创作者发布必需**） |
| `DOUYIN_PROXY` | 代理，如 `http://127.0.0.1:7890` |
| `DY_FP_*` / `DY_HTTP_PROFILE` | 浏览器指纹覆盖（UA/屏幕/核数），需与 cookie 所属浏览器一致 |

## 失败处理

| 场景 | 处理 |
|---|---|
| Docker 未安装 | 建议安装 Docker，或改用二进制 / 源码方式 |
| Docker 里 Cookie 传不进去 | 用 `DOUYIN_COOKIES_FILE=/app/data/cookies.txt` 并挂载 `./data:/app/data`，或在容器内跑一次 `douyin-login` |
| Go 未安装（源码方式） | 引导安装 Go 1.26+（`brew install go`） |
| 端口 18080 被占用 | 换端口（`-port 127.0.0.1:18081`）并同步修改 MCP 配置地址 |
| 服务启动即退出 | 查看日志（LaunchAgent 默认 `/tmp/douyin-mcp.log`），常见原因是端口占用或 cookie 文件权限 |
| 工具调用返回 401 | 服务设了 `AUTH_TOKEN`，客户端需带 `Authorization: Bearer <token>` |
| 工具调用返回 403/404 或人机验证 | 说明 cookie 失效或请求被风控，重新登录或更换出口 IP |
| 挑战页需要 `__ac_signature` | 本机装 Node.js 后服务自动求解；否则会返回清晰报错，可用 `DY_AC_SIGNATURE` 注入 |
| 重启后工具仍不可用 | 提示重启客户端会话 |
