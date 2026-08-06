# Agent Bootstrap — Orbicrew

**Read this file at the start of every new agent session** before implementing anything.

This is the onboarding note for the Orbicrew development workspace (Cursor + Claude Code).

---

## 1. What you are working on

**Orbicrew, by Leangine** — multi-agent AI office platform:

- Office Manager orchestrates specialist “employee” agents
- Interfaces: Standard Dashboard (default early), Orbit View (later differentiator), Telegram/Discord/WhatsApp
- Cost-first architecture: model router + budget guard + self-hosted LiteLLM
- Overnight autonomy with approval gates and morning summaries
- End state: multi-tenant SaaS (Managed + BYO-API)

Product docs (source of truth):

1. `docs/leangine-office-docs/tech/00_INDEX.md`
2. `docs/leangine-office-docs/01_AI_Office_Platform_Requirements.md`
3. Then deep-dive as needed (`03_system_design`, `04_database_design`, `05_api_design`, brand/UI, Orbit View, security)

---

## 2. Workspace map

```
orbicrew_development/          ← personal GitHub (master workspace)
├── AGENTS.md / CLAUDE.md
├── .cursor/rules/
├── docs/development/          ← YOU ARE HERE (process)
├── docs/leangine-office-docs/ ← product requirements
├── repos/                     ← org GitHub project clones
│   ├── orbicrew-web/
│   ├── orbicrew-api/
│   ├── orbicrew-channels/
│   └── orbicrew-infra/
└── local/                     ← private; DO NOT READ unless user asks
```

| GitHub | What |
|---|---|
| Personal | This master workspace (docs + agent config) |
| Org | Nested repos under `repos/` |

---

## 3. Development workflow

| When | Action |
|---|---|
| Before development | Follow `phase_by_phase_development_plan.md` — only current phase work unless user overrides |
| After each development | Update `manual-test-guide.md` if verification steps changed |
| After completing work | Append to `development-tracker.md` |

Also respect root `AGENTS.md` / `CLAUDE.md` and `.cursor/rules/`.

---

## 4. Branch strategy (`repos/*`)

- `main` — production-ready default branch
- `stage` — staging
- `dev` — active development (default working branch)

Feature work: branch from `dev`, merge back to `dev`, promote `dev` → `stage` → `main` via the team’s promotion process (not automated until infra is ready).

---

## 5. Where code goes

| Work | Repo |
|---|---|
| Web UI, dashboard, Orbit View | `repos/orbicrew-web` |
| FastAPI, LangGraph, router, DB, workers | `repos/orbicrew-api` |
| Messaging adapters | `repos/orbicrew-channels` |
| Compose, deploy, shared ops | `repos/orbicrew-infra` |

Scaffold only until Phase development starts — do not invent large feature code ahead of the phase plan.

---

## 6. Hard constraints

- Do **not** read `local/` unless the user explicitly asks.
- Do **not** commit secrets.
- Do **not** commit nested `repos/*` contents into the master workspace git (master ignores them).
- Do **not** force-push protected branches unless asked.
- Do **not** fork Agent Town into the product — Orbit View is original IP.
- Prefer cost-control infrastructure before flashy UI or a huge agent roster.

---

## 7. Ignore files (know the difference)

| File | Ignores `local/`? | Ignores `repos/` content? | Must ignore `.env`? |
|---|---|---|---|
| `.gitignore` | Yes | Yes (keeps README/.gitkeep) | Yes |
| `.cursorignore` | **No** | No | Yes |
| `.claudeignore` | **No** | No | Yes |

---

## 8. After this bootstrap

1. Check `development-tracker.md` for latest session notes / current focus.
2. Open the active phase in `phase_by_phase_development_plan.md`.
3. Confirm which `repos/<service>` you will touch.
4. Implement → update manual test guide → update tracker.
