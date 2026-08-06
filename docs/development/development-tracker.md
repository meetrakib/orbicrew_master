# Development Tracker — Orbicrew

Chronological log of workspace and product development sessions. **Agents: append an entry after completing work.** Do not rewrite history; add corrections as new notes if needed.

Newest entries at the **top**.

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
