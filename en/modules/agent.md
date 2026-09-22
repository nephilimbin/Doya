# Agent Guide

> [中文](../../modules/agent.md) · English

> Recommended way to work: most of Doya's main features (adding feed sources, tuning filtering and parsing, searching the knowledge base, data queries, etc.) can be delegated to an agent in natural language — it invokes the corresponding capabilities on your behalf, so you don't have to work through each settings page item by item. Entry point: the **"Agent"** page in the left main navigation (requires a supported agent CLI tool installed locally; see "Prerequisites").

## What It Is

The "Agent" page is Doya's built-in AI agent terminal: you talk to an AI agent in the web page, and the agent operates the program directly through the built-in `doya` command line. — Adding sources, fetching, filtering, parsing, querying the database, exporting subscriptions — all done conversationally.

## Prerequisites

**A supported agent CLI tool installed locally** (installed agent tools are auto-detected; more will be added over time):
- **Claude Code**: [official installation docs](https://docs.claude.com/en/docs/claude-code/setup)
- **Codex**: [GitHub open-source project](https://github.com/openai/codex)


> [!WARNING]
> The agent does **not** use the LLM platforms and API keys configured in Doya Settings → Models — those serve the article filter/parse pipeline. The agent uses its own account authentication and settings.

## Getting Started

1. Open the "Agent" page in the left main navigation.
2. Select an installed agent in the left column (uninstalled ones are greyed out with install links; click "Refresh" right after installing to re-detect)
3. Click **"New Conversation"** — a terminal appears in the right column
4. Give instructions directly in natural language, for example:

| What you want to do | What to say to the agent |
| ---- | ---- |
| Add an RSS subscription | "Add the RSS feed at https://example.com/feed" |
| Write a website spider | "Write a spider for https://xxx to fetch the article list" |
| Search the knowledge base | "Search the knowledge base for statements about XX" |
| Check running status | "Check the system status and the health of each source" |
| Export subscriptions | "Export my subscriptions as OPML" |

Sessions are **persistent**: closing the page does not kill the agent process; reopen the page to return to the session and continue (Claude Code supports resuming; with Codex, once the process ends you need a new conversation).

## What the Agent Can Do

| Area | Capabilities |
| ---- | ---- |
| Feed source management | Add/list/run RSS and Spider sources; enable/disable, delete, clear data, health checks, template reuse; source creation includes robots.txt checks and data sample confirmation |
| Filtering and parsing | Run filter/parse; one-shot webpage-to-Markdown fetch; view processing statistics |
| Knowledge base | Retrieval-based Q&A (read-only) |
| Data queries | Read-only SQL queries (no direct database connection) |
| Subscription import/export | OPML import/export |
| Model management | List platforms/models/prompt templates |
| System | Status, version, hot reload, paths, schema queries |

## Security Boundaries (agent behavior constraints)

The agent works in a constrained environment; these rules are injected into every session:

- **Reads and writes only your data directory** (`~/.doya/`), never other locations
- Write operations (downloads, deletions, file writes) **rehearse first (`--dry-run`), then execute**; reads/writes involving key files (`.env`, `config.local.yaml`) require your confirmation
- New sources are **checked for duplicates first**, and the configuration must be confirmed by you before it is stored
- Preferences and habits are captured in `agent/preferences.md` in the data directory — kept across sessions and preserved through upgrades

## Working Directory

Each agent session works in the `<data root>/agent/` directory (default `~/.doya/agent/`).
The "Working Directory" button in the left column of the "Agent" page opens it directly (only available for local direct connections; for remote access, operate on the server side).

> [!NOTE]
> Official content in the working directory (`CLAUDE.md`, `.claude/skills/doya-cli/`, etc.) is copied/updated by "Sync Official Toolkit" after your confirmation: you may modify it; the update dialog will flag "your changes will be overwritten" and let you choose to update or keep. Your self-built skills and configuration artifacts are **never touched, never deleted**. `.product-sync.json` is the sync state record — do not edit it by hand. Personal preferences can also be written into `agent/preferences.md` (the agent reads it automatically).

## FAQ

- **Just installed an agent tool but the page still shows it as not installed?** Click the "Refresh" button in the left column to force re-detection (results are otherwise cached for 5 minutes).
- **Authentication failure in the terminal?** When the session shows "[authentication failed, please log in again]", log in to Doya again and start a new session.
- **Can I use the Agent page when accessing remotely (phone/other devices)?** Conversations work, but the "Working Directory" button is only available on a local direct connection.
- **Do agent sessions consume resources?** Idle sessions use no compute; long-unused sessions can be deleted from the session list.

## Related

- Have the agent add sources or write Spider scripts → [Feed Source Configuration](rss.md)
- Model configuration for AI filtering/parsing (unrelated to the agent, see the warning at the top) → [Model Management](llm.md)
