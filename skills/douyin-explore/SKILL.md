---
name: douyin-explore
description: |
  浏览抖音推荐流、查看作品详情和评论（含二级回复）。
  当用户想刷推荐、看首页内容、查看某个视频/图文的详情或评论、或已拿到 aweme_id/链接想获取完整内容时使用。
---

## 输入判断

- 用户想刷推荐 → 步骤 1
- 用户给了 `aweme_id` 或作品链接 → 步骤 2
- 用户想看评论 / 某条评论的回复 → 步骤 3
- 想看关注/朋友/精选/合集，或导出弹幕、看某音乐下的视频 → 步骤 4 / 5

## 执行流程

### 1. 推荐流

调用 `get_homefeed`：

- `count`（string，可选）— 数量，默认 20
- `refresh_index`（string，可选）— 刷新索引，默认 2（换一批用更大值）

展示每条作品的文案、作者、互动数据，并给出 `aweme_id` 供后续操作。

### 2. 作品详情

调用 `get_video_detail`：

- `video`（string，必填）— 作品 `aweme_id` **或**作品链接（`https://www.douyin.com/video/...`）

返回含：文案、作者、发布时间、互动数据、**无水印播放直链**、封面；图文作品返回图片直链列表。

### 3. 评论

- 单页评论：`get_video_comments`（`video` 必填，`cursor`、`count` 翻页）
- 全部一级评论：`get_all_video_comments`
  - `video`（必填）
  - `limit`（int，可选）— 最多取多少条一级评论，`0` 表示全部
  - `include_replies`（bool，可选）— 是否同时抓取二级回复
- 某条评论的回复：`get_sub_comments`（`video`、`comment_id` 必填，`cursor`、`count` 翻页）

展示评论时包含：评论内容、作者、点赞数、发布时间、`comment_id`（回复时要用）。

### 4. 精选 / 推荐 / 关注 / 朋友 tab

除推荐流外，这些页签各有独立接口（多数参数可选、带合理默认值，直接调用即可）：

| 目的 | 工具 | 必填参数 |
|---|---|---|
| 精选频道模块流 | `get_channel_module_feed` | 无（`module_id` 默认 3003101，`refresh_index` 换一批） |
| 关注视频流 | `get_follow_feed` | 无（`cursor`、`count`） |
| 朋友视频流 | `get_familiar_feed` | 无 |
| 朋友推荐流 | `get_familiar_recommend_feed` | 无 |
| 关注中的直播（顶栏） | `get_follow_live_top` | 无 |
| 关注/朋友在播流 | `get_follow_live_feed` | 无 |
| 公开课分类 | `get_course_category_tags` / `get_course_category_videos` | 无（`offset`、`size` 翻页） |
| 精选页资源位 / 多播配置 | `get_solution_resources` / `get_multicast_config` | 无 |
| 合集列表 | `get_mix_list_collection` | 无（`cursor`、`count`） |
| 表情列表 | `get_emoji_list` | 无 |
| 投稿入口角标 | `get_publish_highlight` | 无 |
| 页脚内链 / 学习笔记 | `get_seo_inner_link` / `get_study_notes` | 无 |

### 5. 弹幕与音乐页

- 视频弹幕：`get_danmaku`（`item_id` 必填；`start_time`、`end_time`、`duration`、`authentication_token` 可选）— 用于导出某条视频的弹幕
- 弹幕配置：`get_danmaku_conf`（无参数）— 拿到 `authentication_token`
- 某音乐下的视频：`get_music_aweme`（`music_id` 必填，`cursor`、`count`）
- 音乐详情：`get_music_detail`（`music_id` 必填）

播放埋点类工具（`report_history_write`、`report_aweme_stats`、`mark_following_seen`、`report_series_watch`、`page_turn_offline`）会向服务端写入上报数据，**只在用户明确要求复现上报行为时调用**，日常浏览不要调用。

## 约束

- 只读操作，可直接执行（除上面列出的上报/埋点类写操作）
- `limit=0` / 大数量抓取会翻很多页，先告知用户预计耗时与量级，避免无意义的大量请求
- 作品链接和 `aweme_id` 二选一即可，不要同时编造

## 失败处理

| 场景 | 处理 |
|---|---|
| 未登录 | 引导使用 douyin-login |
| 作品已删除 / 不可见 / 需要权限 | 告知用户该作品无法访问 |
| 评论为空 | 可能已关闭评论或确无评论，如实告知 |
| 翻页卡住或重复 | 停止抓取，返回已获取部分并说明 |
| 人机验证 / 403 | 降低频率，等待后重试 |
