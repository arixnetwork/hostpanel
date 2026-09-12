# Rabby Host

A simple, beautiful, reliable local/self-hosting control panel for Debian
Linux (and other systemd + Docker hosts). Monitor your box at a glance, manage
systemd services, install/start/stop self-hosted apps, and route HTTPS traffic
through a reverse proxy — all from one lightweight web UI.

- **No build step.** Plain Node.js + Express backend and a dependency-free
  vanilla-JS frontend. Everything runs from source.
- **Read-only by default.** Monitoring reads `/proc`, `df`, `ps`, `systemctl`
  and `journalctl`.
- **Opt-in power.** Installing apps (Docker Compose), stopping services and
  editing the reverse proxy require a configured password and, where systemd
  actions need it, passwordless sudo.
- **Secure by design.** First boot generates a random password; config is
  stored 0600; sessions use `HttpOnly; SameSite=Lax` cookies; every free-text
  value passed to a host command is validated or sanitized.

## Requirements

- Debian 11/12 (any systemd Linux works for the monitoring bits)
- Node.js >= 18
- Docker + Docker Compose plugin (only if you install apps)
- nginx (only if you want the reverse-proxy section)

## Quick start

```bash
git clone .../rabby-host /opt/rabby-host
cd /opt/rabby-host
npm ci --omit=dev
node src/server.js
```

On first boot the server prints a generated login password to the console.
Open http://127.0.0.1:3737, log in, and change the password in **Settings**.

### Run as a service

```bash
sudo bash scripts/install-service.sh            # installs to /opt/rabby-host
systemctl status rabby-host
```

### CLI

```bash
bin/rabby-host                                  # start (alias of npm start)
RABBY_PORT=8080 RABBY_BIND_HOST=0.0.0.0 bin/rabby-host
```

## Configuration

| Environment | Default | Meaning |
|-------------|---------|---------|
| `RABBY_PORT` | `3737` | HTTP listen port |
| `RABBY_BIND_HOST` | `127.0.0.1` | Bind address (use `0.0.0.0` to expose) |
| `RABBY_CONFIG_DIR` | `~/.config/rabby-host` | Config + password store |
| `RABBY_DATA_DIR` | `~/.local/share/rabby-host` | App registry, compose files |

## Features

- **Dashboard** — live CPU / memory / disk / network / load / uptime with
  sparklines, top processes, mount table.
- **Services** — list systemd units, filter, view state + main PID, stream
  logs, and start/stop/restart (needs sudo for privileged units).
- **Apps** — one-click install of 11 curated Docker Compose apps (Portainer,
  Uptime Kuma, Nginx Proxy Manager, Gitea, Nextcloud, MariaDB, Redis, Grafana,
  AdGuard Home, MinIO, Home Assistant). Install, start/stop/restart, logs,
  uninstall; Compose files live in the data dir and are reused on reinstall.
- **Reverse proxy** — manage nginx vhosts (`server_name` + `proxy_pass`,
  optional WebSocket upgrade) with `nginx -t` validation before each reload.
- **Settings** — change password, toggle self-check details.

## API

Documented surface (JSON, `/api/health` is open; everything else needs a
session cookie):

```
GET  /api/health
POST /api/auth/setup | /login | /logout | /change         GET /api/auth/status
POST /api/system/overview      GET /api/system/info | processes | uptime
GET  /api/services[/:name[/logs]]   POST /api/services/:name/:action
GET  /api/apps[/:id]   POST /api/apps/:id/install|:action
GET  /api/proxy        POST /api/proxy/sites   … reload / test
GET/PATCH /api/settings         GET /api/self-check
```

## Development

```bash
npm run dev                # watch mode
npm test                   # 55 unit + API tests (fake exec, no host calls)
RABBY_INTEGRATION=1 npm run test:integration   # read-only checks against this box
```

The automated suite injects a fake process runner so unit tests never touch
the host; the integration suite is strictly read-only (no installs, no
service actions).

## Security notes

- Master password is stored as a salted scrypt hash.
- Sessions are in-memory (a server restart logs everyone out).
- Bind to `127.0.0.1` unless you really trust your LAN; consider a reverse
  proxy + TLS (or Rabby's own proxy section) before exposing to the internet.
- Host commands never pass through a shell; all arguments are array-based.
- nginx configs and app variables are rendered from strict allow-lists.

## License

MIT