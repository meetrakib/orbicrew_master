# `resources/` — Shared master-workspace assets

Tracked in the **master** workspace only. Nested org repos must not assume this path exists.

## Contents

| Path | Purpose |
|---|---|
| `resources/logos/` | Brand logos: light/dark wordmarks (`light-logo.svg`, `dark-logo.svg`) and icons (`light-icon.svg`/`png`, `dark-icon.svg`/`png`) |
| `resources/stitch_orbicrew_ui_ux_guide/` | Stitch UI/UX exports for Standard Dashboard screens |
| `resources/orbicrew_dev_infra/` | Lean Docker Compose for shared local dependencies (Postgres+pgvector, Redis) |

### Logos

Tracked under `resources/logos/`:

- `light-logo.svg`, `dark-logo.svg` — wordmarks
- `light-icon.svg`, `light-icon.png`, `dark-icon.svg`, `dark-icon.png` — app/mark icons

Copy into `orbicrew-web` / `orbicrew-admin` as needed; do not depend on this path from nested repos.

### Stitch guide screens

- `orbicrew/` — design tokens / `DESIGN.md`
- `orbicrew_agent_configuration/`
- `orbicrew_agent_roster/`
- `orbicrew_approvals_inbox/`
- `orbicrew_dashboard_overview/`
- `orbicrew_hire_ai_employees/`
- `orbicrew_message_agents/`
- `orbicrew_settings/`
- `orbicrew_task_detail/`
- `orbicrew_usage_billing/`

Each screen folder typically includes `code.html` and `screen.png` (except the design folder).

### Dev infra Compose

See [`orbicrew_dev_infra/README.md`](orbicrew_dev_infra/README.md). Start shared deps with `docker compose up -d`. Run application services from `repos/*` natively — not in this Compose file.

## Agent usage

When building tenant UI in `orbicrew-web` or platform-operator UI in `orbicrew-admin`, use logos/Stitch for branding and layout reference. Copy needed assets into the app repo as appropriate; do not hard-code absolute machine paths or references from nested repos back to this folder.
