# `repos/` — Org project repositories

Clone **Leangine org** GitHub repositories into this directory. Contents of nested repos are **gitignored by the master workspace** so each service keeps its own git history.

Tracked in the master repo: this `README.md` (and optional `.gitkeep`) only.

## Repository URLs

| Repo | GitHub | Local path |
|---|---|---|
| `orbicrew-web` | https://github.com/leangine/orbicrew-web | `repos/orbicrew-web` |
| `orbicrew-admin` | https://github.com/leangine/orbicrew-admin | `repos/orbicrew-admin` |
| `orbicrew-api` | https://github.com/leangine/orbicrew-api | `repos/orbicrew-api` |
| `orbicrew-channels` | https://github.com/leangine/orbicrew-channels | `repos/orbicrew-channels` |
| `orbicrew-infra` | https://github.com/leangine/orbicrew-infra | `repos/orbicrew-infra` |

Master workspace (personal): https://github.com/meetrakib/orbicrew_master

## Expected layout

```
repos/
├── README.md              ← this file (tracked by master)
├── orbicrew-web/          ← Next.js + React + TypeScript (dashboard, tenant settings, Orbit View)
├── orbicrew-admin/        ← Next.js platform operator console (separate app)
├── orbicrew-api/          ← Python FastAPI + LangGraph + workers
├── orbicrew-channels/     ← Telegram / Discord / WhatsApp thin adapters
└── orbicrew-infra/        ← Docker Compose (deploy topology), shared ops
```

## Why these repos

| Repo | Why it exists |
|---|---|
| `orbicrew-web` | Customer interface: Standard Dashboard, tenant settings, Orbit View |
| `orbicrew-admin` | Dedicated platform operator console — separate deploy/auth from tenant UI |
| `orbicrew-api` | Orchestration core (Office Manager, router, budget guard, DB, billing); workers ship from same Python codebase initially |
| `orbicrew-channels` | Multi-channel adapters must stay thin and independently releasable; proves “one backend, many faces” |
| `orbicrew-infra` | Staging/prod Compose + ops without coupling scripts to a single app repo |

Tenant-facing settings stay in `orbicrew-web`. Platform operators use `orbicrew-admin` (privileged APIs on `orbicrew-api`).

## Local shared dependencies (master-only)

During app development, start Postgres + Redis from master `resources/orbicrew_dev_infra/` (`docker compose up -d`). Run api / web / admin / channels **natively** on the Mac — do not put those apps in Docker while coding. Nested repos must not point at that master path; they describe Postgres/Redis as “local or your own Compose.”

## Nested repos must stand alone

Each `repos/<name>/` is independently consumable on GitHub. Files inside a nested repo must **not** reference this master workspace, `docs/`, `resources/`, `local/`, or sibling local paths. Sibling links by GitHub URL are fine.

## Branch strategy (every repo)

- `main` — production-ready (GitHub default)
- `stage` — staging
- `dev` — **daily working branch** — commit and push here by default

Do not push feature work to `main` or `stage` unless explicitly asked. Promote `dev` → `stage` → `main` only on request.

## Clone

```bash
cd repos
git clone https://github.com/leangine/orbicrew-web.git
git clone https://github.com/leangine/orbicrew-admin.git
git clone https://github.com/leangine/orbicrew-api.git
git clone https://github.com/leangine/orbicrew-channels.git
git clone https://github.com/leangine/orbicrew-infra.git
```

If scaffolds already exist locally, remotes are:

```bash
cd repos/orbicrew-web && git remote set-url origin https://github.com/leangine/orbicrew-web.git
# repeat for admin, api, channels, infra with matching URLs
```

**If cloning a fresh machine and `orbicrew-admin` already exists on GitHub**, use the clone command above. Local scaffolds use:

```bash
cd repos/orbicrew-admin && git remote set-url origin https://github.com/leangine/orbicrew-admin.git
# then push main/stage/dev if you created content locally first
```
