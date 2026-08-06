# Phase-by-Phase Development Plan — Orbicrew

Living engineering plan for this workspace. Derived from product docs under `docs/leangine-office-docs/` (especially `tech/07_phased_development_plan.md`, `01_AI_Office_Platform_Requirements.md`, and `tech/03_system_design.md`).

**Agents:** follow this plan before feature work. Prefer finishing a phase’s exit criteria over jumping ahead. Update this file when sequencing decisions change.

**Related process docs:**

- Bootstrap: `AGENT_BOOTSTRAP.md`
- Manual tests: `manual-test-guide.md`
- Session log: `development-tracker.md`

---

## Guiding sequencing principles

1. **Cost-control infrastructure before feature breadth** — router + budget guard before a large agent roster.
2. **Reliability before Orbit View** — Standard Dashboard first; Orbit View is differentiation (later).
3. **Multi-tenant schema thinking early** — include `tenant_id` from day one even when Phase 0 is single-user; turn on RLS/billing in Phase 2.
4. **One backend, many faces** — adapters stay thin; no agent logic in channels or UI.
5. **Exit criteria are usage-based**, not “code exists.”
6. **Prefer reusable shared abstractions** (DRY / shared packages) when building across services — follow stack best practices.

---

## Admin surfaces (decision)

Product docs call for (a) tenant-side admin (roles, billing, per-agent kill switches) and (b) Leangine-side ops (aggregate cost monitoring, platform-wide kill switch). **Decision: dedicated `orbicrew-admin` repo from day one** (not embedded `/admin` in `orbicrew-web`).

| Surface | Audience | Home | Phase |
|---|---|---|---|
| Tenant settings | Tenant owner/admin | `orbicrew-web` Standard Dashboard / settings (billing, seats, roles, agent kill switches) | Phase 2 |
| Platform operator console | Leangine operators | Dedicated `orbicrew-admin` app (separate authz / deploy; privileged `/v1/ops/*` on `orbicrew-api`) | Scaffold/auth shell early; full features Phase 2 |
| Operator APIs | Same | `orbicrew-api` privileged endpoints (RLS-aware, operator role) | Phase 2 |

UI references for dashboard/settings/billing: master `resources/stitch_orbicrew_ui_ux_guide/` and `resources/logos/` (copy into web or admin as needed; never hard-link nested repos to master paths).

---

## Local shared dependencies

- **Deps Compose (master):** `resources/orbicrew_dev_infra/` — Postgres+pgvector, Redis. Use `docker compose up -d` anytime.
- **Apps:** implement and run in `repos/*` **natively** during development (api, web, admin, channels). Do not run those app services in Docker day-to-day.
- **`orbicrew-infra`:** staging/prod-oriented Compose and shared ops — not the day-to-day shared-deps path.

---

## Repo ownership by phase

| Phase | Primary repos |
|---|---|
| Setup | Master workspace + scaffold all `repos/*` (including `orbicrew-admin`) |
| 0 — Personal tool | `orbicrew-api`, `orbicrew-web`; shared deps via `resources/orbicrew_dev_infra`; optional admin auth shell |
| 1 — Overnight + channels | + `orbicrew-channels` (Telegram/WhatsApp); deepen api |
| 2 — Multi-tenancy & pricing | `orbicrew-api`, `orbicrew-web` (tenant settings), `orbicrew-admin` (operator console) |
| 3 — Differentiation | `orbicrew-web` (Orbit View), channels (Discord), integrations |
---

## Phase -1 — Workspace & agentic setup

**Goal:** reproducible agentic coding environment; empty-but-branched project repos; process docs ready.

| Task | Detail | Status |
|---|---|---|
| Master git + ignores | Root git; ignore `local/`, `repos/*` contents, secrets; track `resources/` | Done — remote `https://github.com/meetrakib/orbicrew_master` |
| Agent instructions | `AGENTS.md`, `CLAUDE.md`, `.cursor/rules`, bootstrap | Done (nested isolation + DRY principles) |
| Process docs | This plan, tracker, manual test guide | Done |
| Scaffold org repos | `orbicrew-web`, `orbicrew-api`, `orbicrew-channels`, `orbicrew-infra`, `orbicrew-admin` with `main`/`stage`/`dev` | Done — remotes pushed to `leangine/*` |
| Nested README scrub | Org repos self-contained (no master-workspace references) | Done |
| Admin planning | Dedicated `orbicrew-admin`; tenant settings stay in `orbicrew-web` | Done — decision reversed from admin-inside-web |
| Dev deps Compose | `resources/orbicrew_dev_infra` (Postgres+pgvector, Redis); apps run natively | Done |

**Exit criteria:** a new agent can read `AGENT_BOOTSTRAP.md` and know where to work without a human re-briefing the product. **Met.** Org remotes connected and `main`/`stage`/`dev` pushed.

---

## Phase 0 — Personal tool (validate for yourself first) *(current)*

**Goal:** founder uses Orbicrew daily on real work before multi-tenant/sellable product work.

**Primary stack for this phase:**

- Postgres + pgvector (core tables; `tenant_id` present but single tenant)
- FastAPI + LangGraph Office Manager + 2–3 specialists
- Model router + budget guard + LiteLLM (or equivalent local/dev path)
- Basic Standard Dashboard (chat + task status) — **no Orbit View yet**
- Optional: personal voice STT/TTS once chat path works
- Shared deps via `resources/orbicrew_dev_infra` Compose; apps run natively

| # | Task | Detail | Depends on | Repos | Status |
|---|---|---|---|---|---|
| 0.1 | Local deps + app skeletons | Postgres+pgvector + Redis via `resources/orbicrew_dev_infra`; api/web health locally | — | master deps, api, web | **Done** — Compose healthy; FastAPI `/health`+`/ready`; Next.js status shell |
| 0.2 | Core schema | Tables from `tech/04_database_design.md` minus full billing surface | 0.1 | api | **Done** — SQL migrations in `repos/orbicrew-api/migrations/`, applied to local Postgres; RLS deferred to Phase 2 |
| 0.3 | LangGraph supervisor | Office Manager routes to hardcoded specialists | 0.2 | api | **Done** — `office_manager.py` graph + `POST/GET /v1/tasks`, seeded dev tenant/user in `bootstrap.py` |
| 0.4 | Model router + budget guard | Classify → tier; hard per-task cap | 0.3 | api | **Done** — `model_router.py` (tier + model + cost estimate) + `budget_guard.py` (hard per-task cap) wired as graph nodes in `office_manager.py`; over-cap tasks return `status: paused`, no spend recorded |
| 0.5 | 2–3 specialists | Pick from real need (e.g. writing, research, coding) | 0.4 | api | Pending |
| 0.6 | Standard chat GUI | Text I/O + task status | 0.4 | web | Pending |
| 0.7 | OpenAPI + TS client | Typed contract web ↔ api | 0.3–0.6 | api, web | Pending |
| 0.8 | Voice (optional early) | Whisper STT + TTS for founder languages | 0.6 | api, web | Pending |
| 0.9 | Daily-use soak | 2–3 weeks real work before Phase 1 | all above | — | Pending |

**Exit criteria:** genuinely prefer using this over doing recurring task types yourself.

---

## Phase 1 — Overnight autonomy + multi-channel

**Goal:** prove safe unattended execution and “one backend, many faces.”

| # | Task | Detail | Depends on | Repos |
|---|---|---|---|---|
| 1.1 | Task queue + checkpointing | Redis + Celery (or chosen queue); LangGraph persistence | Phase 0 | api, infra |
| 1.2 | Budget hardening | Nightly aggregate caps; retry-then-escalate | 1.1 | api |
| 1.3 | Approval gates | `approvals` flow end-to-end | 1.2 | api, web |
| 1.4 | Morning summary | Consolidated overnight report on preferred channel | 1.3 | api, channels/web |
| 1.5 | Telegram adapter | First multi-channel proof | Core API stable | channels |
| 1.6 | WhatsApp adapter | Start Business API verification early (external lead time) | 1.5 pattern | channels |
| 1.7 | Research agent + tools | Web search/fetch wired for real research | Phase 0 agents | api |

**Exit criteria:** hand off a real project at night; wake to a morning summary; at least one messaging channel works alongside the chat GUI.

---

## Phase 2 — Multi-tenancy & pricing

**Goal:** sellable product; onboard 3–5 pilot customers.

| # | Task | Detail | Depends on | Repos |
|---|---|---|---|---|
| 2.1 | Tenant model + RLS | Full multi-tenant enforcement | Phase 1 stable | api |
| 2.2 | Billing / subscriptions | Stripe (or similar) + usage records / tiers; tenant billing UI | 2.1 | api, web |
| 2.3 | BYO key management | Encrypted keys; LiteLLM virtual/tenant routing | 2.1 | api, web |
| 2.4 | Agent CRUD UI | User create/edit/delete agents (Memory/Skills/Soul/Setting) | 2.1 | web, api |
| 2.5 | Tenant settings admin | Seats/roles, per-agent kill switches, plan limits in Dashboard settings | 2.1 | web, api |
| 2.6 | Platform operator console | Full `orbicrew-admin` features: cross-tenant cost, tenant lifecycle, platform-wide kill switch (build on early auth shell) | 2.1 | admin, api |
| 2.7 | Project isolation | `project_id` scoping per hallucination/isolation docs | 2.1 | api, web |
| 2.8 | Pilot | 3–5 real agencies/freelancers | all above | — |

**Exit criteria:** ≥3 paying or committed pilots generating real usage data; operators can monitor aggregate cost and halt autonomy platform-wide.

---

## Phase 3 — Differentiation & scale

**Goal:** moat and retention from real pilot feedback — not guessed features.

| # | Task | Detail | Depends on | Repos |
|---|---|---|---|---|
| 3.1 | Orbit View | Original Phaser + DOM HUD per `tech/18_*` — not Agent Town fork | Phase 2 + adapters proven | web, api |
| 3.2 | Discord adapter | Same thin-adapter pattern | Adapter pattern | channels |
| 3.3 | Deeper integrations | Gmail/Drive/Slack/Notion/etc. prioritized by pilots | Pilots underway | api |
| 3.4 | Meta-agent | System-proposed agent configs with human approval | Agent CRUD stable | api, web |
| 3.5 | Multilingual STT/TTS tuning | Provider choice from real usage data | Pilots | api |

**Exit criteria:** visible differentiation in usage/retention (cost transparency, Orbit View engagement, voice where relevant).

---

## Phase 4+ — Later / explicit non-goals for now

Do **not** prioritize unless re-scoped:

- Full visual workflow builder (n8n-style)
- Open third-party tool marketplace
- Full UI localization (product chrome languages) — multilingual **chat** is earlier; chrome i18n is later
- Kubernetes before real multi-instance need
- LangGraph Platform / LiteLLM Enterprise paid locks — prefer core OSS + own app layer (`tech/16_open_source_licensing_compliance.md`)

---

## Suggested immediate next engineering slice

1. ~~Connect org GitHub remotes for scaffolding repos and push `main`/`stage`/`dev`.~~ **Done** (including `orbicrew-admin`).
2. ~~Phase 0.1: deps Compose + native api/web health.~~ **Done**.
3. ~~Phase 0.2–0.4: schema + Office Manager + router/budget guard vertical slice.~~ **Done**.
4. Phase 0.5: 2–3 real specialists, replacing the canned stub output.
5. Phase 0.6: thinnest chat UI that submits a task and shows status.
6. Early: `orbicrew-admin` auth shell (empty operator layout) when convenient; full ops UI remains Phase 2.6.

Update the development tracker after each completed slice.
