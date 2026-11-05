# Reddit MCP：让 Claude、ChatGPT、Cursor 读 Reddit

[English](README.md)

接上一个托管的 MCP 地址，AI 就能搜 Reddit、读某个版块的最新帖子、读帖子下面的评论。**只读公开内容，不发帖。** 不用登记 Reddit 应用，不用 client ID，也不用你的 Reddit 账号。

这个仓库放的是配置方法、好用的问法和限制。服务由 SocialToAI 托管，这里没有需要安装的东西。SocialToAI 团队发布。

## 接入

地址：`https://mcp.socialtoai.com/mcp`

```bash
# Claude Code：运行后输入 /mcp，选 socialtoai，登录
claude mcp add --transport http socialtoai https://mcp.socialtoai.com/mcp

# Codex：浏览器会打开登录页
codex mcp add socialtoai --url https://mcp.socialtoai.com/mcp
```

```json
// Cursor：~/.cursor/mcp.json
{ "mcpServers": { "socialtoai": { "url": "https://mcp.socialtoai.com/mcp" } } }
```

Claude 网页版和桌面版：自定义 → 连接器 → 添加自定义连接器 → 粘贴地址。ChatGPT：设置 → 应用（开发者模式）→ 创建 → 粘贴地址，认证选 OAuth。其他客户端在请求头带 `Authorization: Bearer <YOUR_API_KEY>`，[在这里创建 Key](https://socialtoai.com/console/keys/?utm_source=github&utm_medium=repo&utm_campaign=reddit-mcp-20261106)。

登录时自动建账户，送 100 积分，不用绑卡。

## 哪个问题用哪个调用

Reddit 的搜索和版块列表是两个调用，选对了能省积分。

| 你想要 | 可以这样问 | 调用 | 积分（每页条数） |
| --- | --- | --- | --- |
| 全站提到某件事的帖子 | “搜一下 Reddit 最近一个月讲自建 RSS 阅读器的帖子” | `search`，配 `sort`、`published_within` | 2（约 7 条） |
| 某个版块的新帖 | “读 r/selfhosted 的最新帖子” | `call` `reddit.subreddit_posts`，`sort: latest` | 2（约 25 条） |
| 某个帖子下面大家怎么说 | “读这个帖子的评论，把吐槽归类” | `comments`，传链接或 `t3_` ID | 2（约 8 条） |
| 版块的规模和主题 | “r/selfhosted 有多大，主要聊什么？” | `call` `reddit.subreddit` | 2 |
| 某个账号发了什么 | “这个用户最近发了什么？” | `user_posts` | 2（约 25 条） |
| 今天的 Popular 或热搜词 | “Reddit 现在什么最热？” | `trending` | 2 |
| 视频帖的字幕 | “这个 Reddit 视频里说了什么？” | `detail` 带 transcript | 2 |

两个容易踩的点：

- **`search` 没有版块参数。** 想只看一个版块，用 `reddit.subreddit_posts` 再让 AI 过滤，或者在问题里写清版块，让 AI 核对每条结果发在哪个版块。
- **长帖要翻好几页。** 每页约 8 条评论，读 100 条约 13 次调用，26 积分。传 `limit: 100` 一次请求就能做完，用不到的页不收费。要总结多个帖子，现成工具 `comment_insights` 会先报价再跑，Reddit 的 small 档从 80 积分起。

## 一段真实录制

2026-10-18 在线上真实跑的，完整调用清单见[指南页](https://socialtoai.com/zh/docs/guides/read-reddit-and-x-for-product-complaints/?utm_source=github&utm_medium=repo&utm_campaign=reddit-mcp-20261106)。

> **问：** Reddit 和 X 上大家怎么吐槽升降桌？
>
> **调用：** Reddit `search`（7 条，2 积分）、X `search`（11 条，2 积分）、对讨论最多的 3 个 Reddit 帖子跑 `comment_insights`（173 条评论，84 积分）。
>
> **答案要点：** 能判断的 162 条评论里有 55 条在吐槽。腰背和脖子疼最多（12%，20 条），其次是忘记切换坐站（11%，18 条），然后是站久了脚疼（8%，13 条）。每一点都能点回原评论。
>
> **花费：** 88 积分，$0.29。

## 每天盯一个版块

对 AI 说“每天早上帮我看 r/selfhosted”。它会先告诉你跑什么、最多花多少，你同意才建。新帖可以发邮箱、发飞书、企业微信或 Slack 群，也可以用私有 RSS 地址放进阅读器。Reddit 每天跑一次 2 积分，一个版块一个月最多 62 积分（$10 档约 $0.21）。免费账户可以留 1 个监控，100 积分够跑约 50 次。按天或按周运行，不是实时。批量迁移原有订阅见 [reddit-rss-to-watch](https://github.com/socialtoai/reddit-rss-to-watch)。

## 费用

2026-11-05 查询 [`/v1/capabilities`](https://api.socialtoai.com/v1/capabilities)，价格版本 `pv_29229c6acba5`。这个地址不用 Key，随时可以查到当前价格。

| 计费项 | 积分 | $10 档折合 |
| --- | --- | --- |
| `reddit.search` | 2 | $0.0067 |
| `reddit.subreddit_posts` | 2 | $0.0067 |
| `reddit.comments` | 2 | $0.0067 |
| `reddit.detail` | 2 | $0.0067 |
| `reddit.user_posts` | 2 | $0.0067 |
| `reddit.trending` | 2 | $0.0067 |

$10 = 3,000 积分。失败不扣。正常查询但没有结果也算一次。每个回答都写明这次花了多少、还剩多少。

## 限制

- **只读。** 不能发帖、评论、投票或发消息。
- **只读公开内容。** 不碰登录后才能看的东西：没有私密版块、私信，也没有你自己的首页。
- **有些字段拿不到。** Reddit 搜索结果里，作者昵称、粉丝数、浏览、分享、收藏、转发都返回 `null`。AI 会把它们当成“没有提供”，不会当成 0。
- **数据的用途仍受 Reddit 条款约束。** 商业用途请对照 Reddit 自己的条款。

## Reddit 自己的路

如果你的应用要用 Reddit API，先走 Reddit：登记应用，读 [Data API Terms](https://redditinc.com/policies/data-api-terms)。据 2026-09-30 的报道（[TechCrunch](https://techcrunch.com/2026/09/30/reddit-is-killing-rss-feeds-ending-public-api-access-because-of-ai-bots/)），RSS 11-13 停止，未登记的应用 2027-01-12 后失去 API 访问，公开访问 2027 年 3 月结束。日期可能变动，以 Reddit 官方页面为准。我们整理的时间线和来源：[Reddit API 变更](https://socialtoai.com/zh/data/reddit-api-changes/?utm_source=github&utm_medium=repo&utm_campaign=reddit-mcp-20261106)。

## 链接

- [Reddit MCP 页面，含全部调用](https://socialtoai.com/zh/mcp/reddit/?utm_source=github&utm_medium=repo&utm_campaign=reddit-mcp-20261106)
- [全部 18 个平台](https://github.com/socialtoai/socialtoai-mcp)
- 支持：support@socialtoai.com

本仓库的文字和示例采用 MIT 许可。SocialToAI 团队发布。
