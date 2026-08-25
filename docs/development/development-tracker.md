# Development Tracker — Orbicrew

Chronological log of workspace and product development sessions. **Agents: append an entry after completing work.** Do not rewrite history; add corrections as new notes if needed.

Newest entries at the **top**.

---

## 2026-08-25 — Follow-up: Task 1.8 widened from OpenRouter-only to a multi-provider in-app registry (DeepInfra + OpenRouter, extensible to Bedrock/direct keys)

**Agent / operator:** Claude Code
**Phase:** Phase 1 — plan/docs only, revises task 1.8 from the entry immediately below (same day); no application code touched
**Scope:** `docs/development/phase_by_phase_development_plan.md`, `docs/development/AGENT_BOOTSTRAP.md`, `docs/leangine-office-docs/01_AI_Office_Platform_Requirements.md`, `docs/leangine-office-docs/tech/13_byo_provider_architecture.md`.

### Context
Two things surfaced after the OpenRouter-direct decision below was recorded: (1) the user found that DeepInfra now lists Claude on its pricing page — verified live at deepinfra.com/pricing: Claude Sonnet 5 at $2/$10 per M tokens, *cheaper* than Anthropic's own $3/$15 list price, with no aggregator markup or card-funding fee since DeepInfra owns its inference stack; DeepSeek V3.2 there runs a few cents higher than OpenRouter's, close enough not to matter. DeepInfra doesn't list MiniMax, but that's not a gap — the router only needs one good cheap-tier model. (2) The user then asked, correctly, why this has to be either/or: "can't we have both options, use whatever we need," extensible later to Bedrock and direct platform keys, user-choosable. That reframed task 1.8 from "pick one provider" to "build the provider-agnostic registry `tech/13_byo_provider_architecture.md` already designs for BYO/multi-tenant — just as in-app code for one tenant now, instead of a separately-hosted LiteLLM server." Important distinction worth keeping straight for future sessions: **self-hosting** (a separate gateway process to run/monitor — still not being done) is a different axis from **multi-provider support** (application code that calls different `base_url`s — genuinely free to have regardless of the self-hosting decision).

### Done
- `phase_by_phase_development_plan.md`: task 1.8 rewritten from "OpenRouter-backed cost-tier routing" to **"Multi-provider cost-tier routing"** — an in-app provider registry starting with DeepInfra + OpenRouter, auto-cheapest-per-tier with fallback, per-agent manual override reusing the `agents.model_mode`/`manual_model` columns from 1.4a (made provider-qualified), explicitly designed to add Bedrock/direct-Anthropic/direct-OpenAI as new registry entries without an architecture change. Phase 0 stack description and the "suggested immediate next slice" list updated to match.
- `tech/13_byo_provider_architecture.md`: §0 rewritten to frame Phase 1.8's registry as a lighter first increment of *this same document's* Pattern A/B/C shape (not a separate, throwaway approach) — so Phase 2.3's eventual self-hosted-LiteLLM migration is an upgrade of the existing registry pattern, not a rewrite.
- `01_AI_Office_Platform_Requirements.md`: Layer 4 row, the §7.2 LiteLLM bullet, and the §11.2 tooling table all updated — DeepInfra added as a registered provider (with the verified pricing rationale), OpenRouter kept for catalog breadth/fallback, AWS Bedrock added as a documented-but-not-yet-registered future candidate (real option, priced higher on open models, worth reconsidering only if AWS consolidation or heavy Claude volume changes the calculus).
- `AGENT_BOOTSTRAP.md`: cost-architecture line updated to describe the registry instead of a single provider.
- No `manual-test-guide.md` change — still no code touched.

### Follow-ups
- Task 1.8 implementation itself is still **Pending** — needs both a DeepInfra key and an OpenRouter key in `.env` before work starts.
- AWS Bedrock and direct Anthropic/OpenAI keys are documented as registry-ready but not registered — add them opportunistically if a real need shows up (AWS infra consolidation, wanting Bedrock's non-marked-up Claude pricing at higher frontier-tier volume), not preemptively.

### Files / repos touched
- `docs/development/phase_by_phase_development_plan.md`, `docs/development/AGENT_BOOTSTRAP.md`, `docs/development/development-tracker.md` (this entry)
- `docs/leangine-office-docs/01_AI_Office_Platform_Requirements.md`, `docs/leangine-office-docs/tech/13_byo_provider_architecture.md`

---

## 2026-08-25 — Decision: OpenRouter-direct cost routing now, self-hosted LiteLLM deferred to Phase 2; SaaS scope explicitly kept, not paused

**Agent / operator:** Claude Code
**Phase:** Phase 1 (Overnight autonomy + multi-channel) — plan/docs only, adds task 1.8; no application code touched this session
**Scope:** `docs/development/phase_by_phase_development_plan.md`, `docs/development/AGENT_BOOTSTRAP.md`, `docs/leangine-office-docs/01_AI_Office_Platform_Requirements.md`, `docs/leangine-office-docs/tech/13_byo_provider_architecture.md`.

### Context
User's actual day-to-day motivation surfaced this session, distinct from the sellable-SaaS framing the rest of the docs are written around: cut personal AI-subscription cost, consolidate everything (coding, research, content, ideation, product-dev thinking) into one place, and do it fast without rebuilding what already works. A wide-ranging discussion (cheap/open-source LLM options, inference API aggregators vs. self-hosting, and a specific Contabo CPU-only VPS someone floated) landed on a concrete, bounded decision rather than a re-architecture:

- **Wire real cost-tier routing via OpenRouter, directly — not self-hosted LiteLLM, not yet.** Investigating the live code confirmed `model_router.py`'s own comment already flagged this as deferred work: every tier today (`trivial`/`cheap`/`mid`/`frontier`) calls Anthropic directly (`llm_client.py`), meaning Orbicrew's own cost-routing design has never actually been in effect — every task submitted through `/tasks` so far paid full Anthropic pricing regardless of complexity. `01_AI_Office_Platform_Requirements.md` §12's original Phase 0 spec already called for "OpenRouter-based model routing" — the implementation just never got there. This session's task 1.8 (below) closes that gap; it's a course-correction back to the original plan, not a new idea.
- **Self-hosted LiteLLM is a real Phase 2.3 thing, not a Phase 0/1 thing.** `tech/13_byo_provider_architecture.md`'s gateway design (per-tenant virtual keys, three BYO patterns, budget enforcement as a second layer) only pays for the infrastructure-to-run cost once there are actual multiple tenants/BYO customers. At single-tenant personal-use volume, OpenRouter alone already delivers the same "one account, one bill, every model behind one key" property LiteLLM would provide, with zero server to operate. Added a phased-rollout note (§0) to that document so a future agent doesn't read it and assume the gateway is already running.
- **Self-hosting the LLMs themselves was separately evaluated and rejected for now** — a Contabo "Cloud VPS 18" (18 vCPU/96GB RAM/€39/mo) the user was considering is CPU-only (no GPU line item on that plan); CPU inference of anything in the 30B–70B quality range realistically runs ~1–5 tok/s, unusable for an agentic tool-calling loop, and self-hosting only beats *cheap* open-weight API pricing (DeepSeek/MiniMax-class, ~$0.14–0.50/M tokens via OpenRouter) at roughly 50M+ tokens/day — enterprise volume, not personal use. Not recorded as a standing decision in any doc (no doc claimed self-hosted inference was planned for this stage), just noting it here so the reasoning isn't lost if it comes up again.
- **Explicit user instruction: do not park/deprioritize the SaaS-only scope** (multi-tenancy, billing tiers, admin console, BYO-API key UI, marketplace, brand/marketing, Orbit View) — Phase 2/3 stand as already documented, untouched this session. The personal-use-first framing changes near-term sequencing (1.8 pulled ahead of 1.5–1.7 below) and the routing backend choice, not the long-run product scope.
- **Coding-agent work stays external, deliberately.** Orbicrew's own "coding" specialist (`llm_client.py`) is a single system-prompted LLM call with no file/tool access — not comparable to opencode/Claude Code, which the user already uses for real repo-level work. Decision: keep using those tools separately, pointed at the same OpenRouter key once 1.8 lands, rather than trying to rebuild a repo-aware coding agent inside Orbicrew. Recorded in the 1.8 task description so it doesn't get reinvented later.

### Done
- `phase_by_phase_development_plan.md`: Phase 0 "Primary stack" bullet updated to say OpenRouter-direct with LiteLLM explicitly deferred and why. New **task 1.8 — OpenRouter-backed cost-tier routing** (`llm_client.py`/`model_router.py`/`settings.py` swap Anthropic-direct for OpenRouter's OpenAI-compatible endpoint; trivial/cheap/mid → a DeepSeek V3.2/MiniMax M2.5-class model, frontier → Claude, one key). "Suggested immediate next engineering slice" reordered: 1.8 sequenced ahead of 1.5 (Telegram)/1.6 (WhatsApp)/1.7 (research agent) at explicit user request — direct cost impact on real daily usage outweighs another channel or agent right now.
- `AGENT_BOOTSTRAP.md`: "what you are working on" cost-architecture line updated (OpenRouter now, self-hosted LiteLLM from Phase 2, pointer to 1.8).
- `01_AI_Office_Platform_Requirements.md`: §3.1 Layer 4 row, §7.2's LiteLLM bullet, and §11.2's LiteLLM/OpenRouter tooling rows all updated to describe the phased approach (OpenRouter direct now, LiteLLM as the Phase 2.3 target) instead of implying LiteLLM is the current state.
- `tech/13_byo_provider_architecture.md`: new §0 "Phased rollout" note up front — the whole document is the Phase 2+ target design, not what's running; points to `model_router.py` + Phase 1.8 for the actual current/near-term path.
- No `manual-test-guide.md` change — no code touched, no test steps changed this session.

### Follow-ups
- Task 1.8 itself (the actual `llm_client.py`/`settings.py`/`model_router.py` code change) is still **Pending** — needs the user's OpenRouter API key in `.env` before implementation starts; not done this session by explicit instruction ("not coding").
- When 1.8 lands, opencode/Claude Code should be pointed at the same OpenRouter key (external tooling change, outside this repo — not a tracked Orbicrew task, just noted so the "one place" goal isn't forgotten).
- Phase 2.3's BYO key management task is where self-hosted LiteLLM actually gets stood up — no change needed to that row, it already says "LiteLLM virtual/tenant routing."

### Files / repos touched
- `docs/development/phase_by_phase_development_plan.md`, `docs/development/AGENT_BOOTSTRAP.md`, `docs/development/development-tracker.md` (this entry)
- `docs/leangine-office-docs/01_AI_Office_Platform_Requirements.md`, `docs/leangine-office-docs/tech/13_byo_provider_architecture.md`

---

## 2026-08-07 — Correction: reverted Digest's scheduled-generation cron job + snapshot storage

**Agent / operator:** Claude Code
**Phase:** Phase 1 (Overnight autonomy + multi-channel) — corrects the 1.4c work from the immediately-preceding entry below, same day
**Scope:** `orbicrew-api` (remove cron job, schedule endpoints, snapshot table) + `orbicrew-web` (remove schedule UI, replace snapshot-based history with a live date-range picker) + docs.

### Context
User pushback, verbatim: "why the fuck we need scheduler? it will burn cpu... isn't it simply just a db query? can't we manage in that way?" — correct, and the prior entry's design was wrong. The `generate_scheduled_digests` hourly `arq` cron job scanned `tenants` forever for a feature that delivered nothing (no email/Telegram/Slack channel exists to push a generated digest to), and the `digest_snapshots` table existed only to give that scheduler something to write. Meanwhile "history" never needed either of those — `tasks`/`approvals` rows already persist forever, so browsing an arbitrary past period is just the `build_digest(pool, tenant_id, period_start, period_end)` helper (already built for the on-demand route) called with an older range. A schedule preference with zero functional effect is also exactly the "fabricated affordance" this codebase explicitly avoids elsewhere (see the 1.4a Agent Roster entry's "Generate with AI" omission). Reverted rather than kept-but-unused.

### Done
- **`orbicrew-api`**: removed `generate_scheduled_digests` and its `cron_jobs` registration from `worker.py` (back to just `run_task_job`, no background scheduler beyond the existing task queue). Removed `GET/PUT /v1/dashboard/digest/schedule` and `GET /v1/dashboard/digest/history` plus the `DigestSchedule`/`DigestSnapshot` models from `digest.py` — kept `GET /v1/dashboard/digest` (default + explicit `period_start`/`period_end`) and `POST .../viewed` exactly as before. Deleted `orbicrew_api/timezones.py` (no longer needed with the schedule feature gone). Rewrote `migrations/0006_digest_rename_and_schedule.sql` in place to only do the rename (safe — uncommitted, local-only, never applied anywhere but this dev DB) instead of adding a throwaway `0007_undo` migration; manually reconciled the local dev DB (dropped `digest_snapshots` and the two schedule columns) to match.
- **`orbicrew-web`**: deleted `schedule-form.tsx` and `history-list.tsx`. New `history-range-picker.tsx` — four preset buttons (Today/Yesterday/Last 7 days/Last 30 days), computed client-side from the browser's local clock via plain `Date` arithmetic, `.toISOString()`'d into the existing `period_start`/`period_end` query params — no new endpoint, no stored state. `api-digest.ts` trimmed back to `getDigest`/`markDigestViewed` only. `page.tsx` no longer fetches schedule/history in parallel; just the one `getDigest` call plus the picker component when not already viewing a historical range.
- **Tests**: removed the 3 `generate_scheduled_digests` cases from `test_worker.py` and the 5 schedule/history cases from `test_digest.py`; kept the explicit-range test (renamed its docstring to spell out that this *is* the history mechanism). 53/53 backend tests passing (down from 62, correctly — no coverage lost, coverage removed for removed code).
- **Verified live**: restarted `orbicrew-api`, confirmed the OpenAPI spec now lists only `/v1/dashboard/digest` and `/v1/dashboard/digest/viewed`; regenerated `api-schema.d.ts`; `npx tsc --noEmit`, `npm run lint`, `npm run build` all clean; started the dev server and confirmed `/digest` renders the four history preset buttons and no schedule form, and that clicking through to an explicit range still shows "back to current."

### Follow-ups
- Same as before: a real custom delivery time + scheduler is legitimate once a delivery channel (Telegram, Phase 1.5+) exists to actually push something — build the schedule preference and the job that acts on it together, at that point, not separately.
- `phase_by_phase_development_plan.md`'s 1.4c row and `manual-test-guide.md`'s 1.4c row (added in the immediately-preceding session) were removed rather than marked failed, since the underlying code no longer exists — the 1.4 rows in both were updated in place to note that history *is* the explicit-range query, no separate mechanism.

### Files / repos touched
- `repos/orbicrew-api`: `migrations/0006_digest_rename_and_schedule.sql`, `src/orbicrew_api/digest.py`, `src/orbicrew_api/worker.py`, `tests/test_digest.py`, `tests/test_worker.py`; deleted `src/orbicrew_api/timezones.py`
- `repos/orbicrew-web`: `src/lib/api-digest.ts`, `src/app/digest/page.tsx`, `src/app/digest/actions.ts`, new `src/app/digest/history-range-picker.tsx`; deleted `src/app/digest/schedule-form.tsx`, `src/app/digest/history-list.tsx`
- `docs/leangine-office-docs/`: `01_AI_Office_Platform_Requirements.md`, `tech/03_system_design.md`, `tech/07_phased_development_plan.md`
- `docs/development/`: `phase_by_phase_development_plan.md`, `manual-test-guide.md`, `development-tracker.md` (this entry)

---

## 2026-08-07 — Phase 1.4/1.4c: "Morning summary" renamed to Digest + custom schedule + history

**Agent / operator:** Claude Code
**Phase:** Phase 1 (Overnight autonomy + multi-channel) — reworks 1.4, adds 1.4c
**Scope:** `orbicrew-api` (rename + new migration + new endpoints + cron job) + `orbicrew-web` (route rename + new schedule/history UI) + product docs.

### Context
User pushback on the Phase 1.4 "morning summary" naming: the feature isn't actually time-of-day-scoped (same-day tasks finish outside "morning" too), "morning" is undefined for a global multi-timezone user base, and a fixed daily-cron framing doesn't fit a product where users can check in whenever they want. Worked through the redesign with the user before touching code: (1) rename to **Digest** (locked as a single word for the sidebar nav label), (2) default view stays "since I last checked" (already the watermark-based design from the original 1.4 slice), (3) add an *optional* per-user custom local delivery hour/timezone on top of that default, (4) keep history of past digests, browsable, (5) explicit standing rule going forward: backend/DB/API always UTC, frontend always renders in the viewer's local timezone (`Date.toLocaleString()` already did this correctly, confirmed rather than changed). All confirmed as no-AI-cost, DB-only work before implementation started.

### Done
- **Migration `0006_digest_rename_and_schedule.sql`**: renames `tenants.last_summary_viewed_at` → `last_digest_viewed_at`; adds `tenants.digest_schedule_hour_local` (0–23, checked) + `digest_schedule_timezone` (IANA name, validated via `zoneinfo` at the API layer); adds `digest_snapshots` table (tenant_id, period_start/end, generated_at, jsonb payload) for persisted history.
- **`orbicrew-api`**: `morning_summary.py` → `digest.py`. Router prefix moved to `/v1/dashboard/digest`; models renamed (`DigestResponse`/`DigestTask`/`DigestViewed`, new `DigestSchedule`/`DigestSnapshot`). Live aggregation logic extracted into a shared `build_digest(pool, tenant_id, period_start, period_end)` so the on-demand route and the new scheduled job never diverge. `GET /v1/dashboard/digest` now accepts optional `period_start`/`period_end` to browse an arbitrary historical range live (400 if only one side is given) without ever touching the watermark — the default (no params) path is unchanged "since last viewed, or 24h lookback on first view." New `GET/PUT /v1/dashboard/digest/schedule` (set/clear hour+timezone together, 400 on an unknown IANA zone or one-sided input) and `GET /v1/dashboard/digest/history`. New `orbicrew_api/timezones.py` (`validate_iana_timezone`, `local_hour`) shared between the route and the worker.
- **`orbicrew-api` worker**: new `generate_scheduled_digests` hourly `arq` cron job (`WorkerSettings.cron_jobs`) — for each tenant with a schedule set, checks whether `now` matches their configured local hour (via `zoneinfo`), and if so (and not already generated within the current UTC hour, to survive worker restarts), computes a digest via the shared `build_digest` helper, persists it to `digest_snapshots`, and advances the watermark. This is generation + storage only — **no email/Telegram/Slack delivery channel exists yet** to push it anywhere; the point was to make "custom time" and "history" real and testable now rather than half-build a preference with no effect, using only infrastructure (arq, already-running worker, Postgres) that already exists.
- **`orbicrew-web`**: `api-summary.ts` → `api-digest.ts`; `/summary` → `/digest` (`page.tsx`, `actions.ts`, `mark-viewed-button.tsx` renamed/updated). New `schedule-form.tsx` (hour + timezone picker, timezone `<datalist>` from `Intl.supportedValuesOf("timeZone")` with the browser's own zone as the default, "Turn on"/"Save"/"Turn off" states) and `history-list.tsx` (past snapshots, each linking back into the page as an explicit `period_start`/`period_end` view). The page accepts `searchParams` (`Promise<{period_start?, period_end?}>` per this Next.js version's async-params convention) to switch between "live since last checked" and "browsing history" modes — history mode hides the mark-viewed button and schedule form and shows a "back to current" link. Sidebar nav label/href updated to `Digest`/`/digest`.
- **Tests**: `test_morning_summary.py` → `test_digest.py` (renamed cases plus new coverage for explicit-range browsing, one-sided-range rejection, schedule get/set/clear, invalid-timezone rejection, history listing). `test_worker.py` gained three cases for `generate_scheduled_digests` (fires at the tenant's local hour, skips outside it, skips a tenant already generated this hour). 62/62 backend tests passing.
- **Verified live**: ran the new migration against the real local Postgres; started `orbicrew-api` and hit all four digest endpoints via `curl` (including a rejected bad timezone and a watermark-preserving explicit-range query); directly invoked `generate_scheduled_digests` against the real DB with a tenant's schedule set to the current hour and confirmed a real `digest_snapshots` row landed and then appeared via `GET .../history`; regenerated `api-schema.d.ts` from the live OpenAPI spec; `npx tsc --noEmit`, `npm run lint`, `npm run build` all clean; started the Next dev server and confirmed `/digest` renders (schedule form, history section, mark-as-reviewed), the old `/summary` route now 404s, other pages (`/agents`, `/tasks`, `/approvals`) are unaffected, and that browsing an explicit historical range renders UTC-stored timestamps converted to the server's local time (confirming the UTC-backend/local-frontend display convention holds without any code change, since `toLocaleString()` already did this).
- **Docs**: renamed "morning summary" → Digest throughout `01_AI_Office_Platform_Requirements.md`, `03_system_design.md` (§7 retitled "Unattended autonomous execution flow" with a new paragraph documenting the watermark/schedule/history mechanism), `05_api_design.md`, `07_phased_development_plan.md`, `10_ui_ux_guide.md`, `17_marketing_plan.md` (kept the "wake up to a report" narrative in marketing copy — that's intentional messaging, not a technical claim — but renamed the product term), `AGENT_BOOTSTRAP.md`. `phase_by_phase_development_plan.md` 1.4 row updated in place (renamed, not a log) with a new 1.4c row for the schedule/history addition; `manual-test-guide.md` 1.4 row updated in place plus a new 1.4c row, changelog appended (old Pass entry for "Morning summary" left untouched, per that file's own append-only changelog convention).

### Follow-ups
- No delivery channel wired for the custom schedule yet — Telegram (Phase 1.5) is the first candidate; when it lands, `generate_scheduled_digests` is the natural place to also push the snapshot instead of only persisting it.
- Schedule/watermark are still tenant-level (`DEFAULT_TENANT_ID`), consistent with the rest of Phase 0/1's single-tenant scaffolding — becomes per-user once real auth/multi-user lands (Phase 2), at which point `last_digest_viewed_at`/`digest_schedule_*` likely move from `tenants` to `users`.
- The `[[feedback-timezone-display]]` UTC-backend/local-frontend convention is now a standing rule for all future timestamp work, not just Digest — recorded in memory.

### Files / repos touched
- `repos/orbicrew-api`: `migrations/0006_digest_rename_and_schedule.sql` (new), `src/orbicrew_api/digest.py` (new, replaces `morning_summary.py`), `src/orbicrew_api/timezones.py` (new), `src/orbicrew_api/worker.py`, `src/orbicrew_api/main.py`, `tests/test_digest.py` (new, replaces `test_morning_summary.py`), `tests/test_worker.py`
- `repos/orbicrew-web`: `src/lib/api-digest.ts` (new, replaces `api-summary.ts`), `src/app/digest/` (new: `page.tsx`, `actions.ts`, `mark-viewed-button.tsx`, `schedule-form.tsx`, `history-list.tsx`; replaces `src/app/summary/`), `src/components/sidebar-nav.tsx`
- `docs/leangine-office-docs/`: `01_AI_Office_Platform_Requirements.md`, `tech/03_system_design.md`, `tech/05_api_design.md`, `tech/07_phased_development_plan.md`, `tech/10_ui_ux_guide.md`, `tech/17_marketing_plan.md`, `tech/15_performance_training_and_settings.md`
- `docs/development/`: `AGENT_BOOTSTRAP.md`, `phase_by_phase_development_plan.md`, `manual-test-guide.md`, `development-tracker.md` (this entry)

---

## 2026-08-07 — Additional Stitch design references added

**Agent / operator:** Claude Code
**Phase:** n/a (docs/resources only, no code)
**Scope:** `resources/stitch_orbicrew_ui_ux_guide/` (master-only) + `docs/development/AGENT_BOOTSTRAP.md` + `resources/README.md`.

User dropped a new `resources/stitch_orbicrew_ui_ux_guide/additional-designs/` folder (with briefs
in a sibling `additional_designs_guide/`) covering gaps the original Stitch set didn't design:
`orbicrew_agent_configuration_setting_tab` (the previously-undesigned Setting tab), the
`orbicrew_agent_roster_master_promotion_modal`, `orbicrew_live_voice_mode` (extends
`orbicrew_message_agents`), an **updated** `orbicrew_settings` (adds a Slack Connected-Channels
row), a `shader` background effect, and a duplicate (identical) `orbicrew` DESIGN.md. The first
three map directly to the two "planning-doc update still pending" items flagged in the 1.4a entry
below (Slack-as-channel-adapter, live voice mode) plus the Master-promotion UI that entry
implemented without a visual spec at the time. Recorded the new folder set in
`docs/development/AGENT_BOOTSTRAP.md` and `resources/README.md` (same places the original Stitch
folder list lives) and in memory ([[feedback-stitch-design-references]]) so future sessions know
to check `additional-designs/` alongside the original folders before building/restyling the
affected pages. No app code touched this session.

---

## 2026-08-07 — Agent Roster + Configuration UI, pulled forward from Phase 2.4; Tasks page header parity

**Agent / operator:** Claude Code
**Phase:** Phase 1 (Overnight autonomy + multi-channel) — task 1.4a/1.4b, pulling Phase 2.4 (Agent CRUD UI) forward at explicit user request.
**Scope:** `orbicrew-api` (new agents endpoints + migration) + `orbicrew-web` (`/agents`, `/agents/new`, `/agents/[id]`, `/tasks` header).

### Context
User compared the app against the Stitch design references (`resources/stitch_orbicrew_ui_ux_guide/orbicrew_agent_roster`, `orbicrew_agent_configuration`, `orbicrew_message_agents`) and asked for the real Agents page and a closer Tasks-page match. Investigating first: the `agents` table has existed since `0001_core_schema.sql` but was completely unused — `office_manager.py` routes purely on keyword-classified `specialist` strings, no API/UI ever read or wrote an `agents` row. Building a pixel-matching Agents page with fabricated card data would have violated the project's no-fabricated-content convention (`docs/development/development-tracker.md`'s 1.3a entry), so the real blocker — and the actual scope of this slice — was building the Agent CRUD backend that Phase 2.4 already called for, just earlier than sequenced. Confirmed with the user before writing any migration (schema changes are harder to reverse) via three scoped decisions: (1) Slack should become a chat-channel adapter like Telegram/Discord, not just a Phase 3.3 tool integration — **not yet implemented, planning-doc update still pending**; (2) any agent should be promotable to Master Agent, not just a fixed default — **implemented this session**; (3) live/continuous voice mode should be available on the plain chat GUI, not only the Phase 3 Orbit View avatar — **not yet implemented, planning-doc update still pending**.

### Done
- **Migration `0005_agent_master_and_setting.sql`**: adds `agents.is_master`, `tone_notes`, `model_mode` (`'auto'|'manual'`), `manual_model`, plus a partial unique index (`idx_agents_one_master_per_tenant`) enforcing exactly one Master Agent per tenant at the DB level, not just in application code.
- **`orbicrew-api`**: new `src/orbicrew_api/agents.py` — `GET/POST /v1/agents`, `GET/PATCH /v1/agents/{id}`, `POST /v1/agents/{id}/promote` (transactional demote-then-promote), `GET/POST /v1/agents/{id}/memories` + `DELETE .../{memory_id}` (using the pre-existing but previously-unused `agent_memory` table). A tenant's first-ever agent auto-becomes Master (there's no one else to delegate to). `bootstrap.py` now seeds a default "Office Manager" Master agent alongside the existing default tenant/user, so the roster isn't empty on first load. Router wired into `main.py`. Deliberately **not** wired into `office_manager.py`'s actual task routing/execution — that's separate follow-up work; this slice is the CRUD/data model + UI only, and the Agents page copy doesn't claim otherwise.
- **`orbicrew-web`**: new `src/lib/api-agents.ts` client (same result-wrapper pattern as `api-approvals.ts`); new `/agents` (roster grid — Master card with accent border, other agent cards with an Edit/Make-Master/Disable menu, dashed "Create Agent" card; deliberately omits the mockup's "Generate with AI" card since that flow isn't built, per the no-fabricated-affordances convention), `/agents/new` (manual create form), `/agents/[id]` (Soul/Skills/Memory/Setting tabs per `10_ui_ux_guide.md` §13b — Setting tab is new: Auto/Manual model mode, per-task budget cap, Master Agent promote/status control, and the action-whitelist toggles, all previously undesigned even in the Stitch mockup). Sidebar's "Agents" link and "Hire New Agent" button are no longer disabled placeholders.
- **`/tasks` header parity**: page title now shows the active task's input text (truncated) plus a status badge ("Active Task"/"Completed"/"Paused"/"Failed"), matching `orbicrew_message_agents`'s per-task title bar, instead of a static "Tasks" heading.
- **Verified live**: ran `orbicrew-api-migrate`, started both `orbicrew-api` and `orbicrew-web` dev servers against the real local Postgres/Redis, and drove the full flow with Playwright — create agent → redirect to its config page → check a Skill → save → reload → still checked → add a Memory entry → back on the roster, promote it to Master via the card menu → Master badge moved and the previous Master's card returned to normal → zero console errors throughout. Submitted a real task from `/tasks` and confirmed the new header/badge render correctly. Test agents/memories/tasks created during verification were cleaned up; the seeded "Office Manager" was restored as Master.

### Follow-ups
- The Slack-as-channel-adapter and live-voice-everywhere decisions above are recorded here but **not yet reflected** in `01_AI_Office_Platform_Requirements.md`, `03_system_design.md`, `10_ui_ux_guide.md`, or the phase plan's Phase 3 rows — do that pass before Phase 3 planning starts, so the docs stay authoritative.
- Agent rows are not yet wired into actual task routing (`office_manager.py` still classifies by keyword, ignoring `tasks.agent_id`/the `agents` table's `system_prompt`/`tool_whitelist`/`model_mode`) — the Agents page today manages configuration, it doesn't yet change what happens when a task runs. That wiring is real follow-up work, not a bug.
- Skills tab tool catalog (`web_search`, `code_execution`, `image_generation`, `file_access`, `browser_automation`, `email_send`) is a placeholder list in both `agents.py` (backend) and `agent-config-tabs.tsx` (frontend) — grows as real tool integrations land per `14_universal_task_coverage.md`; the two lists are manually kept in sync today, not a single source of truth.
- No AI-generate-agent flow (the mockup's "Generate with AI" card) — manual creation only.

---

## 2026-08-07 — Phase 1.4: Morning summary

**Agent / operator:** Claude Code
**Phase:** Phase 1 (Overnight autonomy + multi-channel) — task 1.4
**Scope:** `orbicrew-api` (one new endpoint pair + migration) + `orbicrew-web` (new `/summary` page).

### Done
- **Watermark**: new migration `0004_morning_summary.sql` adds `tenants.last_summary_viewed_at`
  (nullable — null means "never viewed"). No per-tenant preferences table existed to hang this off;
  chose `tenants` over `users` since the summary is a tenant-level concept in this still-single-user
  scaffolding.
- **`orbicrew-api`**: new `src/orbicrew_api/morning_summary.py` — `GET /v1/dashboard/morning-summary`
  splits `tasks` with `status in ('done','paused','failed')` and `completed_at >= period_start` into
  three lists (all three statuses set `completed_at` on the same `worker.py` update, confirmed before
  relying on it), sums their spend, and lists `approvals` with `requested_at >= period_start`
  regardless of resolution status (reuses `approvals.ApprovalResponse`/`_to_response` rather than a
  duplicate model). `period_start` is the watermark, or `now() - 24h` on first-ever view (so a new
  tenant doesn't get a report spanning all of history). `POST /v1/dashboard/morning-summary/viewed`
  is a **separate** explicit action that advances the watermark — deliberately not auto-triggered by
  the `GET`, so page polling/reloads can't silently clear it before a human has actually seen it.
  New `tests/test_morning_summary.py` (split-by-status, spend sum, no-watermark fallback, empty
  state, mark-viewed).
- **`orbicrew-web`**: new `src/lib/api-summary.ts` wrapper (same shape as `api-dashboard.ts`); new
  `/summary` page mirroring `/approvals`'s server-component structure — stat row (completed/paused/
  failed counts + total spend), three task-list sections, and a "New Approvals" section that reuses
  the existing `ApprovalItem` component for still-pending rows and a plain read-only row for
  already-resolved ones (rendering `ApprovalItem` for a resolved approval would offer a dead
  Approve/Reject that 409s). "Mark as reviewed" is a `useActionState`/`useFormStatus` form
  (`src/app/summary/actions.ts` + `mark-viewed-button.tsx`) copied from `approval-item.tsx`'s
  resolve-button pattern. Added `{ href: "/summary", label: "Morning Summary", icon: "summarize" }`
  to `sidebar-nav.tsx`, right after Home.
- **Verified live** (both dev servers were already running from prior sessions, real accumulated
  task/approval history): `curl` against `GET /v1/dashboard/morning-summary` returned real
  split-by-status data (37 done, 4 paused, 2 failed tasks; 4 approvals, 2 approved/2 rejected); the
  SSR'd `/summary` HTML contained the expected task text and approval badges; `POST .../viewed`
  advanced the watermark and a follow-up `GET` + page reload correctly showed all four sections
  empty. No browser/screenshot tool was available this session, so visual confirmation was via
  fetching the rendered HTML rather than a real browser — worth a Playwright pass before calling the
  UI fully done.

### Follow-ups
- No Stitch mockup exists for this page (`resources/stitch_orbicrew_ui_ux_guide/` has no
  morning-summary folder) — styling was extrapolated from `/approvals`'s card language rather than a
  reference design.
- 1.5 Telegram adapter is the next pending Phase 1 slice per the phase plan.

---

## 2026-08-07 — Phase 1.3a follow-up: theming, global header, real dashboard, task chat page

**Agent / operator:** Claude Code
**Phase:** Phase 1 (Overnight autonomy + multi-channel) — completing task 1.3a
**Scope:** `orbicrew-web` (theming, layout, dashboard, new `/tasks` page) + `orbicrew-api` (one new
read-only aggregate endpoint). Same-day follow-up to the 1.3a entry below, after the user reviewed
the first pass live and asked for the remaining pieces (logos, dark/light/system mode, a real
global header, sidebar polish, and moving chat into a proper message-thread page).

### Done
- **Brand token correction**: `docs/leangine-office-docs/tech/09_brand_identity.md` §6.3 (the
  brand doc CLAUDE.md calls locked) turned out to have slightly different hex values — and real
  dark-mode values — than the Stitch mockups' own `DESIGN.md`/embedded Tailwind config that 1.3a's
  token set was copied from (Stitch's is a Material-3-style approximation it generated for its own
  exports, light-mode only). Reconciled `globals.css`: locked hexes now anchor
  `background/surface/on-surface/on-surface-variant/primary/secondary(gold)/error/success/warning/
  outline` in both themes; the extra M3 tonal container/outline scale is hand-derived from those
  anchors (same relative steps Stitch used) since the locked doc doesn't define that finer scale.
  Saved as [[reference-brand-token-source]] memory for future sessions.
- **Dark mode**: `@custom-variant dark (&:where(.dark, .dark *));` (Tailwind v4's class-based dark
  mode escape hatch) + a `.dark` override block in `globals.css`. New `src/components/theme-toggle.tsx`
  — cycles **dark → light → system** (dark is default), icon reflects current state
  (`dark_mode`/`light_mode`/`computer` — user asked for a literal desktop icon for "system", not
  the auto-brightness glyph), persisted to `localStorage`. A blocking inline script in
  `layout.tsx`'s `<head>` applies the class before hydration to avoid a flash of the wrong theme;
  `<html>` needs `suppressHydrationWarning` for this (React otherwise correctly flags a real
  server/client class attribute mismatch — the standard `next-themes`-style fix, confirmed via a
  Playwright console-error check before and after).
- **Logos/icons**: copied `resources/logos/*.svg` into `orbicrew-web/public/`. Caught and fixed a
  naming-direction bug before shipping — `dark-*.svg` is a dark-**colored** mark (goes on light
  backgrounds), `light-*.svg` is light-colored (goes on dark backgrounds), the opposite of my first
  (wrong) assumption; renamed the copied files to `orbicrew-{wordmark,icon}-for-{light,dark}-bg.svg`
  so the mapping can't be misread again, and saved [[feedback-logo-icon-naming]] to memory. Sidebar
  now shows the real wordmark (theme-swapped via `dark:hidden`/`dark:block`, no JS needed). Favicon
  is theme-aware via two `<link rel="icon" media="(prefers-color-scheme: ...)">` tags (Next's
  single-file `icon.svg` convention doesn't support that), with a static fallback for browsers that
  ignore the `media` attribute.
- **Global header**: was mobile-only (`md:hidden`) in 1.3a; now always rendered
  (`src/components/app-shell.tsx`), holding a disabled notification bell (no backend), the theme
  toggle, and a new `src/components/user-menu.tsx` — click the account avatar for a dropdown
  showing the seeded Phase-0 user's real email (`founder@orbicrew.local`, matches `bootstrap.py`,
  not fabricated) and a disabled "Log out" (nothing real to end yet — no session exists in
  `orbicrew-web`, unlike `orbicrew-admin`'s real cookie auth). Notification/theme-toggle/avatar are
  all fixed `h-8 w-8` boxes so hover states render as true circles and all three sit on the same
  baseline (both were visual bugs the user caught: an oval hover shape and slight vertical
  misalignment, both fixed by the same change). Footer trimmed back down to just the copyright line
  — the API-health status this same session had briefly added there was removed per feedback: this
  is now purely a global layout element, not a status surface, and `src/lib/api-status.ts` (now
  unreferenced anywhere) was deleted rather than left as dead code.
- **Sidebar**: active nav item now gets a real background fill (`bg-surface-container`) in addition
  to the border/bold/color treatment 1.3a already had — every Stitch mockup shows this and 1.3a
  missed it.
- **Real Overview dashboard** (`src/app/page.tsx`, per `orbicrew_dashboard_overview`): new
  `GET /v1/dashboard/summary` (`src/orbicrew_api/dashboard.py`) returns `active_tasks` (live count
  of `queued`/`running`), `spend_today_usd`/`daily_cap_usd` (reuses the existing rolling-24h query,
  extracted out of `worker.py` into a shared `src/orbicrew_api/usage.py::get_tenant_spend_24h` so
  both call sites share one query instead of two copies), and `recent_tasks` (last 5, lightweight,
  no `task_steps` join). The page renders three real stat cards, a real "Live Activity" feed from
  `recent_tasks`, and a real "Needs Attention" panel — reusing `ApprovalItem` directly against
  `listApprovals("pending")` rather than adding a redundant pending-count field to the new endpoint.
  Deliberately **not** ported from the mockup: the fictional "Agents Status busy/idle" stat (no
  live "Agent" entity exists — the `agents` table is schema-only, confirmed via a full grep of
  `orbicrew-api`, nothing reads or writes it) and "Monthly Spend" (no monthly cap exists to show a
  real progress bar against; "Spend today" against the real rolling-24h `tenant_daily_cap_usd` was
  used instead, since that's the one real enforced number this system has).
- **New `/tasks` page** (per `orbicrew_message_agents` — that mockup's own sidebar copy shows
  "Tasks" as the active tab, confirming it's the destination for the sidebar's existing "Tasks" nav
  item, not a new concept): `chat-form.tsx` moved from `src/app/` to `src/app/tasks/` (same for its
  `actions.ts`) with all submit/poll/record/TTS logic untouched — only the render output changed,
  from a card+`<dl>` to a message-bubble thread (user bubble → three-dot typing indicator while
  queued/running → specialist agent bubble with a real initial-letter avatar, real output/model/
  cost once done, or an inline warning/error card for paused/failed instead of a fake agent reply).
  No multi-turn/history was added — the backend is still one `input_text` → one `output` per task,
  so this is one exchange per page load, not a persisted conversation.
- Saved four memories for future sessions (see `MEMORY.md`):
  [[feedback-logo-icon-naming]], [[feedback-stitch-design-references]] (the
  folder-per-page-purpose convention, explicitly requested), [[reference-brand-token-source]],
  [[feedback-global-layout-and-unbuilt-features]] (global `AppShell` + the "show unbuilt features
  disabled, never omit or fake" pattern used throughout this session).

### Manual tests run
- `uv run pytest` (orbicrew-api) — 47 passed (2 new `test_dashboard.py` cases).
- `npx tsc --noEmit`, `npm run lint`, `npm run build` (orbicrew-web) — all clean.
- Playwright against the live dev server: dark-mode screenshots of `/`, `/tasks`, `/approvals` at
  1440px; theme cycled dark→light→system via real header clicks with light-mode screenshots
  re-captured; a dedicated console-error check confirmed the hydration-mismatch warning was real
  (present before the `suppressHydrationWarning` fix, gone after); zoomed header screenshots
  confirmed the notification/theme-toggle/avatar circles are now equal-sized and aligned, and the
  theme-toggle hover state is a true circle, not an oval.
- Live end-to-end round trip: started `resources/orbicrew_dev_infra` deps (already up) plus a
  temporary `arq` worker (not left running afterward — stopped at the end of this check) since none
  was active; submitted a task from `/tasks`, confirmed via `curl` the task reached `status: "done"`
  with real `claude-haiku-4-5` output and `$0.0002` spend, then re-screenshotted the page to confirm
  the same real output rendered correctly in the agent bubble with the right specialist label/model/
  cost.
- Dashboard stat cards cross-checked against the same live data (4 active/recent tasks visible,
  `$0.30 / $20.00` spend bar, `0` pending approvals matching the empty Approvals inbox).

### Next recommended work
1. Phase 1.4 — morning summary: consolidated overnight report; `AppShell`/`SidebarNav` and the
   dashboard's `recent_tasks`/approvals patterns are ready to extend for this.
2. When Agents/Tasks/Billing/Settings pages actually ship (per their matching
   `resources/stitch_orbicrew_ui_ux_guide/` folders), flip the relevant `sidebar-nav.tsx`
   `NAV_ITEMS` entry from label-only to a real `href`.
3. When real auth/tenancy lands (Phase 2), wire `user-menu.tsx`'s "Log out" for real and replace
   the seeded placeholder email with the actual signed-in user's.

### Files / repos touched
- `repos/orbicrew-web`: `src/app/globals.css`, `src/app/layout.tsx`, `src/app/page.tsx`,
  `src/app/tasks/` (new: `page.tsx`, `chat-form.tsx`, `actions.ts`; moved out of `src/app/`),
  `src/app/approvals/actions.ts`, `src/components/app-shell.tsx`, `src/components/sidebar-nav.tsx`,
  `src/components/theme-toggle.tsx` (new), `src/components/user-menu.tsx` (new),
  `src/lib/api-dashboard.ts` (new), `src/lib/api-schema.d.ts`, `src/lib/api-status.ts` (deleted,
  unreferenced), `src/app/actions.ts` / `src/app/chat-form.tsx` / `src/app/favicon.ico` (deleted,
  moved/replaced), `public/orbicrew-{wordmark,icon}-for-{light,dark}-bg.svg` (new)
- `repos/orbicrew-api`: `src/orbicrew_api/dashboard.py` (new), `src/orbicrew_api/usage.py` (new),
  `src/orbicrew_api/worker.py`, `src/orbicrew_api/main.py`, `tests/test_dashboard.py` (new)
- `docs/development/manual-test-guide.md`, `docs/development/phase_by_phase_development_plan.md`,
  `docs/development/development-tracker.md` (this entry)
- Master memory: `MEMORY.md` + 4 new memory files (see Done above)

---

## 2026-08-07 — Phase 1.3a: Web UI design parity with Stitch

**Agent / operator:** Claude Code
**Phase:** Phase 1 (Overnight autonomy + multi-channel) — task 1.3a
**Scope:** `orbicrew-web` only — restyle `/` and `/approvals` to the Stitch design language and land
a reusable sidebar/shell primitive. No behavior changes: chat submit/poll/voice and approvals
list/approve/reject are untouched, only markup/classNames and a new shared shell wrapping them.

### Done
- Two scope calls confirmed with the user via `AskUserQuestion` before implementation (see plan
  file this session wrote and the user approved): sidebar shows all six Stitch nav items
  (Home/Agents/Tasks/Approvals/Billing/Settings) + "Hire New Agent", but items with no real page
  yet render **disabled** (muted `<span>`, no `href`) rather than being omitted; top header chrome
  with no backing functionality yet (search, notifications, avatar) is **omitted** rather than
  shipped as non-functional decoration.
- `src/app/globals.css`: replaced the old ~10-token placeholder palette with the full
  `resources/stitch_orbicrew_ui_ux_guide/orbicrew/DESIGN.md` Material-3-style token set (`surface`,
  `surface-container-lowest/low/…/highest`, `on-surface`, `on-surface-variant`, `outline(-variant)`,
  `primary(-container)`, `secondary(-container)`, `error(-container)`, plus `on-*` pairs) exposed
  through Tailwind v4's existing `@theme inline` block — same mechanism, more tokens. Also added
  DESIGN.md's 8px spacing scale (`base`/`gutter`/`margin-mobile`/`margin-desktop`/`stack-sm/md/lg`)
  and radius scale (`sm` 0.25rem / `DEFAULT` 0.5rem / `md` 0.75rem / `lg` 1rem / `xl` 1.5rem) as
  theme tokens, using Tailwind v4's actual key names confirmed by reading its own `theme.css`
  (`--radius` for the unnamed `rounded` utility, not `--radius-DEFAULT`; `max-w-*` reads from
  `--container-*`, which would have collided with the built-in `max-w-max` utility if named
  carelessly — used an arbitrary `max-w-[1280px]` value instead of a new theme key there). Old
  simplified aliases (`bg-bg-primary`, `text-text-secondary`, `accent-primary`, `border-border`,
  etc.) were dropped outright, not shimmed — every call site was being rewritten in this same
  slice, so no backwards-compat alias was needed; confirmed via grep that no stale references
  remained anywhere in `src/`.
- New `src/components/sidebar-nav.tsx` (`"use client"`, `usePathname()`-driven active state) and
  `src/components/app-shell.tsx`, mounted once in root `src/app/layout.tsx` wrapping `{children}` —
  every current route wants the shell and there's no auth/onboarding route in this app yet (unlike
  `orbicrew-admin`'s `(operator)` route-group split), so a route-group split would be speculative;
  revisit if a shell-less route is ever actually needed. Disabled nav items and the "Hire New
  Agent" button use `aria-disabled`/`disabled` + a `title="Coming soon"` rather than dead links.
- `src/app/layout.tsx`: added a `<link>` for Material Symbols Outlined (Google Fonts CSS API, same
  URL the Stitch exports use) — not routed through `next/font/google` like the two text fonts,
  since Material Symbols is a variable icon font (FILL/wght/GRAD/opsz axes via
  `font-variation-settings`) that doesn't fit `next/font`'s static-subsetting model. This trips
  `@next/next/no-page-custom-font` (a Pages Router-era lint rule that still fires on root-layout
  `<link>` tags in the App Router); added a targeted `eslint-disable-next-line` with a one-line
  rationale comment, matching the precedent set in the 1.3 approvals-page session for this exact
  kind of unavoidable-but-correct lint warning.
- `/` (`page.tsx` + `chat-form.tsx`) and `/approvals` (`page.tsx` + `approval-item.tsx`) restyled
  into the Stitch card language (`rounded-xl` / `border-outline-variant` / `surface-container-*`
  cards, `primary` buttons, badge/pill styling for status and approval-type) with all real data and
  behavior kept as-is. Explicitly did **not** port the Stitch mockups' fictional content — the
  dashboard mockup's "Live Activity" feed and "Needs Attention" panel (fake numbers, no backing
  data source on this page) and the approvals mockup's two-column code-diff preview (a fictional
  "Coding Agent PR" example with no matching data shape in this app) were left out; only the
  shell/card visual language was adopted, not invented content.
- Approvals page header now shows a real `"N Pending"` pill (`result.approvals.length`), not a
  hardcoded mockup number.

### Manual tests run
- `npx tsc --noEmit`, `npm run lint` (0 errors, 0 warnings after the targeted eslint-disable),
  `npm run build` — all clean; build output unchanged route list (`/`, `/approvals`,
  `/api/voice/*`).
- Playwright (headless Chromium, installed as a dev dependency already present from a prior
  session, run via a temporary uncommitted script) against the live dev server at 1440px and
  390px viewports: confirmed active-state highlighting swaps correctly between `/` and
  `/approvals`, disabled nav items/Hire button render visibly muted, sidebar hides and the mobile
  top bar appears below `md`, and the approvals empty state renders correctly. A live task
  submission through the restyled chat form (`POST /v1/tasks` against the real running API) showed
  the polled task card rendering `QUEUED` status correctly inside the new card styling (task didn't
  reach a terminal status in the test window because no `arq` worker process happened to be running
  at the time — unrelated to this UI-only change; not something this slice needed to fix).
- `curl` smoke on both routes (200s) plus grep confirmed the new shell markup and no leftover
  references to the old token names anywhere in `src/`.

### Next recommended work
1. Phase 1.4 — morning summary: consolidated overnight report; the new shell/nav primitives
   (`AppShell`, `SidebarNav`) are ready for this and future Phase 2 pages to mount into directly.
2. When Agents/Tasks/Billing/Settings pages actually ship, flip their `NAV_ITEMS` entries in
   `sidebar-nav.tsx` from label-only to a real `href` — no other shell change needed.
3. When auth/search/notifications land, revisit the "omit non-functional chrome" call for the top
   header (currently mobile-logo-only) and wire the real search bar/notification bell/avatar.

### Files / repos touched
- `repos/orbicrew-web`: `src/app/globals.css`, `src/app/layout.tsx`, `src/app/page.tsx`,
  `src/app/chat-form.tsx`, `src/app/approvals/page.tsx`, `src/app/approvals/approval-item.tsx`,
  `src/components/sidebar-nav.tsx` (new), `src/components/app-shell.tsx` (new)
- `docs/development/manual-test-guide.md`, `docs/development/phase_by_phase_development_plan.md`,
  `docs/development/development-tracker.md` (this entry)

---

## 2026-08-07 — Plan update: Phase 1.3a added (Stitch UI design parity)

**Agent / operator:** Claude Code
**Scope:** `docs/development/phase_by_phase_development_plan.md` only (no code changes).

User flagged that `orbicrew-web` (home page + `/approvals`) doesn't match the Stitch mockups in
`resources/stitch_orbicrew_ui_ux_guide/` — those screens were consulted only loosely so far;
pages were built functional-first (see the 1.3 entry below: "not the full sidebar nav shown in
the Stitch mockup... explicitly out of scope for this slice"). That was a deliberate per-slice
scope call, not an omission from the plan, but there was no dedicated tracked task to come back
and do the visual pass — Phase 2's UI tasks (2.4/2.5/2.2) only cover *new* CRUD/settings/billing
surfaces, not restyling what already exists.

Added **Phase 1.3a — Web UI design parity with Stitch** to the plan, between 1.3 and 1.4:
restyle the existing pages to the Stitch design language (sidebar nav shell, layout, typography,
color, components) and land reusable layout/nav primitives so 1.4+ and Phase 2 pages start from
that shell instead of bare markup. Sequenced before 1.4 (morning summary) so a third page doesn't
land un-styled too. Also updated the "suggested immediate next slice" list. No code touched in
this session.

---

## 2026-08-07 — Phase 1.3: Approval gates (resolve the Phase 1.2 escalations)

**Agent / operator:** Claude Code
**Phase:** Phase 1 (Overnight autonomy + multi-channel) — task 1.3
**Scope:** `orbicrew-api` (new approvals router) + `orbicrew-web` (first second page, `/approvals`).

### Done
- Explicit scope decision (user-confirmed before implementation): the product docs
  (`tech/03_system_design.md`, `tech/05_api_design.md`) describe approval gates as a generic
  pre-action gate — a task blocks *before* a risky/external action runs
  (`AwaitingApproval → Running`/`Cancelled`), driven by an `agents.action_whitelist` that nothing
  in the codebase enforces yet. That doesn't fit what exists: no specialist performs an
  external/irreversible action (they only return LLM text), so a pre-action whitelist gate would
  have nothing real to guard. Built the narrower, concrete thing instead: a real resolve flow for
  the two escalation types Phase 1.2 already writes into `approvals`
  (`tenant_daily_cap_exceeded`, `task_failure_escalation`). The broader pre-action gate is
  deferred until a specialist actually needs one.
- Second explicit decision: the LangGraph checkpoint for a paused/failed thread is already at
  `END`, so "approve" re-enqueuing the same `task_id` would no-op (nothing left for that thread to
  run). Approve instead spawns a **fresh task row** with the same input — a new `task_id` means a
  new checkpoint thread, no collision — bypassing the specific cap that caused the escalation for
  that one run. The original task/approval remain as the historical record, linked via new
  `approvals.resulting_task_id`. Reject just marks the approval resolved; no task mutation.
- `migrations/0003_approval_gates.sql`: `tasks.bypass_tenant_daily_cap boolean default false`;
  `approvals.resulting_task_id uuid references tasks(id)`; a `check (status in ('pending',
  'approved', 'rejected'))` constraint on `approvals.status` (previously unconstrained).
- `src/orbicrew_api/tasks.py`: extracted `create_and_enqueue_task(...)` (insert task row + arq
  enqueue, including the existing enqueue-failure → `status='failed'` handling) out of
  `_enqueue_task`'s body so both `POST /v1/tasks` and the new approve handler share one
  insert-and-enqueue path instead of duplicating it.
- New `src/orbicrew_api/approvals.py`: `GET /v1/approvals?status=` (default `pending`, "morning
  inbox" framing from the docs), `POST /v1/approvals/{id}/approve`, `POST
  /v1/approvals/{id}/reject` — tenant-scoped via `DEFAULT_TENANT_ID`, 404 on missing/wrong-tenant,
  409 (new status code in this codebase) on an already-resolved approval. Approve calls
  `create_and_enqueue_task(..., bypass_tenant_daily_cap=(approval_type ==
  'tenant_daily_cap_exceeded'))`. Wired into `main.py`.
- `office_manager.py`/`worker.py`: `bypass_tenant_daily_cap` threaded through
  `OfficeManagerState` as a real boolean, checked in `_route_condition` to skip the aggregate
  check entirely rather than mathematically forcing it to always pass.
- `orbicrew-web`: `src/lib/api-approvals.ts` (mirrors `api-tasks.ts`'s `{ok,...}` convention);
  `src/app/approvals/page.tsx` (server component, lists pending approvals or an empty state),
  `approval-item.tsx` (client component, `useActionState` + bound server actions per
  approve/reject button, matching `chat-form.tsx`'s pattern), `actions.ts` (server actions calling
  `revalidatePath("/approvals")` on success so the resolved item drops out of the list on the next
  render — no manual `router.refresh()` needed). Regenerated `api-schema.d.ts`. Added a plain
  `/approvals` link from the home page — **not** the full sidebar nav shown in the Stitch mockup
  (`resources/stitch_orbicrew_ui_ux_guide/orbicrew_approvals_inbox/`); building that shell was
  explicitly out of scope for this slice.
- Tests: new `tests/test_approvals.py` (list/approve/reject/404/409, mocked `pg_pool`/`arq_pool`,
  same style as `test_tasks.py`); `test_worker.py` and `test_office_manager.py` extended for the
  bypass flag.

### Bug caught during manual testing (fixed before landing)
- First implementation passed `tenant_daily_cap_usd=float("inf")` to bypass the aggregate check
  instead of a real boolean. This crashed the *second* worker run with
  `psycopg.errors.InvalidTextRepresentation: invalid input syntax for type json` /
  `Token "Infinity" is invalid` — the LangGraph Postgres checkpointer writes graph state as a JSON
  blob, and Postgres's strict JSON parser rejects the literal `Infinity` token that Python's
  default float-to-JSON path produces (confirmed live: `run_office_manager` worked fine
  standalone since no checkpointer was involved in that reproduction, but failed the moment a real
  worker run persisted state to Postgres). Fixed by adding a real `bypass_tenant_daily_cap: bool`
  to `OfficeManagerState` and short-circuiting the aggregate check in `_route_condition`, instead
  of smuggling the bypass through a numeric sentinel. Worth remembering generally: don't pass
  `float("inf")`/`nan` into any LangGraph-checkpointed state — it round-trips through Postgres
  JSON.

### Manual tests run
- `uv run pytest` — 45 passed (up from 36).
- `uv run orbicrew-api-migrate` — `0003_approval_gates.sql` applied cleanly against the live local
  Postgres (no existing rows violated the new `approvals.status` check constraint).
- Live end-to-end round trip (api + worker running natively): reused a `tenant_daily_cap_exceeded`
  approval from Phase 1.2's own manual testing, `POST .../approve` → new task row created,
  completed for real (`status: done`, real Anthropic output, `usage_records` written) despite the
  tenant cap still being exceeded; confirmed the original approval was `status: approved` with
  `resulting_task_id` pointing at the new task. `POST .../reject` on a `task_failure_escalation`
  row → `status: rejected`, no new task, original task untouched. Double-approve/double-reject on
  an already-resolved id → 409; approve/reject on a random id → 404.
- `orbicrew-web`: `npx tsc --noEmit`, `npm run lint` (0 errors/warnings after adding targeted
  `eslint-disable` comments for the two trailing `useActionState`-required-but-unused server
  action params — same pattern React requires elsewhere in this codebase), `npm run build` all
  clean; `/approvals` confirmed server-rendering both the empty state and a populated approval
  card (via a live Postgres row) with correct fields and Approve/Reject buttons present.
- Process hygiene note: mid-session discovered the user's own long-running `orbicrew-api` (with
  `reload=True`) and `orbicrew-web`/`orbicrew-admin` dev servers were already running in separate
  terminals throughout this session; this agent's own ad hoc `uv run uvicorn --port 8000`
  instances almost certainly lost the port-8000 bind race each time and exited quietly, meaning
  the user's own reload-enabled server (auto-picking up each file edit) served all of this
  session's manual API testing. Functionally harmless (same code, same DB) but worth knowing for
  future sessions — check `lsof -nP -iTCP:8000` before assuming a background `uvicorn` command
  actually bound the port.

### Next recommended work
1. Phase 1.4 — morning summary: consolidated overnight report; a natural place to also surface
   `approvals` rows created overnight (currently only visible by visiting `/approvals` or polling
   `GET /v1/approvals`).
2. If a specialist ever needs to perform a real external/irreversible action, revisit the broader
   pre-action `AwaitingApproval` gate described in the docs — this slice deliberately did not
   build it.

### Files / repos touched
- `repos/orbicrew-api`: `migrations/0003_approval_gates.sql` (new), `src/orbicrew_api/approvals.py`
  (new), `src/orbicrew_api/tasks.py`, `src/orbicrew_api/worker.py`, `src/orbicrew_api/office_manager.py`,
  `src/orbicrew_api/main.py`, `tests/test_approvals.py` (new), `tests/test_worker.py`,
  `tests/test_office_manager.py`, `README.md`
- `repos/orbicrew-web`: `src/lib/api-approvals.ts` (new), `src/app/approvals/` (new: `page.tsx`,
  `approval-item.tsx`, `actions.ts`), `src/app/page.tsx`, `src/lib/api-schema.d.ts`
- `docs/development/manual-test-guide.md`, `docs/development/phase_by_phase_development_plan.md`,
  `docs/development/development-tracker.md` (this entry)

---

## 2026-08-07 — Phase 1.2: Budget hardening (nightly aggregate caps + retry-then-escalate)

**Agent / operator:** Claude Code
**Phase:** Phase 1 (Overnight autonomy + multi-channel) — task 1.2
**Scope:** `orbicrew-api` only.

### Done
- `migrations/0002_budget_hardening.sql`: `tasks` gains `retry_count int not null default 0`; `approvals` (previously schema-only — zero application code touched it) gains `tenant_id uuid not null references tenants (id)` (backfilled from `tasks.tenant_id`) and `approval_type text not null default 'manual'`, plus `idx_approvals_tenant_status`.
- `settings.py`: `tenant_daily_cap_usd` (default `$20.00`), `task_max_retries` (default `3`), `task_retry_base_delay_seconds` (default `30`), `task_retry_max_delay_seconds` (default `300`) — all `.env`-overridable.
- `budget_guard.check_budget` is unchanged (same math, docstring updated) and is now called **twice** from `office_manager._route_condition`: once with the existing per-task inputs, once with per-tenant rolling-24h aggregate inputs (`tenant_spend_24h_usd`/`tenant_daily_cap_usd`, computed by `worker.py` from `usage_records` and passed into `run_office_manager` — the graph itself still has no DB access, matching the existing pattern for `spend_so_far_usd`). A new `daily_cap_paused` node/edge mirrors `budget_paused` but sets `pause_type: "daily_cap"` (vs `"per_task"`) on the returned state, which `worker.py` reads to decide whether to escalate.
- `worker.py`: `run_task_job` no longer marks a task `failed` and re-raises on the first exception. New `_handle_task_failure` increments `tasks.retry_count`; under `task_max_retries` it sets `status = 'queued'` and re-enqueues the same `task_id` via `ctx["redis"].enqueue_job(..., _defer_by=delay)` (arq always injects `ctx["redis"]`, the same pool the worker runs on — no new pool needed) with exponential backoff (`min(base * 2**(retry_count-1), max_delay)`); once exhausted it sets `status = 'failed'` and inserts an `approvals` row (`approval_type = 'task_failure_escalation'`) with the exception message. Separately, after a successful graph run, a `pause_type == "daily_cap"` result also inserts an `approvals` row (`approval_type = 'tenant_daily_cap_exceeded'`). A per-task pause (`pause_type == "per_task"`) does **not** escalate — that's an immediate response already visible to the task submitter via `GET /v1/tasks/:id`, not an unattended-run scenario.
- Tests: `tests/test_worker.py` gained daily-cap-escalation, retry-under-limit, and retry-exhausted-escalation cases (replacing the old single "marks failed on exception" test, which no longer matches worker behavior); `tests/test_office_manager.py` gained a daily-cap-pause routing test and a `pause_type` assertion on the existing per-task pause test.

### Decisions / assumptions
- **Rolling 24h window, not calendar-day reset** — avoids timezone/reset-boundary design questions and needs no cron job; the aggregate check is inline in the same place the per-task check already runs, using the existing `idx_usage_records_tenant_created` index (`select coalesce(sum(cost_usd),0) from usage_records where tenant_id=$1 and created_at >= now() - interval '24 hours'`).
- **`check_budget` reused as-is for both scopes** — the remaining-budget math is identical; only the inputs' semantics differ. No new budget-math function was added (DRY per this workspace's coding principles).
- **Manual/app-level retry, not arq's built-in retry** — `run_task_job` never re-raises anymore, so arq's own per-function retry (unused/unconfigured here) never engages; all retry state (`retry_count`) lives in the `tasks` row so it survives across separate arq job instances (each retry is a fresh `enqueue_job` call, not the same job retried by arq).
- **Escalation via the existing (previously unused) `approvals` table**, not a new notification mechanism — no email/Slack/webhook exists yet (that's future scope, likely alongside Phase 1.4's morning summary); `approvals` rows are currently write-only from this slice, with no read/resolve API or UI (that's explicitly Phase 1.3).
- **Retries are cheap because of the Phase 1.1 checkpointer** — re-enqueuing reuses the same `task_id` as the LangGraph `thread_id`, so a retry resumes from the last node that completed successfully (e.g. a specialist-call failure retries just that call, not `classify`/`route`).

### Manual tests run
- `uv run pytest` — 36 passed (up from 33; net +3: two exception-path tests replaced one, plus one new daily-cap-escalation worker test and one new daily-cap-pause office_manager test — see exact breakdown in Done above).
- `uv run orbicrew-api-migrate` against the live local Postgres — applied `0002_budget_hardening.sql` cleanly (no pre-existing `approvals` rows to backfill).
- Live end-to-end round trip (API + worker running natively against `resources/orbicrew_dev_infra`): restarted the worker with `TENANT_DAILY_CAP_USD` set just above existing 24h spend (`$0.31` vs `$0.3007` already spent), submitted a task → ended `status: "paused"`, `steps` ending in `daily_cap_pause`, and confirmed via `psql` an `approvals` row (`approval_type = 'tenant_daily_cap_exceeded'`) referencing that task.
- Restarted the worker with an invalid `ANTHROPIC_API_KEY` and `TASK_MAX_RETRIES=2`/fast backoff, submitted a task → confirmed via `psql` polling that `tasks.retry_count` climbed `0→1→2→3` across attempts (status cycling `running`→`queued`→`running`...), the task ended `status: "failed"` after exhausting retries, and an `approvals` row (`approval_type = 'task_failure_escalation'`, containing the real Anthropic 401 error message) was written.

### Next recommended work
1. Phase 1.3 — approval gates: a real read/resolve API + UI for the `approvals` rows this slice now writes (currently write-only; nothing surfaces `status: 'pending'` approvals anywhere).
2. Phase 1.4 — morning summary: a natural place to also surface `approvals` rows created overnight, once 1.3 exists.

### Files / repos touched
- `repos/orbicrew-api`: `migrations/0002_budget_hardening.sql` (new), `src/orbicrew_api/settings.py`, `src/orbicrew_api/budget_guard.py`, `src/orbicrew_api/office_manager.py`, `src/orbicrew_api/worker.py`, `tests/test_worker.py`, `tests/test_office_manager.py`, `README.md`
- `docs/development/manual-test-guide.md`, `docs/development/phase_by_phase_development_plan.md`, `docs/development/development-tracker.md` (this entry)

---

## 2026-08-07 — Phase 1.1: Task queue + checkpointing

**Agent / operator:** Claude Code
**Phase:** Phase 1 (Overnight autonomy + multi-channel) — task 1.1
**Scope:** `orbicrew-api` (async task queue + LangGraph checkpointing) + `orbicrew-web` (poll for task completion instead of treating the submit response as final).

### Done
- `repos/orbicrew-api`: added `arq` and `langgraph-checkpoint-postgres` deps. `POST /v1/tasks` (`src/orbicrew_api/tasks.py`) now inserts the task row (`status: "queued"`), enqueues `run_task_job` via `state.arq_pool` (new `arq_pool: ArqRedis` on `AppState`, `deps.py`), and returns immediately — it no longer runs the Office Manager graph inline. New `src/orbicrew_api/worker.py` (arq `WorkerSettings`, run via `uv run arq orbicrew_api.worker.WorkerSettings`) owns execution: `run_task_job` loads the task, runs `run_office_manager`, and persists steps/`usage_records`/final status — the same persistence logic that used to live in the HTTP handler, moved wholesale (not duplicated). `office_manager.py`'s `build_graph`/`run_office_manager` gained `checkpointer`/`thread_id` params; the graph is compiled with `langgraph-checkpoint-postgres`'s `AsyncPostgresSaver` (against the same `database_url`, keyed by `thread_id = task_id`), and `run_office_manager` checks `checkpointer.aget_tuple(config)` to decide between a fresh `ainvoke(initial_state, config)` and a resuming `ainvoke(None, config)`. `TaskResponse.status` gained `"queued"`.
- `repos/orbicrew-web`: `chat-form.tsx` no longer treats the `POST /v1/tasks` response as final (it's now `"queued"` immediately) — it polls `GET /v1/tasks/:id` (new `getTask()` in `api-tasks.ts`) every 1.5s until a terminal status, merging into the existing `state.task` render path so the rest of the UI (spend/output/TTS) is unaffected. Regenerated `api-schema.d.ts`.

### Decisions / assumptions
- **`arq`, not Celery** (the product docs literally say "Redis + BullMQ/Celery", bootstrap doc says "or chosen queue") — user's explicit choice after a trade-off review. The whole stack is already async (`asyncpg`, `redis.asyncio`, LangGraph's `ainvoke`); arq's worker functions are plain `async def`s that drop straight in, vs. Celery's sync worker model needing `asyncio.run()` wrapping everywhere plus a separate result-backend config. `arq==0.25.0` was pinned by dependency resolution (`arq>=0.26` requires `redis<6`, which conflicts with the `redis>=6.2.0` already in use for readiness checks).
- **Postgres-backed checkpointer, not in-memory** — `tech/03_system_design.md`'s "every long-running task is resumable... checkpointed state, not in-memory-only execution" principle rules out `MemorySaver` for real use; reuses the existing Postgres instance rather than adding a new datastore. The checkpointer's own tables (`checkpoints`, `checkpoint_writes`, etc.) are created via `AsyncPostgresSaver.setup()` on worker startup — deliberately **not** one of the hand-rolled `migrations/*.sql` files, since that schema is owned/versioned by the `langgraph-checkpoint-postgres` library, not this app.
- **Resume semantics rely on arq's default in-progress lock timeout, not custom heartbeat/redelivery logic** — if a worker is killed, the job's in-progress marker in Redis expires on arq's default schedule before another worker instance can pick it back up; no custom crash-detection was built (would be premature for a single-user phase). Manually verified end-to-end instead (see below) by re-enqueueing the same `task_id` after a kill, which is the same code path a redelivery would take.
- Task-step persistence (the `task_steps` table) is still an all-at-once write after the graph fully returns, not per-node — the actual resumability mechanism is LangGraph's own checkpoint tables, not `task_steps`. This means a crash before the graph returns leaves `task_steps` empty for that task even though checkpoints exist; acceptable since `task_steps` is a post-hoc summary, not the source of truth for resume.

### Manual tests run
- `uv run pytest` (orbicrew-api) — 33 passed (up from 28; new `test_worker.py` adds 4, `test_office_manager.py` gains 1 resume test, `test_tasks.py`'s POST tests were rewritten in place for enqueue behavior — net count unchanged there).
- `npx tsc --noEmit`, `npm run lint`, `npm run build` (orbicrew-web) — all clean.
- Real end-to-end round trip: started Postgres/Redis (already up via `resources/orbicrew_dev_infra`), ran the API and a worker natively, `POST /v1/tasks` → confirmed immediate `status: "queued"`, polled `GET /v1/tasks/:id` → `running` → `done` with a real Anthropic `claude-haiku-4-5` response and correct `usage_records`/step trace.
- Crash/resume: submitted a second task, `kill -9`'d the worker ~50ms later (mid-specialist-call). Confirmed via `docker exec orbicrew-postgres psql`: task stuck at `status: "running"`, 4 rows in the `checkpoints` table for that `thread_id`, 0 rows in `task_steps`. Restarted the worker, manually re-enqueued the same `task_id` (simulating redelivery) — it resumed and completed in one more checkpoint (5 total, not 4 fresh ones) with exactly 3 `task_steps` rows (`classify`/`route`/`specialist_execute`, no duplicates) — confirms `classify`/`route` were not re-run.

### Next recommended work
1. Phase 1.2 — budget hardening (nightly aggregate caps, retry-then-escalate).
2. If real overnight (multi-hour) autonomous runs become common, revisit the "no custom heartbeat/redelivery" assumption above — arq's default in-progress lock timeout may be too long or too short depending on real task durations.

### Files / repos touched
- `repos/orbicrew-api`: `src/orbicrew_api/tasks.py`, `src/orbicrew_api/office_manager.py`, `src/orbicrew_api/worker.py` (new), `src/orbicrew_api/deps.py`, `pyproject.toml`, `uv.lock`, `tests/test_tasks.py`, `tests/test_worker.py` (new), `tests/test_office_manager.py`, `README.md`
- `repos/orbicrew-web`: `src/lib/api-tasks.ts`, `src/app/chat-form.tsx`, `src/lib/api-schema.d.ts`
- `docs/development/manual-test-guide.md`, `docs/development/phase_by_phase_development_plan.md`, `docs/development/development-tracker.md` (this entry)

---

## 2026-08-07 — Phase 0.8: Voice (Whisper STT + TTS)

**Agent / operator:** Claude Code
**Phase:** Phase 0 (Personal tool) — task 0.8
**Scope:** `orbicrew-api` (two new endpoints) + `orbicrew-web` (mic recording, auto-submit, auto-play).

### Done
- `repos/orbicrew-api`: added `openai` + `python-multipart` deps. New `src/orbicrew_api/voice.py`: `POST /v1/voice/transcribe` (multipart audio upload → OpenAI `whisper-1` → `{text}`) and `POST /v1/voice/speak` (JSON `{text}` → OpenAI `tts-1` → raw `audio/mpeg` bytes). `settings.py` gained `openai_api_key`; `.env.example` documents `OPENAI_API_KEY`. Wired into `main.py`. Tests in `tests/test_voice.py` (mocked OpenAI client): happy path + empty-input 422s + upstream-error 502 for both endpoints.
- `repos/orbicrew-web`: two Route Handlers (`src/app/api/voice/transcribe/route.ts`, `.../speak/route.ts`) proxy to `orbicrew-api` server-side — same "avoid browser CORS + keep the API URL server-only" pattern as `submitTaskAction`, and deliberately Route Handlers rather than Server Actions because Server Actions cap request bodies at 1MB by default (recorded audio can exceed that) and aren't a natural fit for streaming a binary MP3 response.
- `chat-form.tsx`: added a "Record" button using `MediaRecorder` (prefers `audio/webm`) to capture mic audio; on stop, uploads the blob to `/api/voice/transcribe`, fills the (uncontrolled, ref-based) textarea with the transcript, and calls `formRef.current.requestSubmit()` to auto-submit — no second manual click needed (matches the "full voice loop" scope the user picked over a minimal fill-only version). A `useEffect` keyed on `state.task` fetches `/api/voice/speak` for the completed task's output once (deduped via a ref holding the last-spoken task id), turns the MP3 into a blob URL, and renders `<audio autoPlay controls>` — `controls` stays visible so a browser autoplay block still leaves a manual play option.
- Regenerated `src/lib/api-schema.d.ts` (`npm run codegen:api-types`) so the new endpoints are in the typed OpenAPI contract even though the voice Route Handlers use plain `fetch`/`FormData`, not `openapi-fetch` (binary/multipart bodies aren't a good fit for that client).

### Decisions / assumptions
- **OpenAI Whisper API + OpenAI TTS, not self-hosted Whisper or ElevenLabs** — user's explicit choice. Self-hosted Whisper adds infra (model download, CPU/GPU) not worth it for single-user Phase 0; OpenAI TTS reuses the one API key already needed for STT instead of a second provider/account. Revisit provider choice in Phase 3.5 (multilingual tuning from real pilot usage) per the phase plan — this is a placeholder pick, not a locked-in architecture decision.
- **Full voice loop (auto-submit + auto-play), not minimal (fill-box-only)** — user's explicit scope choice over the more conservative "record → fill box → still need Send click" option.
- **Route Handlers, not Server Actions, for both voice endpoints** — Server Actions' default 1MB body cap and lack of streaming-binary-response ergonomics made them a worse fit than a plain Route Handler proxy; documented in `node_modules/next/dist/docs` per this repo's `AGENTS.md` (Next 16 conventions may differ from training data).
- **No DB persistence for voice calls** — unlike text tasks (`tasks`/`usage_records`), STT/TTS calls aren't written to Postgres or counted against `budget_guard`. Phase 0.8 scope is "founder can talk to the chat instead of typing," not cost-tracking voice usage; revisit if voice becomes the primary interaction mode (would need a `usage_records`-style entry per call for real cost visibility).
- Browser autoplay policies can block `<audio autoPlay>` without a fresh user gesture; kept `controls` on the element so the reply is always at least one click away rather than silently lost.

### Manual tests run
- `uv run pytest` (orbicrew-api) — 28 passed (up from 23; 5 new voice tests).
- `npx tsc --noEmit`, `npm run lint`, `npm run build` (orbicrew-web) — all clean; build output confirms `ƒ /api/voice/speak` and `ƒ /api/voice/transcribe` routes.
- Direct `curl` round-trip against the live API: `POST /v1/voice/speak {"text":"Hello from Orbicrew."}` → real MPEG audio (128kbps, confirmed via `file`) → fed that file into `POST /v1/voice/transcribe` → `{"text":"Hello from Orbicrew."}` (exact match).
- End-to-end browser test via a temporary (not committed) Playwright script: launched Chromium with `--use-fake-device-for-media-stream --use-fake-ui-for-media-stream --use-file-for-fake-audio-capture=<synthesized WAV via macOS `say`+`afconvert`>`, granted mic permission, clicked Record → Stop on `http://localhost:3000`, confirmed the real Whisper transcript filled the textarea, the task auto-submitted and completed for real (`writing` specialist, `claude-haiku-4-5`, real haiku output), and the `<audio>` element received a populated `blob:` src from a real `tts-1` response. See manual test guide §0.D.

### Next recommended work
1. Phase 0.9: daily-use soak — 2-3 weeks of real personal use (text + voice) before starting Phase 1.
2. If voice becomes a primary interaction path during the soak, reconsider persisting voice call cost/usage (currently untracked, see assumptions above).

### Files / repos touched
- `repos/orbicrew-api`: `src/orbicrew_api/voice.py`, `src/orbicrew_api/settings.py`, `src/orbicrew_api/main.py`, `tests/test_voice.py`, `pyproject.toml`, `uv.lock`, `.env.example`
- `repos/orbicrew-web`: `src/app/chat-form.tsx`, `src/app/api/voice/transcribe/route.ts`, `src/app/api/voice/speak/route.ts`, `src/lib/api-schema.d.ts`
- `docs/development/manual-test-guide.md`, `docs/development/phase_by_phase_development_plan.md`, `docs/development/development-tracker.md` (this entry)

---

## 2026-08-07 — `orbicrew-admin` auth shell

**Agent / operator:** Claude Code
**Phase:** Early admin work (per `AGENT_BOOTSTRAP.md` §5 / plan §"Admin surfaces") — not a numbered Phase 0 task, done alongside it since Phase 0.7 wrapped up.
**Scope:** `orbicrew-admin` only — Next.js scaffold matching `orbicrew-web`'s conventions, plus a shared-secret operator login gating an empty operator console layout. No `orbicrew-api` calls (privileged `/v1/ops/*` endpoints don't exist yet — that's Phase 2.6).

### Done
- Scaffolded `orbicrew-admin` as a Next.js (App Router) + React + TypeScript + Tailwind app, mirroring `orbicrew-web`'s `package.json`/`tsconfig.json`/`eslint.config.mjs`/`postcss.config.mjs`/brand fonts (Bricolage Grotesque + Hanken Grotesk) and palette (`globals.css`).
- `src/lib/session.ts`: HMAC-SHA256-signed, time-limited (`ORBICREW_ADMIN_SESSION_SECRET`) session cookie helpers (`createSessionCookieValue`/`isSessionValid`), and `verifyOperatorPassword` — constant-time comparison (via `timingSafeEqual` on fixed-length SHA-256 digests, so unequal-length inputs don't short-circuit) against a single shared `ORBICREW_ADMIN_OPERATOR_PASSWORD`.
- `src/app/login/`: `page.tsx` + client `login-form.tsx` (`useActionState`/`useFormStatus`, same pattern as `orbicrew-web`'s `ChatForm`) + `actions.ts` server action — checks the password, sets an httpOnly/`sameSite=lax` session cookie (`secure` in production), redirects to `/`.
- `src/app/(operator)/`: route group — `layout.tsx` reads the session cookie server-side and `redirect("/login")` if missing/invalid/expired, otherwise renders an operator header (nav placeholders: Overview/Tenants/Costs/Kill switch, Sign out) around `page.tsx` (an empty console placeholder explaining Phase 2.6 will wire real data). `actions.ts` holds `logoutAction` (clears the cookie, redirects to `/login`).
- Root `src/app/layout.tsx` sets metadata to "Orbicrew Admin" and applies the same font/palette setup as `orbicrew-web`.
- README updated: dev quickstart (`npm install` / `cp .env.example .env` / `npm run dev`) and a "Status" section describing the auth shell and what's still missing.

### Decisions / assumptions
- **Shared-secret password, not per-operator accounts** — there's no operator identity model yet (that's Phase 2.6, against `orbicrew-api`'s privileged `/v1/ops/*`). A single env-var passphrase is enough to keep the empty console from being wide open on a dev/early-access deploy without over-building auth ahead of the real requirement. Documented as a placeholder in the README, this tracker entry, and a code comment in `session.ts`.
- **Auth check lives in a Server Component layout, not `middleware.ts`** — Next.js Middleware runs on the Edge runtime by default, which doesn't have Node's `crypto` module (only Web Crypto), so HMAC verification there would need a separate Web Crypto implementation. Doing the check in `(operator)/layout.tsx` (a normal Server Component, Node runtime) keeps one code path in `session.ts` and avoids that duplication — acceptable since this app has no other routes yet that would benefit from edge-level gating.
- **Session cookie is a signed `expiresAt.hmac` pair, not a JWT/library** — the only claim needed right now is "a valid operator signed in before this timestamp"; a hand-rolled HMAC pair is simpler than pulling in a JWT dependency for that one bit of state, and can be swapped for real session/identity data once Phase 2.6 lands.
- `openapi-fetch`/`openapi-typescript` were **not** added yet (unlike `orbicrew-web`, Phase 0.7) — there's no `orbicrew-api` endpoint for this app to call yet; add the typed client when Phase 2.6 wires `/v1/ops/*`.

### Manual tests run
- `npm install`, `npx tsc --noEmit` (clean), `npm run lint` (clean), `npm run build` (clean; `/`, `/_not-found`, `/login` routes generated).
- Playwright (headless Chromium, installed with `--no-save` for this check only, not committed) against `npm run dev`: unauthenticated `/` → redirects to `/login`; wrong password → stays on `/login` with the error message; correct password → session cookie set, redirects to `/` operator console; reload stays authenticated; "Sign out" clears the cookie and redirects to `/login`; `/` after sign-out redirects to `/login` again. See manual test guide §0.E.

### Next recommended work
1. Phase 0.8: voice (optional) once the text chat path is in daily use.
2. When Phase 2.6 (platform operator console) starts: replace the shared-secret login with real per-operator auth against `orbicrew-api`'s privileged `/v1/ops/*` endpoints, add the `openapi-typescript`/`openapi-fetch` typed client (same pattern as `orbicrew-web`, Phase 0.7), and wire the Overview/Tenants/Costs/Kill switch nav placeholders to real data.

### Files / repos touched
- `repos/orbicrew-admin`: full initial scaffold — `package.json`, `tsconfig.json`, `next.config.ts`, `eslint.config.mjs`, `postcss.config.mjs`, `.env.example`, `AGENTS.md`/`CLAUDE.md` (Next.js-generated), `src/lib/session.ts`, `src/app/layout.tsx`, `src/app/globals.css`, `src/app/login/{page,login-form,actions}.tsx`, `src/app/(operator)/{layout,page,actions}.tsx`, `README.md`
- `docs/development/manual-test-guide.md`, `docs/development/phase_by_phase_development_plan.md`, `docs/development/development-tracker.md` (this entry)

---

## 2026-08-07 — Phase 0.7: OpenAPI + generated TS client

**Agent / operator:** Claude Code
**Phase:** Phase 0 (Personal tool) — task 0.7
**Scope:** `orbicrew-web` only — replace the hand-written `Task`/`TaskStep`/`SubmitTaskRequest`-shaped types and manual `fetch` calls in `api-status.ts`/`api-tasks.ts` with a generated OpenAPI contract. No `orbicrew-api` code changes (FastAPI already exposes `/openapi.json` for free).

### Done
- Added `openapi-typescript` (devDependency) and `openapi-fetch` (dependency) to `orbicrew-web`.
- `package.json`: new `codegen:api-types` script — `openapi-typescript "${ORBICREW_API_URL:-http://localhost:8000}/openapi.json" -o src/lib/api-schema.d.ts --default-non-nullable false`. Fetches from a running local API rather than reading `orbicrew-api`'s source directly, since the two are independent nested repos (nested-repo isolation) and this keeps `orbicrew-web` buildable from a clone that only has the API's public HTTP contract, not its source.
- `src/lib/api-schema.d.ts` (new, generated, committed): `paths`/`components`/`operations` types for `/health`, `/ready`, `/v1/tasks`, `/v1/tasks/{task_id}`.
- `src/lib/api-client.ts` (new): `apiBaseUrl()` (moved here, still `ORBICREW_API_URL`-driven) + a module-level `apiClient = createClient<paths>({ baseUrl: apiBaseUrl() })` (`openapi-fetch`) shared by both callers.
- `src/lib/api-status.ts` / `src/lib/api-tasks.ts`: rewritten to import types from `components["schemas"][...]` instead of hand-written shapes, and to call `apiClient.GET`/`apiClient.POST` instead of raw `fetch` + manual JSON parsing/casting.
- `src/app/page.tsx`: one-line fix — `apiStatus.ready.checks` is now optional in the generated type (`checks?`), so `Object.entries(...)` needed `?? {}`.

### Decisions / assumptions
- **`--default-non-nullable false`** on the codegen command — by default `openapi-typescript` renders any schema property that has a `default` (e.g. `SubmitTaskRequest.channel`, `.input_type`, `.input_lang`) as non-optional in the generated TS type, even though FastAPI's actual OpenAPI `required` array correctly excludes them. That default behavior is meant for response schemas ("the server will always populate this"); for a *request* body type it's wrong — it would force every caller to pass those fields even though the API happily defaults them. Turning the flag off restores the correct optionality from the real `required` array.
- **`GET /health` and `GET /ready` don't use the `if (res.error)` pattern** — both endpoints document only a `200` response in the OpenAPI schema, so `openapi-fetch`'s `error` field types as unreachable (`never`) for them; TypeScript's control-flow analysis then treats the `if (res.error) {...}` block itself as unreachable dead code and narrows `res` to `never` inside it, breaking any further property access. Used `!res.response.ok || !res.data` instead (checking the raw `Response.ok`, which isn't narrowed away) so a real-world non-2xx (proxy error, unhandled 500) still gets caught at runtime even though the types say it "can't" happen. `POST /v1/tasks` does document a `422` response, so the ordinary `if (error || !data)` pattern works there unchanged.
- **Generated `api-schema.d.ts` is committed, not gitignored** — regenerating it requires a running `orbicrew-api`, which isn't guaranteed at `orbicrew-web` clone/build time (e.g. CI, or a checkout without the sibling API up). Regenerate via `npm run codegen:api-types` whenever the API's request/response shapes change, then commit the diff.
- Chose `openapi-fetch` (a typed wrapper around `fetch`) over a heavier generated SDK (e.g. `openapi-generator`) — it's a thin, dependency-light layer that keeps the existing server-action-based, no-browser-CORS calling convention from Phase 0.6 unchanged; only the call site's type-safety improved.

### Manual tests run
- `npm run codegen:api-types` against the live local API — regenerates cleanly.
- `npx tsc --noEmit` — clean.
- `npm run lint` — clean.
- `curl http://localhost:3000/` — status section still renders `healthy` / postgres+redis `ok` through the new typed client.
- Directly invoked `submitTask()` (via `npx tsx`, pointed at the live API) with a real prompt — returned a real, non-canned `TaskResponse` (`specialist: "writing"`, `tier: "cheap"`, `model: "claude-haiku-4-5"`, real `spend_so_far_usd`, ordered `steps`), confirming the `openapi-fetch` POST call and generated `TaskResponse` type match the live API exactly.

### Next recommended work
1. Phase 0.8: voice (optional) once the text chat path is in daily use.
2. Whenever `orbicrew-api`'s `SubmitTaskRequest`/`TaskResponse` (or new endpoints) change, re-run `npm run codegen:api-types` in `orbicrew-web` and commit the regenerated `api-schema.d.ts` — it will not update itself.
3. If `GET /v1/tasks/:id` ever gets used from `orbicrew-web` (e.g. polling once Phase 1.1 lands), reuse `apiClient.GET("/v1/tasks/{task_id}", ...)` — the type is already generated.

### Files / repos touched
- `repos/orbicrew-web`: `package.json`, `package-lock.json`, `src/lib/api-schema.d.ts` (new), `src/lib/api-client.ts` (new), `src/lib/api-status.ts`, `src/lib/api-tasks.ts`, `src/app/page.tsx`
- `docs/development/manual-test-guide.md`, `docs/development/phase_by_phase_development_plan.md`, `docs/development/development-tracker.md` (this entry)

---

## 2026-08-07 — Phase 0.6: standard chat GUI

**Agent / operator:** Claude Code
**Phase:** Phase 0 (Personal tool) — task 0.6
**Scope:** `orbicrew-web` only — thinnest chat UI that submits a task to `orbicrew-api` and shows status/tier/model/output. No OpenAPI/typed client yet (0.7), no polling/streaming (backend still runs each task synchronously in one request).

### Done
- `src/lib/api-tasks.ts` (new): `Task`/`TaskStep` types mirroring `orbicrew-api`'s `TaskResponse`/`TaskStepResponse`, and `submitTask(inputText)` — server-side `fetch` (`ORBICREW_API_URL`) to `POST /v1/tasks`, same no-browser-CORS convention as the existing `fetchApiStatus`. Extracted `apiBaseUrl()` out of `api-status.ts` so both modules share it (was previously private to that file).
- `src/app/actions.ts` (new): `submitTaskAction(prevState, formData)` Server Action — trims/validates `input_text`, calls `submitTask`, returns `{ task, error }`.
- `src/app/chat-form.tsx` (new): `"use client"` component — `useActionState` + `useFormStatus` around a `<textarea>` + submit button (React Server Actions form pattern, per `node_modules/next/dist/docs/01-app/02-guides/forms.md` since this Next.js 16.3 pin is ahead of training data per the repo's `AGENTS.md` warning). Renders task id/status, specialist/tier/model/spend as a `<dl>` grid, and the raw output text.
- Wired `<ChatForm />` into `src/app/page.tsx` below the existing API-status section (replacing the old "arrives in the next slice" placeholder paragraph).
- `npx tsc --noEmit` and `npm run lint` both clean.

### Decisions / assumptions
- **Server Actions, not a client-side `fetch` + browser CORS** — consistent with the Phase 0.1 decision that browser code never talks to `ORBICREW_API_URL` directly; keeps `orbicrew-api` CORS config unchanged.
- **No polling** — `POST /v1/tasks` already runs the whole LangGraph synchronously and returns the final `TaskResponse` in one round trip (Phase 0.3/1.1 decision), so the Server Action's response *is* the finished task; a queue-backed async flow (Phase 1.1) is the natural point to add polling or SSE.
- Hand-wrote the `Task`/`SubmitTaskRequest`-shaped types in `api-tasks.ts` instead of generating them — Phase 0.7 ("OpenAPI + TS client") is explicitly the next plan item for replacing this with a generated contract; added it to the plan's "suggested immediate next slice" list since it wasn't there before.
- Found and fixed a bug during manual verification: initially exported a plain object (`initialSubmitTaskState`) from the `"use server"` file `actions.ts`, which Next.js rejects ("a use server file can only export async functions") — moved that constant into `chat-form.tsx` instead, since only functions (and Server Action-safe types) may live in a Server Actions module.

### Manual tests run
- `npx tsc --noEmit` — clean; `npm run lint` — clean
- Playwright (headless Chromium, installed for this check) against the running dev server + live API/Postgres/Redis: submitted "Write a two-line haiku about clouds" → real non-canned output, `specialist: writing`, `tier: cheap`, `model: claude-haiku-4-5`, real `spend_so_far_usd` (screenshot captured, matches expected UI)
- Empty/whitespace-only submission (bypassing the HTML `required` attribute to reach the server-side check) → `"Enter a task before submitting."` error rendered, no API call made

### Next recommended work
1. Phase 0.7: OpenAPI schema from `orbicrew-api` + generated TS client for `orbicrew-web`, replacing the hand-written types in `api-tasks.ts`.
2. Phase 0.8: voice (optional) once the text chat path is in daily use.
3. When Phase 1.1 (task queue) lands, revisit this chat form — synchronous single-request submission won't hold once tasks are queued/checkpointed.

### Files / repos touched
- `repos/orbicrew-web`: `src/lib/api-status.ts`, `src/lib/api-tasks.ts` (new), `src/app/actions.ts` (new), `src/app/chat-form.tsx` (new), `src/app/page.tsx`
- `docs/development/manual-test-guide.md`, `docs/development/phase_by_phase_development_plan.md`, `docs/development/development-tracker.md` (this entry)

---

## 2026-08-07 — Phase 0.5: real specialists (Anthropic API)

**Agent / operator:** Claude Code
**Phase:** Phase 0 (Personal tool) — task 0.5
**Scope:** `orbicrew-api` only — replace the canned specialist stub with real Anthropic API calls; reconcile cost against real token usage. No LiteLLM gateway, no OpenRouter free-tier routing yet.

### Done
- `src/orbicrew_api/llm_client.py` (new): `complete(specialist, model, input_text)` — calls `client.messages.create` (Anthropic SDK, `anthropic.AsyncAnthropic`, lazily constructed via `lru_cache` so a missing API key doesn't break app startup) with a per-specialist system prompt (coding/writing/research/general), returns `CompletionResult(text, cost_usd, input_tokens, output_tokens)`. Cost is computed from real `response.usage` against a static `MODEL_PRICING_USD_PER_MTOK` table (Anthropic first-party per-1M-token pricing).
- `office_manager.py`: specialist nodes are now `async` and call `llm_client.complete()` instead of returning `f"[{name}] noted: ..."`. `OfficeManagerState` gained `cost_usd`/`input_tokens`/`output_tokens` (the *actual* spend/usage, separate from `estimated_cost_usd`, which stays the Model Router's pre-call flat estimate used only for the Budget Guard gate). `SPECIALISTS` simplified from a display-name dict to a tuple of keys (display names were unused once the canned-string output went away).
- `model_router.py`: `TIER_MODELS` now maps each tier to a real Anthropic model ID — `trivial`/`cheap` → `claude-haiku-4-5`, `mid` → `claude-sonnet-5`, `frontier` → `claude-opus-5`. `trivial` reuses Haiku rather than the placeholder `openrouter/free-tier` string, since OpenRouter/LiteLLM routing isn't wired yet.
- `tasks.py`: `usage_records` now gets real `input_tokens`/`output_tokens` (previously hardcoded `0, 0`) and `spend_so_far_usd`/`cost_usd` come from the specialist's actual call, not the router's estimate.
- `settings.py` / `.env.example`: added `ANTHROPIC_API_KEY` (optional at the settings layer; required at first real specialist call).
- Tests: `test_office_manager.py` and `test_tasks.py` now patch `orbicrew_api.office_manager.complete` with a fake `CompletionResult` instead of hitting the network; assertions updated to check actual (mocked) cost/tokens instead of the old flat per-tier numbers. `uv run pytest` — 23 passed.
- Manual verification against the live API with a real `ANTHROPIC_API_KEY` (user-provided): coding-routed and research-routed phrasing both returned real, non-canned model output; `psql` on `usage_records` confirmed non-zero `input_tokens`/`output_tokens` and a `cost_usd` matching the real-usage calculation (not the old flat `0.01`/`0.25` placeholders).

### Decisions / assumptions
- **Direct Anthropic SDK, not a self-hosted LiteLLM gateway** — explicit user choice (asked via AskUserQuestion). The system design doc's cost-first architecture calls for a self-hosted LiteLLM proxy as the long-term model-calling surface; that's deferred to a later cost-hardening slice (fits naturally alongside Phase 1.2 budget hardening) since it's a materially bigger unit of work (new service, Compose entry, provider keys registered in LiteLLM config) than "make 3 specialists real." The specialist interface (`llm_client.complete`) is the natural swap point when that happens.
- All four routing outcomes (coding/writing/research/general) now call a real model — not just the 2–3 the phase-plan task line names — since it was near-zero incremental code to include `general` and leaving it canned would've been an inconsistent seam.
- Kept the Model Router's flat per-tier `estimated_cost_usd` as pre-call-only, used solely for the Budget Guard gate (which must decide before any spend happens); actual `cost_usd` from real token usage is what's persisted everywhere else (`usage_records`, `tasks.spend_so_far_usd`, the API response). This was flagged as the expected next step in the 0.4 tracker entry.
- `anthropic.AsyncAnthropic` client is constructed lazily (`@lru_cache` on a getter, not at module import time) so the app still boots and existing endpoints work even before `ANTHROPIC_API_KEY` is set; it only raises when a specialist call is actually attempted.

### Manual tests run
- `uv run pytest` — 23 passed
- Live API: `POST /v1/tasks` for a writing-shaped prompt (routed to `coding` due to a pre-existing `_KEYWORDS` false positive — `"description"` contains the substring `"script"` — not a Phase 0.5 issue, left alone) and a research-shaped prompt — both returned real Claude Haiku output and real cost
- `psql -c 'select model_used, input_tokens, output_tokens, cost_usd from usage_records'` — new rows show real token counts and cost; older rows from before this change still show the old flat placeholders (`claude-opus`/`claude-haiku`, `0`/`0` tokens), confirming the shift is live-forward only, as expected for a schema with no backfill

### Next recommended work
1. Phase 0.6: thinnest chat UI in `orbicrew-web` that submits a task and shows status/tier/model/output.
2. Known pre-existing bug (not fixed here, out of this slice's scope): the Phase 0.3 keyword classifier in `office_manager.classify()` has substring false positives (e.g. "description" matches the "script" keyword) — worth a real cheap-classifier-model pass whenever that's prioritized, per the existing code comment.
3. Self-hosted LiteLLM gateway swap-in, when cost-hardening work starts (Phase 1.2 territory) — `llm_client.complete()` is the intended seam.

### Files / repos touched
- `repos/orbicrew-api`: `src/orbicrew_api/llm_client.py` (new), `src/orbicrew_api/office_manager.py`, `src/orbicrew_api/model_router.py`, `src/orbicrew_api/tasks.py`, `src/orbicrew_api/settings.py`, `.env.example`, `.env`, `pyproject.toml`, `tests/test_office_manager.py`, `tests/test_tasks.py`, `README.md`
- `docs/development/manual-test-guide.md`, `docs/development/phase_by_phase_development_plan.md`, `docs/development/development-tracker.md` (this entry)

---

## 2026-08-07 — Phase 0.4: Model Router + Budget Guard

**Agent / operator:** Claude Code
**Phase:** Phase 0 (Personal tool) — task 0.4
**Scope:** `orbicrew-api` only — tier classification, per-task budget enforcement, real cost persistence. No real model calls yet (specialists are still the Phase 0.3 canned stub).

### Done
- `src/orbicrew_api/model_router.py`: `classify_tier(specialist, input_text)` heuristic (hard-keyword match → `frontier`; short general text → `trivial`; long text → `mid`; else `cheap`) + a static `TIER_MODELS` table mapping each tier to a placeholder model name and flat per-task cost estimate. `route()` returns a `RouteDecision(tier, model, estimated_cost_usd)`.
- `src/orbicrew_api/budget_guard.py`: `check_budget(estimated_cost_usd, spend_so_far_usd, budget_cap_usd) -> BudgetDecision` — pure function, hard per-task ceiling only (no retry-then-escalate or per-tenant aggregate caps yet — those are Phase 1 budget hardening).
- `office_manager.py` graph gains two nodes between `classify` and the specialist nodes: `route` (calls the Model Router, stamps `tier`/`model`/`estimated_cost_usd` into state) and a conditional edge that sends the task to `budget_paused` instead of the specialist when `check_budget` denies it. `budget_paused` is a terminal node with a `[Budget Guard] Task paused: ...` output and a `budget_pause` step.
- `tasks.py`: `SubmitTaskRequest` gained an optional `budget_cap_usd` override (defaults to the `tasks.budget_cap_usd` column default, `$1.00`); `task_steps.model_used`/`cost_usd` are now populated from the graph's step details instead of always being null/0; a `usage_records` row is inserted after a successful (non-paused) run; `tasks.status` can now be `paused`, and `spend_so_far_usd` reflects the real estimate (0 when paused). `TaskResponse` gained `tier`/`model` fields.
- Tests: `tests/test_model_router.py`, `tests/test_budget_guard.py` (new); `tests/test_office_manager.py` and `tests/test_tasks.py` updated for the new `route` step and a budget-paused path. 23 passed (was 13).
- Manual verification against live Postgres/Redis: cheap-tier task → `done`, `$0.01` in `usage_records`; frontier-tier task (hard-keyword phrasing) with default $1 cap → `done`, `$0.25` in `usage_records`; same phrasing with `budget_cap_usd: 0.01` → `status: "paused"`, `spend_so_far_usd: 0`, **no** `usage_records` row written. Confirmed via `psql` on `tasks` and `usage_records`.

### Decisions / assumptions
- Tier→model→cost is a static lookup table, not a real cheap-classifier-model call or per-token LiteLLM costing — that's explicitly deferred until specialists make real model calls (Phase 0.5+); this slice's job was the classify→tier→budget-gate→record pipeline, not real routing intelligence.
- `spend_so_far_usd` always starts at `0` for a submitted task — this endpoint still runs the whole graph synchronously in one request (Phase 0.3 decision, unchanged). Accumulating spend across queued/retried steps is Phase 1.1 (task queue) + 1.2 (budget hardening) territory.
- A paused task writes no `usage_records` row and leaves `spend_so_far_usd` at 0 — budget denial happens before "execution," so nothing was actually spent.
- Added `paused` as a new `tasks.status` value (alongside the existing `running`/`done`/`failed`) — no schema change needed since `status` was already free-text with no CHECK constraint.

### Manual tests run
- `uv run pytest` — 23 passed
- Live API: 3 `POST /v1/tasks` cases (cheap/default-budget, frontier/default-budget, frontier/tiny-budget) — see Done section
- `psql -c 'select task_id, model_used, cost_usd from usage_records'` — exactly the 2 non-paused runs present
- `psql -c 'select id, status, spend_so_far_usd, budget_cap_usd from tasks'` — paused row shows `spend_so_far_usd = 0`, others match their tier's cost

### Next recommended work
1. Phase 0.5: 2–3 real specialists (pick from actual need — writing/research/coding are the current stub categories) replacing the canned stub output; this is also where `estimated_cost_usd` should start being reconciled against real token usage instead of the flat per-tier estimate.
2. Phase 0.6: thinnest chat UI that submits a task and shows status/tier/model.

### Files / repos touched
- `repos/orbicrew-api`: `src/orbicrew_api/model_router.py`, `src/orbicrew_api/budget_guard.py`, `src/orbicrew_api/office_manager.py`, `src/orbicrew_api/tasks.py`, `tests/test_model_router.py`, `tests/test_budget_guard.py`, `tests/test_office_manager.py`, `tests/test_tasks.py`, `README.md`
- `docs/development/manual-test-guide.md`, `docs/development/phase_by_phase_development_plan.md`, `docs/development/development-tracker.md` (this entry)

---

## 2026-08-07 — Phase 0.1: local deps + api/web health skeletons

**Agent / operator:** Cursor agent  
**Phase:** Phase 0 (Personal tool) — task **0.1**  
**Scope:** Start shared Postgres/Redis; scaffold runnable `orbicrew-api` + `orbicrew-web` with health/status probes

### Done

- Confirmed `resources/orbicrew_dev_infra` Compose: Postgres (pgvector/pg16) + Redis healthy on `localhost:5432` / `6379`.
- **`orbicrew-api` (dev):** uv + FastAPI skeleton — `GET /health` (liveness), `GET /ready` (Postgres + Redis probes), CORS for web, `.env.example`, pytest for health stubs, README runbook.
- **`orbicrew-web` (dev):** Next.js App Router + Tailwind shell with Deep Violet tokens + Bricolage/Hanken; home page probes API `/health` + `/ready` via `ORBICREW_API_URL`.
- Manual verification: API ready HTTP 200 with both deps ok; web HTTP 200 showing **healthy** / postgres+redis ok.
- Updated phase plan (0.1 Done), manual test guide §0.A.

### Decisions / assumptions

- Python **3.12** via uv (host default is 3.14; pin for ecosystem stability).
- Health routes at API root (`/health`, `/ready`) for early scaffolding; versioned `/v1/*` product routes come with later slices.
- Web uses server-side `ORBICREW_API_URL` (default `http://localhost:8000`) — no browser CORS required for the status page fetch.

### Manual tests run

- 0.A.1–0.A.4 — pass (see `manual-test-guide.md`)
- `uv run pytest` in api — 2 passed

### Blockers

- None

### Next recommended work

1. Phase 0.2: core Postgres schema (`tenant_id` from day one; billing surface deferred).
2. Then 0.3–0.4 vertical slice: Office Manager + model router + budget guard.
3. Commit/push `dev` on api + web (+ master docs) when asked.

### Files / repos touched

- `repos/orbicrew-api` — package layout under `src/orbicrew_api/`, tests, lockfile, README
- `repos/orbicrew-web` — Next.js app, status page, brand CSS tokens, README
- Master: `docs/development/phase_by_phase_development_plan.md`, `manual-test-guide.md`, this tracker

---

## 2026-08-07 — Dev-branch workflow + admin tech docs

**Agent / operator:** Cursor agent  
**Phase:** Phase -1 (workspace & agentic setup)  
**Scope:** Persist `dev`-only daily git workflow (master + org repos); reinforce dedicated `orbicrew-admin` in product/tech docs; verify admin remote — **no product features**

### Done

- Confirmed `repos/orbicrew-admin` origin is `https://github.com/leangine/orbicrew-admin.git` with `main`/`stage`/`dev` on origin.
- Master: created/pushed `dev` + `stage` (same three-branch model as org repos); day-to-day work on **`dev`**.
- Documented critical branch rule in `AGENTS.md`, `CLAUDE.md`, `AGENT_BOOTSTRAP.md`, `.cursor/rules/*`, root/`repos` READMEs, phase plan, manual tests, nested READMEs: commit/push only on `dev` unless user explicitly asks for `stage`/`main` or promote.
- Tech docs: architecture diagram + components (`03`), privileged `/v1/ops/*` (`05`), devops services (`08`), requirements admin/ops note (`01`), index quick link (`00`), kill-switch surfaces (`12`).
- Nested README branch language + api/admin privileged-API notes; infra mentions admin in first deliverable.

### Decisions / assumptions

- Master `orbicrew_master` uses `main` / `stage` / `dev` like org repos; **daily default is `dev`**.
- Platform admin remains a dedicated app; auth via privileged APIs on `orbicrew-api`, not `/admin` in web.

### Manual tests run

- Admin `git remote -v` → leangine/orbicrew-admin (pass)
- Nested remotes/branches already present from prior scaffold (pass)

### Blockers

- None

### Next recommended work

1. Phase 0.1: deps Compose + native api/web health endpoints (on `dev`).
2. Early admin auth shell when convenient.

### Files / repos touched

- Master: docs, agent rules, README, `repos/README.md` (commit on `dev`)
- Nested README updates on `dev`: web, admin, api, channels, infra

---

## 2026-08-07 — Dedicated admin repo + local deps Compose

**Agent / operator:** Cursor agent  
**Phase:** Phase -1 (workspace & agentic setup)  
**Scope:** Revert admin-inside-web decision; scaffold `orbicrew-admin`; add `resources/orbicrew_dev_infra` — **no product features**

### Done

- **Admin decision (updated):** dedicated `orbicrew-admin` from day one. Tenant settings stay in `orbicrew-web`; platform operator console is a separate app/repo (scaffold/auth shell early; full ops features Phase 2.6).
- Scaffolded `repos/orbicrew-admin` (Next.js-friendly `.gitignore` + standalone README), branches `main`/`stage`/`dev`, initial commit `ce8c7ac`. Remote: `https://github.com/leangine/orbicrew-admin.git` (private) — all three branches pushed.
- Added master `resources/orbicrew_dev_infra/` — lean Compose (Postgres `pgvector/pgvector:pg16` + Redis `redis:7-alpine`), `.env.example`, README. Convention: deps in Docker; apps native on Mac.
- Reverted/updated all master docs/rules that said platform admin is `/admin` inside web.
- Nested sibling READMEs: remove admin-from-web claims; link `orbicrew-admin` by GitHub URL where useful; tell clone-only users to run Postgres/Redis locally or via their own Compose (no master path).

### Decisions / assumptions

- Operator console is a fifth org repo (web, admin, api, channels, infra).
- Day-to-day shared deps live in master `resources/orbicrew_dev_infra/`; `orbicrew-infra` remains deploy/ops oriented.
- Apps (api/web/admin/channels) run natively during development — not in Docker.

### Manual tests run

- Nested admin README has no master/`docs/`/`resources/` references
- Master accidental local `dev`/`stage` branches (created during failed sandbox git init) deleted — never pushed
- `resources/orbicrew_dev_infra` files present and tracked in master

### Blockers

- None for scaffold push: empty private `leangine/orbicrew-admin` already existed; pushed `main`/`stage`/`dev` at `ce8c7ac` (default branch set to `main`).

### Next recommended work

1. Phase 0.1: bring up deps Compose + native api/web health endpoints.
2. Drop logo files into `resources/logos/` when available.
3. Early admin auth shell when convenient.

### Files / repos touched

- Master: docs, agent rules, README, `resources/orbicrew_dev_infra/**`, `repos/README.md`
- Nested:
  - `orbicrew-admin` `ce8c7ac` (new; pushed main/stage/dev)
  - `orbicrew-web` `efdc601`
  - `orbicrew-api` `8f39885`
  - `orbicrew-infra` `c3056b1`

---

## 2026-08-07 — Admin decision, nested-repo isolation, resources

**Agent / operator:** Cursor agent  
**Phase:** Phase -1 (workspace & agentic setup)  
**Scope:** Docs/rules + nested README scrub + track `resources/` — **no product features**

### Done

- **Admin decision:** no `orbicrew-admin` repo. Tenant settings + platform operator console live in `orbicrew-web` (protected `/admin` for operators). Scoped into Phase 2 (tasks 2.5–2.6). **Superseded** by later 2026-08-07 entry (dedicated admin repo).
- Scrubbed all four nested READMEs to be standalone (GitHub sibling URLs only; no master/`docs/`/`AGENT_BOOTSTRAP` references).
- Persisted agent rules: nested-repo isolation + best-practices/DRY in `AGENTS.md`, `CLAUDE.md`, `AGENT_BOOTSTRAP.md`, `.cursor/rules/orbicrew-workspace.mdc`, `.cursor/rules/nested-repos.mdc`.
- Documented master-only `resources/logos/` (empty, `.gitkeep`) and `resources/stitch_orbicrew_ui_ux_guide/` (Stitch screens listed). Confirmed `.gitignore` does **not** ignore `resources/`.
- Updated README, `repos/README.md`, phase plan, this tracker.

### Decisions / assumptions

- Prefer admin-inside-web over a 5th org repo until operator deploy/auth clearly diverges. **Superseded** — dedicated admin required.
- Nested repos must remain independently consumable; master context stays in master docs only.
- Logos folder currently empty — reserved and tracked for upcoming brand assets.

### Manual tests run

- Nested README greps for master-workspace terms — clean
- `git check-ignore` on `resources/` — not ignored by master `.gitignore`
- Ignore policy unchanged: `local/` + `repos/*` contents ignored; cursor/claude ignore `.env*`

### Next recommended work

1. Begin Phase 0.1: Docker Compose skeleton + health endpoints in `orbicrew-infra` / api / web.
2. Drop logo files into `resources/logos/` when available.

### Files / repos touched

- Master: docs, agent rules, README, `resources/**`
- Nested README commits (pushed `main`/`stage`/`dev`):
  - `orbicrew-web` `a6c147e`
  - `orbicrew-api` `aac72a7`
  - `orbicrew-channels` `7efdbc4`
  - `orbicrew-infra` `27ae773`

---

## 2026-08-07 — Connect GitHub remotes and push scaffolds

**Agent / operator:** Cursor agent  
**Phase:** Phase -1 (workspace & agentic setup)  
**Scope:** Remotes, push branches, document URLs — **no product features**

### Done

- Configured `origin` remotes:
  - Master: `https://github.com/meetrakib/orbicrew_master.git`
  - `orbicrew-api`: `https://github.com/leangine/orbicrew-api.git`
  - `orbicrew-channels`: `https://github.com/leangine/orbicrew-channels.git`
  - `orbicrew-infra`: `https://github.com/leangine/orbicrew-infra.git`
  - `orbicrew-web`: `https://github.com/leangine/orbicrew-web.git`
- Pushed `main`, `stage`, and `dev` for all four org repos (tracking set).
- Updated docs with repo → GitHub URL → local path tables (`README.md`, `repos/README.md`, `AGENTS.md`, `CLAUDE.md`, `AGENT_BOOTSTRAP.md`, phase plan, cursor workspace rule).

### Decisions / assumptions

- HTTPS remotes (not SSH) as provided by the user.
- Nested scaffolds pushed as-is; no nested README commit required for this slice.

### Manual tests run

- `git remote -v` on master + each nested repo — URLs match table above
- Org branch pushes succeeded (`main`/`stage`/`dev`)

### Next recommended work

1. Begin Phase 0.1: Docker Compose skeleton + health endpoints in `orbicrew-infra` / api / web.
2. Confirm GitHub default branch is `main` on each remote if not already.

### Files / repos touched

- Master docs/agent config (this commit)
- Nested repos: remote + push only (no new commits)

---

## 2026-08-07 — Agentic workspace setup (setup session)

**Agent / operator:** Cursor agent (setup pass)  
**Phase:** Phase -1 (workspace & agentic setup)  
**Scope:** Docs, git, ignore files, agent instructions, scaffold org repos — **no product features**

### Done

- Researched Claude Code (`CLAUDE.md`, `.claudeignore`, `.claude/settings`) and Cursor (`AGENTS.md`, `.cursor/rules` `.mdc`, `.cursorignore`) best practices; applied lean root instructions + scoped rules.
- Read product docs under `docs/leangine-office-docs/` (index, requirements, system design, phased plan, devops) to infer services and phasing.
- Wrote/filled:
  - `docs/development/AGENT_BOOTSTRAP.md`
  - `docs/development/phase_by_phase_development_plan.md`
  - `docs/development/manual-test-guide.md`
  - `docs/development/development-tracker.md` (this file)
- Master workspace:
  - Root `.gitignore` (ignores `local/`, `repos/*` contents keeping README/.gitkeep, secrets, deps)
  - `.cursorignore` / `.claudeignore` (ignore `.env*`; do **not** ignore `local/`, `docs/`, `repos/`)
  - `README.md`, `AGENTS.md`, `CLAUDE.md`
  - `.cursor/rules/*.mdc`, `.claude/settings.json` (denies agent RW under `local/`)
  - Git init on `main` + initial commit `d20b7a2` — **not pushed** (no remote)
- Scaffolded nested repos under `repos/` with README + language-appropriate `.gitignore`, branches `main` / `stage` / `dev`, initial commits, **no remotes**:
  - `orbicrew-web` — Next.js/React frontend
  - `orbicrew-api` — FastAPI/LangGraph backend + workers
  - `orbicrew-channels` — messaging adapters
  - `orbicrew-infra` — Compose / deploy / ops

### Decisions / assumptions

- Four org repos (web, api, channels, infra) rather than a single monorepo app — matches “one backend, many faces” and independent deploy units; workers stay in `orbicrew-api` initially.
- Orbit View is original IP (not Agent Town fork), deferred to Phase 3 per product sequencing.
- Master workspace = personal GitHub; nested repos = org GitHub (remotes connected in follow-up entry).

### Manual tests run

- W1: `local/` ignored by master git — pass
- W2: nested `repos/orbicrew-*` contents ignored; `repos/README.md` + `repos/.gitkeep` tracked — pass
- W3: agent docs present — pass
- W4: each nested repo has `main` / `stage` / `dev` — pass

### Next recommended work

1. ~~User creates GitHub org repos and provides remote URLs.~~ Done (follow-up entry).
2. ~~Connect remotes + push `main`/`stage`/`dev` for each nested repo.~~ Done (follow-up entry).
3. Begin Phase 0.1: Docker Compose skeleton + health endpoints.

### Files touched (high level)

- Root: README, AGENTS, CLAUDE, gitignores, cursor/claude config
- `docs/development/*`
- `repos/orbicrew-{web,api,channels,infra}/*` (scaffold only)

---

---

## 2026-08-07 — Brand logos added to master resources

**Agent / operator:** Cursor agent  
**Phase:** Workspace assets (pre–Phase 0)  
**Scope:** Commit Orbicrew light/dark logo and icon assets under `resources/logos/`.

### Done
- Added light/dark wordmark SVGs and icon SVG/PNG pairs to `resources/logos/`.
- Updated `resources/README.md` so the logos inventory is accurate (folder no longer empty).

### Decisions / assumptions
- Logos live in master-only `resources/`; apps copy assets in — no nested-repo path coupling.

### Manual tests run
- None (static assets).

### Next recommended work
1. Begin Phase 0.1 when ready: Docker Compose skeleton + health endpoints (unchanged).
2. Copy logos into `orbicrew-web` / `orbicrew-admin` when scaffolding branded UI.

### Files / repos touched
- `resources/logos/*` (SVGs/PNGs; not `.DS_Store`)
- `resources/README.md`
- `docs/development/development-tracker.md`

---

## 2026-08-07 — Phase 0.2: core schema migration

**Agent / operator:** Claude Code
**Phase:** Phase 0 (Personal tool) — task 0.2
**Scope:** `orbicrew-api` only — core Postgres schema, no billing/BYO-key or Orbit View tables, no application logic yet.

### Done
- Added a plain-SQL migration runner (`src/orbicrew_api/migrate.py`, no ORM/framework — matches the existing raw-asyncpg style): tracks applied versions in a `schema_migrations` table, applies pending `migrations/*.sql` files in order inside a transaction, idempotent re-run.
- Wrote `migrations/0001_core_schema.sql` covering `tech/04_database_design.md` §1–3: `tenants`, `users`, `sessions`, `tools`, `agents`, `tasks`, `task_steps`, `task_artifacts`, `approvals`, `agent_memory`, `memory_embeddings`, `usage_records`, plus the indexes from §3.
- Enabled `pgcrypto` (for `gen_random_uuid()`) and `vector` (pgvector) extensions in the migration.
- Added CLI entry point `orbicrew-api-migrate` in `pyproject.toml`.
- Applied migration to local Postgres (`resources/orbicrew_dev_infra`); verified all 13 tables + FKs/indexes via `psql \dt` / `\d agents`.
- Added `tests/test_migrate.py` (pure-function unit tests for pending-migration selection; no live DB in automated tests, consistent with the existing mocked `test_health.py` pattern).
- Updated `orbicrew-api/README.md` (migrations section + status line) and master `manual-test-guide.md` (0.A.5) / `phase_by_phase_development_plan.md` (0.2 → Done).

### Decisions / assumptions
- Excluded from this migration per "minus full billing surface": `subscriptions` and `api_keys` (BYO key management) — both are Phase 2 concerns. `usage_records` **is** included since it's the cost-tracking table the Phase 0.4 budget guard needs, not a billing/subscription table.
- Excluded Orbit View tables (`orbit_*`) — Phase 3 per repo-ownership table.
- **RLS intentionally not enabled** — phase plan constraint 3 says `tenant_id` present from day one, RLS turned on in Phase 2. All tenant-scoped tables have the column and are ready for `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` later without a schema change.
- No HNSW/ivfflat index on `memory_embeddings.embedding` yet — doc explicitly says add once real memory volume exists; noted as a comment in the migration file.
- Chose hand-rolled SQL migrations over Alembic/SQLAlchemy — the service already uses raw `asyncpg`, not an ORM, so a SQLAlchemy-based migration tool would be an unnecessary added dependency/abstraction.
- No seed data (no default tenant/user) — that belongs with the Office Manager vertical slice (0.3+), not the schema task.

### Manual tests run
- `uv run pytest` — 8 passed (5 existing + 3 new migration tests)
- `uv run orbicrew-api-migrate` — applied `0001_core_schema.sql`; second run → "No pending migrations."
- `docker exec orbicrew-postgres psql -U orbicrew -d orbicrew -c '\dt'` — 13 tables + `schema_migrations`
- `docker exec orbicrew-postgres psql -U orbicrew -d orbicrew -c '\d agents'` — columns, PK, FK to `tenants`, index `idx_agents_tenant_status` all present

### Next recommended work
1. Phase 0.3: LangGraph Office Manager supervisor routing to hardcoded specialists (needs a seed tenant/user — first place to add one).
2. When adding schema changes, use the next numbered file (`migrations/0002_*.sql`); do not edit `0001` after it's applied anywhere.

### Files / repos touched
- `repos/orbicrew-api`: `migrations/0001_core_schema.sql`, `src/orbicrew_api/migrate.py`, `tests/test_migrate.py`, `pyproject.toml`, `README.md`
- `docs/development/manual-test-guide.md`, `docs/development/phase_by_phase_development_plan.md`, `docs/development/development-tracker.md` (this entry)

---

## 2026-08-07 — Phase 0.3: LangGraph Office Manager supervisor

**Agent / operator:** Claude Code
**Phase:** Phase 0 (Personal tool) — task 0.3
**Scope:** `orbicrew-api` only — supervisor routing + task persistence, no real model calls yet.

### Done
- Added `langgraph` (1.2.10) as a dependency.
- `src/orbicrew_api/bootstrap.py`: deterministic (uuid5-derived) dev tenant + user, upserted idempotently on app startup (`lifespan` in `main.py`). Phase 0 is single-user, so this stands in for real signup until Phase 2.
- `src/orbicrew_api/office_manager.py`: compiled `langgraph.graph.StateGraph` — a `classify` node picks one of `coding` / `writing` / `research` / `general` by keyword match, conditional-edges into the matching specialist node, which returns a canned `"[<Specialist Name>] noted: <input>"` string. Both node types append a structured step record (`step_type`, `detail`) to state for the audit trail.
- `src/orbicrew_api/tasks.py`: `POST /v1/tasks` (insert task as `running` → run the graph → insert one `task_steps` row per recorded step → update task to `done`/`failed` with `metadata = {specialist, output}`) and `GET /v1/tasks/:id` (task + ordered step trace, 404 if missing/wrong tenant). Runs synchronously in-request — no queue yet (that's Phase 1.1).
- Wired `tasks_router` and the bootstrap call into `main.py`.
- Tests: `tests/test_office_manager.py` (classification table + one full `ainvoke` through the compiled graph, no DB) and `tests/test_tasks.py` (API tests with a mocked `pg_pool`, mirroring the existing `test_health.py` style — submit/coding-routing, get/found, get/404).
- Manually verified against the live local Postgres: started the API, `POST /v1/tasks` for coding/writing/general phrasing all routed correctly, `GET /v1/tasks/:id` returned the persisted result, confirmed via `psql` that `tenants`/`users`/`tasks`/`task_steps` rows landed correctly.

### Decisions / assumptions
- Routing is deliberately keyword-based, not model-based — this task is explicitly "hardcoded specialists"; the real classifier is the Model Router (0.4). Left an explicit code comment marking it as a placeholder so it isn't mistaken for the real router later.
- Specialist "execution" is a canned string, not a real model call — real specialist behavior is task 0.5. `task_steps.model_used` is left null and `cost_usd` is `0` throughout; `tool_call` (the only structured jsonb column on that table) is reused to hold the routing/step detail since no better-fitting column exists yet.
- No task queue — task 0.3 runs the whole graph synchronously inside the HTTP handler. Queued/checkpointed execution is explicitly Phase 1.1; introducing it now would be scope creep against the phase plan.
- `agent_id` on `tasks` is left `null` — hardcoded specialists are Python-level constants, not `agents` table rows yet. Real agent CRUD (and therefore real `agent_id` values) is Phase 0.5 / 2.4.
- Bootstrap uses fixed uuid5-derived IDs (not a runtime-generated random tenant) so the same dev tenant/user exist across restarts without needing a lookup query on every request.

### Manual tests run
- `uv run pytest` — 13 passed (5 health + 3 migration + 5 new office-manager/tasks)
- Live API against `resources/orbicrew_dev_infra` Postgres: `POST /v1/tasks` for coding/writing/general inputs → correct `specialist` + `output` each time; `GET /v1/tasks/:id` → matches; `GET /v1/tasks/<random-uuid>` → 404
- `psql -c '\dt'` style checks: `tenants`/`users` has exactly the one seeded "Founder Workspace" row; `task_steps` has 2 ordered rows (`classify`, `specialist_execute`) per submitted task

### Next recommended work
1. Phase 0.4: Model Router + Budget Guard — replace the keyword classifier with a cheap-model classification step, add per-task budget cap enforcement before/around model calls.
2. Phase 0.5: real specialists (2–3, picked from actual need) replacing the canned stub output; likely also when `agents` table rows start getting created/seeded for real.

### Files / repos touched
- `repos/orbicrew-api`: `src/orbicrew_api/bootstrap.py`, `src/orbicrew_api/office_manager.py`, `src/orbicrew_api/tasks.py`, `src/orbicrew_api/main.py`, `tests/test_office_manager.py`, `tests/test_tasks.py`, `pyproject.toml`, `README.md`
- `docs/development/manual-test-guide.md`, `docs/development/phase_by_phase_development_plan.md`, `docs/development/development-tracker.md` (this entry)

---

## Template for future entries

```markdown
## YYYY-MM-DD — <short title>

**Agent / operator:**  
**Phase:**  
**Scope:**  

### Done
- 

### Decisions / assumptions
- 

### Manual tests run
- 

### Next recommended work
- 

### Files / repos touched
- 
```
