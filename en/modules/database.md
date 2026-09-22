# Database Configuration Guide

> [中文](../../modules/database.md) · English

> Manage data storage connections: the built-in SQLite is used by default and needs no changes for personal use; switch only when you need multi-machine sharing or already have MySQL. Entry point: Settings → Database. Changes saved on this page take effect after a backend restart (see the switching procedure below).

## Default Setup (SQLite, Zero Configuration)

Works out of the box: data lives in the data directory `~/.doya/data/doya.db`, with no database software to install. See the backup section at the end of this page.

## Switching to MySQL

1. Go to Settings → Database → Configure Database, and switch "Database Type" to `mysql`
2. Fill in the connection parameters:

| Parameter | Default | Description |
| ---- | ---- | ---- |
| Host | 127.0.0.1 | MySQL host |
| Port | 3306 | |
| Username | root | |
| Database Password | — | Password field; environment variable placeholders such as `${MYSQL_PASSWORD}` are accepted, with values read from `.env` in the data directory or from system environment variables |
| Database Name | doya | Alphanumeric and underscores only, ≤64 |

   Advanced parameters such as the connection pool sit in the "Advanced Parameters" collapsible section; the defaults are already sensible and usually need no changes.

3. Click "**Test Connection**" to verify reachability (a warning is shown if the server is reachable but table schemas are missing columns)
4. Click "**Save**" — changes that relocate the database trigger a **"Switch Database"** dialog with three options:

| Option | Semantics | Best for |
| ---- | ---- | ---- |
| **Keep New Database** | No migration; use the new database's existing content as is; the source database is left in place and you can switch back anytime | The new database is already initialized / you just want to start over with an empty database |
| **Migrate and Overwrite** | Move the source database's **configuration-family data** (settings, subscriptions, models, etc. — not articles) into the new database, overwriting it; snapshots of both sides are taken automatically beforehand | Switching databases while keeping your subscriptions and configuration |
| **Restore from Backup** | Load a chosen config snapshot into the new database; the source database is not read | Rescue path when the source database is unreachable |

5. After a successful save, a "Configuration saved — restart to take effect" banner appears on the page; click the "**Restart**" button to finish

> Saving from the page writes to `config.local.yaml` in the data directory (only the keys you changed); the running service keeps using the old database, and the switch completes only after a restart. The factory `config.yaml` inside the installation package is read-only and never touched.

## Backup

The "Backup" group offers two kinds of snapshots, both stored in the server-side data directory `data/backups/`:

- **Config snapshot**: the configuration family — settings, subscriptions, models, etc. (small)
- **Full snapshot**: includes article data

| Parameter | Default | Description |
| ---- | ---- | ---- |
| Config backup interval (minutes) | 60 | 0 = disable scheduled backups |
| Full backup interval (minutes) | 1440 | Once a day |
| Config / full snapshot retention count | 5 / 2 | Excess snapshots are cleaned up automatically |

The inline "**Config Snapshot**" and "**Full Snapshot**" buttons trigger them manually at any time; the "Restore from Backup" option offered when switching databases picks from these snapshots.

## Related

- Listen address / port / mobile and public network access, port-occupancy troubleshooting → [Service Endpoint and Remote Access](service.md)
- Where all data lives → [Installation Guide · Data Directory](../install-guide.md)
