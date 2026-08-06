# `repos/` — Org project repositories

Clone **Orbicrew / Leangine org** GitHub repositories into this directory. Contents of nested repos are **gitignored by the master workspace** so each service keeps its own git history.

Tracked in the master repo: this `README.md` (and optional `.gitkeep`) only.

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

- `main` — production-ready
- `stage` — staging
- `dev` — default development branch

## Connecting remotes (when org URLs exist)

```bash
cd repos/orbicrew-web
git remote add origin git@github.com:<ORG>/orbicrew-web.git
git push -u origin main
git push -u origin stage
git push -u origin dev
# repeat for api, channels, infra
```

Until remotes exist, local scaffolds may already be initialized for agentic prep.
