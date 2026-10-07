---
name: douyin-library
description: |
  查看和管理抖音个人内容库：收藏夹、收藏的作品、观看历史、稍后再看、我的预约。
  当用户提到收藏、收藏夹、我的收藏、观看历史、稍后再看、预约等与个人内容库相关的操作时使用。
---

## 输入判断

- 看收藏夹结构 → 步骤 1
- 看自己/他人收藏的作品 → 步骤 2
- 整理收藏（移动/移除） → 步骤 3
- 历史 / 稍后再看 / 预约 → 步骤 4
- 音乐收藏 → 步骤 5

## 执行流程

### 1. 收藏夹列表

调用 `get_collect_list`（无参数）。返回自己的收藏夹：名称 + **`collect_id`**（移动/移除作品时必填）。

### 2. 收藏的作品

调用 `get_user_favorites`：

- `sec_user_id`（string，必填）— 要查询的用户 `sec_user_id`（查自己时用 `check_login_status` 返回的 sec_uid）
- `max_cursor`（string，可选）— 分页游标，首次 `0` 或留空
- `count`（string，可选）— 每页数量，默认 18

展示：文案、作者、互动数据、`aweme_id`。

### 3. 整理收藏

- 移动到指定收藏夹：`move_collect_video`（`aweme_id`、`collect_name`、`collect_id` 均必填）
- 从收藏夹移除：`remove_collect_video`（`aweme_id`、`collect_name`、`collect_id` 均必填）

`collect_name` 与 `collect_id` 都来自步骤 1 的 `get_collect_list`，不要编造。

### 4. 历史 / 稍后再看 / 预约

| 需求 | 工具 | 参数 |
|---|---|---|
| 观看历史 | `get_watch_history` | `cursor`（string，`"0"` = 第一页）、`offset`（string）、`count`（string，默认 `"20"`） |
| 清空观看历史 | `clear_watch_history` | 无参数（⚠️ 不可恢复，需确认） |
| 稍后再看 | `get_watch_later` | `cursor`（string）、`offset`（string）、`count`（string，默认 `"20"`） |
| 我的预约 | `get_my_appointments` | `appointment_type`（**string**，默认 `"100"`）、`count`（**int**，默认 `-1` = 全部） |

### 5. 音乐收藏

| 需求 | 工具 | 参数 |
|---|---|---|
| 我收藏的音乐 | `get_music_collection` | `cursor`（默认 0）、`count`（默认 20） |
| 收藏 / 取消收藏音乐 | `collect_music` | `music_id` 必填；`action`：`1` 收藏（默认）\| `0` 取消 |

## 约束

- 查询类（`get_*`）只读，可直接执行
- `clear_watch_history`、`move_collect_video`、`remove_collect_video` 改变账号数据，执行前必须确认；`clear_watch_history` 不可恢复
- 收藏夹的 `collect_id` 必须先取，不能猜

## 失败处理

| 场景 | 处理 |
|---|---|
| 未登录 | 引导使用 douyin-login |
| 收藏夹列表为空 | 说明账号暂无收藏夹，建议先创建（客户端操作） |
| `collect_id` / `collect_name` 不匹配 | 重新调用 `get_collect_list` 取准确值 |
| 历史/稍后再看为空 | 可能确实为空，或需要先在 App 打开对应 tab；如实告知 |
| 频繁请求被限流 | 降低频率，等待后重试 |
