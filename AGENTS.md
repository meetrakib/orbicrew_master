# AGENTS.md — Orbicrew workspace

Persistent instructions for coding agents (Cursor, Claude Code, and others) working in this workspace.

**Read first:** [`docs/development/AGENT_BOOTSTRAP.md`](docs/development/AGENT_BOOTSTRAP.md)

Keep this file lean. Deep process detail lives in `docs/development/`. Product detail lives in `docs/leangine-office-docs/`.

---

## Product (one paragraph)

**Orbicrew, by Leangine** is a multi-tenant AI employee office: an Office Manager agent receives tasks via Standard Dashboard, Orbit View (2D office), or messaging channels (Telegram/Discord/WhatsApp), routes work to specialist agents, enforces cost routing and budget caps, and supports overnight unattended execution with approval gates. Stack target: Next.js + React + TypeScript (web), Python + FastAPI + LangGraph (api), Postgres+pgvector, Redis, self-hosted LiteLLM.

Canonical docs: `docs/leangine-office-docs/` — start at `tech/00_INDEX.md` and `01_AI_Office_Platform_Requirements.md`. Brand lock: Orbicrew, Deep Violet, Bricolage Grotesque + Hanken Grotesk (`tech/09_brand_identity.md` §6). Orbit View is **original IP**, not an Agent Town fork (`tech/18_orbit_view_game_ui.md`).

---

## Workspace layout

| Path | Role | Notes |
|---|---|---|
| `docs/leangine-office-docs/` | Product & tech requirements | Source of truth for what to build |
| `docs/development/` | Phase plan, test guide, tracker, bootstrap | Process source of truth |
| `repos/` | Nested org project git repos | Implement features here |
| `local/` | Private scratch / personal materials | **Do not read unless the user explicitly asks** |

**Git ownership:** master workspace = user's **personal** GitHub. Nested `repos/*` = **org** GitHub. Do not commit nested repo contents into the master repo.

---

## Development workflow (mandatory)

1. **Before development:** read `docs/development/phase_by_phase_development_plan.md` and work the current phase / task. Do not jump ahead of exit criteria without user approval.
2. **During development:** implement in the correct `repos/<service>/` on branch `dev` (feature branches off `dev` as needed). Follow `main` / `stage` / `dev` strategy.
3. **After each development slice:** update `docs/development/manual-test-guide.md` if new or changed manual verification steps exist.
4. **After completing work:** append an entry to `docs/development/development-tracker.md`.

Do **not** build product features in the workspace root — only docs, agent config, and scaffolding.

---

## Nested repos

| Repo | Path | Stack (intent) | Owns |
|---|---|---|---|
| `orbicrew-web` | `repos/orbicrew-web` | Next.js, React, TypeScript | Standard Dashboard, auth session UI, Orbit View chrome + Phaser world |
| `orbicrew-api` | `repos/orbicrew-api` | Python, FastAPI, LangGraph | Orchestration, router, budget guard, DB, billing, workers |
| `orbicrew-channels` | `repos/orbicrew-channels` | TypeScript or Python (TBD) | Thin Telegram / Discord / WhatsApp adapters |
| `orbicrew-infra` | `repos/orbicrew-infra` | Docker Compose, scripts | Local stack, deploy topology, shared CI/ops templates |

When editing a nested repo:

- Use that repo's own git history (`cd repos/<name>`).
- Do not `git add` nested `.git` trees into the master workspace commit.
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

## Coding conventions (from product docs)

- Prefer latest stable releases of Next.js, React, FastAPI, etc. at time of build — don't pin forever to versions named in aged docs.
- Frontend owns UI + thin BFF/proxy; backend owns orchestration, LangGraph, LiteLLM calls, DB, billing. Typed OpenAPI + generated TS client preferred.
- Channel adapters must stay thin — no duplicated agent logic per channel.
- UI brand: Deep Violet tokens; Bricolage Grotesque (display) + Hanken Grotesk (body). Global English-first product/UI.

---

## Ignore files reminder

- **`.gitignore`:** ignores `local/`, contents of `repos/*` (keeps README/.gitkeep), secrets, deps, build artifacts.
- **`.cursorignore` / `.claudeignore`:** ignore `.env` variants and build/dep noise; **do not** ignore `local/`, `docs/`, or `repos/`.
