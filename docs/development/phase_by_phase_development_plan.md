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

## Phase 0 — Personal tool (validate for yourself first)

**Goal:** founder uses Orbicrew daily on real work before multi-tenant/sellable product work.

**Primary stack for this phase:**

- Postgres + pgvector (core tables; `tenant_id` present but single tenant)
- FastAPI + LangGraph Office Manager + 2–3 specialists
- Model router + budget guard, calling models through a small **in-app provider registry** — DeepInfra and OpenRouter to start (both give one account/one bill covering cheap-tier DeepSeek-class models and Claude), extensible to Bedrock/direct-Anthropic/direct-OpenAI as new registry entries, auto-cheapest-per-tier by default with per-agent manual override. Self-hosted LiteLLM (a separate gateway *server*) is intentionally deferred — it only earns its keep once Phase 2.3 needs per-tenant virtual keys/BYO routing (`tech/13_byo_provider_architecture.md`); the provider registry itself needs no new infrastructure to run since it's just outbound HTTP calls from the existing FastAPI service
- Basic Standard Dashboard (chat + task status) — **no Orbit View yet**
- Optional: personal voice STT/TTS once chat path works
- Shared deps via `resources/orbicrew_dev_infra` Compose; apps run natively

| # | Task | Detail | Depends on | Repos | Status |
|---|---|---|---|---|---|
| 0.1 | Local deps + app skeletons | Postgres+pgvector + Redis via `resources/orbicrew_dev_infra`; api/web health locally | — | master deps, api, web | **Done** — Compose healthy; FastAPI `/health`+`/ready`; Next.js status shell |
| 0.2 | Core schema | Tables from `tech/04_database_design.md` minus full billing surface | 0.1 | api | **Done** — SQL migrations in `repos/orbicrew-api/migrations/`, applied to local Postgres; RLS deferred to Phase 2 |
| 0.3 | LangGraph supervisor | Office Manager routes to hardcoded specialists | 0.2 | api | **Done** — `office_manager.py` graph + `POST/GET /v1/tasks`, seeded dev tenant/user in `bootstrap.py` |
| 0.4 | Model router + budget guard | Classify → tier; hard per-task cap | 0.3 | api | **Done** — `model_router.py` (tier + model + cost estimate) + `budget_guard.py` (hard per-task cap) wired as graph nodes in `office_manager.py`; over-cap tasks return `status: paused`, no spend recorded |
| 0.5 | 2–3 specialists | Pick from real need (e.g. writing, research, coding) | 0.4 | api | **Done** — `llm_client.py`; coding/writing/research/general specialist nodes call the real Anthropic API at the router-selected model; actual cost from real token usage persisted to `usage_records`/`tasks.spend_so_far_usd` |
| 0.6 | Standard chat GUI | Text I/O + task status | 0.4 | web | **Done** — `ChatForm` client component + `submitTaskAction` server action in `orbicrew-web`; posts to `POST /v1/tasks` server-side (no browser CORS) and renders status/specialist/tier/model/spend/output |
| 0.7 | OpenAPI + TS client | Typed contract web ↔ api | 0.3–0.6 | api, web | **Done** — `orbicrew-web` generates `src/lib/api-schema.d.ts` from `orbicrew-api`'s `/openapi.json` (`npm run codegen:api-types`, `openapi-typescript`) and calls it through a typed `openapi-fetch` client (`src/lib/api-client.ts`); `api-status.ts`/`api-tasks.ts` no longer hand-write `Task`/`SubmitTaskRequest`-shaped types |
| 0.8 | Voice (optional early) | Whisper STT + TTS for founder languages | 0.6 | api, web | **Done** — `orbicrew-api`: `voice.py` (`POST /v1/voice/transcribe` via OpenAI `whisper-1`, `POST /v1/voice/speak` via OpenAI `tts-1`); `orbicrew-web`: mic button in `ChatForm` records via `MediaRecorder`, transcribes through a same-origin Route Handler proxy, auto-submits the task, and auto-plays the spoken reply (with visible `<audio controls>` as an autoplay-blocked fallback) |
| 0.9 | Daily-use soak | 2–3 weeks real work before Phase 1 | all above | — | Pending |
| — | `orbicrew-admin` auth shell | Next.js scaffold + shared-secret operator login gating an empty operator console layout | — | admin | **Done** — `src/lib/session.ts` (HMAC-signed session cookie) + `/login` (shared-secret password check) gate `(operator)` route group; nav placeholders (Overview/Tenants/Costs/Kill switch), no `orbicrew-api` calls yet; full features + real operator auth remain Phase 2.6 |

**Exit criteria:** genuinely prefer using this over doing recurring task types yourself.

---

## Phase 1 — Overnight autonomy + multi-channel *(current)*

**Goal:** prove safe unattended execution and “one backend, many faces.”

Started ahead of the Phase 0.9 daily-use soak, at explicit user request — see `development-tracker.md` for the decision.

| # | Task | Detail | Depends on | Repos | Status |
|---|---|---|---|---|---|
| 1.1 | Task queue + checkpointing | Redis + Celery (or chosen queue); LangGraph persistence | Phase 0 | api, infra | **Done** — `arq` (Redis-backed, async-native) replaces synchronous execution in `POST /v1/tasks` (now enqueues, returns `status: "queued"` immediately); `src/orbicrew_api/worker.py` runs the Office Manager graph off-request; graph compiled with a Postgres-backed LangGraph checkpointer (`langgraph-checkpoint-postgres`) keyed by `thread_id = task_id`, so a crashed/restarted worker resumes from the last completed node instead of restarting. `orbicrew-web`'s `chat-form.tsx` now polls `GET /v1/tasks/:id` until a terminal status |
| 1.2 | Budget hardening | Nightly aggregate caps; retry-then-escalate | 1.1 | api | **Done** — `budget_guard.check_budget` reused for a rolling-24h per-tenant aggregate cap (`settings.tenant_daily_cap_usd`, checked alongside the per-task cap in `office_manager._route_condition`; pauses with `pause_type: "daily_cap"`); `worker.py` no longer fails a task on its first exception — it retries with exponential backoff (`task_max_retries`/`task_retry_base_delay_seconds`/`task_retry_max_delay_seconds`) by re-enqueuing the same `task_id` via arq's `_defer_by` (cheap: the LangGraph checkpointer resumes past already-completed nodes), tracked via new `tasks.retry_count`. Both a daily-cap pause and retries-exhausted failure now write an `approvals` escalation row (`migrations/0002_budget_hardening.sql` adds `tasks.retry_count` and `approvals.tenant_id`/`approval_type`) — the table was previously schema-only |
| 1.3 | Approval gates | `approvals` flow end-to-end | 1.2 | api, web | **Done** — scoped to resolving the two escalation types 1.2 already writes (not the docs' broader pre-action action-whitelist gate, deferred until a specialist performs an external/irreversible action); new `src/orbicrew_api/approvals.py`: `GET /v1/approvals?status=`, `POST /v1/approvals/{id}/approve`, `POST /v1/approvals/{id}/reject` (404/409 handled). Because the LangGraph checkpoint for a paused/failed thread is already at `END`, approve spawns a fresh task with the same input (new `create_and_enqueue_task` helper shared with `POST /v1/tasks`) rather than resuming in place, bypassing the specific cap that escalated via new `tasks.bypass_tenant_daily_cap` (a real boolean threaded through `OfficeManagerState` — an earlier `float("inf")` approach broke the checkpointer's Postgres JSON write and was caught in manual testing). Reject just marks the approval resolved. `migrations/0003_approval_gates.sql` adds `approvals.resulting_task_id` + a status check constraint. `orbicrew-web` gains its first second page, `/approvals` (list + Approve/Reject), linked from the home page |
| 1.3a | Web UI design parity with Stitch | Apply `resources/stitch_orbicrew_ui_ux_guide/` (sidebar nav shell, layout, typography, color, component styling) across existing `orbicrew-web` pages (`/`, `/approvals`) instead of the current unstyled/functional-only shell; establish reusable layout/nav primitives so later pages (1.4+, Phase 2) start from the Stitch design language rather than bare markup | 1.3 | web | **Done** — `globals.css` now carries the full `DESIGN.md` Material-3-style token set (surface/on-surface/primary/secondary/error containers, 8px spacing scale, DESIGN.md's radius scale) via Tailwind v4's `@theme inline`, replacing the old 10-token placeholder set; new `src/components/sidebar-nav.tsx` + `app-shell.tsx` (mounted once in root `layout.tsx`) provide the fixed sidebar (Home/Approvals live, Agents/Tasks/Billing/Settings + "Hire New Agent" rendered disabled since those pages don't exist yet) and a minimal mobile header — no search/notifications/avatar chrome, since none of that is backed by real functionality yet (explicit scope call, see tracker). `/` and `/approvals` restyled into the Stitch card language with their existing real data/behavior unchanged (no fabricated stats or fictional mockup content ported over). **Follow-up round (same day):** dark/light/system theming (locked-brand dark palette added, class-based Tailwind dark mode, header toggle, default dark), real logo/favicon wired from `resources/logos/`, header made global (was mobile-only) with account menu, sidebar active-state background fixed, `/` rebuilt into a real data-backed Overview dashboard (new `GET /v1/dashboard/summary`), and chat moved off `/` onto a new `/tasks` page restyled as a message-bubble thread per `orbicrew_message_agents`. See tracker for full detail |
| 1.4 | Digest | Consolidated activity report — "since last checked," not time-of-day-scoped (see `03_system_design.md` §7) | 1.3a | api, channels/web | **Done**, renamed from "Morning summary" 2026-08-07 — `GET/POST /v1/dashboard/digest*` in `orbicrew-api` (`digest.py`, migration `0004_morning_summary.sql` + `0006_digest_rename_and_schedule.sql`; `tenants.last_digest_viewed_at` is the "since last viewed" watermark, falling back to a 24h lookback on first view); delivered on the `orbicrew-web` dashboard as `/digest` — completed/paused/failed task lists + total spend + approvals requested in that window (any resolution status), with an explicit "Mark as reviewed" action. Explicit `period_start`/`period_end` browses any historical range live without touching the watermark — this **is** the history mechanism, no separate storage or scheduler (a scheduled-generation approach was built and then deliberately reverted the same day — see tracker). All timestamps UTC in the API/DB, converted to the viewer's local timezone for display |
| 1.4a | Agent Roster + Configuration UI (Phase 2.4 pulled forward) | Real `/agents` + `/agents/[id]` backed by the previously-unused `agents`/`agent_memory` tables; Master Agent promotion | 1.3a | api, web | **Done** — see Phase 2.4 row and tracker entry for full detail |
| 1.5 | Telegram adapter | First multi-channel proof | Core API stable | channels | **Done** — new `orbicrew-channels` project (Python, `python-telegram-bot`): thin adapter calling `orbicrew-api`'s real REST endpoints (`POST/GET /v1/tasks`, `GET/POST /v1/approvals/*`) — deliberately **not** the WebSocket status-streaming protocol `tech/05_api_design.md` §5/§6 describes, since that stream isn't implemented in `orbicrew-api` yet; polls `GET /v1/tasks/:id` and edits the reply message in place, same pattern `orbicrew-web`'s chat form already uses. `/approvals` renders pending escalations with inline Approve/Reject buttons. Gated to a single `TELEGRAM_ALLOWED_CHAT_ID` — `orbicrew-api` has no per-tenant auth yet (one hardcoded `DEFAULT_TENANT_ID` everywhere), so this is the Phase 1 personal-use slice explicitly, not the multi-tenant BYO-bot-per-workspace design; that becomes real once Phase 2.1 (tenant model + auth) lands — see tracker for the scope discussion. Verified live against a real bot (@orbicrew_dev_bot) |
| 1.6 | WhatsApp adapter | Start Business API verification early (external lead time) | 1.5 pattern | channels | **Blocked** — code built and unit-tested (34/34 passing: webhook parsing, signature verification, allow-list gating, dispatch handlers), same shape as the Telegram adapter but a FastAPI webhook server instead of a polling bot, since Meta delivers inbound messages via webhook POST (no live-status message-edit either — Cloud API has no edit endpoint for text messages, so updates are separate sends). Cannot verify live: Meta's WhatsApp Cloud API requires **business verification** to issue a real Phone Number ID, which in turn requires an incorporated business — user does not have one yet. This confirms the task's own "external lead time" rationale. Revisit once the business is incorporated and verification clears |
| 1.7 | Research agent + tools | Web search/fetch wired for real research | Phase 0 agents | api | Pending |
| 1.8 | Multi-provider cost-tier routing | Replace `llm_client.py`'s Anthropic-only call with a small **in-app provider registry** (no self-hosted gateway/server — just an HTTP call to a different `base_url` per provider from the existing FastAPI service): starts with **DeepInfra** and **OpenRouter** as registered providers, each mapped against `trivial`/`cheap`/`mid`/`frontier` tiers. **Auto mode (default)**: router picks whichever registered provider is cheapest for the tier it needs, with automatic fallback to the next-cheapest on error/unavailability. **Manual override, per agent**: reuses the `agents.model_mode`/`manual_model` columns already added in 1.4a — `manual_model` becomes provider-qualified (e.g. `deepinfra/claude-sonnet-5`) so a user can pin a specific agent to a specific provider+model. **Designed to extend without a rewrite**: adding AWS Bedrock, a direct Anthropic key, or a direct OpenAI key later is just a new registry entry, not an architecture change — this is deliberately the same shape as the BYO Pattern A/B/C design in `tech/13_byo_provider_architecture.md`, scoped down to single-user for now so Phase 2.3's eventual LiteLLM migration is an upgrade of this pattern, not a rewrite. Closes the gap `model_router.py`'s own comment already flagged ("self-hosted LiteLLM / OpenRouter routing is deferred") and reaffirms the original Phase 0 spec (§12 of `01_AI_Office_Platform_Requirements.md` always called for "OpenRouter-based model routing"). Coding-agent work (opencode/Claude Code) stays external to Orbicrew, pointed at whichever registered provider, rather than duplicated as an in-app specialist | Phase 0.4–0.5, 1.4a | api | Pending |

**Exit criteria:** hand off a real project unattended; check back to a consolidated Digest of what happened; at least one messaging channel works alongside the chat GUI.

---

## Phase 2 — Multi-tenancy & pricing

**Goal:** sellable product; onboard 3–5 pilot customers.

| # | Task | Detail | Depends on | Repos | Status |
|---|---|---|---|---|---|
| 2.1 | Tenant model + RLS | Full multi-tenant enforcement | Phase 1 stable | api | |
| 2.2 | Billing / subscriptions | Stripe (or similar) + usage records / tiers; tenant billing UI | 2.1 | api, web | |
| 2.3 | BYO key management | Encrypted keys; LiteLLM virtual/tenant routing | 2.1 | api, web | |
| 2.4 | Agent CRUD UI | User create/edit/delete agents (Memory/Skills/Soul/Setting) | 2.1 | web, api | **Pulled forward and done (2026-08-07, pre-Phase-2, single-tenant)** — see tracker; `agents`/`agent_memory` tables (unused since 0001) now have a real CRUD API and `/agents` + `/agents/[id]` UI, including Master Agent promotion (any agent promotable, DB-enforced one-per-tenant). Remaining Phase 2.4 scope once multi-tenancy lands: per-tenant RLS on the endpoints (currently single default tenant like the rest of Phase 0/1), and the mockup's "Generate with AI" flow (not built) |
| 2.5 | Tenant settings admin | Seats/roles, per-agent kill switches, plan limits in Dashboard settings | 2.1 | web, api | |
| 2.6 | Platform operator console | Full `orbicrew-admin` features: cross-tenant cost, tenant lifecycle, platform-wide kill switch (build on early auth shell) | 2.1 | admin, api | |
| 2.7 | Project isolation | `project_id` scoping per hallucination/isolation docs | 2.1 | api, web | |
| 2.8 | Pilot | 3–5 real agencies/freelancers | all above | — | |

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
4. ~~Phase 0.5: 2–3 real specialists, replacing the canned stub output.~~ **Done**.
5. ~~Phase 0.6: thinnest chat UI that submits a task and shows status.~~ **Done**.
6. ~~Phase 0.7: OpenAPI + generated TS client (replace the hand-written `Task`/`SubmitTaskRequest` types in `orbicrew-web`).~~ **Done**.
7. ~~`orbicrew-admin` auth shell (empty operator layout).~~ **Done**.
8. ~~Phase 0.8: voice (optional) once the text chat path is in daily use.~~ **Done**.
9. Phase 0.9: daily-use soak — 2-3 weeks of real personal use before starting Phase 1 (superseded — Phase 1 started early at explicit user request; see `development-tracker.md`).
10. ~~Phase 1.1: task queue (`arq`) + LangGraph Postgres checkpointing.~~ **Done**.
11. ~~Phase 1.2: budget hardening — nightly (rolling 24h) aggregate caps + retry-then-escalate.~~ **Done**.
12. ~~Phase 1.3: approval gates — read/resolve API + UI for the `approvals` rows Phase 1.2 now writes.~~ **Done**.
13. ~~Phase 1.3a: web UI design parity — restyle existing pages (`/`, `/approvals`) to match `resources/stitch_orbicrew_ui_ux_guide/` (sidebar nav shell, layout, typography, color, components) before 1.4 adds another un-styled page.~~ **Done**.
14. ~~Phase 1.4: Digest (renamed from "morning summary") — consolidated activity report, "since last checked" plus history via an explicit date-range query (no scheduler — a scheduled-generation approach was tried and reverted the same day, see tracker).~~ **Done**.
15. **Phase 1.8: Multi-provider cost-tier routing** (DeepInfra + OpenRouter now, extensible to Bedrock/direct-Anthropic/direct-OpenAI later) — sequenced ahead of 1.5/1.6/1.7 at explicit user request (2026-08-25): direct cost impact on real daily usage outweighs adding another channel or agent first. See tracker for the full decision context.
16. Phase 1.5: Telegram adapter — first multi-channel proof.
17. Phase 1.6: WhatsApp adapter.
18. Phase 1.7: Research agent + tools — real web search/fetch.

Update the development tracker after each completed slice.
