# `resources/` — Shared master-workspace assets

Tracked in the **master** workspace only. Nested org repos must not assume this path exists.

## Contents

| Path | Purpose |
|---|---|
| `resources/logos/` | Brand logos for Orbicrew / Leangine (add files here as they become available) |
| `resources/stitch_orbicrew_ui_ux_guide/` | Stitch UI/UX exports for Standard Dashboard screens |

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

## Agent usage

When building web or platform-admin UI in `orbicrew-web`, use these assets for branding and layout reference. Copy needed logos/assets into the web repo as appropriate; do not hard-code absolute machine paths or references from nested repos back to this folder.
