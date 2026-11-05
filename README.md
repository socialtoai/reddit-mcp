# Reddit MCP server: read Reddit from Claude, ChatGPT or Cursor

[中文](README.zh.md)

Connect one hosted MCP address and your AI can search Reddit, read a subreddit's newest posts, and read the comments under a thread. **It reads public content and does not post.** There is no Reddit app to register, no client ID, and no Reddit account of yours involved.

This repository holds the setup, the prompts that work and the limits. The server is hosted by SocialToAI, so there is nothing here to install. Published by the SocialToAI team.

## Connect

Address: `https://mcp.socialtoai.com/mcp`

```bash
# Claude Code: run this, then type /mcp, pick socialtoai and sign in
claude mcp add --transport http socialtoai https://mcp.socialtoai.com/mcp

# Codex: the browser opens for sign-in
codex mcp add socialtoai --url https://mcp.socialtoai.com/mcp
```

```json
// Cursor: ~/.cursor/mcp.json
{ "mcpServers": { "socialtoai": { "url": "https://mcp.socialtoai.com/mcp" } } }
```

Claude (web, desktop): Customize → Connectors → Add custom connector → paste the address. ChatGPT: Settings → Apps (developer mode) → Create → paste the address, authentication OAuth. Other clients send `Authorization: Bearer <YOUR_API_KEY>` in a header; [create a key here](https://socialtoai.com/console/keys/?utm_source=github&utm_medium=repo&utm_campaign=reddit-mcp-20261106).

Signing in creates the account with 100 free credits, no card.

## Which call answers which question

Reddit's search and a subreddit's listing are different calls. Picking the right one saves credits.

| You want | Ask something like | Call | Credits (page size) |
| --- | --- | --- | --- |
| Mentions across all of Reddit | "Search Reddit for posts about self-hosted RSS readers from the last month" | `search`, `sort`, `published_within` | 2 (about 7 posts) |
| What is new in one community | "Read the newest posts in r/selfhosted" | `call` `reddit.subreddit_posts`, `sort: latest` | 2 (about 25 posts) |
| What people say under one thread | "Read the comments on this thread and group the complaints" | `comments`, by link or `t3_` id | 2 (about 8 comments) |
| Size and rules of a community | "How big is r/selfhosted and what is it about?" | `call` `reddit.subreddit` | 2 |
| What one account posts | "What has this user posted lately?" | `user_posts` | 2 (about 25 posts) |
| Today's Popular feed, or trending searches | "What is trending on Reddit right now?" | `trending` | 2 |
| The captions of a video post | "What is said in this Reddit video?" | `detail` with the transcript include | 2 |

Two things catch people out:

- **`search` has no subreddit parameter.** To stay inside one community, read `reddit.subreddit_posts` and let your AI filter, or name the community in your question and have your AI check where each result was posted.
- **Long threads take several pages.** At about 8 comments a page, reading 100 comments is about 13 calls, or 26 credits. `limit: 100` does this in one request, and pages it does not need cost nothing. For a summary of several threads, the ready-made `comment_insights` tool quotes its price before it runs; on Reddit its small scale starts at 80 credits.

## A recorded answer

Recorded on the live service on 2026-10-18, published with the full call list at [the guide page](https://socialtoai.com/docs/guides/read-reddit-and-x-for-product-complaints/?utm_source=github&utm_medium=repo&utm_campaign=reddit-mcp-20261106).

> **Asked:** What do people complain about with standing desks on Reddit and X?
>
> **Calls:** `search` on Reddit (7 posts, 2 credits), `search` on X (11 posts, 2 credits), `comment_insights` on the 3 most-discussed Reddit threads (173 comments, 84 credits).
>
> **Answer, in short:** 55 of the 162 comments that could be judged complain about something. Back and neck pain leads (12%, 20 comments), then forgetting to switch between sitting and standing (11%, 18), then sore feet (8%, 13). Each point links to the comment behind it.
>
> **Spent:** 88 credits, $0.29.

## Follow a subreddit every day

Say "watch r/selfhosted every morning" to your AI. It shows what will run and at most what it costs, and makes the watch only after you agree. New posts arrive by email, in a Feishu, WeCom or Slack chat, or as a private RSS feed for your reader. Each daily run on Reddit is 2 credits, so one subreddit is at most 62 credits a month ($0.21 on the $10 pack). A free account keeps one watch, and 100 free credits cover about 50 runs. It runs daily or weekly, so it is not real time. Moving existing feeds in bulk: [reddit-rss-to-watch](https://github.com/socialtoai/reddit-rss-to-watch).

## Credits

Checked on 2026-11-05 against [`/v1/capabilities`](https://api.socialtoai.com/v1/capabilities), price version `pv_29229c6acba5`. That address needs no key and always has the current numbers.

| Price key | Credits | On the $10 pack |
| --- | --- | --- |
| `reddit.search` | 2 | $0.0067 |
| `reddit.subreddit_posts` | 2 | $0.0067 |
| `reddit.comments` | 2 | $0.0067 |
| `reddit.detail` | 2 | $0.0067 |
| `reddit.user_posts` | 2 | $0.0067 |
| `reddit.trending` | 2 | $0.0067 |

$10 buys 3,000 credits. Failed calls are free. A search that ran and found nothing is charged. Every answer says what it cost and what is left.

## Limits

- **Read only.** It cannot post, comment, vote or send messages.
- **Public content only.** Nothing behind a login: no private communities, no messages, no home feed of your own.
- **Some fields are not given.** In Reddit search results, author display name, follower count, views, shares, saves and reposts come back as `null`. Your AI is told to read them as "not given", never as zero.
- **Reddit's rules still apply to what you do with the data.** Commercial use of Reddit content is governed by Reddit's own terms; check them for your case.

## Reddit's own route

If you run an app that needs Reddit's API, start with Reddit: register the app and read the [Data API Terms](https://redditinc.com/policies/data-api-terms). As reported on 2026-09-30 ([TechCrunch](https://techcrunch.com/2026/09/30/reddit-is-killing-rss-feeds-ending-public-api-access-because-of-ai-bots/)), RSS feeds stop on 2026-11-13, unregistered apps lose API access after 2027-01-12, and public access ends in March 2027. Dates can move; Reddit's own pages are the authority. Our notes with sources: [Reddit API changes](https://socialtoai.com/data/reddit-api-changes/?utm_source=github&utm_medium=repo&utm_campaign=reddit-mcp-20261106).

## Links

- [Reddit MCP page, with every call](https://socialtoai.com/mcp/reddit/?utm_source=github&utm_medium=repo&utm_campaign=reddit-mcp-20261106)
- [All 18 platforms](https://github.com/socialtoai/socialtoai-mcp)
- Support: support@socialtoai.com

MIT licence for the text and examples in this repository. Published by the SocialToAI team.
