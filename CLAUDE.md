# CLAUDE.md

Skills 层不包含业务实现代码，所有功能通过调用 douyin-mcp 的 MCP 工具完成。

## MCP 工具映射表

类型说明：**ReadOnly** = 只读采集（可直接执行）；**Destructive** = 会改变账号状态或对外可见（执行前需用户确认）。

| MCP 工具 | 类型 | 对应 Skill | 说明 |
|---|---|---|---|
| `check_login_status` | ReadOnly | douyin-login | 检查登录状态 |
| `get_login_qrcode` | ReadOnly | douyin-login | 获取扫码登录二维码（Base64 + token） |
| `check_login_qrcode` | ReadOnly | douyin-login | 轮询扫码状态，成功后写 cookie |
| `send_phone_code` | Destructive | douyin-login | 发送手机号登录验证码 |
| `login_by_phone` | Destructive | douyin-login | 手机号 + 验证码登录 |
| `delete_cookies` | Destructive | douyin-login | 删除 cookies 重置登录 |
| `publish_content` | Destructive | douyin-publish | 发布图文作品 |
| `publish_with_video` | Destructive | douyin-publish | 发布视频作品 |
| `search_videos` | ReadOnly | douyin-search | 搜索视频（排序/时长/时间/范围/内容形式筛选） |
| `search_users` | ReadOnly | douyin-search | 搜索用户 |
| `search_challenges` | ReadOnly | douyin-search | 搜索话题/挑战 |
| `search_suggest` | ReadOnly | douyin-search | 搜索联想建议 |
| `search_lives` | ReadOnly | douyin-search | 搜索直播间 |
| `get_hot_search_board` | ReadOnly | douyin-search | 抖音热搜榜 |
| `get_hot_search_videos` | ReadOnly | douyin-search | 热搜词下的视频列表 |
| `get_homefeed` | ReadOnly | douyin-explore | 推荐流 |
| `get_video_detail` | ReadOnly | douyin-explore | 作品详情（无水印直链、封面、图文图片） |
| `get_video_comments` | ReadOnly | douyin-explore | 作品评论（单页） |
| `get_all_video_comments` | ReadOnly | douyin-explore | 作品全部一级评论（可选二级回复） |
| `get_sub_comments` | ReadOnly | douyin-explore | 某条评论的回复列表 |
| `get_channel_module_feed` | ReadOnly | douyin-explore | 精选频道模块流 |
| `get_course_category_tags` | ReadOnly | douyin-explore | 精选公开课分类标签 |
| `get_course_category_videos` | ReadOnly | douyin-explore | 精选公开课分类作品 |
| `get_solution_resources` | ReadOnly | douyin-explore | 精选页资源位 |
| `get_multicast_config` | ReadOnly | douyin-explore | 多播配置 |
| `page_turn_offline` | Destructive | douyin-explore | 页面下线开关上报 |
| `get_emoji_list` | ReadOnly | douyin-explore | 表情列表 |
| `get_publish_highlight` | ReadOnly | douyin-explore | 投稿入口角标 |
| `get_mix_list_collection` | ReadOnly | douyin-explore | 合集列表 |
| `get_seo_inner_link` | ReadOnly | douyin-explore | 页脚内链 |
| `get_study_notes` | ReadOnly | douyin-explore | 学习 AI 笔记列表 |
| `get_follow_feed` | ReadOnly | douyin-explore | 关注视频流 |
| `get_familiar_feed` | ReadOnly | douyin-explore | 朋友视频流 |
| `get_familiar_recommend_feed` | ReadOnly | douyin-explore | 朋友推荐流 |
| `get_follow_live_top` | ReadOnly | douyin-explore | 关注中的直播（顶栏） |
| `get_follow_live_feed` | ReadOnly | douyin-explore | 关注/朋友在播流 |
| `get_danmaku` | ReadOnly | douyin-explore | 视频弹幕 |
| `get_danmaku_conf` | ReadOnly | douyin-explore | 弹幕配置 |
| `report_history_write` | Destructive | douyin-explore | 上报播放历史 |
| `report_aweme_stats` | Destructive | douyin-explore | 上报播放统计 |
| `mark_following_seen` | Destructive | douyin-explore | 标记关注列表已读 |
| `report_series_watch` | Destructive | douyin-explore | 上报短剧观看进度 |
| `get_music_aweme` | ReadOnly | douyin-explore | 某个音乐下的视频 |
| `get_music_detail` | ReadOnly | douyin-explore | 音乐详情 |
| `get_user_info` | ReadOnly | douyin-user | 用户资料（昵称、粉丝数等） |
| `get_user_posts` | ReadOnly | douyin-user | 用户作品列表（单页，含播放直链） |
| `get_user_all_posts` | ReadOnly | douyin-user | 用户全部作品（自动翻页） |
| `get_user_followers` | ReadOnly | douyin-user | 用户粉丝列表（单页） |
| `get_all_user_followers` | ReadOnly | douyin-user | 用户全部粉丝（自动翻页） |
| `get_user_following` | ReadOnly | douyin-user | 用户关注列表（单页） |
| `get_all_user_following` | ReadOnly | douyin-user | 用户全部关注（自动翻页） |
| `get_my_profile` | ReadOnly | douyin-user | 自己的资料 |
| `get_user_dashboard` | ReadOnly | douyin-user | 创作者数据概览 |
| `get_social_count` | ReadOnly | douyin-user | 关注/粉丝/获赞计数 |
| `get_user_settings` | ReadOnly | douyin-user | 用户设置（`source=www\|hj`） |
| `get_custom_settings` | ReadOnly | douyin-user | 自定义设置 |
| `get_collected_awemes` | ReadOnly | douyin-user | 收藏的作品列表 |
| `baike_check_worldbook` | ReadOnly | douyin-user | 百科词书权限 |
| `baike_binding_subject` | ReadOnly | douyin-user | 百科绑定主体 |
| `digg_video` | Destructive | douyin-interact | 点赞/取消点赞作品 |
| `collect_video` | Destructive | douyin-interact | 收藏/取消收藏作品 |
| `post_video_comment` | Destructive | douyin-interact | 发表评论或回复评论 |
| `get_collect_list` | ReadOnly | douyin-library | 自己的收藏夹列表 |
| `get_user_favorites` | ReadOnly | douyin-library | 指定用户收藏的作品 |
| `move_collect_video` | Destructive | douyin-library | 移动收藏作品到指定收藏夹 |
| `remove_collect_video` | Destructive | douyin-library | 从收藏夹移除作品 |
| `get_watch_history` | ReadOnly | douyin-library | 观看历史 |
| `clear_watch_history` | Destructive | douyin-library | 清空观看历史 |
| `get_watch_later` | ReadOnly | douyin-library | 稍后再看列表 |
| `get_my_appointments` | ReadOnly | douyin-library | 我的预约 |
| `get_music_collection` | ReadOnly | douyin-library | 我收藏的音乐 |
| `collect_music` | Destructive | douyin-library | 收藏/取消收藏音乐 |
| `get_notices` | ReadOnly | douyin-notices | 消息通知列表（按分组） |
| `get_all_notices` | ReadOnly | douyin-notices | 全部消息通知（自动翻页） |
| `get_notice_count` | ReadOnly | douyin-notices | 通知未读数（各分区） |
| `get_notice_detail` | ReadOnly | douyin-notices | 单条通知详情 |
| `get_notice_digg_list` | ReadOnly | douyin-notices | 某条通知下的点赞用户 |
| `delete_notice` | Destructive | douyin-notices | 删除一条通知 |
| `list_conversations` | ReadOnly | douyin-im | 会话列表（含未读 `unread` / `unread_classes`） |
| `get_conversation_list` | ReadOnly | douyin-im | 查询单个私信会话信息 |
| `get_conversation_info` | ReadOnly | douyin-im | 会话详情（名称、群主、未读数） |
| `get_conversation_history` | ReadOnly | douyin-im | 聊天记录；`unread_only=true` 只返回未读消息 |
| `get_conversation_participants` | ReadOnly | douyin-im | 群成员列表 |
| `get_stranger_conversations` | ReadOnly | douyin-im | 陌生人会话列表 |
| `share_conversation` | ReadOnly | douyin-im | 群聊分享信息 |
| `get_im_user_info` | ReadOnly | douyin-im | 按 sec_uid 批量查昵称/头像 |
| `pull_im_messages` | ReadOnly | douyin-im | 增量拉取私信/群消息（protobuf cmd 2048） |
| `get_im_spotlight_relation` | ReadOnly | douyin-im | 消息面板「互动/关注动态」 |
| `get_im_active_status` | ReadOnly | douyin-im | 私信在线状态（批量） |
| `get_im_active_config` | ReadOnly | douyin-im | 在线状态配置 |
| `get_im_strategy_config` | ReadOnly | douyin-im | 互动资源策略配置 |
| `get_im_resources` | ReadOnly | douyin-im | 互动资源（自定义表情等） |
| `get_im_emoticon_trending` | ReadOnly | douyin-im | 热门表情 |
| `get_im_feedback_entrance` | ReadOnly | douyin-im | 反馈入口 |
| `get_im_messages` | ReadOnly | douyin-im | 已接收消息缓冲（按会话过滤） |
| `create_conversation` | Destructive | douyin-im | 创建私信会话 |
| `send_dm` | Destructive | douyin-im | 发送私信 |
| `send_dm_media` | Destructive | douyin-im | 发送图片/视频/语音/文件 |
| `send_dm_sticker` | Destructive | douyin-im | 发送表情包 |
| `send_dm_card` | Destructive | douyin-im | 发送卡片消息 |
| `send_dm_user_card` | Destructive | douyin-im | 分享用户名片 |
| `share_dm_aweme` | Destructive | douyin-im | 分享作品到会话 |
| `share_dm_web` | Destructive | douyin-im | 分享链接到会话 |
| `mark_conversation_read` | Destructive | douyin-im | 会话标记已读 |
| `set_conversation_setting` | Destructive | douyin-im | 置顶 / 免打扰 |
| `leave_conversation` | Destructive | douyin-im | 退出群聊 |
| `update_conversation_name` | Destructive | douyin-im | 改群名 / 群简介 / 群公告 |
| `kick_conversation_participants` | Destructive | douyin-im | 把成员移出群 |
| `recall_message` | Destructive | douyin-im | 撤回消息 |
| `delete_message` | Destructive | douyin-im | 删除消息 |
| `im_active_heartbeat` | Destructive | douyin-im | 在线心跳 |
| `start_im_listen` | Destructive | douyin-im | 启动私信实时接收（WebSocket） |
| `stop_im_listen` | Destructive | douyin-im | 停止私信实时接收 |
| `get_live_info` | ReadOnly | douyin-live | 直播间信息（主播、拉流地址） |
| `get_live_events` | ReadOnly | douyin-live | 已收集的直播间事件 |
| `get_live_rank_list` | ReadOnly | douyin-live | 直播间榜单列表 |
| `get_live_contribution_rank` | ReadOnly | douyin-live | 直播间贡献榜 |
| `get_live_thousand_ticket_rank` | ReadOnly | douyin-live | 千票榜 |
| `get_live_pk_rank` | ReadOnly | douyin-live | PK 排行榜 |
| `get_live_pk_contribution_rank` | ReadOnly | douyin-live | PK 贡献榜 |
| `get_live_pk_context` | ReadOnly | douyin-live | PK 上下文 |
| `get_live_linkmic_list` | ReadOnly | douyin-live | 连麦列表 |
| `get_live_production` | ReadOnly | douyin-live | 直播间带货商品列表 |
| `get_all_live_production` | ReadOnly | douyin-live | 直播间全部带货商品（自动翻页） |
| `get_live_production_detail` | ReadOnly | douyin-live | 直播间商品详情 |
| `get_product_comments` | ReadOnly | douyin-live | 商品评论 |
| `get_product_comment_counter` | ReadOnly | douyin-live | 商品评论统计 |
| `get_product_sku_list` | ReadOnly | douyin-live | 商品 SKU/规格 |
| `live_room_enter` | Destructive | douyin-live | 进入直播间（预热会话） |
| `start_live_listen` | Destructive | douyin-live | 启动直播间实时监听 |
| `stop_live_listen` | Destructive | douyin-live | 停止直播间监听 |
| `send_live_comment` | Destructive | douyin-live | 发送直播弹幕 |
| `like_live_room` | Destructive | douyin-live | 给直播间点赞 |

## SKILL.md 编写规范

每个 SKILL.md 包含 YAML frontmatter（name + description，可选 argument-hint）和 Markdown 正文。

正文必须包含：输入判断、约束条件、执行流程（含 MCP 工具调用与参数）、失败处理。

编写原则：
- 控制在 200 行以内
- Destructive 操作需用户确认
- 工具名和参数必须与 douyin-mcp 运行时 `tools/list` 一致
- 参数优先接受"id 或链接/名称"（多数工具两者都支持），不要编造参数

## 参考资源

- **douyin-mcp 源码**：`/Users/shiqilin/Repos/yunsur/mcp/douyin-mcp`（Go 版 HTTP + MCP 服务）
- **工具清单**：服务运行时 `POST /mcp` 的 `tools/list`（当前 133 个工具）
- **REST 对照**：`/api/v1/*`（README 有完整表格）
