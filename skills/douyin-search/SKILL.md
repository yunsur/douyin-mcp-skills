---
name: douyin-search
argument-hint: "[搜索关键词]"
description: |
  在抖音搜索内容：视频、用户、话题、直播间，以及搜索联想和热搜榜。
  当用户想在抖音上搜索、查找内容时使用——包括搜视频、找博主、找话题、看热搜、搜直播间、看看有没有某某内容等场景。
---

## 输入判断

- 关键词明确 → 按目标类型调用对应工具
- 只有关键词没说要什么 → 默认 `search_videos`（综合），必要时补充搜索用户/话题
- 想看热点 → `get_hot_search_board`
- 想看某个热搜词底下有哪些视频 → `get_hot_search_videos`（`hotword` + `sentence_id` 从热搜榜结果取）

## 执行流程

### 1. 搜索视频

调用 `search_videos`：

- `keyword`（string，必填）— 搜索关键词
- `count`（int，可选）— **自动翻页抓取数量**，>0 时忽略 `offset`，适合"给我 50 条"
- `offset`（string，可选）— 分页偏移，首次 `0`
- `channel`：`general`（综合，默认）| `video`（视频频道）
- `sort_type`：`0` 综合排序 | `1` 最多点赞 | `2` 最新发布
- `publish_time`：`0` 不限 | `1` 一天内 | `7` 一周内 | `180` 半年内
- `filter_duration`：空 不限 | `0-1` | `1-5` | `5-10000`（分钟）
- `search_range`：`0` 不限 | `1` 最近看过 | `2` 还未看过 | `3` 关注的人
- `content_type`：`0` 不限 | `1` 视频 | `2` 图文

### 2. 其他搜索类型

- 搜用户：`search_users`（`keyword` 必填，`offset`、`count`）
- 搜话题/挑战：`search_challenges`（`keyword` 必填，`offset`、`count`、`channel`、`sort_type`）
- 搜索联想：`search_suggest`（`keyword` 必填）— 用于确认关键词的常见写法
- 搜直播间：`search_lives`（`keyword` 必填，`offset`、`count`）
- 热搜榜：`get_hot_search_board`（无参数）— 返回每个热搜词及其 `sentence_id`
- 某个热搜词下的视频：`get_hot_search_videos`（`hotword` + `sentence_id` 必填，取自热搜榜结果；`offset`、`count`、`entry_name` 可选）

### 3. 展示结果

整理为列表展示，视频每条包含：

- 标题 / 文案、作者（昵称 + `sec_user_id`）、发布时间
- 互动数据：点赞、评论、收藏、分享
- **`aweme_id`**（后续详情、评论、点赞、收藏都要用）

用户可继续：查看作品详情（`douyin-explore`）、进入互动（`douyin-interact`）、看作者主页（`douyin-user`）。

## 约束

- 只读操作，可直接执行
- `aweme_id` / `sec_user_id` 必须来自搜索结果，不能编造
- 单次 `count` 不要过大（建议 ≤100），避免触发风控；大量采集分多次并留间隔

## 失败处理

| 场景 | 处理 |
|---|---|
| 未登录 | 引导使用 douyin-login 登录 |
| 无搜索结果 | 建议调整关键词或用 `search_suggest` 确认写法 |
| 高频请求被拦截（403 / 人机验证） | 等待一段时间再试，降低频率 |
| 结果为空但看起来应有内容 | 换 `channel` / `search_range` / `sort_type` 再试 |
