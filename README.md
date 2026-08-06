# Orbicrew Development Workspace

Personal agentic coding workspace for **Orbicrew, by Leangine** — an AI employee office platform (multi-agent orchestration, cost-controlled model routing, standard dashboard + Orbit View, multi-channel adapters).

This repository is the **master workspace**: docs, agent instructions, and pointers. Application code lives in **nested org repos** under `repos/`.

## Setup philosophy

1. **Clone this master repo** anywhere on your machine (personal GitHub).
2. **Clone org project repos into `repos/`** (one folder per service — see `repos/README.md`).
3. **Start agentic coding** in Cursor and/or Claude Code from this workspace root.
4. Agents should read `docs/development/AGENT_BOOTSTRAP.md` first, then follow the phase plan and tracking docs.

```
orbicrew_development/          ← this repo (personal GitHub)
├── AGENTS.md / CLAUDE.md      ← persistent agent instructions
├── .cursor/rules/             ← Cursor project rules
├── docs/
│   ├── development/           ← phase plan, test guide, tracker, bootstrap
│   └── leangine-office-docs/  ← product & technical requirements
├── repos/                     ← org GitHub clones (gitignored contents)
│   ├── orbicrew-web/
│   ├── orbicrew-api/
│   ├── orbicrew-channels/
│   └── orbicrew-infra/
└── local/                     ← private scratch (gitignored; agents: don't read unless asked)
```

## Ownership

| Location | GitHub | Purpose |
|---|---|---|
| Workspace root (this repo) | **Personal** — [meetrakib/orbicrew_master](https://github.com/meetrakib/orbicrew_master) | Docs, agent config, development process |
| `repos/*` | **Org** — [leangine](https://github.com/leangine) | Product services and infra |

Nested repo contents under `repos/` are **not** committed to the master repo. Only `repos/README.md` (and optionally `.gitkeep`) is tracked so the folder exists when you clone.

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
git clone https://github.com/leangine/orbicrew-api.git
git clone https://github.com/leangine/orbicrew-channels.git
git clone https://github.com/leangine/orbicrew-infra.git
cd ..

# 3. Open in Cursor / Claude Code and follow AGENT_BOOTSTRAP.md
```

## What not to do

- Do not commit secrets (`.env*`), `local/`, or nested `repos/*` application trees into the master repo.
- Do not build product features in the master repo root — implement in the appropriate `repos/<service>/`.
- Do not treat Agent Town as a fork target; Orbit View is original IP (see product docs).
