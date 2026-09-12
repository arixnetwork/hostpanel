# BUILD_STATUS.md

Build status tracker for **Rabby Host v1.0.0** — a simple, beautiful, reliable
local/self-hosting control panel for Debian Linux.

## Legend
- ✅ completed
- 🔨 in progress
- ⏳ pending

---

## Phases

| Phase | Description | Status |
|-------|-------------|--------|
| 1 | Project scaffolding (package.json, structure, docs) | ✅ |
| 2 | Core libs: process runner, config store + password auth, system metrics | ✅ |
| 3 | Express API: system, services, apps, reverse proxy, settings + auth | ✅ |
| 4 | Docker-Compose app catalog (11 apps) | ✅ |
| 5 | Web UI (dashboard, services, apps, proxy, settings, login) | ✅ |
| 6 | Unit tests | ✅ |
| 7 | End-to-end fix loop + real-system verification | ✅ |
| 8 | Packaging (systemd unit, install script, README) | ✅ |

## Current phase
None — v1.0.0 is feature-complete. Final polish: unit + integration suites
are green and the package boots against the real host.

## Tests
- **Unit/API: 55/55 passing** — `npm test` (`node --test "test/unit/*.test.js"`).
  All host commands are routed through the injectable executor and faked in
  unit tests, so the suite never touches the host.
- **Integration: 12/12 passing** — `RABBY_INTEGRATION=1 npm run test:integration`.
  Read-only against the real box: metrics (/proc), df/statfs, ps, systemctl
  list/show, journalctl, docker presence, compose template rendering, nginx
  absence detection.
- Manual smoke passed earlier: boot, banner + one-time password, health,
  auth 401/200 + logout, overview (real cpu/mem/disk), processes, 27 services,
  service detail (ssh, pid 166), journal logs, apps list (11), self-check.

## Tests passed
- All of the above. Notable fixes along the way: Metrics readProc TDZ, wrap TDZ
  in app.js, auth middleware usage, first-boot password banner, exec.js timeout
  (manual timer, no double signal), proxy site-file naming (`<name>.conf`) +
  strict whole-string `DOMAIN_RE` validation + `enabled:false` on failed
  `nginx -t`, systemd `safeUnit` allow-list + non-JSON `list-units` parsing,
  integration test fixes (async `disks`/`topProcesses`, `exec` import).

## Tests failed
- (none remaining)

## Remaining work
- Optional niceties: notifications, export, theming polish — none required for v1.0.0.