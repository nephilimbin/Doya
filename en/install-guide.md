# Doya Installation Guide

> [中文](../install-guide.md) · English

> Applies to macOS (Apple Silicon) and Windows. For detailed configuration of each feature, see "Module Documentation" at the end of this guide.

## 1. What is Doya

Doya is a **customizable information aggregation platform**: it brings the information sources you care about into one place, and AI helps you filter, distill, and search them. Main capabilities:

| Capability | Description |
| ---- | ---- |
| **Multi-source collection** | RSS subscriptions + Spider crawlers (write your own scripts to scrape sites without RSS), fetched automatically on schedule |
| **AI filtering and parsing** | Main-text extraction, summary generation from prompt templates, and more; supports major LLM platforms worldwide and local inference |
| **Knowledge base retrieval** | Turns collected articles into a semantically searchable knowledge base with Q&A-style queries; attach local models to enable hybrid retrieval |
| **AI agent operation** | Built-in agent terminal; direct an AI agent in natural language to handle adding sources, tuning settings, retrieval, and other core operations |
| **Reader** | Web-based reading interface; compatible with the GReader protocol, works with third-party mobile readers |

**Trial edition notes**: free with no time limit, capped at a maximum of **20 feed sources** (disabled sources do not free up slots; once the cap is reached, adding a source produces a clear error message, and the source management page lists all sources). Contact the author to obtain the full edition.
<p align="center">
  <img src="../images/wechat.jpg" width="400" alt="WeChat QR code" /><br/>
  <sub>Scan the QR code to contact the author (WeChat)</sub>
</p>

## 2. Supported Platforms

| Platform | Status |
| ---- | ---- |
| macOS (Apple Silicon, M-series chips) | ✅ Available (installer is arm64) |
| macOS (Intel chips) | ❌ No installer yet |
| Windows 64-bit (x64) | ✅ Available |
| Linux | 🚧 Adaptation in progress |

## 3. Installation

### macOS

1. Double-click `Doya_1.3.0_macOS_arm64_Trial.dmg`
2. In the window that opens, drag the Doya icon into the Applications folder next to it
3. Open the Applications folder and double-click Doya to launch

### Windows

1. Double-click `Doya_1.3.0_Windows_x64_Trial_Setup.exe` and follow the setup wizard
2. Double-click the Doya shortcut on the Desktop (or in the Start menu) to launch

## 4. Security Prompts on First Launch

- **macOS**: A "cannot verify the developer" prompt on first launch is normal. Allow it either way: Settings → Privacy & Security → click "Open Anyway"; or right-click Doya in Applications → choose "Open" → click "Open" again. Subsequent launches will not ask again.
- **Windows**: A blue "Windows protected your PC" (SmartScreen) dialog on first run is normal. Click "More info" → "Run anyway".

## 5. First Use

1. After double-clicking to launch, wait about 10–30 seconds (the first launch initializes and unpacks the built-in browser into the data directory, so only the first run is slower); the browser then opens the page automatically (the auto-opened page always carries the actual address; in the rare case the port is occupied, another port is used automatically). If the page never opens, confirm the process is running: check `doya.bin` in Activity Monitor on macOS, or `doya.exe` in Task Manager on Windows.
2. Log in with the factory account admin / admin (the first login forces a password change).
3. **Recommended: set up the agent first and operate the app through conversation** — all of Doya's main features can be handled by an AI agent, with no need to work through the settings pages manually:
   - Install [Claude Code](https://docs.claude.com/en/docs/claude-code/setup) or [Codex](https://github.com/openai/codex) locally (either one; more agent tools will be added later).
   - Open the **"Agent"** page in the left navigation, select an agent → click "New Session"
   - Give instructions in natural language, for example: "Add the RSS feed for xxx", "Write a spider for site xxx", "Search the knowledge base for xxx" — the agent executes them for you
   - See the [Agent Guide](modules/agent.md) for details

4. For manual configuration, refer to the relevant module document:
   - Add/manage feed sources (RSS) → [Feed Source Configuration](modules/rss.md)
   - Write Spider crawler scripts → [Spider Guide](spiders.md)
   - Configure AI models (for filtering/parsing) → [Model Management](modules/llm.md)
   - Knowledge base and local models → [Knowledge Base Configuration](modules/knowledge.md)
   - Database and backups → [Database Configuration](modules/database.md)
   - Listen address/port, mobile and remote access → [Service Endpoint & Remote Access](modules/service.md)

5. Closing the web page does not quit the application. To stop the service, click "Shut Down" on the Settings → Account page; platform fallbacks:
   - macOS: the Doya icon menu at the top of the menu bar (the recommended fallback; after "Hide menu icon", the icon automatically returns the next time the service restarts). If the icon is unavailable and the web page is unresponsive, open Terminal:
     - To recover the service: `pkill -x doya.bin` (the supervision loop restarts it automatically after 2 seconds, and the hidden menu bar icon is restored as well)
     - To shut down completely: `pkill -f serve.sh && pkill -x doya.bin` (stop the supervision loop first, then the backend, so it is not restarted; the Activity Monitor equivalent: quit the bash process whose command contains `serve.sh` first, then quit each `doya.bin` one by one — multiple processes with the same name is a normal state)
   - Windows: right-click the Doya icon in the taskbar tray at the bottom right → "Exit backend service" (it uses process signals instead of the web page, so it works even when the page fails to open or hangs; if you cannot find the icon, see item 7 below). If the icon is unavailable and the web page is unresponsive:
     - To recover a hung service: **double-click Doya once more** — the app detects the unresponsive service and shows a dialog; click OK to automatically end the hung process and restart (the tray icon returns with it)
     - To shut down completely: on the "Details" tab of Task Manager, first end the cmd.exe whose command line contains `serve.cmd` (the supervision loop; if it is not stopped first, it restarts the backend; the command-line column is hidden by default: right-click the column headers → Select columns → check "Command line"), then end every `doya.exe` (multiple entries with the same name is normal — end them all)
6. Double-clicking again does not start a second service; it only opens the page once more.
7. **Windows tray icon** (bottom right of the taskbar): the status line shows whether the service is running and the actual access URL (click it to open the page); right-click offers "Restart backend service" and "Exit backend service". If the icon is not visible, click the ^ at the bottom right of the taskbar (on Win11, you can also turn Doya on under Settings → Personalization → Taskbar → Other system tray icons)

## 6. Data Directory (`~/.doya`)

All data lives in the `.doya` folder under your home directory.
macOS: `~/.doya/`;
Windows: enter `%USERPROFILE%\.doya` in the File Explorer address bar and press Enter.
**Upgrading, reinstalling, and uninstalling never touch this directory**; deleting it = wiping all data.

```
.doya/
├── config.local.yaml    # 你改过的设置(设置中心保存时自动写)
├── .env                 # 密钥文件(可选,装好模型密钥后自动生成)
├── data/
│   ├── doya.db          # 主数据库(订阅、文章、账号、设置;体积大头)
│   ├── backups/         # 自动/手动备份快照
│   ├── spiders/         # 你的 spider 爬虫脚本
│   └── knowledge/ 等    # 知识库镜像与检索索引
├── logs/                # 运行日志(按日新文件,自动保留 30 天)
├── models/              # 知识库本地模型(放入三件套后启用混合检索)
├── agent/               # agent 会话工作区与偏好记忆
├── browsers/            # 内置浏览器首启解压(macOS/Windows)
└── run/                 # 运行时状态(PID/端口等,可随时删)
```

Key points:

- **Backup**: stop the service before copying — while the app is running, the database is being written to, and a direct copy may be incomplete. Copying the entire `.doya` directory is the safest option; if space is limited, back up at least these three items: `data/` (all data — subscriptions, articles, and accounts), `config.local.yaml` (parameters changed in the settings center), and `.env` (model keys). `models/` is large and can be re-provisioned, and `logs/` and `run/` are runtime traces, so they can be skipped
- **Migration**: when switching computers, stop the service, copy the entire `.doya` directory into the home directory of the new machine, and simply launch. To place it somewhere other than the home directory (for example on another disk), copy it there, set the environment variable `DOYA_DATA_DIR` to the new path, and then launch (by default the app looks for `.doya` only in the user's home directory)
- When you hit a problem, the logs in `logs/` usually contain the specific error cause (one file per day, kept automatically for 30 days); include the relevant error lines when reporting to the author for faster diagnosis (see the end of this guide for feedback channels)

## 7. Uninstallation

> **Uninstalling does not touch your data**: uninstallation only removes the app itself. To keep your data (for example,
> if you plan to reinstall), delete nothing; delete the data directory only for a complete wipe — **deletion cannot be
> undone**, so first make sure it contains nothing you need to keep.

- **macOS**: delete Doya from Applications; delete the data directory if you wish. If "Launch at login" was enabled, disable it in settings before deleting — leaving it on does not affect your data, but a dead login item remains (the system logs one launch-failure entry at login, with no dialog and no repeated retries). If the app is already deleted and you want to clean up afterwards, just remove `~/Library/LaunchAgents/com.doya.server.plist` manually
- **Windows**: if "Launch at login" was enabled, **disable it in settings first**, then uninstall (the uninstaller does not clean up this registry entry, and a leftover entry causes a "script file not found" dialog at every login). Then Settings → Apps → uninstall Doya (when uninstallation finishes, it shows the data directory location); delete the data directory if you wish

## 8. Upgrade

- **macOS**: download the new dmg and drag the new Doya into Applications to overwrite the old version. If the old version is running, it detects the new one within about 30 seconds and exits gracefully on its own (with a notification); double-click the icon to launch the new version and the upgrade is complete — no need to quit the old version manually first
- **Windows**: run the new Setup.exe to install over the old version. The installer detects running Doya processes and walks you through ending them (a cleanup page); just follow the wizard

Your data is unaffected in all cases.

## 9. Module Documentation

| Task | Document |
| ---- | ---- |
| Have an agent act for you in natural language (recommended) | [Agent Guide](modules/agent.md) |
| Add/manage RSS feed sources; tune filtering/parsing/scheduling | [Feed Source Configuration](modules/rss.md) |
| Write Spider crawler scripts | [Spider Guide](spiders.md) |
| Configure AI model platforms and keys | [Model Management](modules/llm.md) |
| Build the knowledge base and set up local retrieval models | [Knowledge Base Configuration](modules/knowledge.md) |
| Switch databases and manage backups | [Database Configuration](modules/database.md) |
| Listen address/port; mobile, LAN, and public network access | [Service Endpoint & Remote Access](modules/service.md) |

## 10. Feedback

Found a problem or have a feature request? Feel free to open one on GitHub (including the error lines from the `logs/` directory makes diagnosis easier): <https://github.com/nephilimbin/Doya/issues>
