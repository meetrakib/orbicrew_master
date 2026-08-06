# `repos/` — Org project repositories

Clone **Leangine org** GitHub repositories into this directory. Contents of nested repos are **gitignored by the master workspace** so each service keeps its own git history.

Tracked in the master repo: this `README.md` (and optional `.gitkeep`) only.

## Repository URLs

| Repo | GitHub | Local path |
|---|---|---|
| `orbicrew-web` | https://github.com/leangine/orbicrew-web | `repos/orbicrew-web` |
| `orbicrew-api` | https://github.com/leangine/orbicrew-api | `repos/orbicrew-api` |
| `orbicrew-channels` | https://github.com/leangine/orbicrew-channels | `repos/orbicrew-channels` |
| `orbicrew-infra` | https://github.com/leangine/orbicrew-infra | `repos/orbicrew-infra` |

Master workspace (personal): https://github.com/meetrakib/orbicrew_master

## Expected layout

```
repos/
├── README.md              ← this file (tracked by master)
├── orbicrew-web/          ← Next.js + React + TypeScript (dashboard + Orbit View)
├── orbicrew-api/          ← Python FastAPI + LangGraph + workers
├── orbicrew-channels/     ← Telegram / Discord / WhatsApp thin adapters
└── orbicrew-infra/        ← Docker Compose, deploy, shared ops
```

## Why these four

| Repo | Why it exists |
|---|---|
| `orbicrew-web` | Interface layer for Standard Dashboard and Orbit View; separate deploy/scale from Python orchestration |
| `orbicrew-api` | Orchestration core (Office Manager, router, budget guard, DB, billing); workers ship from same Python codebase initially |
| `orbicrew-channels` | Multi-channel adapters must stay thin and independently releasable; proves “one backend, many faces” |
| `orbicrew-infra` | Local Compose + staging/prod topology without coupling ops scripts to a single app repo |

## Branch strategy (every repo)

- `main` — production-ready (GitHub default)
- `stage` — staging
- `dev` — default development branch

## Clone

```bash
cd repos
git clone https://github.com/leangine/orbicrew-web.git
git clone https://github.com/leangine/orbicrew-api.git
git clone https://github.com/leangine/orbicrew-channels.git
git clone https://github.com/leangine/orbicrew-infra.git
```

If scaffolds already exist locally, remotes are:

```bash
cd repos/orbicrew-web && git remote set-url origin https://github.com/leangine/orbicrew-web.git
# repeat for api, channels, infra with matching URLs
```
