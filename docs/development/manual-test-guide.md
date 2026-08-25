# Manual Test Guide — Orbicrew

Living checklist of **manual** verification steps. Automated tests live in each service repo; this guide is for human (or agent-assisted) smoke/acceptance checks after development slices.

**Agents:** update this file when you add or change behavior that needs a repeatable manual check. Tie new sections to the phase/task that introduced them.

---

## How to use

1. Pick the environment: `local` (default) | `stage` | `prod` (prod only when intentional).
2. Run relevant sections after your change — not the whole document every time.
3. Record results briefly in `development-tracker.md` (pass/fail + notes).

**Prerequisites (fill in as stack comes up):**

| Prerequisite | Local how-to | Status |
|---|---|---|
| Shared deps (Postgres, Redis) | `resources/orbicrew_dev_infra` — `docker compose up -d` | Available |
| API reachable | Native: `cd repos/orbicrew-api && uv sync && uv run orbicrew-api` → `http://localhost:8000` | Available (Phase 0.1) |
| Worker running | Native: `cd repos/orbicrew-api && uv run arq orbicrew_api.worker.WorkerSettings` (separate process; executes queued tasks) | Available (Phase 1.1) |
| Web reachable | Native: `cd repos/orbicrew-web && npm install && npm run dev` → `http://localhost:3000` | Available (Phase 0.1) |
| Admin reachable | Native: `cd repos/orbicrew-admin && npm install && npm run dev` → `http://localhost:3000` (or next free port) | Available (auth shell) |
| Test tenant / user | TBD | Not yet |
| Model keys / LiteLLM | `.env` (never commit) | Not yet |

---

## Workspace smoke (always applicable)

| # | Check | Expected |
|---|---|---|
| W1 | Master repo ignores `local/` | `git check-ignore -v local/` succeeds |
| W2 | Master repo ignores nested repo contents | `repos/orbicrew-web/README.md` is ignored by master git (except `repos/README.md`) |
| W3 | Agent docs present | `AGENT_BOOTSTRAP.md`, phase plan, tracker, `AGENTS.md`, `CLAUDE.md` exist |
| W4 | Nested repos have `main`, `stage`, `dev` | `git branch` in each `repos/*` shows all three |
| W5 | Nested READMEs stand alone | No references to `orbicrew_master`, `docs/development`, or `AGENT_BOOTSTRAP` inside `repos/orbicrew-*/` |
| W6 | `resources/` tracked | `resources/README.md`, Stitch guide, and `orbicrew_dev_infra` present; not gitignored |
| W7 | Dev deps Compose | `resources/orbicrew_dev_infra/docker-compose.yml` defines Postgres+pgvector and Redis |
| W8 | Dedicated admin scaffold | `repos/orbicrew-admin` exists with README; platform admin not documented as `/admin` in web |
| W9 | Default branch is `dev` | Working checkout for master + each `repos/*` is `dev` for daily commits/pushes |
| W10 | Master has `main`/`stage`/`dev` | `git branch -a` on master shows all three (local + origin after push) |

---

## Phase 0 — Personal tool

### 0.A Local stack

| # | Check | Expected | Status |
|---|---|---|---|
| 0.A.1 | Deps compose up | `docker compose ps` shows Postgres + Redis **healthy** in `resources/orbicrew_dev_infra` | Pass (2026-08-07) |
| 0.A.2 | API liveness | `curl http://localhost:8000/health` → `{"status":"ok",...}` | Pass (2026-08-07) |
| 0.A.3 | API readiness | `curl http://localhost:8000/ready` → `status: ready`, postgres+redis `ok` (HTTP 200) | Pass (2026-08-07) |
| 0.A.4 | Web status shell | `http://localhost:3000` shows Orbicrew status with API **healthy** and postgres/redis ok | Pass (2026-08-07) |

### 0.A.5 Core schema migration

| # | Check | Expected | Status |
|---|---|---|---|
| 0.A.5.1 | Apply migrations | `cd repos/orbicrew-api && uv run orbicrew-api-migrate` → applies `0001_core_schema.sql` | Pass (2026-08-07) |
| 0.A.5.2 | Idempotent re-run | Running again prints "No pending migrations." | Pass (2026-08-07) |
| 0.A.5.3 | Tables present | `docker exec orbicrew-postgres psql -U orbicrew -d orbicrew -c '\dt'` shows all 13 core tables + `schema_migrations` | Pass (2026-08-07) |

### 0.A.6 Typed API contract (OpenAPI + TS client)

| # | Check | Expected | Status |
|---|---|---|---|
| 0.A.6.1 | Codegen from live API | `cd repos/orbicrew-web && npm run codegen:api-types` (API running) → regenerates `src/lib/api-schema.d.ts` with no errors | Pass (2026-08-07) |
| 0.A.6.2 | Typecheck | `npx tsc --noEmit` in `orbicrew-web` — clean | Pass (2026-08-07) |
| 0.A.6.3 | Status page still works through typed client | `http://localhost:3000` shows API **healthy**, postgres/redis `ok` | Pass (2026-08-07) |
| 0.A.6.4 | Chat submit still works through typed client | Submitting a prompt returns a real, non-canned task with matching `TaskResponse` shape (specialist/tier/model/spend/output) | Pass (2026-08-07) — verified by calling `submitTask()` directly against the live API |

### 0.B Task path (core)

| # | Check | Expected | Status |
|---|---|---|---|
| 0.B.1 | Submit text task via web | Task appears queued → running → done/failed | Pass (2026-08-07) — `http://localhost:3000` "Ask the Office Manager" form submits via a server action to `POST /v1/tasks` and renders status/specialist/tier/model/spend/output; verified with Playwright (real haiku prompt → `writing`/`cheap`/`claude-haiku-4-5` real output) |
| 0.B.2 | Office Manager routes | Specialist appropriate to task is selected | Pass (2026-08-07, via API) — `POST /v1/tasks` with coding/writing/research/general phrasing returns the matching `specialist` |
| 0.B.3 | Budget guard | Task over cap pauses / fails safely (no runaway spend) | Pass (2026-08-07) — `POST /v1/tasks` with `budget_cap_usd: 0.01` against a frontier-tier phrasing returns `status: "paused"`, `spend_so_far_usd: 0`, and no `usage_records` row is written |
| 0.B.4 | Router prefers cheap tier | Simple task does not hit frontier model (verify in usage logs) | Pass (2026-08-07) — short/simple phrasing routes to `cheap`/`trivial` tier (`claude-haiku` / free tier); only hard-keyword phrasing (e.g. "debug production architecture") escalates to `frontier` (`claude-opus`); confirmed via `usage_records` rows |
| 0.B.5 | Task persisted with step trace | `GET /v1/tasks/:id` returns `classify` + `specialist_execute` steps in order; unknown id → 404 | Pass (2026-08-07) |
| 0.B.6 | Specialists call a real model | `POST /v1/tasks` (with `ANTHROPIC_API_KEY` set) returns real, non-canned specialist output; `usage_records` row has non-zero `input_tokens`/`output_tokens` and a `cost_usd` computed from them (not the flat per-tier estimate) | Pass (2026-08-07) — verified for coding- and research-routed phrasing; `psql` confirmed real token counts on the `usage_records` row |

### 0.C Agents

| # | Check | Expected | Status |
|---|---|---|---|
| 0.C.1 | Specialist A happy path | Completes a real thin task with artifact or clear result | Pending |
| 0.C.2 | Specialist B happy path | Same | Pending |
| 0.C.3 | Failure handling | Failed tool/model call surfaces clear status, no silent hang | Pending |

### 0.D Voice

| # | Check | Expected | Status |
|---|---|---|---|
| 0.D.1 | STT | Spoken input becomes correct-enough text in primary language | Pass (2026-08-07) — Playwright + Chromium fake mic (`--use-fake-device-for-media-stream` + `--use-fake-ui-for-media-stream`) fed a real synthesized-speech WAV; `POST /v1/voice/transcribe` (OpenAI `whisper-1`) returned the correct text and filled the chat textarea |
| 0.D.2 | Auto-submit after transcription | Recording stop transcribes then submits the task without a manual Send click | Pass (2026-08-07) — task auto-submitted via `formRef.requestSubmit()`, routed to the writing specialist, real `claude-haiku-4-5` output rendered |
| 0.D.3 | TTS | Reply plays back | Pass (2026-08-07) — on task completion, `POST /v1/voice/speak` (OpenAI `tts-1`) returns real MP3 bytes; web auto-plays via a blob-URL `<audio autoPlay controls>` (controls stay visible as a manual-replay/autoplay-blocked fallback) |
| 0.D.4 | Direct API round-trip | `POST /v1/voice/speak` → `POST /v1/voice/transcribe` on the resulting audio returns matching text | Pass (2026-08-07) — `curl` round-trip: "Hello from Orbicrew." → real MP3 → transcribed back to "Hello from Orbicrew." |

### 0.E Admin auth shell

| # | Check | Expected | Status |
|---|---|---|---|
| 0.E.1 | Unauthenticated root redirect | Visiting `/` in `orbicrew-admin` without a session cookie redirects to `/login` | Pass (2026-08-07) |
| 0.E.2 | Wrong password rejected | Submitting an incorrect operator password on `/login` stays on `/login` and shows "Incorrect operator password." | Pass (2026-08-07) |
| 0.E.3 | Correct password signs in | Submitting the correct `ORBICREW_ADMIN_OPERATOR_PASSWORD` sets a signed session cookie and redirects to `/` (operator console placeholder) | Pass (2026-08-07) |
| 0.E.4 | Session persists on reload | Reloading `/` after signing in stays on `/` (cookie still valid) | Pass (2026-08-07) |
| 0.E.5 | Sign out clears session | Clicking "Sign out" redirects to `/login`; visiting `/` again afterward redirects back to `/login` | Pass (2026-08-07) |

---

## Phase 1 — Overnight + channels

| # | Check | Expected | Status |
|---|---|---|---|
| 1.1 | Overnight handoff | `POST /v1/tasks` → `status: queued`; submit, `kill -9` the worker mid-run, confirm the task is stuck at `running` with checkpoint rows in Postgres (`checkpoints` table, keyed by `thread_id = task_id`); restart the worker, re-enqueue the same task id; task completes without re-running `classify`/`route` (checkpoint count grows by exactly one, `task_steps` has no duplicates) | Pass (2026-08-07) |
| 1.2a | Tenant daily cap | Lower `TENANT_DAILY_CAP_USD` (or submit enough tasks) so `sum(usage_records.cost_usd)` over the trailing 24h plus a new task's estimated cost exceeds the cap → task ends `status: paused` with `steps` ending in `daily_cap_pause`, no new `usage_records` row, and an `approvals` row (`approval_type = 'tenant_daily_cap_exceeded'`) appears for the task | Pass (2026-08-07) |
| 1.2b | Retry-then-escalate | Force execution failures (e.g. temporarily set `ANTHROPIC_API_KEY` to an invalid value, optionally lower `TASK_MAX_RETRIES`/`TASK_RETRY_BASE_DELAY_SECONDS` for a fast test) → `tasks.retry_count` increments and `status` cycles `running`→`queued`→`running` on each attempt; once `retry_count` exceeds `TASK_MAX_RETRIES`, task ends `status: failed` and an `approvals` row (`approval_type = 'task_failure_escalation'`) appears with the error message | Pass (2026-08-07) |
| 1.3a | Approve daily-cap escalation | `GET /v1/approvals` lists a pending `tenant_daily_cap_exceeded` row; `POST /v1/approvals/{id}/approve` → approval becomes `status: approved` with `resulting_task_id` set; the new task completes normally (`status: done`, `usage_records` written) despite the tenant cap still being exceeded; original task/approval untouched otherwise | Pass (2026-08-07) |
| 1.3b | Reject escalation | `POST /v1/approvals/{id}/reject` on a `task_failure_escalation` row → `status: rejected`, `resulting_task_id` stays null, no new task row created, original task stays `failed` | Pass (2026-08-07) |
| 1.3c | Approval conflict/missing | `POST .../approve` or `.../reject` on an already-resolved id → 409; on a missing id → 404 | Pass (2026-08-07) |
| 1.3d | Web UI shell (design parity) | `/` and `/approvals` render inside the shared Stitch-styled sidebar shell: correct nav item is highlighted active on each route, nav items with no page yet (Agents/Tasks/Billing/Settings) and "Hire New Agent" are visibly muted and non-interactive, sidebar hides below `md` with a mobile top bar taking its place, existing chat submit/poll and approvals list still render real data through the new card styling | Pass (2026-08-07) — Playwright screenshots at 1440px and 390px widths against the live dev server confirmed active-state highlighting, disabled nav items, responsive sidebar/mobile-header swap, and a live-submitted task rendering its polled status inside the restyled card |
| 1.3e | Web UI: theming, global header, real dashboard, task chat | Dark/light/system theme toggle (header, default dark) applies instantly with no flash-of-wrong-theme on reload; brand wordmark/favicon swap correctly by theme; header (notifications/theme toggle/account menu) and footer are global on every page (`/`, `/tasks`, `/approvals`), not page-local; `/` is a real Overview dashboard (`GET /v1/dashboard/summary`) — Active Tasks / Spend today vs daily cap / Pending Approvals stat cards, a real recent-tasks Live Activity feed, and a real pending-approvals Needs Attention panel (Approve/Reject inline); chat moved to `/tasks`, restyled as a message-bubble thread (user bubble → typing indicator → specialist agent bubble with real output/model/cost) | Pass (2026-08-07) — Playwright screenshots (dark + light, 1440px) of `/`, `/tasks`, `/approvals`; theme cycle dark→light→system verified via header button clicks with no hydration-mismatch console error (`suppressHydrationWarning` on `<html>`); live end-to-end task submission from `/tasks` (arq worker started for this check only) completed for real (`claude-haiku-4-5`, real haiku output, real `$0.0002` spend) and rendered correctly in the bubble thread; dashboard stat cards cross-checked against real `tasks`/`usage_records` rows |
| 1.4 | Digest | `GET /v1/dashboard/digest` splits tasks completed/paused/failed since the tenant's last-viewed watermark (`tenants.last_digest_viewed_at`, falling back to a 24h lookback on first-ever view) into their own lists, sums their spend, and lists approvals requested in that window (any status); `/digest` in `orbicrew-web` renders it with a "Mark as reviewed" action; after marking, a fresh `GET` (and page reload) shows the empty state until new activity occurs. Not time-of-day-scoped — "since last checked," any time of day. Passing explicit `period_start`/`period_end` browses an arbitrary historical range without touching the watermark — this **is** the history feature (Today/Yesterday/Last 7 days/Last 30 days preset buttons on the page, computed client-side in the viewer's local timezone then sent as UTC), no separate storage or background job. All timestamps render in the viewer's local timezone (`toLocaleString()`) while the API/DB stay UTC throughout | Pass (2026-08-07) — verified against real data accumulated from earlier manual runs (37 done, 4 paused, 2 failed tasks; 4 approvals split 2 approved/2 rejected) via `curl` against the live API and the SSR'd `/digest` page; `POST /v1/dashboard/digest/viewed` advanced the watermark and a follow-up `GET`/page reload correctly showed all four sections empty; explicit-range browsing verified to leave the watermark untouched; history preset buttons confirmed present on the rendered page |
| 1.4b | Tasks page header parity | `/tasks` header shows the active task's input text (truncated) as the page title plus a status badge (Active Task / Completed / Paused / Failed) instead of a static "Tasks" heading, matching `orbicrew_message_agents`'s per-task title bar; empty state still shows the generic "Tasks" heading and helper copy | Pass (2026-08-07) — Playwright: empty `/tasks` shows generic heading; submitting a real task ("Draft a 3-page summary on minimalist SaaS UI trends") updates the heading to the task text and shows an "Active Task" badge while `status` is `queued`/`running` |
| 1.4a | Agent Roster + Configuration (pulled forward from 2.4) | `GET /v1/agents` lists real agents (no fabricated cards); `POST /v1/agents` creates one, first-ever agent for a tenant auto-becomes Master; `POST /v1/agents/{id}/promote` demotes the previous Master and promotes the target inside one transaction (DB has a partial unique index enforcing exactly one Master per tenant); `PATCH /v1/agents/{id}` persists Soul/Skills/Setting tab edits; `POST`/`DELETE /v1/agents/{id}/memories` add/remove Memory tab entries; `/agents` and `/agents/[id]` in `orbicrew-web` render this real data styled per `resources/stitch_orbicrew_ui_ux_guide/orbicrew_agent_roster` and `orbicrew_agent_configuration` | Pass (2026-08-07) — `curl` round-trip against the live API confirmed create/list/promote/memory CRUD and the one-master-per-tenant DB constraint; Playwright screenshots of `/agents` (roster grid, Master badge, dashed Create-Agent card) and all four `/agents/[id]` tabs (Soul, Skills, Memory, Setting) against the live dev server, plus an end-to-end browser flow (create agent → check a Skill → reload → still checked → add a Memory entry → promote via the roster card menu → Master badge moved) with zero console errors; test agents/memories cleaned up afterward, seeded "Office Manager" restored as Master |
| 1.5 | Telegram round-trip | A message sent to the bot submits a real task (`channel: "telegram"`) via `POST /v1/tasks`; the bot's "⏳ queued…" reply is edited in place as status changes (polling `GET /v1/tasks/:id` every `TASK_POLL_INTERVAL_SECONDS`, no WebSocket stream — none exists yet in `orbicrew-api`) until it lands on a terminal status with real output/model/cost; `/approvals` lists real pending approvals with inline Approve/Reject buttons wired to `POST /v1/approvals/{id}/approve\|reject`; the bot ignores any chat other than `TELEGRAM_ALLOWED_CHAT_ID` (no multi-tenant auth exists yet in `orbicrew-api` — single-user gate only, see tracker) | Pass (2026-08-25) — real bot (@orbicrew_dev_bot) against live local infra + `orbicrew-api` + worker: a real Telegram message ("hi") produced a real task (`general`/`trivial`, `claude-haiku-4-5`, `$0.0004`) and the placeholder message was edited to the real output, confirmed by the user in the live Telegram client; `/approvals` empty-state reply confirmed live (no pending approvals to test the button callbacks against real data, so that path relies on `orbicrew-channels`' 15/15 passing unit tests — approve/reject callback handling, timeout/terminal polling branches, disallowed-chat gate — instead of a live run) |
| 1.6 | Cross-channel continuity | Task started on web visible on Telegram (or vice versa) | Pending |
| 1.7 | WhatsApp round-trip | When Business API ready | Blocked — Meta requires business verification (needs an incorporated business the user doesn't have yet) before issuing a real Phone Number ID; code is built and covered by 34 unit tests in `orbicrew-channels` but not runnable against a live Meta app |
| 1.8 | Research Agent tools (phase plan's task 1.7 — numbering here has drifted from the plan file, see this row's label) | Submit a research-classified task with `TAVILY_API_KEY` set (e.g. "research and compare the pricing of the top 3 project management tools"); the task's `steps` include one or more `tool_call` entries (`web_search`/`web_fetch`, with `input`/`result_summary` in `detail`) before the final `specialist_execute` step; the output is grounded in real search results, not just the model's own knowledge; a bad/unreachable URL the model tries to fetch produces an `is_error` tool result the model works around instead of crashing the task | Pass (2026-08-25) — real task ("research and compare current web search provider pricing for Tavily and Brave, cite your sources") against live local infra + `orbicrew-api` + worker + a real `TAVILY_API_KEY`: classified `research`, ran 4 real tool calls (2× `web_search`, 2× `web_fetch` against `tavily.com/pricing` and Brave's docs) logged as their own `tool_call` steps before `specialist_execute`, output cited real fetched pricing tables with source URLs, `$0.0161` real spend. Note: the first attempt was misclassified as `coding` because the input text contained the word "API" (a coding keyword in `office_manager.py`'s keyword classifier) — not a bug in this task, just a reminder that the Phase 0.3 keyword classifier is a known placeholder; rewording the prompt to avoid coding keywords fixed it. No bad-URL/`is_error` path exercised live (covered by unit tests only) |
| 1.9 | Multi-provider search registry (same session, same day as 1.8 above) | With `TAVILY_API_KEY`/`LINKUP_API_KEY`/`PARALLEL_API_KEY`/`EXA_API_KEY`/`BRAVE_API_KEY` all set and `SEARCH_PROVIDER_MODE=auto`: (a) each configured provider's `.search()` called directly returns real results; (b) `web_search()` in auto mode picks the first configured provider in priority order (`tavily`); (c) with the Tavily key deliberately invalidated in-process, `web_search()` falls through to a real Linkup call and still returns real results, proving the fallback path works against live APIs, not just mocks | Pass (2026-08-25) — all 5 configured providers returned real results individually (`tavily`, `linkup`, `parallel`, `exa`, `brave`; `serpapi` correctly skipped, no key provided); auto mode picked `tavily` first as expected; a forced real Tavily 401 fell through to a real Linkup call returning real results (`"current Anthropic funding round"` → real Business Times article). A full task through the API/worker ("research and compare current pricing for Linkup, Parallel, and Exa search, cite your sources") also completed end-to-end using `tavily` for all `web_search` calls, `$0.0235` real spend. Manual-mode provider pinning not exercised live (covered by unit tests only) |
| 1.10 | Multi-provider model registry (Phase 1.8) | With `GROQ_API_KEY`/`CEREBRAS_API_KEY`/`DEEPINFRA_API_KEY`/`OPENROUTER_API_KEY` set: submit a `general`/`trivial` task (e.g. "hi") → `specialist_execute` step's `model_used` is `groq/openai/gpt-oss-20b`, real spend `$0`; submit a short non-`general` task that lands on `cheap` tier → `model_used` is `deepinfra/deepseek-ai/DeepSeek-V4-Flash` with real (small) spend; force a Groq auth failure in-process → falls through to a real Cerebras call; with none of the four keys set, `trivial`/`cheap` tasks still complete normally via direct Anthropic Haiku (`model_used: claude-haiku-4-5`), unchanged from pre-1.8 behavior; a `mid`/`frontier` task's `model_used` stays a plain Anthropic model name (registry never invoked) | Pending — code built and covered by 9 new unit tests (`test_model_providers.py`, `test_llm_client.py`) covering provider selection, the trivial→cheap fallthrough, all-providers-failing, and the registry→Anthropic-Haiku fallback in `llm_client.complete()`, but not yet run against real Groq/Cerebras/DeepInfra/OpenRouter keys — none were in `.env` this session |

---

## Phase 2 — Multi-tenancy & billing

| # | Check | Expected | Status |
|---|---|---|---|
| 2.1 | Tenant isolation | Tenant A cannot read Tenant B data/tasks/memory | Pending |
| 2.2 | Project isolation | Two projects under one tenant do not cross-contaminate context | Pending |
| 2.3 | Billing meters | Usage increments correctly; tier limits enforce | Pending |
| 2.4 | BYO keys | Tenant key used when billing_mode=byo; never logged in plaintext | Pending |
| 2.5 | Agent CRUD | Create/edit/disable/delete agent via UI persists correctly | Pending |

---

## Phase 3 — Differentiation

| # | Check | Expected | Status |
|---|---|---|---|
| 3.1 | Orbit View assign | In-world assign moves task through visible states | Pending |
| 3.2 | Discord round-trip | Same adapter contract as other channels | Pending |
| 3.3 | Meta-agent | Proposed agent requires human approval before activation | Pending |

---

## Regression notes

Add dated notes when a manual bug is found in the wild:

| Date | Area | Issue | Fixed? |
|---|---|---|---|
| — | — | (none yet) | — |

---

## Changelog (manual guide)

| Date | Change |
|---|---|
| 2026-08-07 | 1.4 renamed "Morning summary" → Digest (not time-of-day-scoped); a scheduled-generation + custom-delivery-time approach was built, then reverted the same day (unnecessary background job, no delivery channel to use it) in favor of history-as-a-live-date-range-query — see tracker |
| 2026-08-07 | 1.4 marked Pass — Morning summary (`GET`/`POST /v1/dashboard/morning-summary*` + `orbicrew-web` `/summary` page) verified live |
| 2026-08-07 | 1.3 replaced with 1.3a/1.3b/1.3c (approve/reject/conflict for the approvals resolve flow), all Pass |
| 2026-08-07 | Phase 1 table renumbered — 1.2a/1.2b (tenant daily cap, retry-then-escalate) inserted; former 1.2–1.6 shifted to 1.3–1.7 |
| 2026-08-07 | 0.E added — Admin auth shell checks, all Pass; Admin prerequisite row updated |
| 2026-08-07 | 0.A.6 added — OpenAPI + TS client (Phase 0.7) typed contract checks, all Pass |
| 2026-08-07 | 0.B.1 marked Pass — Standard chat GUI (Phase 0.6) verified live via Playwright |
| 2026-08-07 | 0.B.3/0.B.4 marked Pass — Model Router + Budget Guard (Phase 0.4) verified live |
| 2026-08-07 | Initial structure created during workspace agentic setup |
