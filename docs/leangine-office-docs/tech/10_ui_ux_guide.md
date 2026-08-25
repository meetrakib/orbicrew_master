# UI/UX Guide

Part of the AI Employee Office Platform documentation set. See `00_INDEX.md` for the full document list.

**How to use this document:** this guide is written to be handed, in full, to an AI UI-generation tool (e.g., Google Stitch or similar) as a single prompt/spec. Every page section below is written in enough descriptive detail — layout, content, states, tone — that a generator working from this text alone should be able to produce the actual interface, not just a rough sketch. Copy Section 0 (brand identity) and the rest of this document together as one paste; they're written to be consumed as a unit. If the brand direction changes, update Section 0 from the template in `09_brand_identity.md` Section 5 before regenerating anything, so every page stays visually consistent with the current brand decision.

---

## 0. Brand identity in force for this build

**Status: locked.** The product name is **Orbicrew, by Leangine**. This section is a direct copy of the current, authoritative template from `09_brand_identity.md` Section 6.7 — if the brand direction ever changes, update it there first, then re-paste the block here so the two documents never drift out of sync.

```
BRAND NAME: Orbicrew, by Leangine
TAGLINE: Hire AI employees, not another chatbot

TYPOGRAPHY:
  Display/heading face: Bricolage Grotesque, 600 weight
  Body face: Hanken Grotesk, 400 weight

COLOR TOKENS (light / dark):
  --bg-primary:      #F7F5FB / #181233
  --bg-surface:      #FFFFFF / #221A47
  --text-primary:    #201A3D / #EAE6F7
  --text-secondary:  #5C5480 / #A79FCB
  --accent-primary:  #534AB7 / #7F77DD
  --accent-gold:     #E0A94C / #E8BC6C
  --warning:         #D68A2E / #E4A248
  --danger:          #C94F4F / #D96666
  --success:         #4C9A6A / #5FA875
  --border:          #E4E0F0 / #362C5C

LOGO DIRECTION: Three-tier lockup — (1) wordmark-only (default, ≥160px), (2) icon + wordmark horizontal lockup (compact spaces), (3) compact orb-with-integrated-O icon (favicon/app-icon, 32-64px only). No robot/gear/circuit icon, no gradient-blob, no tagline in the mark itself. Full spec: `09_brand_identity.md` Section 6.5, and Section 1.6 below.

VOICE: Plain verbs, no hype. Cost-honest language (real numbers, never vague "credits"). Global English, no regional idiom. Premium through restraint — no gradients-as-decoration, no stock AI imagery, no exclamation-heavy copy.
```

**Color palette rationale, briefly:** both direct competitors researched (Lindy, Manus) default to near-monochrome black/white/gray branding with no distinct color identity — Deep Violet was chosen specifically to give Orbicrew a memorable, premium visual anchor that neither competitor has, while avoiding blue (the oversaturated default across the wider AI-SaaS category) and avoiding green (which reads as a "savings/fintech" signal, the wrong association for an agent-orchestration product). Full competitive analysis and the two runner-up palettes considered (Dusty Terracotta, Ocean Teal) are documented in `09_brand_identity.md` Section 6.2, kept there for reference if this direction is ever revisited.

**Overall aesthetic direction, stated plainly for the generator:** minimal, elegant, modern, premium. Generous whitespace. Confident typography doing most of the visual work rather than decoration, icons, or color doing it. No gradients used as decoration (gradients are acceptable only as extremely subtle, functional depth cues — e.g., a barely-there surface elevation — never as a hero-section background effect). No stock "AI" visual clichés: no glowing orbs, no neural-network line-art, no circuit-board textures, no robot mascots. The product should look like it was designed by people who trust the content to carry the page, the way a well-made physical product (a good notebook, a well-built tool) earns trust through restraint rather than embellishment. If in doubt on any page below, choose the plainer, calmer option over the more decorated one.

**Market/language note for the generator:** this is a global, English-first product. All UI chrome, marketing copy, and default language is English. The product's chat feature lets end users converse with their AI agents in any language they choose (this is a runtime chat capability, not a UI localization feature) — reflect this only where the chat/task-submission surface is described below (a language auto-detect indicator on the chat input), not anywhere else in the interface. Do not add a language switcher to marketing pages, onboarding, or dashboard chrome — those stay English-only for this build.

---

## 1. Design system foundations

### 1.1 Theming approach

- Every color, spacing, and radius value is a CSS custom property (design token, per Section 0 above), never hardcoded in component code — this is what makes dark/light mode a data swap, not a rebuild.
- Respect the OS-level `prefers-color-scheme` on first load, then let the user override and persist their choice (stored per-user).
- Do not simply invert light-mode colors for dark mode — dark backgrounds use the dedicated dark token values from Section 0, not a mathematical inversion of the light values, and accent colors should read as very slightly desaturated in dark mode so they don't vibrate against the dark background.

### 1.2 Type scale

| Role | Font | Size (desktop) | Size (mobile) |
|---|---|---|---|
| Display / hero | Display face, 600 weight | 48px | 32px |
| Page heading (H1) | Display face, 600 weight | 32px | 24px |
| Section heading (H2) | Display face, 500 weight | 22px | 18px |
| Body | Body face, 400 weight | 16px | 15px |
| Small/caption | Body face, 400 weight | 13px | 12px |

### 1.3 Logo usage

Orbicrew uses a three-tier lockup system — matching the pattern both direct competitors (Lindy, Manus) use, where a compact icon exists only as a small-format fallback and the wordmark leads everywhere else. Full rationale and file specs live in `09_brand_identity.md` Section 6.5; the summary an interface generator needs is below.

**Primary — wordmark only.** "Orbicrew" set in Bricolage Grotesque 600 weight, in `--accent-primary`. Default for anything ≥160px width: marketing header, dashboard top bar (desktop), footer, email signature, social profiles. No icon accompanies this form.

**Secondary — icon + wordmark, horizontal.** The compact icon (below) sits to the left of the wordmark, vertically centered, gap = 0.4× icon height. Use where height is constrained but width allows both — compact in-product headers, mobile top bar, button labels needing a brand anchor.

**Compact icon only — 32-48px contexts.** A circular orb motif doubling as the letter "O": a violet ring with a smaller solid violet core dot offset toward the top, plus one small gold satellite dot near the ring's upper edge — simultaneously readable as "O" and as a simplified orbit diagram. Sits on a rounded-square tile using `--bg-primary` (light mode) or the dark-mode `--bg-surface` value as the tile background. Favicon and app-icon use only — never used standalone above ~64px; past that size, step up to the secondary lockup.

| Context | Lockup |
|---|---|
| Marketing site header, footer | Primary (wordmark only) |
| Dashboard top bar, desktop (≥`md`) | Primary (wordmark only) |
| Dashboard top bar, mobile / compact nav | Secondary (icon + wordmark) |
| Browser favicon, app icon (mobile home screen, PWA) | Compact icon only |
| Email signature, social profile avatar | Compact icon only (avatar/square contexts) or Primary (signature/banner contexts) |
| Loading/splash screen | Compact icon only, centered, no animation beyond a simple fade-in |

Never place any lockup over a busy photo or gradient background — always a flat surface from the token set (`--bg-primary` or `--bg-surface`) behind the mark, in both light and dark mode.

### 1.4 Responsive breakpoints

| Breakpoint | Width | Primary target |
|---|---|---|
| `sm` | ≥ 640px | Large phones, landscape phones |
| `md` | ≥ 768px | Tablets |
| `lg` | ≥ 1024px | Small laptops |
| `xl` | ≥ 1280px | Desktops |
| `2xl` | ≥ 1536px | Large monitors |

Mobile-first CSS throughout — base styles target the smallest viewport, breakpoints add complexity upward, never the reverse. Every page in this document must be specified for both a desktop/tablet layout and a mobile layout; where a page below only describes one, apply the standard collapse rules from Section 1.5.

### 1.5 Standard responsive collapse rules (apply to every page unless a page explicitly overrides this)

- Multi-column layouts collapse to a single column below `md`.
- A left sidebar/nav collapses to a bottom tab bar below `md` (not a hamburger menu — a hamburger hides exactly the navigation items, like Agents and Approvals, that need to stay visible and discoverable at a glance).
- Data tables collapse to a stacked card list below `md`, one card per row, with the table's column labels becoming inline field labels within each card.
- Multi-series charts simplify to a single key number plus a simple bar/sparkline below `md`, rather than attempting a dense chart at a cramped width.

### 1.6 Accessibility and motion baseline (non-negotiable, applies to every page)

- Visible keyboard focus states on every interactive element, in both light and dark themes.
- Reduced-motion respected (`prefers-reduced-motion`): the live-updating activity feed fades rather than slides/animates in, and Orbit View's sprite movement (Section 14) snaps to position rather than walking, when this is set.
- Color is never the only signal for status — every status badge or indicator pairs color with an icon or text label, since color-blind users otherwise can't distinguish states like "done" (green) from "failed" (red) in a task list.
- Minimum touch target size 44×44px on all mobile interactive elements.

---

## 2. Full page inventory

The product has four top-level areas: **Marketing** (public, pre-signup), **Onboarding** (first-run), **Dashboard** (the core authenticated product — Standard GUI and Orbit View are both permanently-available presentations of it, never a forced either/or, see Section 14), and **Settings**. Every page below belongs to one of these.

Marketing Landing Page → Sign Up/Login → Onboarding Wizard → Dashboard Home, which branches to: Agent Roster (→ Agent Detail, → Create Agent Wizard, → AI-Generate Agent), Task List (→ Task Detail), Chat GUI, Orbit View, Approvals Inbox, Usage & Billing, and Settings (→ Connected Channels, Plan & Billing, API Keys, Team Members, Interface Preference).

---

## 3. Marketing landing page

**Purpose:** convert a visiting agency owner or freelancer into a signup within one screen of scrolling. This is the single most important page for first impressions of the brand — apply Section 0's restraint principle here hardest, since marketing pages are where "AI-generated feeling" clichés show up most.

**Hero section:** not a generic large-headline-plus-gradient-background template. Instead: a calm, confident headline using the tagline or a close variant ("Hire AI employees, not another chatbot"), a one-sentence subhead explaining the core mechanic in plain words ("Hand off real work to a roster of AI agents. Watch it get done. See exactly what it cost."), a single clear call-to-action button, and — as the visual centerpiece instead of a stock illustration or gradient blob — a small, real-feeling animated demo panel: a task card being assigned, a status indicator moving through its states, and a real-looking dollar figure (e.g., "$0.03") ticking in next to it as the task completes. This directly dramatizes the cost-transparency differentiator instead of describing it. Background is a flat or near-flat surface color from the token set — no decorative gradient wash behind the hero text.

**Below the fold, three-column section:** three columns, each with a short label, one sentence of explanation, and a small supporting visual (a simple icon or a tiny illustrative screenshot crop, not a large illustration) — "Hire agents" (assign work the way you'd delegate to a real employee), "Talk your way" (voice or text, and converse in whatever language you prefer), "Sleep while they work" (overnight execution with a Digest waiting for you). These map directly to the three brand pillars.

**Pricing section:** the real tier table (Starter/Growth/Studio/Enterprise for managed, Starter/Growth/Studio for BYO — see `02_cost_and_pricing.md` for current numbers), shown plainly as a simple comparison table or card row, real dollar figures visible without a click-to-reveal step. No "contact us for pricing" opacity except on the genuinely custom Enterprise tier.

**Social proof section (once real customers exist):** a small number of real, specific testimonials (a name, a role, and a concrete number — "saved me 6 hours a week" — not a vague quote) rather than a wall of generic logos.

**Footer:** standard — links to pricing, docs, login, and legal, in a plain, unadorned layout.

**Responsive:** below `md`, the hero's live demo animation simplifies to a static illustrative screenshot rather than attempting the live animation on a small, likely lower-powered screen — this is a performance and clarity decision, not just a space one.

---

## 4. Sign up / login

A minimal, single-purpose page: email + password fields, or a Google OAuth button, nothing else on this screen. Do not add marketing copy, testimonials, or feature lists here — the person has already decided to sign up; this page's only job is to get them through with minimal friction. A small dark/light mode toggle is visible here (not hidden inside settings only), since first impressions matter and some visitors will have a strong preference immediately.

---

## 5. Onboarding wizard

A five-step first-run flow, one focused decision per screen, with a visible step indicator (e.g., "Step 2 of 5") so the person always knows how much is left.

1. **Choose your pricing track.** Two large, clearly labeled cards: "Managed" (we handle model access, simple monthly plan) and "Bring your own API" (you connect your own model provider, lower flat fee). Plain, short explanatory text under each, not a wall of comparison detail — the full comparison lives on the pricing page they've already seen.
2. **Choose how you want to work.** Cards for each interface mode: Standard Dashboard (recommended, described as "a clean list-and-chat view") and Orbit View (described as "a visual, spatial view of your agents you can customize," see Section 14), and note that Telegram/Discord/WhatsApp can be connected later from Settings. This selection sets a default, not a permanent lock — both views stay available side by side at all times regardless of which is chosen here, make that explicit in the copy ("you can switch anytime, both stay available").
3. **Pick your first agents.** A simple grid of starter agent templates (Writing, Research, Coding, and a few others) as selectable cards, each with a one-line description of what it does. A visible "skip, I'll add these later" option — this step should never feel mandatory.
4. **Connect your model access.** Only shown if "BYO" was chosen in step 1: a simple form to paste an API key, with a "we validate this immediately" confirmation once entered, and clear reassurance copy about how the key is stored (encrypted, never logged — matches `06_security_and_scalability.md`).
5. **Try it.** A single, prominent text input with placeholder copy like "Try assigning your first task — for example: 'Draft a welcome email for new clients.'" Submitting this takes the person straight into the Dashboard with that task now visibly running — the onboarding flow's entire purpose is to get someone to this first real moment as fast as possible.

**Responsive:** each step remains a single full-screen focus on mobile, same content, larger touch targets, no layout changes beyond the standard stacking rules.

---

## 6. Dashboard home (Standard GUI mode)

**Purpose:** the default landing screen after login for anyone using the Standard Dashboard interface preference — a calm overview, not a dense data wall.

**Desktop/tablet layout (≥`lg`):** a persistent left sidebar containing primary navigation (Home, Agents, Tasks, Approvals, Usage & Billing, Settings) plus a top bar above the main content area (brand wordmark, a global search field, a notifications icon, the dark/light toggle, and the account avatar/menu on the far right). The main content area itself is organized top-to-bottom as: a row of three or four compact stat cards (active tasks right now, this month's spend against budget as a simple progress indicator, agents currently busy vs. idle), then a live-updating activity feed showing recent task events as they happen (agent name, a plain-language description of what it just did, a timestamp, and — if relevant — a cost figure), then, only if there are pending approvals, a compact "needs your attention" widget summarizing them with a link into the full Approvals Inbox.

**Mobile layout (<`md`):** top bar simplifies to just the wordmark and avatar; the stat cards stack into a single scrollable row or a 2-up grid; the activity feed remains the primary scrollable content; navigation moves to a bottom tab bar (Home, Agents, Tasks, More) per the standard collapse rule in Section 1.5. Uses the secondary (icon + wordmark) lockup per Section 1.3, since the mobile top bar is a compact-height context.

**Behavioral notes:** the activity feed updates live via a persistent connection, not on manual refresh — new events should visually settle in gently (a soft fade, not a jarring pop) to avoid feeling chaotic when many agents are active at once. The spend-vs-budget stat is always visible near the top of this page specifically, not buried in a billing sub-page, because reinforcing "you can always see what this is costing you" on the single most-viewed page in the product is core to the brand's trust positioning.

---

## 7. Chat GUI (task submission)

**Purpose:** the primary, lowest-effort way most people will interact with the product day to day — functionally similar to a modern chat app, but every message either comes from or results in an actual agent doing actual work, not just a text reply.

**Layout:** a single-column conversation thread taking up most of the screen, showing task history and agent responses in chronological order. Each agent message is prefixed with a small avatar and the agent's name (e.g., "Research Agent"), and when an agent is actively working, its message renders as a status line that updates in place ("Research Agent is searching the web...", then "Research Agent found 6 sources, drafting summary...") rather than a spinner alone — this gives the person something concrete to read while waiting, consistent with the brand's plain-language, no-mystery voice. Below the thread, a fixed input bar contains: a microphone icon for voice input (tap to record), a paperclip/attach icon for images or files, a text field, and a send button, all in a single row. A small, unobtrusive language indicator sits near the input field (e.g., "EN" with a subtle globe icon) that auto-detects the language being typed or spoken and can be tapped to override manually — this exists purely to reassure the person that typing or speaking in another language will work correctly, not as a general UI language switcher.

**Responsive:** on mobile, the input bar becomes a fixed element pinned to the bottom of the screen exactly like a standard messaging app, with the thread scrolling underneath it — a pattern virtually every visitor will already know intuitively.

**Dark mode note:** agent status bubbles use a very slightly elevated surface color with a subtle 1px border in dark mode, rather than sitting flush with the background — this keeps them clearly legible as distinct message blocks rather than blending into the page.

---

## 8. Task list / history

**Layout (desktop):** a filterable table with columns for status, agent, channel (which interface the task came in through), date, and cost. Status is shown as a small pill/badge that pairs a color with a short label and a matching icon (queued: neutral gray circle icon; running: accent-green pulsing dot; awaiting approval: warning-amber clock icon; done: success-green checkmark; failed: danger-red icon) — never color alone, per the accessibility baseline. Clicking any row opens the Task Detail page.

**Layout (mobile):** the same information as a stacked card per task — status badge and cost prominent at the top of each card, agent/channel/date as secondary text below.

---

## 9. Task detail (execution trace)

**Purpose:** the audit-trail view — this is the page that answers "what did my AI employee actually do today," which is a core trust feature, so it should feel transparent and calm, not like a raw debug log.

**Layout:** a header showing the task's short description, its current status badge, which agent handled it, which channel it came in through, and its total cost, all in one line or a tight cluster at the top. Below that, a vertical, numbered sequence of steps the agent actually took, written in plain language rather than technical trace format — e.g., "Classified as moderate complexity, routed to a mid-tier model" rather than a raw model-ID string; "Searched the web for 'product launch trends'" rather than a raw tool-call JSON blob. Tool calls are shown collapsed by default (a small expandable row) except for any step that failed or was flagged, which expands automatically so the person sees exactly where something went wrong without having to hunt for it. Each step shows its own small cost figure where relevant, and the header total is the sum — cost transparency applied at the most granular level available, not just as a single top-line number. At the bottom, any real output artifacts (a generated document, an image, a link to a pull request) are shown as clearly labeled, directly clickable/downloadable items, not buried in the step log.

---

## 10. Approvals inbox

**Purpose:** the "morning inbox" — the single place where anything an agent wants to do that's risky, costly, or irreversible surfaces for a yes/no decision, especially after unattended overnight work.

**Layout:** a simple vertical list, each pending item showing: a plain-language description of what's being requested (e.g., "Coding Agent wants to open a pull request titled 'Add checkout validation'"), enough context to decide without leaving this screen (a diff preview for code, a rendered preview for a social post, the full recipient list for an email blast), and two clear action buttons, Approve and Reject, styled distinctly (Approve as the primary accent color, Reject as a plain outlined button, never both looking equally weighted). Nothing on this page should require clicking through to a different page to understand what's being asked — the entire decision-relevant context is inline.

**Empty state:** when there's nothing pending, this should read as calm reassurance, not a blank page — a short, warm line like "All caught up — nothing needs your review right now," matching the voice principles in `09_brand_identity.md` Section 5 (empty states as reassurance, not dead space).

---

## 11. Usage & billing dashboard

**Managed-plan view:** current billing period's spend shown as a progress bar against the plan's included quota, a breakdown of cost by agent (which agents are driving spend), and a breakdown of cost by model tier — this last one is a genuine differentiator worth surfacing prominently, since showing "80% of your tasks ran on a cost-efficient model tier" is direct, visible proof that the platform's routing/cost-control system is actually working, not just a marketing claim. An upgrade prompt appears only when genuinely approaching the plan's limit, phrased plainly ("You're at 85% of this month's included usage") rather than as urgency-manufacturing dark-pattern copy.

**BYO-plan view:** since there's no platform-side usage cost to show, this simplifies to: current platform-fee plan status, and a clearly-labeled *estimate* of the person's own provider spend (pulled from the same per-task cost-estimation used elsewhere), explicitly marked as an estimate since their actual billing happens on their own provider account, not ours.

**Responsive:** on mobile, multi-series charts simplify to the single most important number plus a basic bar view, per the standard collapse rule.

---

## 12. Settings

A simple settings shell with a left-hand (desktop) or top-tab (mobile) list of sub-pages: Connected Channels, Plan & Billing, API Keys, Team Members, Interface Preference.

- **Connected Channels:** connection status and connect/disconnect controls for Telegram, Discord, and WhatsApp, each shown as a simple row with a status indicator and a single primary action button.
- **Plan & Billing:** current tier, upgrade/downgrade controls, payment method on file, and a plain invoice history list.
- **API Keys (BYO only):** add, rotate, or remove provider API keys. Keys are always shown masked after initial entry (e.g., `sk-...ab12`) and never displayed in full again — each key row shows a simple valid/invalid status indicator.
- **Team Members:** invite/remove users and assign roles (owner/admin/member) — this sub-page is only visible at all on plans that support multiple users (Studio tier and above); it simply doesn't appear in the settings list otherwise, rather than appearing greyed-out.
- **Interface Preference:** the default-view control described in Section 5's step 2, framed here as "your default view when you log in," with a short reassurance line that every connected channel (chat GUI, Orbit View, Telegram, etc.) remains simultaneously usable regardless of which is set as default.

---

## 13a. Agent roster

**Purpose:** the customer's roster of configured AI employees — the core "hire, don't prompt" mental model made visible.

**Layout (desktop):** a card grid, one card per agent, three or four columns wide. Each card shows the agent's avatar (a simple, calm illustrated or abstract icon per agent, not a photorealistic face), its name and role (e.g., "Writing Agent"), a live status indicator (Active/Idle/Disabled), a lightweight usage figure for the current period ("12 tasks this month"), and two compact quick-action links ("Edit," "Disable"). Alongside the agent cards, two special cards are always present and visually distinct (e.g., a dashed border rather than a solid one): a plain "+ Create Agent" card for manual creation, and a "✨ Generate with AI" card that opens the meta-agent flow (Section 13c) — both are first-class, equally discoverable actions, not one hidden behind the other, since letting the system propose an agent from a plain-language description is meant to feel just as normal as building one by hand.

**Layout (mobile):** the grid collapses to two columns on a tablet-sized screen and to a single stacked list on a phone-sized screen, same card content, larger touch targets.

**Master Agent, shown distinctly:** exactly one agent in the roster is the Master Agent — the one the person actually talks to about anything, who in turn delegates to or spins up the specialist agents. Its card is visually distinguished from the rest of the roster (e.g., a subtle accent-colored border or a small "Master" label) and is pinned first in the grid, always. Clicking it opens the same Chat GUI thread described in Section 7, since the Master Agent is who most conversations happen with by default — specialist agents are usually worked with indirectly, through the Master Agent's delegation, rather than chatted with directly, though direct access to a specialist's own thread remains available from its own card for people who want it.

---

## 13b. Agent detail / config editor

**Purpose:** the configuration surface for a single agent, organized as four clearly labeled tabs (or, on mobile, a vertically stacked accordion instead of horizontal tabs, since tab labels get cramped at narrow widths): **Soul**, **Skills**, **Memory**, **Setting**.

- **Soul tab:** a short role description field, an expandable system-prompt text area for more detailed instruction, and a free-text "tone/persona notes" field — this is where the agent's personality and purpose are defined in the person's own words.
- **Skills tab:** a checklist of available tools this agent is allowed to use (web search, code execution, image generation, file access, and so on — the full tool catalog per `14_universal_task_coverage.md`), shown as simple labeled checkboxes, not a dense permissions matrix.
- **Memory tab:** a plain list of what this agent currently remembers (client names, brand guidelines, past decisions), each entry removable individually, plus a way to add a memory entry manually.
- **Setting tab:** the agent's default model-selection mode (Auto or Manual, per `03_system_design.md` Section 5), its budget cap, and — the most safety-critical control on this page — an action whitelist listing every irreversible or external action type (sending an email, publishing a post, deploying code) with a clear per-action toggle between "runs automatically" and "requires my approval first," defaulted to requiring approval for everything external.

---

## 13c. Create agent wizard / AI-generate agent

**Manual creation:** a short multi-step form mirroring the Soul/Skills/Memory/Setting structure from Section 13b, with sensible, conservative defaults already filled in (a modest budget cap, an approval-required action whitelist) so a new agent can never accidentally be created with wide-open permissions.

**AI-generate flow:** a single, prominent text input inviting a plain-language description — e.g., "I need someone who edits product photos and writes Instagram captions." Submitting this shows a generated draft configuration (filled-in Soul/Skills/Memory/Setting sections, same shape as the manual form) presented for review, every field editable inline before confirming. This review step is never skippable — a system-generated agent always lands in an editable draft state and requires an explicit confirm action before it goes live, never auto-activating.

---

## 14. Orbit view — the spatial office UI (summary — full spec in `18_orbit_view_game_ui.md`)

**This entire feature now has its own dedicated document, `18_orbit_view_game_ui.md`.** It grew far beyond what belongs in a page-by-page interface guide — a living, decorated, multi-room 2D world with idle-time NPC behavior, amenity rooms, a full room/decor editor, and a scripting layer needs its own spec, not a subsection here. What follows is a short summary so this document stays self-contained for anyone reading it in isolation; hand `18_orbit_view_game_ui.md` to Stitch (or whichever generator builds this specific feature) alongside this file. For a dedicated, copy-paste Stitch prompt pack (every screen/state, animation matrix, acceptance checklist), use `19_orbit_view_stitch_prompts.md`.

**What it is:** an additional, permanently-available way to view and work with the agent roster — a living 2D virtual office, not a static diagram. Multiple connected rooms (desks, a break room, a lounge with TVs, washrooms, a kitchen/food area), each agent visibly present and doing something at all times: working at a desk when assigned a task, or wandering to the lounge, kitchen, or break room and doing something believable there when idle — the same instantly-recognizable "real, populated office" feeling as the Agent Town reference screenshot, executed in Orbicrew's own brand language and taken further with rooms, amenities, and interactivity the reference project doesn't have.

**What it is not:** a replacement for the Standard Dashboard (both stay permanently available, selectable per Section 12's Interface Preference or via a persistent top-bar icon), and not a 3D scene — flat 2D, rendered efficiently by construction (canvas/WebGL, texture atlases, viewport culling, throttled off-screen animation), so a large, richly decorated, many-agent office stays light to run rather than becoming a resource hog as it grows. Full technical rationale, the room/amenity catalog, idle-time NPC behavior rules, the decoration and company-branding system, the safe UI-only scripting layer, and zoom/navigation behavior are all in `18_orbit_view_game_ui.md`.

---

## 15. Empty, loading, and error states (apply across every page above)

- **Empty states** are written as a warm invitation to act, in the product's own voice, never a blank gray box — e.g., an empty Agent Roster reads "No agents yet — hire your first one," with the create/generate actions immediately visible right there, not just described in text.
- **Loading states** use skeleton screens for list/table views (matching the eventual real layout's shape) rather than a bare spinner — this reduces perceived wait time and prevents the layout from jumping once real content arrives.
- **Error states** explain what happened and exactly how to fix it, in the product's calm, plain voice, never a generic apology — e.g., "Task paused: budget cap reached. Increase the cap or approve additional spend to continue," which is directly actionable, versus a bare "Something went wrong."
