---
name: douyin-im
description: |
  抖音私信与群聊：查看会话列表与未读数、读取聊天记录（含只看未读）、给用户发私信/图片/表情包/名片/分享作品与链接、群成员、已读标记、置顶免打扰、实时接收新消息。
  当用户提到私信、消息、聊天记录、群、群聊、未读、发消息、@我、实时收消息等与抖音 IM 相关的操作时使用。
---

## 输入判断

- 看会话 / 有多少未读 → 步骤 1
- 读某会话的聊天记录（含只看未读） → 步骤 2
- 给某人发消息 / 分享内容 → 步骤 3
- 群信息与群管理 → 步骤 4
- 实时接收新消息 → 步骤 5
- 增量拉取 / 消息面板（在线状态、互动动态、热门表情） → 步骤 6

## 关键参数

- `conversation`：会话 **id 或名称**都能用（群 id 是纯数字；私聊形如 `0:1:a:b`；名称如 `九号mz5闲聊群`，支持模糊匹配）
- `to_user_id`：**对方的数字 uid**（不是 sec_uid）。获取方式：`get_user_info` 返回的 uid、会话消息里的 `sender`、或会话列表里的 `owner_uid`
- `sec_uid`：用于查昵称（`get_im_user_info`）与名片分享
- 自己的 uid / sec_uid 来自 `check_login_status`

## 执行流程

### 1. 会话列表与未读

调用 `list_conversations`：

- `name`（string，可选）— 按名称模糊过滤，留空返回全部
- `type`（int，可选）— `1` 私聊 | `2` 群聊 | `0` 不限

每条返回：`name`、`conversation_id`、`conversation_type`、最近消息，以及：

- `unread`：服务端未读数（消息类 1+2 之和）
- `unread_classes`：分类明细 `{类: 数量}`（类 2 = 普通消息，3/4 = 群/系统类）
- `cached`：`true` 表示该会话来自本地累积（服务端只返回变化过的会话）

未登录/新账号刚查询时列表可能较少，重复查询会逐步累积（服务端接口只回增量）。

### 2. 读聊天记录

- 普通翻页：`get_conversation_history`
  - `conversation`（必填）、`cursor`（0 = 最新一页，返回 `next_cursor` 继续往前翻）、`count`（默认 50，上限 50）
- **只看未读**：`get_conversation_history` + `unread_only=true`
  - 按服务端未读数从最新往回自动翻页，返回 `unread`、`unread_classes`、`messages`、`fetched`、`truncated`
- 会话详情：`get_conversation_info`（`conversation`）— 名称、类型、id、群主、未读数
- 群成员：`get_conversation_participants`（`conversation`、`offset`、`count`）
- 陌生人会话：`get_stranger_conversations`（无参数）
- 昵称解析：`get_im_user_info`（`sec_uids` 列表）— 把消息里的 `sender_sec_uid` 换成昵称

### 3. 发送消息

所有发送类工具都需要 `to_user_id`（数字 uid），会**自动创建会话**：

| 目的 | 工具 | 参数 |
|---|---|---|
| 文本私信 | `send_dm` | `to_user_id`、`content` |
| 图片/视频/语音/文件 | `send_dm_media` | `to_user_id`、`kind`（image\|video\|audio\|file）、`path`（本地绝对路径） |
| 表情包 | `send_dm_sticker` | `to_user_id`、`sticker` |
| 卡片消息 | `send_dm_card` | `to_user_id`、`card` |
| 分享用户名片 | `send_dm_user_card` | `to_user_id`、`sec_uid` |
| 分享作品 | `share_dm_aweme` | `to_user_id`、`aweme_id` |
| 分享链接 | `share_dm_web` | `to_user_id`、`url` |

**发送前必须把内容展示给用户确认**（以用户身份发给真人，无法撤回）。
指定会话查询可用 `create_conversation` / `get_conversation_list`（`to_user_id`，可选 `conversation_short_id`）。

### 4. 群信息与群管理

- 群分享信息：`share_conversation`（`conversation`）— 群名、会话 id、分享链接与文案
- 标记已读：`mark_conversation_read`（`conversation`）
- 置顶 / 免打扰：`set_conversation_setting`（`conversation`，`pin`、`mute` 可只传一个）
- 退出群聊：`leave_conversation`（`conversation`）— ⚠️ 需确认

群管理 / 消息管理（都是写操作，**执行前必须确认**，需要群主或管理员权限）：

| 目的 | 工具 | 参数 |
|---|---|---|
| 改群名 / 群简介 / 群公告 | `update_conversation_name` | `conversation` 必填；`name`、`desc`、`notice` 按需传（不改的不传） |
| 把成员移出群 | `kick_conversation_participants` | `conversation`、`user_ids`（数字 uid 列表，取自 `get_conversation_participants`）均必填 |
| 撤回消息 | `recall_message` | `conversation`、`server_message_id`（取自 `get_conversation_history`；服务端限 **3 分钟内**） |
| 删除消息（仅自己侧） | `delete_message` | `conversation`、`message_id`（取自 `get_conversation_history`） |

### 5. 实时接收

1. `start_im_listen`（无参数）— 建立 WebSocket，收到的消息进入服务端缓冲
2. `get_im_messages`（`conversation_id`、`conversation_type` 可选过滤）— 取缓冲里的新消息
3. 结束监听：`stop_im_listen`（无参数）

实时缓冲是"启动监听之后"收到的消息；历史消息用 `get_conversation_history`。

### 6. 增量拉取与消息面板

| 目的 | 工具 | 参数 |
|---|---|---|
| 增量拉取私信/群消息 | `pull_im_messages` | `cursor`、`timestamp`（微秒，0 = 当前） |
| 互动/关注动态 | `get_im_spotlight_relation` | `conv_ids`（会话 id 数组，可为空） |
| 在线状态（批量） | `get_im_active_status` | `sec_user_ids`（列表） |
| 在线心跳 | `im_active_heartbeat` | `new_user_login`（默认 0）— 写操作 |
| 在线状态配置 | `get_im_active_config` | 无 |
| 互动资源策略配置 | `get_im_strategy_config` | `scenes`（默认 `["interactive_resources"]`） |
| 互动资源（自定义表情等） | `get_im_resources` | `scenes`（默认 `CUSTOM_STICKER_PAGE`）、`custom_cursor`、`custom_limit` |
| 热门表情 | `get_im_emoticon_trending` | `cursor`、`count`、`group_id` |
| 反馈入口 | `get_im_feedback_entrance` | `entrance`（默认 `IM6383-3586`） |

## 约束

- 只读（`get_*` / `list_conversations` / `share_conversation`）可直接执行；发送、标记已读、置顶免打扰、退群、监听启停需谨慎，发送类必须先确认内容
- 群管理（改群名、踢人）与消息管理（撤回、删除）影响他人可见状态，**必须先向用户确认目标群、目标成员/消息**；撤回仅限自己发出且 3 分钟内的消息
- **限流**：IM 接口容易返回 `409 too many requests`（浏览器端也在轮询同一批接口）。遇到时等待 20–60 秒再重试，不要连续重试
- 不要群发、不要对陌生人批量发消息（平台风控 + 骚扰）
- 监听是有状态长连接：用完必须 `stop_im_listen` / `stop_live_listen`，避免常驻占用

## 失败处理

| 场景 | 处理 |
|---|---|
| 未登录 | 引导使用 douyin-login |
| `409 too many requests` | 退避 20–60 秒后重试 |
| 找不到会话 | 用 `list_conversations` 确认名称/id（名称支持模糊匹配） |
| 发送失败 / 需要验证 | 展示错误；可能是对方未关注、账号受限或风控，建议稍后重试 |
| 未读数为 0 但用户认为有消息 | 说明"未读"以服务端计数为准（红点可能含其它来源），可改用普通翻页查看最近消息 |
| 消息里只有 uid 没有昵称 | 用 `get_im_user_info` 按 `sender_sec_uid` 批量解析 |
