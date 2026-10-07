---
name: douyin-live
description: |
  抖音直播间：获取直播间信息、实时监听弹幕/礼物/进场/点赞/PK、发送弹幕、点赞、查看各类榜单、连麦与 PK 数据、直播间带货商品与商品评论。
  当用户提到直播、直播间、弹幕、监听直播、榜单、榜一、PK、带货、直播商品等与抖音直播相关的操作时使用。
---

## 输入判断

- 想知道直播间信息 / 找 room_id → 步骤 1
- 想实时收弹幕、礼物、进场 → 步骤 2
- 想发弹幕 / 点赞 → 步骤 3
- 想看榜单 / PK / 连麦 → 步骤 4
- 想看直播带货商品 → 步骤 5

## 参数来源（重要）

- `web_rid`：直播间号（`https://live.douyin.com/<直播间号>` 里的数字）——`get_live_info`、`start_live_listen`、`live_room_enter`、PK 相关用它
- `room_id` / `anchor_id` / `sec_anchor_id`：由 `get_live_info` 返回，榜单、弹幕、带货都依赖它
- `channel_id`：PK / 连麦的会话 id，由 `get_live_pk_context`、`get_live_linkmic_list` 返回
- 不要编造这些 id

## 执行流程

### 1. 直播间信息

调用 `get_live_info`（`web_rid` 必填）→ 直播间号、主播（`anchor_id` / `sec_anchor_id`）、状态、拉流地址。

进入直播间（预热会话，部分接口需要）：`live_room_enter`（`web_rid`）。

### 2. 实时监听

1. 启动：`start_live_listen`（`web_rid`）— 监听弹幕、礼物、进场、点赞、关注、房间热度、PK
2. 取数据：`get_live_events`（无参数）— 返回已收集的事件（服务端缓冲）
3. 停止：`stop_live_listen`（无参数）

监听期间应周期调用 `get_live_events` 汇总给用户；**用完必须停止**，避免长连接常驻。

### 3. 互动

- 发弹幕：`send_live_comment`（`room_id`、`content` 均必填）— ⚠️ 以用户身份公开出现，发送前确认文案
- 点赞：`like_live_room`（`room_id`，`count` 默认 1）

### 4. 榜单 / PK / 连麦

| 目的 | 工具 | 参数 |
|---|---|---|
| 榜单列表 | `get_live_rank_list` | `room_id`，`anchor_id`，`sec_anchor_id` |
| 贡献榜 | `get_live_contribution_rank` | `room_id`，`anchor_id`，`sec_anchor_id` |
| 千票榜 | `get_live_thousand_ticket_rank` | `room_id`，`web_rid` |
| PK 排行榜 | `get_live_pk_rank` | `web_rid`，`side`（`current` 默认 / `other`） |
| PK 贡献榜 | `get_live_pk_contribution_rank` | `channel_id`，`anchor_id` |
| PK 上下文 | `get_live_pk_context` | `web_rid`，`channel_id` |
| 连麦列表 | `get_live_linkmic_list` | `room_id`，`channel_id` |

### 5. 直播带货

- 商品列表：`get_live_production`（`page_url` 必填 = `https://live.douyin.com/<房间号>`，`room_id`、`author_id`、`offset`）
- 全部商品：`get_all_live_production`（同上，自动翻页）
- 商品详情：`get_live_production_detail`（`page_url`、`promotion_id` 必填，`origin_type` 默认 `638303`）
- 商品评论：`get_product_comments`（`product_id`、`shop_id` 必填，`cursor`）
- 评论统计：`get_product_comment_counter`（`product_id`、`shop_id` 必填，`stat_id`）
- 商品 SKU / 规格：`get_product_sku_list`（`product_id`、`shop_id` 必填）— 拿商品的规格与价格档位

`promotion_id` 来自商品列表结果；`product_id` / `shop_id` 来自商品详情。

## 约束

- 查询类只读可直接执行；`send_live_comment` 必须先确认文案；监听启停会占用服务端连接
- 监听 + 轮询 `get_live_events` 属于高频操作，建议间隔 ≥5 秒，避免风控
- 点赞 `count` 不要过大（默认 1），批量刷赞违规
- 用完监听一定 `stop_live_listen`，否则连接常驻、事件越积越多

## 失败处理

| 场景 | 处理 |
|---|---|
| 未登录 | 引导使用 douyin-login |
| 直播间未开播 / 已结束 | 告知用户当前不可监听 |
| 缺少 `room_id` / `channel_id` | 先调 `get_live_info` / `get_live_pk_context` 获取，不要编造 |
| 弹幕发送失败 | 可能是弹幕权限、频率限制或风控，展示错误并建议稍后重试 |
| 监听期间无事件 | 确认直播间在播，且 `start_live_listen` 成功；必要时重建监听 |
| 频繁请求被限流 | 降低轮询频率，等待后重试 |
