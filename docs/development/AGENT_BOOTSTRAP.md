# Agent Bootstrap — Orbicrew

**Read this file at the start of every new agent session** before implementing anything.

This is the onboarding note for the Orbicrew development workspace (Cursor + Claude Code).

---

## 1. What you are working on

**Orbicrew, by Leangine** — multi-agent AI office platform:

- Office Manager orchestrates specialist “employee” agents
- Interfaces: Standard Dashboard (default early), Orbit View (later differentiator), Telegram/Discord/WhatsApp
- Cost-first architecture: model router + budget guard, calling models through a small in-app provider registry (DeepInfra + OpenRouter to start, extensible to Bedrock/direct-Anthropic/direct-OpenAI — auto-cheapest by default, per-agent manual override); self-hosted LiteLLM (a separate gateway server) is deferred until Phase 2's multi-tenant/BYO routing actually needs it — see `phase_by_phase_development_plan.md` Phase 1.8 and `docs/leangine-office-docs/tech/13_byo_provider_architecture.md`
- Unattended autonomy with approval gates and a Digest ("since last checked," any time of day)
- End state: multi-tenant SaaS (Managed + BYO-API)
- Admin surfaces: tenant settings in `orbicrew-web`; platform operator console in dedicated `orbicrew-admin`

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
├── resources/                 ← logos, Stitch UI/UX, local deps Compose (master-only)
│   └── orbicrew_dev_infra/    ← Postgres + Redis Compose
├── repos/                     ← org GitHub project clones
│   ├── orbicrew-web/
│   ├── orbicrew-admin/
│   ├── orbicrew-api/
│   ├── orbicrew-channels/
│   └── orbicrew-infra/
└── local/                     ← private; DO NOT READ unless user asks
```

| GitHub | What |
|---|---|
| Personal — [meetrakib/orbicrew_master](https://github.com/meetrakib/orbicrew_master) | This master workspace (docs + agent config + resources) |
| Org — [leangine](https://github.com/leangine) | Nested repos under `repos/` |

### Repository URLs

| Repo | GitHub | Local path |
|---|---|---|
| Master workspace | https://github.com/meetrakib/orbicrew_master | workspace root |
| `orbicrew-web` | https://github.com/leangine/orbicrew-web | `repos/orbicrew-web` |
| `orbicrew-admin` | https://github.com/leangine/orbicrew-admin | `repos/orbicrew-admin` |
| `orbicrew-api` | https://github.com/leangine/orbicrew-api | `repos/orbicrew-api` |
| `orbicrew-channels` | https://github.com/leangine/orbicrew-channels | `repos/orbicrew-channels` |
| `orbicrew-infra` | https://github.com/leangine/orbicrew-infra | `repos/orbicrew-infra` |

### Master-only resources

| Path | Use for |
|---|---|
| `resources/logos/` | Brand logos (folder may start empty; tracked) |
| `resources/stitch_orbicrew_ui_ux_guide/` | Stitch screen exports for dashboard/admin UI |
| `resources/orbicrew_dev_infra/` | Shared local deps: Postgres+pgvector, Redis |

Stitch folders: `orbicrew` (DESIGN), `orbicrew_agent_configuration`, `orbicrew_agent_roster`, `orbicrew_approvals_inbox`, `orbicrew_dashboard_overview`, `orbicrew_hire_ai_employees`, `orbicrew_message_agents`, `orbicrew_settings`, `orbicrew_task_detail`, `orbicrew_usage_billing`.

`resources/stitch_orbicrew_ui_ux_guide/additional-designs/` — later Stitch exports covering gaps in the
original set (briefs in the sibling `additional_designs_guide/` folder): `orbicrew_agent_configuration_setting_tab`
(Agent Configuration's undesigned Setting tab), `orbicrew_agent_roster_master_promotion_modal`
(promote/demote Master Agent modal, adds to `orbicrew_agent_roster`), `orbicrew_live_voice_mode`
(continuous voice overlay, extends `orbicrew_message_agents`), `orbicrew_settings` (updated —
adds a Slack row to Connected Channels), `shader` (WebGL background shader, no `screen.png`), plus
a duplicate `orbicrew` DESIGN.md (identical to the original — same tokens).

### Local infra convention

1. `cd resources/orbicrew_dev_infra && docker compose up -d` — start shared dependencies anytime.
2. Develop apps under `repos/<github-repo>/` and run them **natively on the Mac** (api, web, admin, channels).
3. Do **not** run those app services in Docker during development — saves memory and disk.
4. Nested repos must not reference this master path; they say “Postgres/Redis locally or your own Compose.”

---

## 3. Development workflow

| When | Action |
|---|---|
| Before development | Follow `phase_by_phase_development_plan.md` — only current phase work unless user overrides |
| During development | Checkout **`dev`**, implement, commit, push **`origin dev`** (master + every nested org repo) |
| After each development | Update `manual-test-guide.md` if verification steps changed |
| After completing work | Append to `development-tracker.md` |

Also respect root `AGENTS.md` / `CLAUDE.md` and `.cursor/rules/`.

---

## 4. Branch strategy (critical)

Long-lived branches everywhere: **`main`** (prod) · **`stage`** (staging) · **`dev`** (active development).

| Rule | Detail |
|---|---|
| Daily default | Work, commit, and push **only on `dev`** — master workspace (`orbicrew_master`) and all five org repos under `repos/*` |
| Do not | Push feature work directly to `main` or `stage` |
| Exception | Only when the user **explicitly** asks to use `stage` / `main`, or to merge/promote |
| Features | Branch from `dev`, merge back to `dev`, then promote `dev` → `stage` → `main` only on explicit request |

---

## 5. Where code goes

| Work | Repo |
|---|---|
| Web UI, dashboard, tenant settings, Orbit View | `repos/orbicrew-web` |
| Platform operator console | `repos/orbicrew-admin` |
| FastAPI, LangGraph, router, DB, workers | `repos/orbicrew-api` |
| Messaging adapters | `repos/orbicrew-channels` |
| Deploy Compose / shared ops | `repos/orbicrew-infra` |
| Local shared deps Compose | `resources/orbicrew_dev_infra` (master-only) |

**Admin decision:** dedicated `orbicrew-admin` from the start (not `/admin` inside web). Scaffold/auth shell early; full operator features stay phase-aligned with multi-tenancy. Tenant admin stays in Standard Dashboard settings (`orbicrew-web`). **Auth/API:** `orbicrew-admin` calls privileged operator endpoints on `orbicrew-api` (e.g. `/v1/ops/*`); tenant JWTs must not access those routes.

Scaffold only until Phase development starts — do not invent large feature code ahead of the phase plan.

---

## 6. Hard constraints

- Do **not** read `local/` unless the user explicitly asks.
- Do **not** commit secrets.
- Do **not** commit nested `repos/*` contents into the master workspace git (master ignores them).
- Do **not** force-push protected branches unless asked.
- Do **not** fork Agent Town into the product — Orbit View is original IP.
- Prefer cost-control infrastructure before flashy UI or a huge agent roster.
- **Nested repo isolation:** when editing under `repos/<name>/`, never reference master-only paths (`docs/`, `resources/`, `local/`, `orbicrew_master`, workspace layout). Nested READMEs/code must stand alone. Sibling **GitHub** URLs are OK.
- **Coding principles:** follow best practices; prefer reusable shared abstractions (DRY) over copy-paste.

---

## 7. Ignore files (know the difference)

| File | Ignores `local/`? | Ignores `repos/` content? | Ignores `resources/`? | Must ignore `.env`? |
|---|---|---|---|---|
| `.gitignore` | Yes | Yes (keeps README/.gitkeep) | **No** (tracked) | Yes |
| `.cursorignore` | **No** | No | No | Yes |
| `.claudeignore` | **No** | No | No | Yes |

---

## 8. After this bootstrap

1. Check `development-tracker.md` for latest session notes / current focus.
2. Open the active phase in `phase_by_phase_development_plan.md`.
3. Confirm which `repos/<service>` you will touch; **`git checkout dev`** before committing.
4. Start shared deps from `resources/orbicrew_dev_infra/` if needed; run apps natively.
5. Implement → update manual test guide → update tracker → push **`dev`**.
6. If building UI: consult `resources/stitch_orbicrew_ui_ux_guide/` and `resources/logos/`; copy assets into `orbicrew-web` or `orbicrew-admin` as needed.
