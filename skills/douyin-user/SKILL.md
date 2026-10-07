---
name: douyin-user
description: |
  查看抖音用户资料与作品：昵称、简介、粉丝/关注/获赞数据、作品列表、粉丝与关注列表（支持全量翻页）。
  当用户想看某个博主、作者、用户的主页信息、作品或粉丝关注时使用。
---

## 输入判断

- 有 `sec_user_id` 或主页链接 → 直接查询
- 只有昵称 → 先用 `search_users` 找到用户，拿到 `sec_user_id` 再查
- 看自己的资料 / 设置 / 收藏的作品 / 创作者数据 → 步骤 4

## 执行流程

### 1. 用户资料

调用 `get_user_info`：

- `user`（string，必填）— 用户 `sec_user_id` **或**主页链接（`https://www.douyin.com/user/...`）

返回：昵称、头像、简介、认证信息、粉丝数、关注数、获赞数、作品数。

### 2. 作品列表

- 单页：`get_user_posts`（`user` 必填，`max_cursor` 翻页游标，`count`）— 含播放直链
- 全量：`get_user_all_posts`（`user` 必填，`limit` 最多返回数量，`0` 表示全部，自动翻页）

展示每条：文案、发布时间、互动数据、`aweme_id`。

### 3. 粉丝 / 关注

| 需求 | 工具 | 参数 |
|---|---|---|
| 粉丝（单页） | `get_user_followers` | `user`、`max_time`、`count` |
| 粉丝（全量） | `get_all_user_followers` | `user`、`limit`（0 = 全部） |
| 关注（单页） | `get_user_following` | `user`、`max_time`、`count` |
| 关注（全量） | `get_all_user_following` | `user`、`limit`（0 = 全部） |

展示昵称、`sec_user_id`、粉丝数（若有）、是否互关（若返回）。

### 4. 自己的资料、设置与收藏

| 目的 | 工具 | 参数 |
|---|---|---|
| 自己的资料 | `get_my_profile` | 无 |
| 创作者数据概览 | `get_user_dashboard` | 无 |
| 关注/粉丝/获赞计数 | `get_social_count` | 无 |
| 用户设置 | `get_user_settings` | `source`：`www`（默认）\| `hj` |
| 自定义设置 | `get_custom_settings` | 无 |
| 自己收藏的作品 | `get_collected_awemes` | `max_cursor`、`count` |
| 百科词书权限 / 绑定主体 | `baike_check_worldbook` / `baike_binding_subject` | 无（`account_uid` 默认当前用户） |

## 约束

- 只读操作，可直接执行
- `sec_user_id` 必须来自搜索结果或主页链接，不能编造 `MS4wLjABAAAA...`
- 全量抓取（`limit=0`）在大账号上会非常慢且请求量大：先说明规模，必要时先取前 N 条
- 查询他人粉丝/关注属于敏感数据，仅按用户明确要求执行，不用于骚扰或批量营销

## 失败处理

| 场景 | 处理 |
|---|---|
| 未登录 | 引导使用 douyin-login |
| 用户不存在 / 已注销 | 告知用户主页不可访问 |
| 隐私设置不可见 | 如实说明该用户未公开粉丝/关注列表 |
| 只有昵称无法定位 | 用 `search_users` 列出候选让用户确认 |
| 频繁请求被限流 | 降低频率，等待后重试 |
