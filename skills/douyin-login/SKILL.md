---
name: douyin-login
description: |
  管理抖音登录状态：检查是否已登录、扫码登录、手机号验证码登录、重置登录切换账号。
  当用户提到登录、扫码、验证码登录、账号、切换账号、退出登录、登录状态检查，或其他 skill 报告"未登录"需要先登录时使用。
---

## 执行流程

### 1. 检查登录状态

调用 `check_login_status`（无参数）。返回：

```json
{ "logged_in": true, "sec_uid": "MS4wLjABAAAA...", "uid": 85390087718 }
```

- 已登录 → 告知用户当前账号（uid / sec_uid）
- 未登录 → 进入步骤 2

### 2. 登录

#### 方式 A：扫码登录（推荐）

1. 调用 `get_login_qrcode`（无参数）。返回两部分：
   - 文本：`token` 与超时提示
   - 图片：PNG 二维码（MCP image content，Base64）
2. **展示二维码**：客户端能渲染图片就直接展示；纯文本终端则落盘让用户打开：

```bash
echo "<base64_data>" | base64 -d > /tmp/douyin-qrcode.png
open /tmp/douyin-qrcode.png        # macOS
xdg-open /tmp/douyin-qrcode.png    # Linux
```

   提示用户：打开抖音 App → 扫一扫 → 确认登录。二维码有过期时间，过期重新获取。
3. 用户扫码后，调用 `check_login_qrcode`，参数 `token`（步骤 1 返回的 token）轮询状态；成功后 **Cookie 自动写入服务端本地**（`cookies.txt`）。
4. 调用 `check_login_status` 确认登录成功。

#### 方式 B：手机号 + 验证码

1. 向用户要手机号，调用 `send_phone_code`（`phone`，必填）。
2. 用户提供收到的短信验证码后，调用 `login_by_phone`（`phone`、`code`，均必填）。
3. 调用 `check_login_status` 确认。

### 3. 重新登录 / 切换账号

1. 调用 `delete_cookies`（⚠️ 需用户确认）——清除当前登录状态
2. 按步骤 2 重新登录
3. `check_login_status` 确认新账号

## 约束

- `delete_cookies`、`send_phone_code`、`login_by_phone` 会改变账号/会话状态，执行前必须确认
- 扫码需要用户手动用抖音 App 完成，无法自动完成
- 登录成功后 cookie 由服务写入本地文件；重启服务仍是登录态
- 同一 cookie 频繁切换 IP/设备会触发风控，尽量避免

## 失败处理

| 场景 | 处理 |
|---|---|
| MCP 工具不可用 | 引导用户使用 `/setup-douyin-mcp` 完成部署和连接配置 |
| 二维码超时 | 重新调用 `get_login_qrcode` 获取新二维码与新 token |
| `check_login_qrcode` 一直未成功 | 确认用户是否已扫码；无效则重新获取二维码 |
| 手机验证码收不到 | 确认手机号格式（+86 11 位）；稍后重试，避免频繁请求 |
| 登录后调用其他工具仍失败（401/403/人机验证） | cookie 未生效或被风控：重新登录，必要时更换出口 IP |
