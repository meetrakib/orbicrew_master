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

### Additional designs (later Stitch drop)

`resources/stitch_orbicrew_ui_ux_guide/additional-designs/` — exports covering gaps left by the
original set. Each has a brief in the sibling `additional_designs_guide/` folder explaining what
it's for and where it attaches:

- `orbicrew_agent_configuration_setting_tab/` — the Setting tab for Agent Configuration (model
  mode, Master Agent control, budget cap, action whitelist); only the Soul tab was designed before.
- `orbicrew_agent_roster_master_promotion_modal/` — "Make Master Agent" confirmation modal, adds
  to `orbicrew_agent_roster`.
- `orbicrew_live_voice_mode/` — full-screen continuous/live voice overlay, extends
  `orbicrew_message_agents` (distinct from that mockup's existing one-shot dictation mic).
- `orbicrew_settings/` — **updated** version of `orbicrew_settings` above: adds a Slack row to the
  Connected Channels list (Slack as a chat-channel adapter, alongside Telegram/Discord/WhatsApp).
- `shader/` — a WebGL background shader (`code.html` only, no `screen.png`).
- `orbicrew/` — duplicate `DESIGN.md`, identical to the top-level `orbicrew/DESIGN.md` (same tokens).

### Dev infra Compose

See [`orbicrew_dev_infra/README.md`](orbicrew_dev_infra/README.md). Start shared deps with `docker compose up -d`. Run application services from `repos/*` natively — not in this Compose file.

## Agent usage

When building tenant UI in `orbicrew-web` or platform-operator UI in `orbicrew-admin`, use logos/Stitch for branding and layout reference. Copy needed assets into the app repo as appropriate; do not hard-code absolute machine paths or references from nested repos back to this folder.
