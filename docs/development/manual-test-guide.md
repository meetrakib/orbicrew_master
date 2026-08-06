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
| Web reachable | Native: `cd repos/orbicrew-web && npm install && npm run dev` → `http://localhost:3000` | Available (Phase 0.1) |
| Admin reachable | Native process in `repos/orbicrew-admin` (TBD URL) | Not yet |
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

### 0.B Task path (core)

| # | Check | Expected | Status |
|---|---|---|---|
| 0.B.1 | Submit text task via web | Task appears queued → running → done/failed | Pending (web GUI not built yet — API path passes, see 0.B.2) |
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

### 0.D Voice (when implemented)

| # | Check | Expected | Status |
|---|---|---|---|
| 0.D.1 | STT | Spoken input becomes correct-enough text in primary language | Pending |
| 0.D.2 | TTS | Reply plays back | Pending |

---

## Phase 1 — Overnight + channels

| # | Check | Expected | Status |
|---|---|---|---|
| 1.1 | Overnight handoff | Long task resumes after worker restart (checkpoint) | Pending |
| 1.2 | Approval gate | Restricted action waits for approval; does not execute | Pending |
| 1.3 | Morning summary | Summary lists done / blocked / needs decision with links | Pending |
| 1.4 | Telegram round-trip | Message → task → status/result on Telegram | Pending |
| 1.5 | Cross-channel continuity | Task started on web visible on Telegram (or vice versa) | Pending |
| 1.6 | WhatsApp round-trip | When Business API ready | Pending |

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
| 2026-08-07 | 0.B.3/0.B.4 marked Pass — Model Router + Budget Guard (Phase 0.4) verified live |
| 2026-08-07 | Initial structure created during workspace agentic setup |
