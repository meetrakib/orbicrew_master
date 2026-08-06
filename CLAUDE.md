# CLAUDE.md — Orbicrew workspace

Instructions for Claude Code. Cross-tool shared guidance also lives in `AGENTS.md` — keep them consistent; prefer editing both when changing policy.

**Session start:** read `@docs/development/AGENT_BOOTSTRAP.md` then the current phase in `@docs/development/phase_by_phase_development_plan.md`.

Imported shared instructions: `@AGENTS.md`

---

## What this workspace is

Master **personal** git repo ([meetrakib/orbicrew_master](https://github.com/meetrakib/orbicrew_master)) for Orbicrew agentic development. Product code is in nested Leangine org repos under `repos/` (see URL table in `AGENTS.md` / `repos/README.md`). Product requirements are in `docs/leangine-office-docs/`. Process docs are in `docs/development/`. Brand/UI assets and local deps Compose are in `resources/` (master-only).

Do not implement application features at the workspace root.

---

## Critical constraints

1. **Do not read `local/`** unless the user explicitly asks.
2. **Before coding features:** follow `docs/development/phase_by_phase_development_plan.md`.
3. **After a development slice:** update `docs/development/manual-test-guide.md` when test steps change.
4. **After finishing work:** update `docs/development/development-tracker.md`.
5. Nested `repos/*` = Leangine org GitHub; this root = personal GitHub (`https://github.com/meetrakib/orbicrew_master`). Commit inside the nested repo for code changes; commit here for docs/agent config/resources.
6. Branches in org repos: `main` (prod), `stage`, `dev` (default work).
7. No secrets in commits. No force-push to protected branches unless asked.
8. **Nested repo isolation:** when editing under `repos/<name>/`, never reference master-only paths (`docs/`, `resources/`, `local/`, workspace layout, `orbicrew_master`). Nested repos must stay independently consumable. Sibling GitHub URLs are OK.
9. **Coding principles:** follow best practices; prefer reusable shared abstractions (DRY / shared packages) over copy-paste.
10. **Local infra:** shared deps via `resources/orbicrew_dev_infra/`; run apps natively (not in Docker) during development.
11. **Platform admin:** dedicated `repos/orbicrew-admin` — not embedded `/admin` in web.

---

## Where to work

| Need | Go to |
|---|---|
| Dashboard / tenant settings / Orbit View | `repos/orbicrew-web` |
| Platform operator console | `repos/orbicrew-admin` |
| API, LangGraph, router, DB, workers | `repos/orbicrew-api` |
| Telegram / Discord / WhatsApp adapters | `repos/orbicrew-channels` |
| Deploy topology / shared ops | `repos/orbicrew-infra` |
| Local Postgres + Redis Compose | `resources/orbicrew_dev_infra` |
| Requirements / architecture | `docs/leangine-office-docs/` |
| Phase / tests / session log | `docs/development/` |
| Logos / Stitch UI references | `resources/logos`, `resources/stitch_orbicrew_ui_ux_guide` |

Platform operator UI is the dedicated `orbicrew-admin` repo (scaffold/auth shell early; full features with Phase 2). Tenant settings remain in `orbicrew-web`.

Launch Claude Code from the nested service directory when doing deep work on that service to keep context focused. Root `CLAUDE.md` / `AGENTS.md` still apply for master-side work; do not bake master paths into nested-repo files.

---

## Product pointers (do not duplicate full specs here)

- Index: `docs/leangine-office-docs/tech/00_INDEX.md`
- Requirements: `docs/leangine-office-docs/01_AI_Office_Platform_Requirements.md`
- System design / stack: `docs/leangine-office-docs/tech/03_system_design.md`
- Brand (locked): `docs/leangine-office-docs/tech/09_brand_identity.md` §6 — **Orbicrew**, Deep Violet, Bricolage Grotesque + Hanken Grotesk
- Orbit View: `docs/leangine-office-docs/tech/18_orbit_view_game_ui.md` (original IP; Agent Town is reference only)
- UI references: `resources/stitch_orbicrew_ui_ux_guide/` + `resources/logos/`

Guiding priorities: (1) cost control, (2) reliability over flash, (3) multi-tenant from day one, (4) global English-first, (5) user-extensible agents.

---

## Claude Code notes

- Keep this file short (<200 lines). Put long procedures in `docs/development/` and path-scoped rules under `.claude/rules/` if added later.
- Project settings: `.claude/settings.json`. Personal overrides: `.claude/settings.local.json` (gitignored).
- `.claudeignore` excludes secrets and build noise; it does **not** ignore `local/`, `docs/`, `repos/`, or `resources/`.
