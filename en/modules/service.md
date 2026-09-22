# Service Endpoint and Remote Access Guide

> [中文](../../modules/service.md) · English

> Controls how the service listens: local-only, or open to phones / LAN / public-internet devices. Entry point: Settings → Account → Service Endpoint. By default only the local machine can access it; no changes are needed for personal use. Saved changes take effect after a backend restart.

## Field Reference

| Field (UI name) | Default | Description |
| ---- | ---- | ---- |
| Listen Address | 127.0.0.1 | The network interface address the service process listens on: `127.0.0.1` keeps access local; `0.0.0.0` or a LAN IP opens it to other devices (an access key must be configured as well; `0.0.0.0` recommended — it includes loopback and does not affect local use) |
| Listen Port | 9696 | If the port is occupied, the service automatically switches to an available one (see "Port Drift"); after a change, the port in the browser's "Connection Address" must be updated to match |
| Access Key | (empty) | Signing key for login tokens: empty = local access only; **required when opening the service to other devices** (≥18 characters, at least three of uppercase / lowercase / digits / symbols; a "Random Generate" button is available). After changing it, a restart is required and **all login sessions are invalidated — you must log in again**; you can also enter `${ENV_VAR_NAME}` to manage it through an environment variable |
| Trusted Proxies | (empty) | Whitelist of inbound reverse proxies: the `X-Forwarded-For` header is trusted to recover the real client IP (for rate limiting / auditing) only when the request arrives directly from one of these IPs (e.g., a local nginx). Unrelated to the **outbound proxy** on the "Account" page; keep it empty when no reverse proxy is deployed; once configured, an access key must be configured too; separate multiple entries with commas |

If the listen address is opened to non-local devices without an access key, validation blocks the save (the UI shows the corresponding message).

## Phone / LAN Device Access

1. Change the default password (admin): Settings → Account
2. Set the listen address to `0.0.0.0` and set an access key (you can click "Random Generate")
3. Save, then restart the service when prompted
4. From other devices, open `http://<computer-IP>:<port>`

> [!WARNING]
> **Before exposing the service to the public internet**: Doya is a single-user system with no built-in HTTPS or abuse protection. For public deployment, put it behind a reverse proxy (nginx / caddy, etc.) with proper HTTPS, and declare the proxy through the "Trusted Proxies" field; listening on `0.0.0.0` means any host that can route to this machine may attempt to log in — proceed at your own risk.

## Port Drift

1. When the actual listening port differs from the configured one (e.g., the configured port is occupied), the service automatically switches to an available port.
2. The top of the "Service Endpoint" section shows a notice bar with the actual port, the drift reason, and the occupier, plus **"Adopt Actual Port"** to write the actual port back into the configuration in one click — avoiding the confusion of "the page opens but the configured port does not match".
3. To pin a specific port: change the listen port under Settings → Account → Service Endpoint and restart, and avoid Windows reserved port ranges — Hyper-V / WSL2 / Docker Desktop randomly reserve several ranges within the system dynamic port range (reshuffled on reboot), and ports falling inside a reserved range cannot be listened on. The service automatically falls back to a high port (unaffected by reserved ranges) and returns to the configured port once it becomes available again — no manual handling required; run `netsh interface ipv4 show excludedportrange protocol=tcp` to view the current reserved ranges.

## Related

- Database connections and backups → [Database Configuration](database.md)
