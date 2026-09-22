# Spider 爬虫脚本指南

> [English](en/spiders.md) · 中文

> Spider 用于抓取没有 RSS 的网站。**推荐让 agent 代写**：在「代理」页描述目标网站，agent 会编写脚本、核查 robots.txt、用真实数据样本与你确认后直接建源（见 [Agent 使用指南](modules/agent.md)）。本指南面向需要自己编写或修改脚本的用户。

## 脚本放哪

spider 脚本不随安装包分发，由用户自行管理，放数据目录：

- macOS:`~/.doya/data/spiders/`
- Windows:`%USERPROFILE%\.doya\data\spiders\`

程序按**文件路径动态加载**，改/加脚本后重载或重启生效，**无需重装程序**。

## 怎么写

脚本为单个 Python 文件，**文件名即 spider 名**（如 `video_site.py`），基本骨架：

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

### 接口规范

| 要素 | 说明 |
| ---- | ---- |
| 基类 | 继承 `BaseSpider`（随程序内置，import 写法见上例，无需自行安装） |
| `name` | 类属性，注册键；**必须与文件名一致**，不一致会导致运行时找不到该 spider |
| `start_urls` | 起始抓取地址列表 |
| `parse(self, response)` | 核心方法：解析页面，逐条 `yield` 数据；`response.css(...)` 为 CSS 选择器（`::text` 取文本、`::attr(...)` 取属性），`response.urljoin(...)` 把相对链接补全为绝对地址 |
| `_build_item_dict(...)` | 数据条目构造器，字段：`channel` / `title` / `url` / `description` / `guid` / `author`；**`title`、`url`、`guid` 必填**（缺失条目应跳过）；**不采集发布时间**（列表页上的日期信息不入库） |
| `_build_channel(response)` | 渠道元数据构造，`parse` 开头调用一次，传入 `_build_item_dict` |
| `concurrent_requests` / `download_delay` | 并发数与抓取间隔，控制对目标站的压力 |

### 会话档位（按需升级）

不写 `configure_sessions` 时默认为**纯 HTTP 会话**——速度快、零额外依赖，大多数站点足够。页面内容需 JavaScript 渲染时才升级**浏览器档**（覆写 `configure_sessions` 构造 scrapling 动态会话）；chromium 已随安装包内置，无需另装。具体写法与选型参考：<占位：作者提供的模板 / 详细参考文档>。

### 密钥与令牌

目标站需要令牌 / API Key / Cookie 时，**不写进脚本文件**——通过建源时的「额外参数」传入（前端「添加 Spider」弹窗，或让 agent 配置），脚本内以下写法读取：

```python
def configure_sessions(self, manager):
    api_token = self.extra.get("api_token")   # 令牌
    cookies = self.extra.get("cookies")       # Cookie(仅浏览器档支持)
    headers = self.extra.get("headers")       # 自定义请求头
```

### 测试

- **推荐交给 agent 验证**：在「代理」页让 agent 试爬你写的脚本，它会代为运行并展示抓取样本
- 本机装有 Python 环境的，可在脚本尾部添加 `if __name__ == "__main__":` 直跑块独立执行预览（参考模板见上方占位链接）

脚本就位后，到【设置 → 源管理】点「**添加 Spider**」建立采集源（字段说明见[采集源配置](modules/rss.md)）；也可以直接让 agent 完成建源。

## 依赖库

spider 依赖的本地库（如签名脚本库 `_vendor/`）放在 `data/spiders/` 下与脚本同级，脚本按自身位置定位引用。例如：

```
data/spiders/
├── video_site.py              # spider 脚本
└── _vendor/                   # 依赖库(Node + jsdom 签名)
    ├── sign_tool/
    │   ├── client.py
    │   └── sign_server.js
    └── node_modules/
```

## 运行时依赖

- **浏览器类 spider**（需渲染页面的网站）：chromium 已随安装包内置，开箱即用（首次启动自动解压，无需安装）
- **需要 Node.js 的 spider**（如需本地执行签名脚本的网站）：需自行安装 Node.js，并确保 `node` 命令可用

## 相关

- 建源与源管理（前端操作）→ [采集源配置](modules/rss.md)
- 让 agent 代写 spider 与建源 → [Agent 使用指南](modules/agent.md)
