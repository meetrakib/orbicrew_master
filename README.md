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
| Workspace root (this repo) | **Personal** | Docs, agent config, development process |
| `repos/*` | **Org** (Orbicrew / Leangine) | Product services and infra |

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

## Quick start for a new machine

```bash
# 1. Clone master workspace (personal)
git clone <your-personal-remote-url> orbicrew_development
cd orbicrew_development

# 2. Clone org repos into repos/ (after org remotes exist)
cd repos
git clone <org>/orbicrew-web.git
git clone <org>/orbicrew-api.git
git clone <org>/orbicrew-channels.git
git clone <org>/orbicrew-infra.git
cd ..

# 3. Open in Cursor / Claude Code and follow AGENT_BOOTSTRAP.md
```

Until org remotes exist, local scaffolding under `repos/` may already be initialized with `main` / `stage` / `dev` — connect remotes and push when GitHub org repos are ready.

## What not to do

- Do not commit secrets (`.env*`), `local/`, or nested `repos/*` application trees into the master repo.
- Do not build product features in the master repo root — implement in the appropriate `repos/<service>/`.
- Do not treat Agent Town as a fork target; Orbit View is original IP (see product docs).
