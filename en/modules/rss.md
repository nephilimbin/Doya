# Feed Source (RSS / Spider) Configuration Guide

> [中文](../../modules/rss.md) · English

> Doya gathers information from two kinds of sources: **RSS subscriptions** and **Spider crawlers**. Adding and maintaining them happens in Settings → Source Management; fetch cadence and filter/parse behavior are tuned in Settings → Source Behavior.

> [!TIP]
> Operations like adding sources are best delegated to the agent: open the "Agent" page in the left navigation and type "help me add the RSS for xxx". See the [Agent Guide](agent.md). Manual configuration is described below.

## Add an RSS Source

1. Settings → Source Management → click "**Add RSS**"
2. Fill in:

| Field | Description |
| ---- | ---- |
| Name | Unique identifier of the source; must not duplicate another source |
| Feed URL | RSS subscription address (starting with `http(s)://`) |
| Website URL | Homepage of the site, used to generate OPML |
| Category | Dropdown of existing categories (categories are created in "Manage Categories") |
| Tags | Comma-separated, optional |
| Cron expression | Fetch cadence, 5-field; default `0 */12 * * *` (every 12 hours) |
| Fetch limit | Max items per fetch, default 999 |
| Enable / Filter / Parse | Three switches, all on by default |

3. After saving, the source enters the list and is fetched automatically on its Cron; you can also trigger a check manually next to the "Enable" switch in its row.

## Add a Spider Source

Spiders are for sites without RSS (requires writing a Python crawler script).

1. First put the spider script (`.py`) into the data directory `~/.doya/data/spiders/` (how to write scripts and templates: see the [Spider Management Guide](../spiders.md))
2. Settings → Source Management → click "**Add Spider**", and besides the common fields, fill in:

| Field | Description |
| ---- | ---- |
| Entry file | Spider script file name (e.g. `my_spider.py`) |
| URL template | Address with `{key}` placeholders, e.g. `https://api.x.com/{cat}/list` |
| POST parameters | Values that fill the placeholders (JSON), default `{}` |
| Extra parameters | Parameters passed to the script (JSON, e.g. cookie, api_token), default `{}` |

> [!TIP]
> The dialog reminds you too: adding sources via the agent (skill tool) is recommended — give the target URL to the agent, and it will write the script, check robots.txt, confirm a data sample, and create the source for you.

## Manage Existing Sources

- **Enable/disable**: the switch in the "Enable" column toggles on click; select multiple rows then use "Enable Selected / Disable Selected"
- **Delete**: delete a single source or select rows then "Delete Selected"; the dialog offers two choices:
  - **Delete source only (unsubscribe)** — historical articles are kept
  - **Source and articles** — cascading delete of fetches, filter records and all articles; irreversible
- **Categories**: "Manage Categories" supports add/rename/delete; when a category is deleted, its sources move to `unknown`
- **Find**: type filter (All/RSS/Spider), search box, and the "Displayed Fields" panel (choose columns and their order)
- **Health**: a red exclamation mark in the "Status" column means the latest fetch failed; hover for the reason; hovering the "Cron" column shows the raw expression

## Source Behavior (Filter / Parse / Scheduler)

Settings → Source Behavior controls fetch and processing behavior for all sources, in three groups:

### Filter

Politeness and concurrency parameters at fetch time; applied automatically after saving (no restart needed):

| Parameter | Default | Description |
| ---- | ---- | ---- |
| Enable filtering | On | Turning it off prompts "also pause parsing" |
| Content extraction algorithm | readability | Three choices: markitdown (whole-page conversion, lightest) / trafilatura (pure-Python body extraction) / readability (Mozilla reader view, requires node) |
| Download delay (seconds) | 3.0 | Polite interval toward the site |
| Max concurrency / per-domain concurrency | 10 / 1 | Concurrent fetch control |
| Batch size / max retries | 10 / 3 | Per-batch volume and retries |
| Fetch timeout (seconds) / retries | 30 / 1 retry (2 seconds apart) | Per-page fetch fault tolerance |

### Parse

AI parsing (requires a configured model; see [Model Management](llm.md)):

- The **"Parse Model"** selector at the top of the page: pick a chat model from the enabled platforms; switching takes effect in the next parse cycle
- Common parameters: model temperature (0.3), per-request timeout (100 seconds), requests per minute limit (60), remove links inside articles (off; turning it on saves tokens)
- Parse prompt templates are managed in Settings → Prompts and can be bound per source (choose "Parse Template" when editing a source)

### Scheduler

Takes effect only after a **backend restart**: scheduler thread count (20), timezone (default Asia/Shanghai, searchable), startup delay (1 second).

## Related

- Writing Spider scripts → [Spider Management Guide](../spiders.md)
- Let the agent manage sources → [Agent Guide](agent.md)
