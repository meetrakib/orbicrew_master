# Brief: Agent Roster — promote/demote Master Agent

## Where this lives
Same page as `orbicrew_agent_roster` (Orion Core / DevBot_9 / DataDive / Lexi example). Reuse that exact grid, card style, and header unchanged. This brief adds one new interaction to it.

## Why it's needed
The current mockup shows exactly one card with a "MASTER" badge (Orion Core) but gives no way to change which agent holds that role. Product decision: any agent should be promotable to Master, and it should be reassignable, not fixed.

## What to add

1. **Per-card menu action** — on every non-Master agent card (DevBot_9, DataDive, Lexi in the reference mockup, via their existing "⋮" three-dot menu), add a new menu item: **"Make Master Agent."**

2. **Confirmation modal** (triggered by that menu item, or by "Promote to Master" from the Setting tab per `01_agent_configuration_setting_tab.md`):
   - Title: "Make [Agent Name] your Master Agent?"
   - Body copy: "Only one agent can be Master. [Current Master Name] will no longer be your default point of contact — you can still message it directly from its own card."
   - Two actions: a primary confirm button ("Make Master") and a secondary cancel/dismiss.

3. **Result state**: after confirming, the "MASTER" badge (accent-bordered card treatment, same violet border/label already used on Orion Core) moves to the newly promoted agent's card, and that card becomes pinned first in the grid. The previously-Master card returns to the normal card style (like DevBot_9/DataDive today).

## Style
No new visual language needed beyond the modal — reuse the existing violet accent border + "MASTER" pill badge exactly as shown on the Orion Core card today, just make clear in the mockup that it's a movable badge, not fixed to one card.
