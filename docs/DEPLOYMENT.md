# Deploying Lunel

Use the self-hosted deployment instructions and environment-variable reference below.
Keep workers on a private network and expose only the console through HTTPS.

### Railway (optional per-instance-domains mode)

Set `LUNEL_RAILWAY_TOKEN`, and project/environment IDs (auto-injected when
the console itself runs on Railway). Each instance is then deployed as its
own Railway service with a generated public domain — verified against the
current Railway GraphQL API. Without those variables, Railway uses the same
single-service behavior as above.

---

## 3. Self-hosted Docker

```bash
export LUNEL_PG_PASSWORD=$(openssl rand -hex 16)
export LUNEL_SECRET_KEY=$(openssl rand -hex 32)
export LUNEL_WORKER_TOKEN=$(openssl rand -hex 32)
export LUNEL_PUBLIC_URL=https://lunel.example.com
export LUNEL_GITHUB_CLIENT_ID=...
export LUNEL_GITHUB_CLIENT_SECRET=...
export LUNEL_ADMIN_GITHUB_LOGIN=yourlogin
export LUNEL_COOKIE_SECURE=1

cd deploy/docker && docker compose up -d --build
```

- Console: `http://127.0.0.1:8080` (put your own TLS proxy in front, or
  uncomment the `caddy` service for automatic Let's Encrypt).
- Worker: uses the **process driver** by default inside its container.
  For real container isolation enable the Docker driver:
  uncomment the docker.sock mount (read-only, worker only) and set
  `LUNEL_WORKER_DRIVER=docker`, then build the Core image:
  `docker build -t lunel/core:latest ../../core`.
- Optional wildcard endpoints: point `*.lunel.example.com` at the host and
  uncomment the Caddy service (`deploy/proxy/Caddyfile.template`).

---

## Environment variables reference

### Console / unified service

| Variable | Default | Purpose |
|---|---|---|
| `PORT` | `8080` | Public listen port (platform-injected) |
| `LUNEL_DATABASE_URL` | falls back to `DATABASE_URL` | PostgreSQL DSN |
| `LUNEL_SECRET_KEY` | auto-generated + persisted | Session hashing context |
| `LUNEL_WORKER_TOKEN` | auto-generated (unified) / required (split) | Shared secret with workers |
| `LUNEL_GITHUB_CLIENT_ID/SECRET` | — | OAuth login |
| `LUNEL_PUBLIC_URL` | `http://127.0.0.1:$PORT` | Canonical console origin (OAuth redirect, endpoint URLs) |
| `LUNEL_ADMIN_GITHUB_LOGIN` | — | Bootstrap admin |
| `LUNEL_COOKIE_SECURE` | `0` | Set `1` behind HTTPS |
| `LUNEL_LOCAL_WORKER_URL` | auto (unified) / `http://127.0.0.1:9100` | Default worker API |
| `LUNEL_DOMAIN_ROOT` | `lunel.app` | Informational for provider domains |

### Worker (split deployments)

| Variable | Default | Purpose |
|---|---|---|
| `LUNEL_WORKER_TOKEN` | — (required) | Shared secret |
| `LUNEL_CONSOLE_URL` | — | Heartbeat target (optional) |
| `LUNEL_NODE_ID` / `LUNEL_NODE_REGION` | `local` | Scheduler identity |
| `LUNEL_WORKER_DRIVER` | auto (`docker` if available, else `process`) | Isolation driver |
| `LUNEL_WORKER_DATA` | `/var/lib/lunel/instances` | Instance data root |
| `LUNEL_WORKER_PORT_START/END` | `19000-19999` | Port allocation range |
| `LUNEL_CORE_PYTHON` / `LUNEL_CORE_CWD` | `.venv/bin/python` / — | How to launch Core (process driver) |
| `LUNEL_NODE_CAPACITY` | `20` | Max instances reported to scheduler |

### Core

| Variable | Default | Purpose |
|---|---|---|
| `PORT` | `8000` | Listen port (allocated automatically by the worker) |
| `LUNEL_CORE_API_TOKEN` | — | Management API bearer (required for it to be enabled) |
| `LUNEL_STATE_PATH` | `/data/state.json` | Persistence |
| `LUNEL_LOG_LEVEL` / `LUNEL_LOG_JSON` | `info` / `0` | Logging |
| `LUNEL_VERSION` / `LUNEL_BUILD` / `LUNEL_COMMIT` | `1.0.0`/`dev`/`unknown` | Version report |
