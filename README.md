# Orbicrew Development Workspace

Personal agentic coding workspace for **Orbicrew, by Leangine** — an AI employee office platform (multi-agent orchestration, cost-controlled model routing, standard dashboard + Orbit View, multi-channel adapters).

This repository is the **master workspace**: docs, agent instructions, shared `resources/`, and pointers. Application code lives in **nested org repos** under `repos/`.

## Setup philosophy

1. **Clone this master repo** anywhere on your machine (personal GitHub).
2. **Clone org project repos into `repos/`** (one folder per service — see `repos/README.md`).
3. **Start shared deps** from `resources/orbicrew_dev_infra/` (`docker compose up -d` for Postgres + Redis).
4. **Start agentic coding** in Cursor and/or Claude Code from this workspace root; run apps natively.
5. Agents should read `docs/development/AGENT_BOOTSTRAP.md` first, then follow the phase plan and tracking docs.

```
orbicrew_development/          ← this repo (personal GitHub)
├── AGENTS.md / CLAUDE.md      ← persistent agent instructions
├── .cursor/rules/             ← Cursor project rules
├── docs/
│   ├── development/           ← phase plan, test guide, tracker, bootstrap
│   └── leangine-office-docs/  ← product & technical requirements
├── resources/                 ← logos, Stitch UI/UX, local deps Compose (tracked)
│   └── orbicrew_dev_infra/    ← Postgres + Redis Compose (apps run natively)
├── repos/                     ← org GitHub clones (gitignored contents)
│   ├── orbicrew-web/
│   ├── orbicrew-admin/
│   ├── orbicrew-api/
│   ├── orbicrew-channels/
│   └── orbicrew-infra/
└── local/                     ← private scratch (gitignored; agents: don't read unless asked)
```

## Ownership

| Location | GitHub | Purpose |
|---|---|---|
| Workspace root (this repo) | **Personal** — [meetrakib/orbicrew_master](https://github.com/meetrakib/orbicrew_master) | Docs, agent config, development process, `resources/` |
| `repos/*` | **Org** — [leangine](https://github.com/leangine) | Product services and infra |

Nested repo contents under `repos/` are **not** committed to the master repo. Only `repos/README.md` (and optionally `.gitkeep`) is tracked so the folder exists when you clone.

## Admin surfaces (decision)

| Surface | Where | Phase |
|---|---|---|
| Tenant settings (billing, seats, roles, per-agent kill switches) | `orbicrew-web` Standard Dashboard / settings | Phase 2 |
| Platform operator console (cross-tenant cost, tenant lifecycle, platform-wide kill switch) | Dedicated `orbicrew-admin` app/repo | Scaffold early; full features Phase 2 |

Platform operator console is a **separate** org repo from day one — not embedded `/admin` in `orbicrew-web`.

## Local development infra

| Need | Where |
|---|---|
| Postgres + pgvector, Redis (shared deps) | `resources/orbicrew_dev_infra/` — `docker compose up -d` |
| Application coding (api, web, admin, channels) | `repos/<name>/` — run **natively** on the Mac |
| Staging/prod-style multi-service Compose | `repos/orbicrew-infra` (later) |

Do **not** run app services in Docker during day-to-day development.

## Resources (master-only)

| Path | Contents |
|---|---|
| [`resources/logos/`](resources/logos/) | Brand logos (may be empty until assets are added) |
| [`resources/stitch_orbicrew_ui_ux_guide/`](resources/stitch_orbicrew_ui_ux_guide/) | Stitch UI/UX screen exports for dashboard work |
| [`resources/orbicrew_dev_infra/`](resources/orbicrew_dev_infra/) | Lean Docker Compose for shared local dependencies |

See [`resources/README.md`](resources/README.md). Agents building web/admin UI should use logos/Stitch; copy into the relevant app repo as needed. Nested org repos must not hard-depend on this folder.

## Docs map

| Doc | Path | When to use |
|---|---|---|
| Agent bootstrap (read first) | [`docs/development/AGENT_BOOTSTRAP.md`](docs/development/AGENT_BOOTSTRAP.md) | Every new agent session |
| Phase-by-phase plan | [`docs/development/phase_by_phase_development_plan.md`](docs/development/phase_by_phase_development_plan.md) | Before starting feature work |
| Manual test guide | [`docs/development/manual-test-guide.md`](docs/development/manual-test-guide.md) | After each development slice |
| Development tracker | [`docs/development/development-tracker.md`](docs/development/development-tracker.md) | After completing work |
| Product docs index | [`docs/leangine-office-docs/tech/00_INDEX.md`](docs/leangine-office-docs/tech/00_INDEX.md) | Requirements & architecture |

## Branch strategy (org repos)

Each project under `repos/` uses three long-lived branches:

- `main` — production-ready
- `stage` — staging / pre-prod
- `dev` — active development default

Work feature branches off `dev` unless a change is explicitly stage/prod-only.

## Repository URLs

| Repo | GitHub | Local path |
|---|---|---|
| Master workspace (personal) | https://github.com/meetrakib/orbicrew_master | workspace root |
| `orbicrew-web` | https://github.com/leangine/orbicrew-web | `repos/orbicrew-web` |
| `orbicrew-admin` | https://github.com/leangine/orbicrew-admin | `repos/orbicrew-admin` |
| `orbicrew-api` | https://github.com/leangine/orbicrew-api | `repos/orbicrew-api` |
| `orbicrew-channels` | https://github.com/leangine/orbicrew-channels | `repos/orbicrew-channels` |
| `orbicrew-infra` | https://github.com/leangine/orbicrew-infra | `repos/orbicrew-infra` |

## Quick start for a new machine

```bash
# 1. Clone master workspace (personal)
git clone https://github.com/meetrakib/orbicrew_master.git orbicrew_development
cd orbicrew_development

# 2. Clone org repos into repos/
cd repos
git clone https://github.com/leangine/orbicrew-web.git
git clone https://github.com/leangine/orbicrew-admin.git
git clone https://github.com/leangine/orbicrew-api.git
git clone https://github.com/leangine/orbicrew-channels.git
git clone https://github.com/leangine/orbicrew-infra.git
cd ..

# 3. Start shared deps (Postgres + Redis only)
cd resources/orbicrew_dev_infra && docker compose up -d && cd ../..

# 4. Open in Cursor / Claude Code and follow AGENT_BOOTSTRAP.md
#    Run api / web / admin / channels natively — not in Docker.
```

Org repos are also cloneable **standalone** — their READMEs must not require this master workspace.

## What not to do

- Do not commit secrets (`.env*`), `local/`, or nested `repos/*` application trees into the master repo.
- Do not build product features in the master repo root — implement in the appropriate `repos/<service>/`.
- Do not put master-workspace paths into nested org repo files.
- Do not treat Agent Town as a fork target; Orbit View is original IP (see product docs).
