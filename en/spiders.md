# Spider Script Guide

> [中文](../spiders.md) · English

> Spiders crawl websites that offer no RSS. **The recommended route is to have an agent write them**: describe the target website on the "Agent" page, and the agent will write the script, check robots.txt, confirm with you against real data samples, and create the source directly (see the [Agent Guide](modules/agent.md)). This guide is for users who need to write or modify scripts themselves.

## Where Scripts Live

Spider scripts do not ship with the installation package; users manage them on their own, inside the data directory:

- macOS: `~/.doya/data/spiders/`
- Windows: `%USERPROFILE%\.doya\data\spiders\`

The program loads scripts **dynamically by file path**: after editing or adding a script, reload or restart for changes to take effect — **no reinstall needed**.

## How to Write One

A script is a single Python file whose **filename is the spider name** (e.g., `video_site.py`). Basic skeleton:

```python
from scrapling.spiders import Response

from core.spider.base_spider import BaseSpider


class VideoSiteSpider(BaseSpider):
    name = "video_site"          # 必须与文件名(去 .py)完全一致
    start_urls = ["https://example.com/list"]

    concurrent_requests = 1      # 低并发,避免触发目标站限制
    download_delay = 3.0         # 抓取间隔(秒)

    async def parse(self, response: Response):
        channel = self._build_channel(response)  # 渠道元数据,开头建一次
        for elem in response.css("列表项CSS选择器"):
            url = response.urljoin(elem.css("a::attr(href)").get(""))
            title = elem.css("a::text").get("").strip()
            if not title or not url:
                continue          # 跳过必填字段缺失的条目
            yield self._build_item_dict(
                channel=channel,
                title=title,
                url=url,
                description=elem.css(".desc::text").get("").strip(),
                guid=url,         # 唯一标识,常用 URL;增量去重靠它
                author=elem.css(".author::text").get(""),
            )
```

### Interface Specification

| Element | Description |
| ---- | ---- |
| Base class | Inherit from `BaseSpider` (built into the program; see the import in the example above — nothing to install) |
| `name` | Class attribute serving as the registration key; **must match the filename** — a mismatch makes the spider unfindable at runtime |
| `start_urls` | List of URLs to start crawling from |
| `parse(self, response)` | Core method: parses the page and `yield`s items one by one; `response.css(...)` is a CSS selector (`::text` extracts text, `::attr(...)` extracts attributes); `response.urljoin(...)` completes relative links into absolute URLs |
| `_build_item_dict(...)` | Item builder with fields `channel` / `title` / `url` / `description` / `guid` / `author`; **`title`, `url`, and `guid` are required** (skip items that miss them); **publish time is not collected** (date information on list pages is not stored) |
| `_build_channel(response)` | Builds channel metadata; call it once at the top of `parse` and pass the result into `_build_item_dict` |
| `concurrent_requests` / `download_delay` | Concurrency and crawl interval, controlling the load put on the target site |

### Session Tiers (Upgrade as Needed)

Without `configure_sessions`, the default is a **plain HTTP session** — fast, zero extra dependencies, and enough for most sites. Upgrade to the **browser tier** only when page content needs JavaScript rendering (override `configure_sessions` to build a scrapling dynamic session); chromium already ships with the installation package, so there is nothing to install. For concrete usage and tier selection, refer to <placeholder: author-provided templates / detailed reference docs>.

### Secrets and Tokens

When the target site requires a token / API key / Cookie, **do not write it into the script file** — pass it through "Extra Parameters" when creating the source (the "Add Spider" dialog in the frontend, or let the agent configure it), and read it inside the script as follows:

```python
def configure_sessions(self, manager):
    api_token = self.extra.get("api_token")   # 令牌
    cookies = self.extra.get("cookies")       # Cookie(仅浏览器档支持)
    headers = self.extra.get("headers")       # 自定义请求头
```

### Testing

- **Prefer having the agent verify**: on the "Agent" page, ask the agent to test-crawl your script; it runs the script for you and shows the crawled samples
- If you have a Python environment on your machine, append an `if __name__ == "__main__":` block at the end of the script to run a standalone preview (see the placeholder link above for template references)

With the script in place, go to Settings → Source Management and click "**Add Spider**" to create a source (field reference in [Feed Source Configuration](modules/rss.md)); or simply let the agent create the source.

## Dependency Libraries

Local libraries a spider depends on (such as the signing-script library `_vendor/`) go under `data/spiders/`, next to the script; scripts resolve them relative to their own location. For example:

```
data/spiders/
├── video_site.py              # spider 脚本
└── _vendor/                   # 依赖库(Node + jsdom 签名)
    ├── sign_tool/
    │   ├── client.py
    │   └── sign_server.js
    └── node_modules/
```

## Runtime Requirements

- **Browser spiders** (sites whose pages need rendering): chromium ships with the installation package and works out of the box (auto-extracted on first launch, no installation)
- **Spiders needing Node.js** (e.g., sites that require running signing scripts locally): install Node.js yourself and make sure the `node` command is available

## Related

- Creating sources and source management (frontend operations) → [Feed Source Configuration](modules/rss.md)
- Having the agent write spiders and create sources → [Agent Guide](modules/agent.md)
