# douyin-mcp-skills

基于 [douyin-mcp](https://github.com/yunsur/douyin-mcp)（Go 版 HTTP + MCP 双协议服务）的 Agent Skills 集合，为抖音提供完整的自动化操作能力：数据采集、互动、私信收发、直播间监听、创作者发布。兼容 [Agent Skills 开放标准](https://agentskills.io)，支持 OpenClaw、Claude Code 等平台。

## 功能

| Skill | 说明 |
|---|---|
| **setup-douyin-mcp** | 安装部署 douyin-mcp 服务并配置 MCP 连接 |
| **douyin-login** | 登录管理（扫码 / 手机号验证码、状态检查、重置登录） |
| **douyin-publish** | 发布图文 / 视频作品到创作者中心 |
| **douyin-search** | 搜索视频 / 用户 / 话题 / 直播 / 热搜 / 联想建议 |
| **douyin-explore** | 浏览推荐流、作品详情（含水印直链）、评论与二级回复 |
| **douyin-interact** | 互动操作（点赞、收藏、发表评论、回复评论） |
| **douyin-user** | 查看用户主页、作品列表、粉丝 / 关注 |
| **douyin-library** | 收藏夹、观看历史、稍后再看、我的预约 |
| **douyin-notices** | 消息通知（未读数、详情、删除） |
| **douyin-im** | 私信与群聊（会话、未读消息、收发媒体、实时接收） |
| **douyin-live** | 直播间（实时监听、弹幕、榜单、带货商品） |
| **douyin-content-plan** | 内容策划（热点分析、对标研究、选题建议） |

## 前置条件

- 已安装并运行 [douyin-mcp](https://github.com/yunsur/douyin-mcp) 服务（默认 `http://127.0.0.1:18080/mcp`）
- 已完成抖音登录（Cookie 或通过 `douyin-login` 扫码 / 验证码登录）
- 如未安装，使用 `setup-douyin-mcp` skill 引导完成

## 安装

### OpenClaw

下载本项目到本地后，解压到 OpenClaw 的 SKILLS 目录下重启会话生效。

### Claude Code

```bash
# 项目级别
cp -r skills/ .claude/skills/

# 或全局级别
cp -r skills/ ~/.claude/skills/
```

### omp

把本仓库的 `skills/` 目录加入 omp 的技能搜索路径，重启会话后生效：

```yaml
skills:
  customDirectories:
    - /绝对路径/douyin-mcp-skills/skills
```

根 `SKILL.md` 可作为统一入口使用，也可以直接使用各子 skill。

## 使用示例

```
# 首次使用：安装 MCP 服务和登录
/setup-douyin-mcp
/douyin-login

# 搜索与浏览
/douyin-search 九号电动车 mz5
/douyin-explore

# 发布内容
/douyin-publish

# 互动
/douyin-interact

# 私信与未读
/douyin-im 看下 Mz5 群未读

# 直播间
/douyin-live 监听 123456789 的弹幕
```

## 许可证

MIT
