# lazy-AutoTeam — maintainer notes

Wrapper for [`cnitlrt/AutoTeam`](https://github.com/cnitlrt/AutoTeam) packaging
the upstream FastAPI + Vue + Playwright/Chromium stack as a single-process
LazyCat app. Follows the standard `lazycat-lpk-wrapper` pattern.

## Lazycat appstore identifiers

- **package id**: `cloud.lazycat.app.autoteam`
- **app_id**: `5346` (recorded 2026-05-08)
- **subdomain**: `autoteam` → `https://autoteam.<box-domain>`
- **bootstrap workflow**: when re-running `bootstrap-app.yml` to
  resubmit a fix, pass `app_id=5346` so the workflow skips
  `/app/create` (which would 500 on duplicate package).

## Architecture

**Single-process**, like `lazy-pixelle-video`:

- Upstream Dockerfile (`vendor/AutoTeam/Dockerfile`) is used **unmodified** —
  it already installs `uv`, `xvfb`, Playwright + Chromium, and exposes :8787.
- Upstream `docker-entrypoint.sh` symlinks `/app/data/.env`,
  `accounts.json`, `state.json`, `auths/`, `screenshots/` from the persist
  bind, then `exec uv run autoteam api`.
- LazyCat reverse-proxies `/` → `http://main:8787`. FastAPI serves both
  `/api/*` and the bundled Vue dist (`src/autoteam/web/dist/`).

**No `patches/` directory.** All wrapper customization is via the lazycat
manifest env block — upstream code is pristine, swappable on
`git subtree pull`.

## Critical wiring

| Manifest env | Behaviour |
|---|---|
| `API_KEY={{.U.API_KEY}}` | required deploy param; gates `/api/*`. Upstream `config.py` uses `os.environ.setdefault(key, value)` on `.env` load, so **manifest env beats `data/.env`** — users can't accidentally lock themselves out by saving an empty value via the UI. |
| `MAIL_PROVIDER={{.U.MAIL_PROVIDER}}` | initial default; user changes in UI persist to `data/.env` (UI-set values win for keys NOT in manifest env). |
| `PLAYWRIGHT_PROXY_URL={{.U.PLAYWRIGHT_PROXY_URL}}` | Chromium SOCKS5 / HTTP proxy. Empty = direct. Lazycat boxes are typically residential IPs, but VPS-hosted boxes will want this. |

## Healthcheck

`GET /api/setup/status` — middleware (`api.py` line 53) lists this in
`_AUTH_SKIP_PATHS`, so it's reachable without an API key. Container-side
healthcheck uses `curl -fsS` against this endpoint.

Do **not** use `/` for healthcheck — the SPA returns 200 even when the
backend is half-broken (FastAPI catch-all serves `index.html` last).

## Data persistence

```
/lzcapp/var/persist/                 mounted at /app/data
  .env                               runtime config (UI-editable)
  accounts.json                      account pool
  state.json                         scheduler state
  auths/                             Codex auth files (one per account)
  screenshots/                       Playwright debug captures
```

Upstream entrypoint `chmod -R 777 /app/data` — this works on lazycat
binds when running as root inside the container.

## Updating upstream

```sh
git subtree pull --prefix=vendor/AutoTeam \
  https://github.com/cnitlrt/AutoTeam.git dev --squash
```

Sanity check after pull:

- `vendor/AutoTeam/Dockerfile` still ends with `EXPOSE 8787` and
  `ENTRYPOINT ["/docker-entrypoint.sh"]`
- `vendor/AutoTeam/docker-entrypoint.sh` still symlinks
  `accounts.json`, `state.json`, `.env`, `auths/`, `screenshots/`
- `src/autoteam/api.py` still has `_AUTH_SKIP_PATHS = {... "/api/setup/status" ...}`

If upstream renames `data/` → something else, or moves the FastAPI
entrypoint command, manifest healthcheck + bind path need to follow.

## Compliance / dual-use

Upstream README explicitly flags potential OpenAI ToS conflict (auto
registration + multi-account management). The `package.template.yml`
locale `usage` and `appstore.yml` `description` both repeat that
disclaimer prominently — keep that text in sync if the upstream
disclaimer wording changes.

This is the same dual-use posture as `lazy-outlookEmailPlus`: ship as-is,
flag the risk, gate access with a required password.
