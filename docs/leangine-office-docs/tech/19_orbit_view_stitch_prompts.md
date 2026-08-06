# Orbit View — Stitch prompt pack (game UI screens & states)

**This document is self-sufficient.** Paste from it alone into Stitch (or a similar UI generator). Everything needed — brand tokens, product chrome rules, Orbit behavior, screen prompts, motion matrix, and acceptance checks — lives here. Do not open other product docs to generate Orbit View frames.

**How to use this document**

1. Paste **Prompt 0 (Brand + art-direction lock)** once at the start of a Stitch session (or into every major generation if sessions are isolated).
2. Generate in the **recommended order** in §6 — foundations first, then world, then chrome overlays, then states.
3. Each numbered block under §7 is a **standalone paste**. Keep the block between the `<<<STITCH` / `STITCH>>>` markers intact.
4. Do **not** ask Stitch for production Phaser/React code. Ask for **high-fidelity UI frames, component sheets, and motion notes**.
5. Prefer **light mode** frames first, then regenerate the same screens with the dark/evening office pass (Prompt 14).
6. When iterating, append §13 Negative constraints and/or Consistency reminder.

---

## 1. Brand identity (locked — use everywhere)

**Product:** Orbicrew, by Leangine  
**Tagline:** Hire AI employees, not another chatbot

### 1.1 Color tokens — Deep Violet

| Token | Light | Dark | Use |
|---|---|---|---|
| `--bg-primary` | `#F7F5FB` | `#181233` | Page / chrome background |
| `--bg-surface` | `#FFFFFF` | `#221A47` | Card / panel / dock background |
| `--text-primary` | `#201A3D` | `#EAE6F7` | Primary text |
| `--text-secondary` | `#5C5480` | `#A79FCB` | Secondary / muted text |
| `--accent-primary` | `#534AB7` | `#7F77DD` | Wordmark, primary CTAs, links, focus rings, Master Agent accents |
| `--accent-gold` | `#E0A94C` | `#E8BC6C` | Sparse highlight — badges, one emphasized number, orbit satellite mote; never a large fill |
| `--warning` | `#D68A2E` | `#E4A248` | Warning / awaiting approval |
| `--danger` | `#C94F4F` | `#D96666` | Danger / failed |
| `--success` | `#4C9A6A` | `#5FA875` | Success / running (status-only — not brand primary) |
| `--border` | `#E4E0F0` | `#362C5C` | Hairline borders |

**Usage discipline:** Violet is the dominant identity color. Gold appears sparingly. Success/warning/danger are status-only. Never invent a second “game HUD” palette.

### 1.2 Typography

| Role | Font | Weight | Desktop | Mobile |
|---|---|---|---|---|
| Display / hero | Bricolage Grotesque | 600 | 48px | 32px |
| Page heading (H1) | Bricolage Grotesque | 600 | 32px | 24px |
| Section heading (H2) | Bricolage Grotesque | 500 | 22px | 18px |
| Body | Hanken Grotesk | 400 | 16px | 15px |
| Small / caption | Hanken Grotesk | 400 | 13px | 12px |

Avoid Inter, Roboto, Arial, or bit-font RPG chrome as the primary UI type.

### 1.3 Logo

1. **Primary — wordmark only** (≥ ~160px width): “Orbicrew” in Bricolage Grotesque 600, `--accent-primary`. Default for desktop top bar.
2. **Secondary — icon + wordmark:** compact icon left of wordmark; gap = 0.4× icon height. Use for mobile / compact nav.
3. **Compact icon only (32–64px):** violet ring (“O”) + offset violet core + small gold satellite on a rounded-square tile (`--bg-primary` light / dark `--bg-surface`). Favicon / splash only — never standalone above ~64px.

**Never:** robot, gear, circuit, neural-net, gradient-blob marks. Tagline is separate copy — not part of the logo file.

### 1.4 Voice

- Plain verbs, no hype, no exclamation-heavy copy.
- **Cost-honest:** real dollar amounts (`$0.04`, `$12.40`) — never vague “credits.”
- Global English for all chrome; no region-specific idiom.
- Premium through restraint — no decorative gradients, no stock AI imagery.

### 1.5 Overall aesthetic (product + world)

Minimal, elegant, modern, premium. Generous whitespace in chrome. Confident type does most of the work. No gradients as decoration. No glowing AI orbs, neural-network line-art, circuit textures, or robot mascots. If unsure, choose the plainer, calmer option.

**World art direction:** flat, geometric, low-detail 2D top-down office (pixel art optional, not mandatory). Soft edges, readable silhouettes, quiet charm. Floors/walls/furniture from brand-adjacent neutrals with violet on focus, selection, Master Agent desk border, and doorway hints. Gold only as tiny highlights.

---

## 2. Product chrome rules that Orbit must match

Orbit View must feel like the same product as the Standard Dashboard — not a bolted-on game skin.

### 2.1 Status badges (icon + label — color alone is never enough)

| Status | Visual |
|---|---|
| queued | Neutral + circle icon + “Queued” |
| running | Success + pulsing dot + short live label (“searching the web…”, “drafting…”) |
| awaiting approval | Warning + clock + “Needs approval” |
| done | Success + check |
| failed | Danger + X + short failure bubble when needed |

### 2.2 Chat GUI (docked in Orbit — same language as Standard chat)

- Thread: agent avatar + name; while working, status lines update **in place** (not spinner-only).
- Streaming answers; tool/step rows collapsed by default; failed steps auto-expanded.
- Per-segment cost figures where relevant.
- Composer row: mic · attach · text · send; language chip (“EN” + globe) auto-detects spoken/typed language (chat capability only — **not** a site-wide language switcher).
- Dark mode: agent bubbles on slightly elevated surface + 1px border so they don’t melt into background.
- Include **Stop** when a task is running.

### 2.3 Empty / loading / error voice

- **Empty:** warm invitation to act — never a blank gray box. Example: “Your office is ready” + clear CTAs.
- **Loading:** skeleton / progress of eventual layout — not a lone cartoon spinner.
- **Error:** what happened + how to fix it. Example: “Task paused: budget cap reached…” — never “Something went wrong.”

### 2.4 Approvals tone

Morning-inbox calm: enough inline context to Approve / Reject without hunting. Empty approvals = “All caught up…”, not dead space.

### 2.5 Accessibility & motion (non-negotiable)

- Visible keyboard focus on interactive chrome.
- `prefers-reduced-motion`: snap agents to positions (no walk cycles); badge/text fades instead of walks/slides; orbit rings omitted.
- Status always icon + label (not color alone).
- Mobile touch targets ≥ 44×44px.

### 2.6 Breakpoints (for layout decisions)

| Breakpoint | Width |
|---|---|
| `sm` | ≥ 640px |
| `md` | ≥ 768px |
| `lg` | ≥ 1024px |
| `xl` | ≥ 1280px |

Orbit **room editing and free camera** are desktop/tablet (≥ `md`). Below that: Overview-only (see §3.6).

### 2.7 Theming

Respect `prefers-color-scheme` initially; allow user override. Dark mode uses dedicated dark tokens — not inverted light art. Orbit needs **two intentional art passes** (day office / evening office), not a color-filter of one pass.

---

## 3. Orbit View product behavior (full source of truth for this pack)

### 3.1 One-paragraph intent

Orbicrew Orbit View is a living, multi-room **2D top-down office** where AI employees are always visibly *somewhere* and doing *something*. The human is **not** a walkable avatar — they are a camera watching a small, charming, brand-true office. Clicking an agent opens a docked chat/task panel (same product language as Standard Dashboard). Agents walk between desks, break room, kitchen, lounge, washrooms, and lobby. Delegation from the Master Agent draws a brief **orbit-ring trail** (brand metaphor). Status, cost, and chat must match the rest of Orbicrew. Aim for clearer status, calmer chrome, richer idle life, and stronger company identity than a generic pixel RPG office HUD.

**One-line test for every visual decision:** does this make the office feel more like a real, lived-in place, without costing meaningfully more to render? If either half fails, cut it from v1 frames (or mark “later” in annotations).

### 3.2 Hard differentiators vs Agent Town–style references (inspire, do not copy)

| Orbicrew | Avoid (reference trap) |
|---|---|
| No player / boss character | Walking RPG protagonist, “Press E” proximity menus |
| Multi-room office with amenities + idle life | Single desk floor, idle-at-desk only |
| Deep Violet / gold brand tokens + product sans type | Separate brown/gold pixel HUD + bit font |
| Cost figures on working desks; approvals / cost toasts | Token/context meters as RPG HP bars |
| Company banner in lobby; decoration editor | Fixed map only |
| Side chat dock reusing product Chat GUI | Full-screen game terminal takeover only |
| Three zoom tiers (Room / Floor / Overview Map) | Continuous zoom without status-preserving tiers |
| Mobile = glanceable Overview only | Requiring keyboard + walk-up interaction |

### 3.3 Rendering approach (for art + frame decisions)

- **2D, not 3D** — top-down, flat, resource-light on purpose.
- **Split surfaces:** world (floors, walls, furniture, agents, paths, orbit rings) = canvas/WebGL; surrounding chrome (top bar, chat dock, minimap, toasts, palette) = ordinary product UI / DOM.
- Texture atlases / sprite batching; viewport culling; throttled off-screen idle simulation (state still updates coarsely so rooms don’t look stale when revisited).
- Grid / tile-based rooms — curated decor, not freeform chaos.
- **Pixel art is optional.** Flat geometric shapes with the same silhouette simplicity are equally valid.
- **Perf intent to design for:** Room/Floor ~60fps with ~20 agents and 3–4 rooms in view; Overview stays cheap via culling (status dots, not scaled furniture).

### 3.4 Room catalog (v1)

| Room type | Purpose | Typical amenities |
|---|---|---|
| **Workroom** | Desks; agents work here when tasked | Desks, chairs, monitors, shelves, optional huddle point |
| **Break room** | Short casual idle pause | Couch / seating, coffee machine, small table |
| **Kitchen** | Idle “eating” destination | Counter, fridge, table + chairs |
| **Lounge** | Social idle; “watch” TV | Seating, 1–2 wall TVs (abstract loop, not real video), rug |
| **Washroom** | Brief visit; no linger / no social | Minimal fixtures |
| **Entrance / lobby** | Front door; company banner; Master Agent default seat | Reception / greeting desk, banner, seating |
| **Meeting room** *(optional)* | Enclosed hand-off / huddle | Table, chairs, whiteboard / screen |

Rooms are rectangular or L-shaped on a tile grid, connected by doorways. Person composes layout from this palette — no single fixed template floor plan. Room *types* are curated; count/size of each type is flexible.

### 3.5 Core interaction model — delegation, not roaming

1. **Idle life** — no task → wander / amenity / socialize (not frozen at desk).
2. **Summoned** — Master or human assigns work → agent walks to their desk (or briefly via conference point) via simple grid paths through doorways.
3. **Working** — at desk: live status badge + status bubble mirroring Chat GUI language; tiny running-cost figure on desk.
4. **Done / failed → idle** — brief terminal badge, then leave desk and resume idle life (do not sit forever waiting).

**Master Agent:** lobby desk by default, accent-bordered like its roster card. Delegation visual: orbit ring from Master → specialist (see §3.7).

**Human interaction:** click (desktop) / tap (tablet) agent → docked chat/task panel on the side — office stays visible. No player avatar. No “Press E.”

### 3.6 Idle-time FSM (per agent, while idle)

- **Wandering** — slow calm walks; weighted destinations (break/kitchen > washroom; lounge more likely if others already there).
- **At amenity** — short random hold + context icon (coffee / food / watching TV).
- **Socializing** — two+ idle agents near each other → decorative “…” / chat icon only (not real dialogue). Distinct from real collab peeks that have readable text.
- **Near desk / cycling** — hover near desk area or start another wander loop until summoned.

**Tick model:** idle *decisions* on a coarse interval (seconds–minute), not every frame. Only walk animation needs frame smoothness.

### 3.7 Ambient world detail

- **Orbit rings** on Master→specialist delegation — signature brand motion.
- **Lounge TVs** — looping abstract pattern / “on” state (not real video).
- **Wall clock** (optional later) — real time.
- **Desk cost chips** — real `$x.xx` only while working.

### 3.8 Zoom & navigation

| Tier | What you see |
|---|---|
| **Room View** (default) | One room; furniture + agent activity highly readable; badges + cost chips visible |
| **Floor View** | 2–3 rooms; agents smaller; badges stay fixed screen size; costs may collapse to icon-only until hover |
| **Overview Map** | Entire office; furniture culled; agents = colored status dots/icons; rooms as labeled polygons; banner still a recognizable lobby mark |

Continuous scroll/pinch between tiers (not hard cuts). Pan via drag / two-finger. Corner **minimap** with violet viewport rectangle; click room to jump.

**Mobile (< `md`):** view-only Overview Map — no pan/zoom fight with page scroll; prompt to open desktop for editing / full Orbit chat. Standard chat remains available on phone.

### 3.9 Decoration & company banner

- Curated placeable set per room type (desks, plants, coffee, TV, banner, etc.) — considered, not infinite sandbox.
- Placing employees at desks is **visual/organizational only** — does not change agent config.
- **Company banner** in lobby: company name + optional logo upload; visible in Room and Floor views.
- Safety copy for any advanced custom-decoration scripting: “Only affects what you see — it can’t change how agents work.” Scripting is cosmetic/sandboxed only (no task, config, or backend access). Desktop/tablet edit mode only.

### 3.10 Dark / light for the world

Two art passes from brand tokens:

- **Day / light:** soft lavender ambient (`#F7F5FB` influenced), pale floors, maximum desk-status legibility.
- **Evening / dark:** deep violet night (`#181233` / `#221A47`), warm lounge TV glow, intentional monitor practical lights — **not** inverted day art.

### 3.11 Data / sync implications (for UI honesty in frames)

- Spatial layout (rooms, furniture, desk assignment, banner) is presentational metadata on top of the same task/agent truth as Standard Dashboard — never a conflicting second status model.
- Idle wander positions are ephemeral / client-side — do not invent “saved footsteps” UI.
- Live task updates should match dashboard events (running badge, toasts, approvals) in the same moment.

### 3.12 Suggested build / validation sequence (for QA of Stitch sets)

1. Single workroom + task states (assign → walk → work → done/fail).
2. Idle life + one amenity room.
3. Full room types + decor + company banner.
4. Zoom tiers + Overview.
5. Sandboxed cosmetic scripting last.

---

## 4. Product intent summary for the generator

Orbicrew Orbit View is premium SaaS chrome wrapped around a spatial world — camera-as-observer, docked product chat, multi-room idle life, cost transparency, approvals morning glance, lobby company identity, and Master Agent orbit rings. Clearer, calmer, and more brand-true than a generic pixel-office HUD.

---

## 5. Recommended generation order

| Step | Generate | Why first |
|---|---|---|
| A | Prompt 0 — brand + art direction | Locks palette, type, silhouette language |
| B | Prompt 1 — master composed desktop canvas | Establishes chrome around the world |
| C | Prompt 2 — empty / first-run office | Onboarding into spatial mode |
| D | Prompt 3 — populated Room View | Hero “alive office” shot |
| E | Prompt 4 — agent state sheet | All body/activity states in one inventory |
| F | Prompts 5–7 — selection, chat dock, conversation | Core interaction loop |
| G | Prompts 8–10 — rooms, zoom tiers, mobile overview | Navigation & layout |
| H | Prompts 11–13 — toasts, loading/offline, night pass | System states |
| I | Prompts 14–16 — decoration editor, Master Agent orbit, approvals | Orbicrew-only features |
| J | Prompt 17 — component inventory sheet | Hand-off for engineering |
| K | Use §10 matrix + §11 checklist to QA Stitch outputs | Acceptance |

*(Prompt numbering in §7: Prompt 0, then Prompts 1–18 as labeled in each block.)*

---

## 6. Prompt 0 — Brand + art-direction lock (paste first)

<<<STITCH PROMPT 0 — BRAND + ART DIRECTION LOCK
You are designing UI frames for **Orbicrew, by Leangine** — Orbit View, a living 2D top-down office for AI employees. This is premium SaaS product UI wrapped around a spatial world, NOT a retro RPG skin.

BRAND (locked — use exact tokens):
- Name: Orbicrew, by Leangine
- Tagline: Hire AI employees, not another chatbot
- Display type: Bricolage Grotesque 600
- Body type: Hanken Grotesk 400
- Light tokens: bg-primary #F7F5FB, bg-surface #FFFFFF, text-primary #201A3D, text-secondary #5C5480, accent-primary #534AB7, accent-gold #E0A94C, warning #D68A2E, danger #C94F4F, success #4C9A6A, border #E4E0F0
- Dark tokens: bg-primary #181233, bg-surface #221A47, text-primary #EAE6F7, text-secondary #A79FCB, accent-primary #7F77DD, accent-gold #E8BC6C, warning #E4A248, danger #D96666, success #5FA875, border #362C5C
- Logo: wordmark “Orbicrew” in accent-primary for desktop chrome; compact orb-O icon (violet ring + offset core + gold satellite) only in tiny contexts; secondary = icon + wordmark on mobile
- Voice: plain verbs, no hype, real dollar amounts never vague “credits”, calm empty/error copy

WORLD ART DIRECTION:
- Flat, geometric, low-detail 2D top-down office (NOT mandatory pixel art). Soft edges, readable silhouettes, quiet charm.
- Floors/walls/furniture built from brand-adjacent neutrals with violet accents on focus, selection, Master Agent desk border, and portal/doorway hints. Gold only as small highlights (status sparkle, banner trim, orbit satellite motes) — never large fills.
- No robot mascots, neural nets, glowing AI orbs, circuit textures, neon cyber grids, purple-gradient hero washes, or cartoon HP bars.
- Agents are simple humanoid silhouettes with distinct hair/outfit color per role; tiny name tags and status badges with icon+label (color alone is never enough).
- Soft daylight wash for light mode; intentionally designed evening office for dark mode (cozy lounge TV glow), NOT an inverted screenshot.

CHROME:
- HUD is calm product UI: translucent or solid surfaces from tokens, 1px borders, generous padding, Hanken Grotesk labels.
- Split: world canvas = spatial scene; surround = React/DOM chrome (top bar, chat dock, minimap, toasts).

CAMERA MODEL:
- User is a floating camera. NO walkable player character. Pan/zoom only. Click agent to select.

STATUS LANGUAGE (locked):
queued · running · awaiting approval · done · failed — each with matching icon + short label.

OUTPUT STYLE FOR ALL LATER PROMPTS:
- High-fidelity product mock frames, 1440×900 desktop unless specified mobile.
- Annotate interactive hit targets lightly when asked.
- Prefer one composition per frame; avoid dashboard clutter on top of the office.
STITCH>>>

---

## 7. Screen & state prompts (copy-paste)

### 7.1 Prompt 1 — Master composed desktop (Room View + chrome)

<<<STITCH PROMPT 1 — MASTER DESKTOP COMPOSITION
Using Prompt 0 brand lock, design the **primary desktop Orbit View** (1440×900).

LAYOUT:
1) Persistent product top bar (not a game title bar): left = Orbicrew wordmark; center-left = view switcher pill “Standard | Orbit” with Orbit active (accent-primary underline); right = notifications, dark/light toggle, account avatar. Optional slim “Today’s spend $12.40” gold-accent number near right (cost honesty).
2) Full remaining area = top-down office canvas showing an L-shaped multi-room office: Entrance/Lobby (company banner “Northwind Studio”), Workroom with 4 desks, Break room (couch + coffee), Kitchen peek, Lounge with wall TV (abstract looping pattern, not real video).
3) Corner minimap (bottom-left of canvas): monochrome floor plan, violet viewport rectangle, clickable room blobs.
4) Bottom-right floating zoom pill: “Room · Floor · Overview” with Room selected.
5) Collapsed chat dock handle on the right edge labeled “Chat”.
6) Soft ambient toast mid-right: “Research Agent finished — $0.04” with success icon.

WORLD CONTENT:
- Master Agent at lobby desk with subtle violet accent border on desk.
- 2 agents working at desks (pulsing success-status dot + short status text “searching the web…”, “drafting…”).
- 1 agent walking through doorway toward break room (calm walk).
- 1 agent on lounge couch with small “watching” TV icon.
- 1 idle agent near plant with “…” social bubble near another idle agent (decorative, no dialogue text).
- Tiny running-cost figures only on working desks (“$0.02”).

STATES TO SHOW SIMULTANEOUSLY (readable, not chaotic): idle wander, working, social hint, lounge amenity.
Do NOT include any player avatar, WASD hints, or “Press E”.
Motion notes (side annotations): subtle agent walk cycle, soft status pulse, toast fade-in, TV ambient loop.
STITCH>>>

### 7.2 Prompt 2 — Empty / onboarding orbit

<<<STITCH PROMPT 2 — EMPTY OFFICE + FIRST-RUN ORBIT
Design the Orbit View **empty / first-run** state for a new Orbicrew account who chose Orbit as preferred view.

SCENE:
- Same chrome as Prompt 1, but office is a simple starter layout: Lobby + one Workroom only. No agents placed. Desks empty. Soft natural light. Company banner shows placeholder “Your company” faded, with dashed “Add company banner” affordance.
- Centered calm empty-state card on the canvas (not a dark game overlay):
  - Display heading: “Your office is ready”
  - Body: “Place agents at desks, then assign work from chat or by selecting an agent.”
  - Primary button (accent-primary): “Place your first agents”
  - Secondary outlined button: “Start from a template office”
  - Tertiary text link: “Switch to Standard view”
- Soft coach marks (3 max): (1) point to decoration/place-agents control, (2) point to chat dock, (3) point to zoom pill. Gold accent rings, not neon.

Avoid RPG tutorial with keyboard prompts. Voice must be warm and practical.
Also generate a second frame: empty office after dismissing the card, with only a persistent bottom-center chip: “No agents placed yet — open roster to place employees.”
STITCH>>>

### 7.3 Prompt 3 — Populated office (hero alive state)

<<<STITCH PROMPT 3 — POPULATED MULTI-ROOM OFFICE
Generate a polished **hero Room/Floor hybrid** frame of Orbicrew Orbit View with ~8 agents across rooms. This is the marketing-and-product “alive office” shot.

ROOMS VISIBLE: Lobby banner “Helio Legal”, Workroom A (desks), Workroom B (conference huddle), Break room, Kitchen, Lounge, Washroom (door ajar, agent briefly exiting), Meeting room optional.

POPULATION BEATS:
- Master Agent summoned two specialists: show a gold+violet **orbit-ring trail** animating from Master desk to a specialist walking to their desk (brand moment — annotate as key motion).
- Agents: walking corridor, typing at desk, coffee at machine, watching TV, two socializing with “…” bubble, one washroom-exit, one blocked/awaiting-approval amber badge “needs approval”.
- Status badges always fixed screen-readable size; include icon + short label.
- Cost chips on working desks with real dollars.
- Minimap shows busy rooms with tinted status dots.

Feeling: calm, productive, slightly charming — never arcade chaotic. No player character.
Produce light-mode frame + annotation strip listing each agent’s current activity.
STITCH>>>

### 7.4 Prompt 4 — Agent activity state sheet

<<<STITCH PROMPT 4 — AGENT STATE SPRITE / POSE SHEET
Create a **component-style character state sheet** for Orbicrew Orbit View agents (flat geometric 2D, brand colors, not pixel-RPG sheet clones).

Show one consistent character (and small variants for Master Agent vs specialist) in a grid:

1. Idle — standing near desk / plant
2. Walking — 4 facing directions, soft walk cycle notes
3. Working at desk — typing / monitor glow
4. Thinking / queued — small “…” emote above head + queued badge
5. Blocked / awaiting approval — amber clock badge
6. Success / done — green check flash, brief hold
7. Failed / error — danger badge + short “Task failed” bubble
8. Summoned / walking to desk (purposeful faster walk)
9. Break-room coffee pose
10. Kitchen / eating pose
11. Lounge watching TV pose
12. Socializing — two agents facing, shared “…” bubble
13. Washroom visit — brief in/out (no linger pose)
14. Selected — soft violet focus ring under feet + elevated name tag
15. Hover (desktop) — lighter focus ring, cursor pointer note
16. Offline / degraded agent — desaturated silhouette + muted “offline” caption
17. Reduced-motion note tile — snap-to-position (no walk), fade badge changes only

Also show status badge chips matching product language:
- queued (neutral + circle icon)
- running (success + pulsing dot)
- awaiting approval (warning + clock)
- done (success + check)
- failed (danger + X)

Include desk-cost micro-label examples: “$0.01”, “$0.18”.
No emote spam, no angry/heart RPG clutter — keep professional-charm.
STITCH>>>

### 7.5 Prompt 5 — Agent selected (detail HUD)

<<<STITCH PROMPT 5 — AGENT SELECTED DETAIL PANEL
Design Orbit View with one agent **selected** in a Workroom (desktop).

INTERACTION:
- Selected agent has violet ground ring + slightly brighter silhouette.
- Right side slides in a **docked detail + chat panel** (~380–420px), office remains visible on the left (no full-screen takeover).
- Panel header: agent avatar/silhouette, name “Maya Chen”, role “Research Agent”, status pill “Running”, live cost “$0.07 this task”.
- Quick actions row: Assign task, Open full chat, View task detail, Pause/stop (if running).
- Below: compact current-task card with status line updating in place (“searching the web…”).
- Below: recent activity list (3 rows) with tiny costs.
- Bottom: chat composer matching product Chat GUI — mic, attach, text field, send; language chip “EN” with globe.
- Optional radial floating shortcuts near agent (Assign · Chat · Focus camera) — only 3 icons, restrained, brand tokens, NOT a heavy game radial.

Also show Master Agent selected variant: accent-bordered panel header “Master Agent”, copy explaining “Delegates work across your crew”.
STITCH>>>

### 7.6 Prompt 6 — Agent-to-agent conversation peek

<<<STITCH PROMPT 6 — AGENT-TO-AGENT THREAD PEEK
Show two agents at a huddle table / meeting room mid-handoff.

VISUALS:
- Small double speech-bubble between them with truncated plain-language peek: “Handing research notes to Writing Agent…”
- Optional thin violet dashed connector between agents.
- Clicking the peek opens a lightweight **thread peek card** anchored near them (not full dock): participants, 3 latest lines, cost to date, “Open full thread” CTA.
- Emphasize this is product-visible collaboration, not decorative RPG chatter. Idle social “…” from amenity rooms must look distinct (icon-only) from real collaboration peeks (has readable text + open affordance).

Provide two frames: (A) in-world peek only, (B) peek + open thread card.
STITCH>>>

### 7.7 Prompt 7 — Human ↔ agent conversation overlay

<<<STITCH PROMPT 7 — HUMAN↔AGENT CHAT DOCK OVERLAY
Design the primary **human-to-agent conversation** in Orbit View.

REQUIREMENTS:
- Right dock open to ~40% width, world dimmed slightly but still animating (agents keep working).
- Conversation thread identical in spirit to Standard Chat GUI: agent avatar + name, in-place status lines while working, streaming text for answers, collapsible tool/step rows (“Searched the web for ‘Q3 trends’”) collapsed by default, failed steps auto-expanded.
- Each completed segment can show a small real cost figure.
- Composer with mic, attach, text, send, language auto-detect chip.
- Header shows connection to selected agent + “Watching live in office” with a tiny live pulse.
- Include Stop task control when running.
- Empty thread state: “Assign work to Maya — she’ll walk to her desk and get started.”

Also generate a **focused assignment modal** variant triggered from “Assign task” on an unselected busy agent: short message field + “Queue for when free” checkbox + cost estimate if available — calm form, not RPG terminal.
STITCH>>>

### 7.8 Prompt 8 — Task assignment / command entry (spatial)

<<<STITCH PROMPT 8 — SPATIAL TASK ASSIGNMENT FLOWS
Produce a 3-frame storyboard of Orbicrew assignment without a player avatar:

Frame A — Select agent → dock opens → user types task → Send.
Frame B — Agent (wherever they are) shows “Summoned”, walks to desk via doorway path; status “returning to desk…”; orbit-ring spark from Master if Master delegated.
Frame C — At desk: Working badge + bubble “drafting welcome email…” + desk cost tick “$0.03”.

Alternate Frame D — Assign from Master Agent: Master stays at lobby; orbit rings fly to specialist; specialist walks; Master shows brief “Delegated” status.

No WASD. Pathing is grid-calm. Include motion timing notes: summon reaction 200ms, walk 2–6s typical room hop, work bubble fade 400ms.
STITCH>>>

### 7.9 Prompt 9 — Room / zone focus & labels

<<<STITCH PROMPT 9 — ZONE FOCUS + ROOM LABELS
Design Floor View (zoom mid) for a multi-room office with clear zone readability.

SHOW:
- Soft room name labels that float at fixed screen size: Workroom · Break room · Kitchen · Lounge · Washroom · Lobby · Meeting room.
- Labels use muted text-secondary; active focused room label uses accent-primary.
- Clicking a room on minimap pans/zooms camera to that room with a smooth ease.
- Focused room: slightly brighter floor wash; unfocused rooms gently dim (not black).
- Doorways read as clear passages.
- Decor reads considered and brand-aligned (plants, shelves, coffee, TV abstract loop).

Also show label collision rules: labels never cover agent badges; badges win z-order.
STITCH>>>

### 7.10 Prompt 10 — Zoom tiers (Room / Floor / Overview)

<<<STITCH PROMPT 10 — THREE ZOOM TIERS
Create a **horizontal triptych** of the same office at Orbicrew’s three navigation tiers:

1) Room View (default): furniture + agents highly readable; badges + cost chips visible.
2) Floor View: 2–3 rooms; agents smaller; badges stay constant screen size; costs may collapse to icon-only unless hovered.
3) Overview Map: entire office plan; furniture culled; agents become colored status dots/icons (idle/working/done/failed/approval) with optional count chip “8 agents · 3 busy”; rooms as labeled polygons; company banner still recognizable as a mark on lobby.

UI constant across: top bar, minimap, zoom pill reflecting current tier.
Annotate continuous scroll/pinch between tiers (not hard cuts).
STITCH>>>

### 7.11 Prompt 11 — Activity toasts / notifications

<<<STITCH PROMPT 11 — AGENT ACTIVITY TOAST SYSTEM
Design a restrained toast / notification system for Orbit View (desktop).

SHOW A VERTICAL STACK (max 3 visible) top-right under the product top bar:
1) Success: “Writing Agent finished ‘Welcome email’ — $0.06” + check icon
2) Warning: “Coding Agent needs approval to open a PR” + clock icon + Approve/Review actions
3) Info: “Research Agent started ‘Competitor scan’” + pulse dot

Rules in annotations:
- Soft fade/slide 200–250ms; never bounce or game-juice spam
- Pair every color with icon + text
- Clicking a toast pans camera to that agent and opens dock
- Stack collapses older toasts into “+2 more” chip
- Matches dashboard notification voice (calm, actionable)

Also design a subtle in-world alternative: desk flash once when task completes, synchronized with toast.
STITCH>>>

### 7.12 Prompt 12 — Loading / reconnecting / offline degraded

<<<STITCH PROMPT 12 — CONNECTION & DEGRADED STATES
Generate three Orbit View system frames:

A) Initial loading: office silhouette skeleton (room outlines + desk rectangles), no fake agents yet; calm caption “Opening your office…”; no cartoon spinner — use subtle violet progress bar.

B) Reconnecting: world visible but slightly desaturated; top banner “Reconnecting — live updates paused”; agents frozen mid-pose or continue cosmetic idle but with “stale” badge; chat composer disabled with helper text.

C) Offline degraded: clear banner “You’re offline. Showing last known office state.” Cost chips grayed. Action buttons disabled except “Retry connection” and “Open Standard view”. Empty-error voice: plain and fixable, never “Something went wrong.”

Include a fourth micro-state: authenticated error budget/cap — toast “Task paused: budget cap reached” with link to Usage.
STITCH>>>

### 7.13 Prompt 13 — Mobile / responsive Overview

<<<STITCH PROMPT 13 — MOBILE ORBIT OVERVIEW
Design Orbit View on a phone (390×844) per product rules: **view-only Overview Map**, no pan/zoom fight with page scroll.

LAYOUT:
- Compact top bar: secondary lockup (orb icon + Orbicrew), notifications, avatar
- Full-bleed simplified floor plan with agent status dots
- Bottom sheet half-collapsed: “3 busy · $1.24 today” + “Open on desktop for room editing & chat in Orbit”
- Primary CTA: “Open Standard chat” (mobile can still work via Standard UI)
- Tap a status dot → mini agent sheet (name, status, cost, “Continue in chat”)

Do not offer decoration editing or continuous camera free-roam on mobile.
STITCH>>>

### 7.14 Prompt 14 — Light vs dark / time-of-day atmosphere

<<<STITCH PROMPT 14 — LIGHT OFFICE + EVENING OFFICE ART PASSES
Create matched side-by-side frames of the same populated office:

Left — Light / day office: soft lavender ambient (#F7F5FB influenced), pale floors, clean daylight.
Right — Dark / evening office: deep violet night (#181233 / #221A47 surfaces), warm lounge TV glow, dimmer workroom monitors as intentional practical lights — a designed evening pass, NOT inverted colors.

Keep HUD chrome synchronized with brand light/dark tokens.
Gold remains sparse accent. Success/warning/danger stay accessible with icon+label.
Annotate: dark mode enhances lounge coziness; day mode maximizes desk-status legibility.
STITCH>>>

### 7.15 Prompt 15 — Decoration editor + company banner

<<<STITCH PROMPT 15 — DECORATION PALETTE + COMPANY BANNER EDITOR
Design Orbit View in **edit mode** (desktop/tablet only).

UI:
- Left or bottom curated palette drawer grouped by room type: Workroom, Break, Kitchen, Lounge, Lobby.
- Placeable items as simple geometric icons (desk, chair, plant, coffee machine, TV, banner, shelf). Curated count, not infinite sandbox.
- Toggle “Place employees” vs “Place decor”.
- Selected object has transform handles (move/rotate) within tile grid snappiness (show faint grid).
- Lobby banner editor modal: company name field + optional logo upload; live preview of wall banner; note “Visible in Room and Floor views”.
- Calm copy: “Arrange your office. This doesn’t change how agents work.”

Also show safety copy near advanced “Custom decoration script” entry (collapsed): “Only affects what you see — it can’t change how agents work.”
STITCH>>>

### 7.16 Prompt 16 — Master Agent orbit delegation (signature moment)

<<<STITCH PROMPT 16 — ORBIT RING DELEGATION SIGNATURE
Storyboard the **signature Orbicrew motion** in 4 keyframes:

1) User (or auto Master) assigns task to specialist currently in lounge.
2) Thin violet ring with gold satellite mote expands from Master Agent desk.
3) Ring travels as a graceful arc/path through rooms toward the specialist (readable at a glance, not a particle storm).
4) Specialist receives soft highlight, status “Summoned”, walks to desk; ring dissolves into a brief gold spark then disappears.

Include “reduced motion” alternative keyframe: ring omitted; specialist badge simply flips to Summoned and camera optional-eases toward them.
This moment should feel like the brand name made visible.
STITCH>>>

### 7.17 Prompt 17 — Approvals & overnight morning glance

<<<STITCH PROMPT 17 — APPROVALS + MORNING OFFICE GLANCE
Orbicrew-specific: overnight work results visible spatially.

Frame A — Morning entry to Orbit: toast stack “2 approvals waiting”, agents with amber badges at desks, Approvals inbox shortcut chip.
Frame B — Clicking amber agent opens dock with Approve/Reject inline (same clarity as Approvals inbox — enough context to decide).
Frame C — After approve, badge turns running/done; soft success toast with cost.

Keep tone calm and trustworthy — this is a spatial morning inbox, not a game quest log.
STITCH>>>

### 7.18 Prompt 18 — Component inventory sheet

<<<STITCH PROMPT 18 — FULL COMPONENT INVENTORY
Produce a single wide **UI kit sheet** of Orbit View DOM chrome components (not the whole office):

- Top bar / view switcher
- Agent roster pills (idle/running/failed/offline)
- Minimap
- Zoom tier control
- Status badges (all states)
- Desk cost chip
- Speech / status bubbles (work status vs social “…” vs collab peek)
- Selection ring + hover ring
- Chat dock (collapsed handle, expanded panel, composer)
- Toast variants (success/warn/info/error)
- Connection banners
- Empty-state card
- Edit-mode palette drawer
- Banner editor fields
- Mobile mini agent sheet
- Reduced-motion callouts

Use exact brand tokens/type. Annotate sizes roughly (badge 20–24px height, toast 320px width, dock 400px).
STITCH>>>

---

## 8. Component inventory (engineering-facing)

| Component | Surface | Notes |
|---|---|---|
| Product top bar + Standard/Orbit switcher | DOM | Always product chrome, not a game title bar |
| Agent roster pills | DOM | Click → camera focus + select |
| World canvas | Canvas/WebGL | Rooms, furniture, agents, paths, orbit rings |
| Selection ring | World | Violet, calm |
| Status badge | World (screen-fixed size) | Icon + label; product status language |
| Desk cost chip | World | Real `$x.xx`; working agents only |
| Activity / status bubble | World→DOM overlay | Work status lines; TTL fade ~400ms |
| Social “…” icon | World | Decorative only |
| Collab peek bubble | World + card | Real thread peek |
| Orbit ring VFX | World | Master→specialist delegation |
| Chat / detail dock | DOM | Same language as Chat GUI (§2.2) |
| Minimap | DOM or world overlay | Jump-to-room |
| Zoom tier control | DOM | Room / Floor / Overview |
| Toast stack | DOM | Activity + approvals |
| Connection banners | DOM | Reconnect / offline / budget |
| Empty/onboarding card | DOM | Warm invitation |
| Decoration palette | DOM | Desktop/tablet edit mode |
| Company banner | World + modal | Lobby identity |
| Mobile overview + sheet | DOM | View-only spatial status |

---

## 9. Animation & state matrix

| Trigger | Visual | Duration / feel | Reduced motion |
|---|---|---|---|
| Hover agent | Soft focus ring | 120ms fade | Instant ring |
| Select agent | Ring + dock slide-in | 200–280ms ease-out | Instant dock |
| Idle wander decision | Walk to amenity | Coarse tick (seconds–minute) | Snap relocate |
| Amenity arrive | Context icon (coffee/food/TV) | Hold 3–10s random | Icon only, no bounce |
| Socialize | Shared “…” | 2–4s | Static icon |
| Task assigned | Summoned badge | Immediate | Same |
| Master delegation | Orbit ring path | ~0.8–1.5s | Badge change only |
| Walk to desk | Calm purposeful walk | Room-dependent | Snap |
| Working | Pulse badge + status text updates | Live | Fade text swaps |
| Cost tick | Micro number update | Soft 150ms | Instant |
| Done | Check flash | Hold ~1.5–3s then idle-life resume | Fade |
| Failed | Danger badge + bubble | Hold then idle | Fade |
| Approval needed | Amber clock | Persistent until resolved | Same |
| Toast appear | Fade/slide | 200–250ms | Fade only |
| Zoom tier change | Continuous camera | Scroll/pinch | Discrete snaps OK |
| Dark mode lounge TV | Ambient loop | Continuous low energy | Static “on” frame |
| Offline | Desaturate + banner | Immediate | Same |
| Prefers-reduced-motion | No walk cycles | — | Required support |

**Agent behavioral FSM:**

```
idle-life (wander | amenity | socialize | near-desk)
   ↓ assign / summon
summoned → walking-to-desk → working (queued visuals → running)
   ↓ complete / fail / stop
brief terminal badge → back to idle-life
   ↘ awaiting-approval (parallel badge) until human acts
```

---

## 10. Acceptance checklist for Stitch outputs

Use this before treating a generation as “done.”

**Brand & product fit**

- [ ] Deep Violet + gold discipline (gold sparse); Bricolage + Hanken present
- [ ] No player/boss avatar; no “Press E” / WASD tutorial metaphors
- [ ] Status badges: queued/running/awaiting approval/done/failed with icon+label
- [ ] Real dollar costs where work is happening
- [ ] Chat dock feels like product Chat GUI (§2.2), not a separate game terminal skin
- [ ] Company banner / Master Agent / orbit rings present in relevant frames
- [ ] Multi-room amenities + idle life visible in populated frames

**Coverage**

- [ ] Empty/onboarding, populated, selection, human chat, agent-agent peek
- [ ] Assignment storyboard, zone labels, all three zoom tiers
- [ ] Toasts, loading/reconnect/offline, approvals morning glance
- [ ] Light + evening art passes
- [ ] Mobile overview-only
- [ ] Edit mode + banner editor
- [ ] Component inventory sheet
- [ ] Reduced-motion notes present

**Clarity & restraint**

- [ ] First glance answers: who’s busy, who’s idle, what it costs, where to click
- [ ] No visual clutter competing with the office (max 3 toasts, limited coach marks)
- [ ] Empty/error copy is calm and actionable

**Differentiation vs generic office-game UIs**

- [ ] Feels like Orbicrew (premium SaaS + charming office), not a reskinned pixel MMO HUD
- [ ] Signature orbit-ring moment is clear and brand-true

---

## 11. Prompt → product rule map (internal QA)

| Stitch pack | Product rules validated |
|---|---|
| Prompts 1–3, 9–10 | Rooms (§3.4), zoom tiers (§3.8) |
| Prompts 4, 8, 16 | Delegation + idle FSM + orbit rings (§3.5–3.7) |
| Prompts 5–7 | Selection + Chat GUI alignment (§2.2, §3.5) |
| Prompts 11, 17 | Live activity + approvals presence (§2.4) |
| Prompt 12 | Loading / reconnect / offline / budget voice (§2.3) |
| Prompt 13 | Mobile Overview-only (§3.8) |
| Prompt 14 | Dual art passes (§3.10) |
| Prompt 15 | Decor + banner + safe scripting messaging (§3.9) |
| Prompt 18 | DOM vs canvas split (§3.3) |

Build order for validating sets: §3.12.

---

## 12. Research appendix — Agent Town vs Orbicrew Orbit

*(Background only — do not paste into Stitch. Informs why prompts choose certain differentiators.)*

### 12.1 What Agent Town does well (keep the learning, not the skin)

From public Agent Town references (spatial-office / open-source demos of the same class):

- **Spatial task visibility:** assign → watch `queued → returning → sending → running → done/failed` with bubbles — the mechanic that makes a spatial view worth having.
- **World + HUD split:** canvas for the room; product UI for chat, seats, connection, tasks.
- **Worker autonomy:** idle wander to POIs (printer, sofa, coffee, whiteboard) with light emotes; busy workers queue; return-to-seat before real work.
- **Readable runtime chrome:** top agent pills with status dots, connection states, streaming chat + collapsible tool rows.
- **Camera craft:** follow/drag, wheel zoom.
- **Onboarding spotlight:** coach mark when setup is incomplete.

### 12.2 Gaps vs Orbicrew needs

| Gap in Agent Town–class UIs | Orbicrew requirement |
|---|---|
| Player “boss” avatar + proximity “Press E” menus | Camera-only observer; click/tap select |
| Pixel RPG HUD palette + bit-font chrome | Deep Violet product design system |
| Mostly one office map / seat model | Multi-room catalog + amenity idle-life as first-class |
| No company branding in-world | Lobby company banner |
| No cost transparency in-world | Desk cost chips + spend honesty |
| Gateway-centric connection panels | Multi-tenant SaaS, Standard↔Orbit parity |
| No approvals / overnight morning UX | Approvals badges + morning glance |
| No Master-Agent visual delegation metaphor | Orbit rings |
| Limited dark-mode art intentionality | Dual art passes |
| Mobile not a first-class glance mode | Overview-only mobile |
| No sandboxed cosmetic scripting / decor editor productization | Curated decor + safe UI-only scripting |

Agent Town-class projects are a **reference for the spatial work-loop**, not a category match. Orbicrew is broader (dashboard, channels, cost, isolation, approvals, branding).

### 12.3 How this Stitch pack differentiates

1. **Observer camera + docked product chat** replaces walk-up RPG interaction while preserving “watch work happen in space.”
2. **Brand-token world + calm SaaS chrome** beats generic pixel HUD for B2B trust without losing charm.
3. **Idle-life rooms + Master orbit rings + cost + approvals + banner** create moments generic office-game UIs never designed for.
4. **Zoom tiers and mobile overview** make “see the whole crew” scale with roster size.
5. **Explicit reduced-motion and accessibility pairing** (icon+label status) are product requirements, not afterthoughts.

---

## 13. Optional paste add-ons (short)

### 13.1 Negative prompt (append to any generation)

<<<STITCH NEGATIVE CONSTRAINTS
Do not include: walkable player character, RPG “Press E” prompts, WASD instructions, HP/mana bars, pixel-font-only HUDs as the primary chrome, neon cyberpunk grids, robot mascots, neural network illustrations, purple decorative gradient washes, dense dashboard widgets covering the office, more than three simultaneous toasts, speech bubbles with lorem-ipsum RPG quest text, or a separate game color system disconnected from Orbicrew tokens.
STITCH>>>

### 13.2 Consistency reminder (append when iterating)

<<<STITCH CONSISTENCY REMINDER
Keep agent silhouettes, furniture scale, badge anatomy, dock width (~400px), and minimap placement identical across frames. Only change the state being demonstrated. Light mode unless the prompt is the evening pass. Status language must stay: queued, running, awaiting approval, done, failed.
STITCH>>>
