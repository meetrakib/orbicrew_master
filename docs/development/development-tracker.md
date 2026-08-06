# Development Tracker — Orbicrew

Chronological log of workspace and product development sessions. **Agents: append an entry after completing work.** Do not rewrite history; add corrections as new notes if needed.

Newest entries at the **top**.

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
