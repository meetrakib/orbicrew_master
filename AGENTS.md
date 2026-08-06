# AGENTS.md — Orbicrew workspace

Persistent instructions for coding agents (Cursor, Claude Code, and others) working in this workspace.

**Read first:** [`docs/development/AGENT_BOOTSTRAP.md`](docs/development/AGENT_BOOTSTRAP.md)

Keep this file lean. Deep process detail lives in `docs/development/`. Product detail lives in `docs/leangine-office-docs/`.

---

## Product (one paragraph)

**Orbicrew, by Leangine** is a multi-tenant AI employee office: an Office Manager agent receives tasks via Standard Dashboard, Orbit View (2D office), or messaging channels (Telegram/Discord/WhatsApp), routes work to specialist agents, enforces cost routing and budget caps, and supports overnight unattended execution with approval gates. Stack target: Next.js + React + TypeScript (web + admin), Python + FastAPI + LangGraph (api), Postgres+pgvector, Redis, self-hosted LiteLLM.

Canonical docs: `docs/leangine-office-docs/` — start at `tech/00_INDEX.md` and `01_AI_Office_Platform_Requirements.md`. Brand lock: Orbicrew, Deep Violet, Bricolage Grotesque + Hanken Grotesk (`tech/09_brand_identity.md` §6). Orbit View is **original IP**, not an Agent Town fork (`tech/18_orbit_view_game_ui.md`).

---

## Workspace layout

| Path | Role | Notes |
|---|---|---|
| `docs/leangine-office-docs/` | Product & tech requirements | Source of truth for what to build |
| `docs/development/` | Phase plan, test guide, tracker, bootstrap | Process source of truth |
| `repos/` | Nested org project git repos | Implement features here |
| `resources/` | Logos, Stitch UI/UX, local deps Compose | Master-only |
| `local/` | Private scratch / personal materials | **Do not read unless the user explicitly asks** |

**Git ownership:** master workspace = personal GitHub ([meetrakib/orbicrew_master](https://github.com/meetrakib/orbicrew_master)). Nested `repos/*` = Leangine org GitHub. Do not commit nested repo contents into the master repo.

### Master-only assets (`resources/`)

| Path | Contents |
|---|---|
| `resources/logos/` | Brand logos (populate as assets arrive; folder tracked via `.gitkeep`) |
| `resources/stitch_orbicrew_ui_ux_guide/` | Stitch exports: `orbicrew` (DESIGN), agent configuration/roster, approvals inbox, dashboard overview, hire AI employees, message agents, settings, task detail, usage/billing |
| `resources/orbicrew_dev_infra/` | Docker Compose for shared deps (Postgres+pgvector, Redis) |

Use logos/Stitch when building Standard Dashboard or platform admin UI. Copy needed files into `orbicrew-web` or `orbicrew-admin` — never make nested repos depend on `resources/` paths.

### Local infra convention

- Start shared dependencies anytime from `resources/orbicrew_dev_infra/` (`docker compose up -d`).
- Develop and run apps in `repos/<github-repo>/` **natively on the Mac** (api, web, admin, channels) — do **not** run those app services in Docker during development.
- `orbicrew-infra` owns staging/prod-oriented Compose and deploy topology, not day-to-day shared deps.

---

## Development workflow (mandatory)

1. **Before development:** read `docs/development/phase_by_phase_development_plan.md` and work the current phase / task. Do not jump ahead of exit criteria without user approval.
2. **During development:** implement in the correct `repos/<service>/` on branch `dev` (feature branches off `dev` as needed). Commit and push to `dev` only unless the user explicitly asks to use `stage` / `main` or to merge/promote.
3. **After each development slice:** update `docs/development/manual-test-guide.md` if new or changed manual verification steps exist.
4. **After completing work:** append an entry to `docs/development/development-tracker.md`.

Do **not** build product features in the workspace root — only docs, agent config, resources, and scaffolding.

---

## Branch workflow (critical — all agents)

Day-to-day: **checkout `dev`, commit on `dev`, push `origin dev`.** Do not push feature work to `main` or `stage` unless the user explicitly says to use those branches or to merge/promote.

| Repo class | Long-lived branches | Daily working / push target |
|---|---|---|
| Nested org repos under `repos/*` | `main` · `stage` · `dev` | **`dev` only** |
| Master workspace `orbicrew_master` | `main` · `stage` · `dev` (same model) | **`dev` only** |

- Feature branches: cut from `dev`, merge back to `dev`.
- Promote `dev` → `stage` → `main` only when the user explicitly requests a merge/promote.
- Never force-push `main` / `stage` (or `master`) unless the user explicitly requests it.

---

## Nested repos

| Repo | GitHub | Path | Stack (intent) | Owns |
|---|---|---|---|---|
| `orbicrew-web` | https://github.com/leangine/orbicrew-web | `repos/orbicrew-web` | Next.js, React, TypeScript | Standard Dashboard, tenant settings, Orbit View |
| `orbicrew-admin` | https://github.com/leangine/orbicrew-admin | `repos/orbicrew-admin` | Next.js, React, TypeScript | Platform operator console |
| `orbicrew-api` | https://github.com/leangine/orbicrew-api | `repos/orbicrew-api` | Python, FastAPI, LangGraph | Orchestration, router, budget guard, DB, billing, workers |
| `orbicrew-channels` | https://github.com/leangine/orbicrew-channels | `repos/orbicrew-channels` | TypeScript or Python (TBD) | Thin Telegram / Discord / WhatsApp adapters |
| `orbicrew-infra` | https://github.com/leangine/orbicrew-infra | `repos/orbicrew-infra` | Docker Compose, scripts | Deploy topology, shared CI/ops templates |

Platform operator console = dedicated `orbicrew-admin` (scaffold/auth shell early; full operator features phase-aligned with multi-tenancy). Tenant-facing settings stay in `orbicrew-web`. Auth: admin uses privileged operator APIs on `orbicrew-api` (`/v1/ops/*` or equivalent) — not tenant JWTs and not routes inside web.

### Nested repo isolation (critical)

When editing files under `repos/<name>/`:

- Never reference the master workspace, `orbicrew_master`, `docs/`, `resources/`, `local/`, or sibling **local** paths.
- Nested repos must remain independently consumable (clone-only from GitHub org).
- Sibling links by **GitHub URL** are OK.
- Use that repo's own git history (`cd repos/<name>`). Do not `git add` nested trees into the master commit.
- Prefer opening / focusing work inside the service directory for scoped context.

---

## Hard rules

- Never read `local/` unless the user explicitly asks.
- Never commit secrets (`.env`, keys, credentials). Prefer `.env.example` only.
- Never force-push `main` / `master` / `stage` unless the user explicitly requests it.
- Prefer open-source infrastructure; build custom for routing, billing, agent UX, Orbit View (see product docs).
- Cost control first: classifier/router and budget guards before speculative agent roster growth.
- Multi-tenant `tenant_id` (and later `project_id`) belongs in data model thinking from day one even when Phase 0 is single-user.

---

## Coding conventions

- Follow best practices for the language and framework in use.
- Prefer reusable / shared abstractions (DRY, shared packages and patterns where appropriate) — extract when duplication is real.
- Prefer latest stable releases of Next.js, React, FastAPI, etc. at time of build — don't pin forever to versions named in aged docs.
- Frontend owns UI + thin BFF/proxy; backend owns orchestration, LangGraph, LiteLLM calls, DB, billing. Typed OpenAPI + generated TS client preferred.
- Channel adapters must stay thin — no duplicated agent logic per channel.
- UI brand: Deep Violet tokens; Bricolage Grotesque (display) + Hanken Grotesk (body). Global English-first product/UI. Prefer Stitch + `resources/` when implementing dashboard/admin screens.

---

## Ignore files reminder

- **`.gitignore`:** ignores `local/`, contents of `repos/*` (keeps README/.gitkeep), secrets, deps, build artifacts. Does **not** ignore `resources/`.
- **`.cursorignore` / `.claudeignore`:** ignore `.env` variants and build/dep noise; **do not** ignore `local/`, `docs/`, `repos/`, or `resources/`.
