# Development Tracker — Orbicrew

Chronological log of workspace and product development sessions. **Agents: append an entry after completing work.** Do not rewrite history; add corrections as new notes if needed.

Newest entries at the **top**.

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
