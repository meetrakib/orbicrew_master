# Brief: Agent Configuration — "Setting" tab

## Where this lives
Same page as `orbicrew_agent_configuration` (Data Analyst Prime example). That mockup only designed the **Soul** tab. This brief is for the **Setting** tab, selected from the same left-hand tab list (Soul / Skills / Memory / Setting) — reuse the exact page shell, header, and tab strip from that mockup unchanged.

## Why it's needed
Per the product spec, every agent's most safety-critical controls live here — nothing external or irreversible should be a surprise. This tab has never been visually designed; only Soul has.

## Sections to design (top to bottom, single scrolling column matching Soul tab's card style)

1. **Model selection mode**
   - Two-state toggle or segmented control: **Auto** (default) vs **Manual**.
   - Helper text under Auto: "Router picks the cheapest model that can handle each task, decided live, per task."
   - When **Manual** is selected, reveal a model picker dropdown (e.g., Claude Opus / Sonnet / Haiku, GPT-4o, Gemini, etc.) — show this as a conditional reveal, not always visible.

2. **Master Agent control**
   - If this agent IS currently the Master: show a static badge/row — "This is your Master Agent" with a small info icon explaining what that means (the one agent the user talks to directly; delegates to others).
   - If this agent is NOT the Master: show a "Promote to Master" button/row instead. Clicking it should be understood to open the confirmation modal specified in `02_agent_roster_master_promotion.md` (same modal, reused here).

3. **Budget cap**
   - A numeric input (currency-prefixed, e.g. "$") labeled "Per-task budget cap" with helper text: "Task pauses and flags for review if it exceeds this."

4. **Action whitelist**
   - A list of irreversible/external action types (e.g., "Send an email," "Publish a social post," "Deploy code," "Make a payment," "Post to a connected channel") — pull the real list from the tool catalog if available, otherwise use these as placeholders.
   - Each row is a label + a two-state toggle: **"Requires my approval"** (default, shown as the safe/selected state) vs **"Runs automatically."**
   - Visually flag this section as the most important on the page — e.g., a subtle warning-tinted border or header icon — since accidentally flipping these to automatic is the main risk this page exists to prevent.

## Style
Match `orbicrew_agent_configuration`'s existing Soul tab exactly: same card container, spacing, input styling (light violet-tinted fields), typography, and the persistent "Save Changes" / "Deactivate" header buttons.
