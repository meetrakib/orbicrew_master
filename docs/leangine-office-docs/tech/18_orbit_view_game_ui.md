# Orbit View — the living 2D office world

Part of the AI Employee Office Platform documentation set. See `00_INDEX.md` for the full document list. Companion to `10_ui_ux_guide.md` (which contains a short summary and a pointer to this file) and `09_brand_identity.md` (brand tokens used throughout).

**How to use this document:** this is written the same way `10_ui_ux_guide.md` is — descriptive prose detailed enough that a generator (Stitch or similar) working from this text alone can build the actual interface, not just a rough sketch. Copy `09_brand_identity.md` Section 6.7 (brand tokens) together with this file as one paste, since every color/type decision below refers back to those tokens rather than restating them.

**Stitch screen pack:** for copy-paste, production-ready Stitch prompts covering every Orbit View screen, HUD state, animation note, and acceptance checklist, use `19_orbit_view_stitch_prompts.md`. That file is the preferred Stitch input for game UI frames; this document remains the product/behavior source of truth the prompts must stay aligned with.

---

## 0. What this is, in one paragraph

Orbit View is a living, populated, multi-room 2D office — not a static diagram, not a decorative animation loop. Every agent is visibly *somewhere* and doing *something* at every moment: at their desk working a task, in the break room on a coffee run, in the lounge watching a tiny TV, at a shared table having a "meeting" with another agent. It should feel like looking into a real, small, slightly charming office from directly above — the same feeling the Agent Town reference screenshot gives at a glance — but built as a genuinely resource-light 2D system, in Orbicrew's own brand language, with rooms, amenities, and interactivity the reference project doesn't have. This document specifies rooms, agent behavior, decoration, branding, zoom/navigation, and the safe customization/scripting layer in enough detail to build from.

**The one-line test for every decision below:** does this make the office feel more like a real, lived-in place, without costing meaningfully more to render? If a feature fails either half of that test, it's out of scope for v1 — flagged as a "later" idea where relevant rather than silently dropped.

---

## 1. Why not just copy Agent Town

Agent Town (`github.com/geezerrrr/agent-town`) proved the core mechanic works — <cite index="68-1">approach a worker, assign a task through direct in-world interaction instead of forms, watch it move through queued, returning, sending, running, and done/failed states</cite> — and that mechanic is worth keeping (Section 4 below). But a direct reskin would be a worse product than something built for what Orbicrew actually is. Concretely, this spec differs in ways that matter:

- **No player-avatar.** Agent Town's model is a boss character the person walks around the map with. Orbicrew has no controllable character at all — the person is a camera, not a body in the scene. This is the single biggest structural difference: it removes an entire category of engineering (player movement, collision, pathing for a controllable character) the product doesn't need, and it reframes the whole thing around *watching agents live their day* rather than *the player walking somewhere to poke an NPC*.
- **A full day, not just a task loop.** The reference project's agents exist in two states: idle-at-desk or working-at-desk. This spec adds a genuine idle-time life — breaks, food, a lounge, socializing at a shared table — so the office reads as populated and alive even during a quiet hour with nothing queued, which is precisely the moment a static reference office looks most dead.
- **Rooms with real amenities, not just desks.** Break room, kitchen, washrooms, a lounge with TVs — Section 3's room catalog is deliberately broader than "one big desk floor," because a believable office has places besides desks.
- **Company identity, visibly present in the world.** A banner/branding system (Section 7) lets a person put their own company name and logo directly into the scene — the reference project has no equivalent, and it's a meaningful way to make each person's office feel like *their* office, not a template.
- **A safe customization/scripting layer.** Section 8 specifies a constrained, UI-only scripting surface so technically-inclined users (or their coding agent) can add custom decorative behavior — a fish tank with swimming fish, a wall clock that keeps real time, a custom desk toy — without ever touching backend logic or task execution. Nothing in the reference project offers user-authored behavior at all.
- **Brand-token visual language, not a separate game palette.** Every wall, floor, and furniture piece pulls from the same Deep Violet / gold / neutral token set as the rest of the product (`09_brand_identity.md` Section 6.7), not a saturated 16-bit game palette bolted on the side.

---

## 2. Rendering approach — why this stays light as it grows

**This is 2D, not 3D, by deliberate choice, not a limitation.** A 3D scene (real geometry, lighting, a 3D camera) costs meaningfully more to build, more to run on an average laptop, and more to maintain every time a new room or furniture piece is added — and delivers no more legibility than a well-executed 2D scene for what this feature actually needs to communicate (who's here, what are they doing). Flat 2D, viewed top-down, gets the same "living world" feeling at a fraction of the engineering and runtime cost.

**The resource-saving techniques, concretely, roughly in order of impact:**

1. **Canvas/WebGL rendering, not hundreds of animated DOM elements.** A single rendering surface for the entire spatial scene (floors, walls, furniture, agents), with all surrounding chrome (top bar, chat dock, minimap, decoration palette) as ordinary DOM/React — matching the split the reference project itself uses <cite index="68-1">between a Phaser game view and a React HUD</cite>. This is the single highest-leverage decision: once a scene has more than a handful of moving pieces (which a populated multi-room office with amenity-seeking agents will, constantly), canvas/WebGL batching is substantially cheaper than DOM animation.
2. **Texture atlases and sprite batching.** All floor tiles, walls, furniture, and character sprites packed into a small number of shared sheets, drawn through the renderer's batching so the GPU issues one draw call per sheet rather than one per object — standard practice, and what keeps a large, richly decorated office cheap regardless of how many objects a person places.
3. **Viewport culling.** Only what's within (or just outside) the visible camera area is drawn or animated each frame. A room the person has scrolled or zoomed away from costs nothing. This matters most for the fully-zoomed-out overview (Section 6) — at that level, detail isn't rendered small, it isn't rendered at all.
4. **Throttled simulation for off-screen agents, not frozen state.** An agent's idle-time wandering and animation only run at full fidelity when on-screen. Off-screen or zoomed-past agents still update their *state* (an idle agent's internal "what am I doing right now" simulation keeps ticking at a coarse interval, per Section 5's tick model) so that scrolling back to a room shows a believable, non-stale scene — but they don't animate every frame while unwatched, since nobody's spending render budget on movement nobody's watching.
5. **A capped, grid-based tile system**, not freeform pixel painting. Every room is built from a fixed-size tile grid using the curated decor catalog (Section 7). This is both what keeps every office looking considered rather than cluttered, and what makes the renderer's cost predictable regardless of how creative a person gets with layout — a large office is "more tiles from the same small sprite atlas," never "more unique unbounded assets."

**Pixel art is not mandatory.** The brand-safe direction is flat, geometric, low-detail 2D shapes built from brand tokens — executable either as genuine low-resolution pixel art (scaled with nearest-neighbor/integer scaling to stay crisp) or as flat vector-style shapes with the same silhouette simplicity, without changing anything about the rendering approach above. Recommend prototyping both directions on the same three or four furniture pieces before committing art direction — a real open decision, not a technical one.

**Concrete performance target:** Room View and Floor View (Section 6) hold a stable 60fps on a mid-range laptop with roughly 20 agents and 3-4 rooms in view. Overview Map holds a stable frame rate *regardless of total office size*, since its entire purpose is a cheap, always-responsive summary — if culling and throttling are implemented correctly, its cost is bounded by what's on-screen (small icons and status colors), not by total room/agent count. This is the property that makes "see the whole office at once" viable at any scale rather than degrading as a roster grows.

---

## 3. The room catalog

A believable office needs places besides desks. The following room types are the v1 curated set — each has a defined *purpose* (what agents actually do there) so the office never has decorative-only space:

| Room type | Purpose | Typical amenities |
|---|---|---|
| **Workroom** | Where desks live — the primary room type, one or more per office. Agents work here when assigned a task. | Desks, chairs, monitors, shelving, a shared conference/huddle point (Section 4) |
| **Break room** | Where idle agents go for a short, casual pause between tasks. | A couch or seating cluster, a coffee machine, a small table |
| **Kitchen** | Where idle agents go to "eat" — a believable, low-key idle activity distinct from the break room's social framing. | Counter, fridge, table and chairs |
| **Lounge** | The most social idle destination — where agents gather, sit, and "watch" a wall-mounted TV together. | Seating, one or two wall-mounted TVs (Section 5's ambient detail), a rug, shelving |
| **Washroom** | Present for realism and completeness — agents visit briefly and leave; no lingering, no social behavior here. | Minimal fixtures, kept small and simple |
| **Entrance/lobby** | The office's front door — where the company banner (Section 7) is most prominently displayed, and where the Master Agent's seat (Section 4) typically defaults. | Reception desk or greeting area, the banner, seating |
| **Meeting room** *(optional, user-buildable)* | An enclosed room with a table, for delegation moments (Section 4) to visually happen somewhere enclosed rather than only at an open conference point in a workroom. | Table, chairs, a whiteboard or screen |

**Room construction:** every room is a rectangular or L-shaped space on the tile grid, resized by dragging its edges within sensible min/max bounds, connected to adjacent rooms via doorways. A person builds their office by adding rooms one at a time and connecting them — a small office might be one workroom plus a break room; a larger one might have three workrooms, a kitchen, a lounge, washrooms, and a dedicated meeting room. There's no fixed template floor plan; the room catalog above is the palette, and layout is the person's to compose.

**Curated, not unlimited.** The set of room *types* is fixed (the table above) so every office reads as a recognizable, real kind of space rather than an arbitrary blank box with a label — but a person can have as many of each type as they want, sized and arranged however they like.

---

## 4. Core interaction model — delegation, not roaming

The reference project's actual task-assignment mechanic is worth replicating precisely because it's what makes a spatial view earn its keep over a plain list: <cite index="68-1">direct interaction rather than forms or dropdowns, with visible execution as tasks move through queued, returning, sending, running, and done or failed states</cite>. Orbicrew keeps this state machine, adapted to the no-player-avatar framing from Section 1:

1. **Idle:** an agent not currently assigned a task is living its idle-time life per Section 5 (wandering, at the break room, in the lounge, etc.) rather than sitting frozen at its desk — this is the key departure from a static idle pose, and the core of what makes the office feel alive.
2. **Summoned:** when the Master Agent (or the person, via the chat dock) assigns a task to a specialist agent, that agent — wherever it currently is in the office — walks to its own desk (or, if the delegation is framed as a hand-off, briefly to a conference point first, per the Master Agent behavior below) to begin work. Pathing is grid-based and simple (move along room tiles, through doorways) — no complex pathfinding is needed for an office-sized grid.
3. **Working:** at its desk, the agent shows its live status badge and a small text bubble mirroring the same status lines used in the Chat GUI ("searching the web…", "drafting…") — matching `10_ui_ux_guide.md` Section 8's queued/running/done/failed iconography exactly, never a separate game-only visual language for status.
4. **Done, back to idle:** on completion, the status badge briefly shows done or failed, then the agent leaves its desk and resumes its idle-time life (Section 5) — it does not sit idle-at-desk waiting for the next task; it goes and does something, exactly like a person would.

**The Master Agent as delegation point:** the Master Agent's desk (always in the entrance/lobby by default, visually distinguished with the same accent-bordered treatment as its roster card) is where most delegation moments visually originate. When the Master Agent hands a task to a specialist, an orbit ring (Section 5's ambient detail) animates from the Master Agent's position to the specialist's new desk — a direct visualization of the brand's own name and the clearest, most legible payoff of the spatial view as a feature.

**Interacting with an agent** (click on desktop, tap on tablet) opens the same underlying chat/task-assignment interface used elsewhere, docked to one side of the scene rather than a full-screen takeover, so the office stays visible and the person can watch the agent walking/working while they type.

---

## 5. Idle-time behavior — what makes the office feel alive

This is the section that most separates Orbicrew's office from a static reference: **an agent without a task is not idle-at-desk, it's living.** This is specified as a simple, cheap finite-state machine per agent — deliberately not a complex AI-driven simulation, since the entire point is a believable, low-cost illusion of life, not a genuine autonomous simulation:

**The idle-time state machine (per agent, runs only while idle):**

- **Wandering:** the default idle state — an agent walks at a slow, calm pace between rooms on a simple randomized-but-weighted schedule (more likely to visit the break room or kitchen than the washroom; more likely to head toward the lounge if another agent is already there, creating natural-looking small gatherings).
- **At an amenity:** once an agent reaches a break room, kitchen, or lounge, it pauses there for a randomized short duration, showing a small contextual animation or icon (a coffee cup, a food icon, a "watching TV" pose) — simple, readable, and reusing the same lightweight animation-and-icon approach as status badges elsewhere in the product, not elaborate character animation.
- **Socializing:** if two or more idle agents end up in the same amenity room, there's a chance they're shown near each other with a small speech-bubble icon (not actual dialogue text — just a light visual cue, like a "…" or a small chat icon) for a few seconds, implying a casual exchange without the cost or complexity of generating actual conversational content for a purely decorative moment.
- **Returning:** after its amenity visit, an agent either wanders to another amenity, heads back toward its desk area (hovering nearby, not sitting rigidly), or starts a new wandering cycle — until summoned per Section 4.

**Tick-based, not per-frame.** Per the "Office Simulator" pattern of calling every actor to act on a coarse interval rather than every frame, each agent's idle-state decision (where to wander next, how long to linger) is recalculated on a slow tick — on the order of every several seconds to a minute of real time, not every animation frame — which is both cheap to compute for a large roster and closer to how a real office actually moves (people don't relocate every second). Only the *walking animation* between decided points needs frame-rate smoothness; the *decision* of where to walk next is cheap and infrequent.

**Ambient world detail** (kept simple and optional, not required for v1 but specified for later):

- **Orbit rings** on delegation (Section 4) — the clearest brand-tied motion moment in the whole feature.
- **Wall-mounted TVs** in the lounge cycle through a small set of simple ambient animations (not real video — a looping abstract pattern or a simple "static/on" icon state) to sell the "agents are watching something" idea cheaply.
- **A wall clock** showing real time, reinforcing that this is a live, current view of the office rather than a static illustration.
- **Cost-aware desk state:** per the brand's cost-transparency positioning, a working agent's desk shows a small, unobtrusive running-cost figure next to its status badge — the same "never a vague abstraction, always a real number" principle applied spatially, and something the reference project has no equivalent of.

---

## 6. Zoom and navigation — seeing the whole office at once

Three navigation tiers, moved between continuously (scroll/pinch), not as discrete hard-cut views:

- **Room View (default):** camera framed on a single room at a scale where furniture, desks, and every agent's current activity are clearly readable — the main working view.
- **Floor View:** camera pulls back to show two or three connected rooms at once. Agents render smaller, but status badges and idle-activity icons stay a fixed screen size regardless of zoom, since legibility of "what's happening" is the entire point.
- **Overview Map (fully zoomed out):** the entire office — every room the person has built, connected by doorways/hallways — visible at once. Individual furniture detail drops out (this is where the culling from Section 2 matters most); agents render as small colored dots or minimal icons at their current position, colored by state (idle/working/done/failed, matching status colors used everywhere else in the product), so the whole office reads as a live, glanceable status overview — genuinely "see every employee in every room," not a detailed scene shrunk down.

**Controls:** scroll-wheel or pinch-to-zoom moves continuously between the three tiers; click-and-drag or two-finger-pan moves the camera; a small persistent minimap in a corner shows the full floor plan with the current viewport outlined, letting a person jump straight to any room by clicking its position on the minimap.

**Responsive behavior:** room editing and fine camera control are desktop/tablet-only (≥`md`, matching `10_ui_ux_guide.md`'s breakpoint conventions). Below that width, Orbit View is available as a simplified, view-only Overview Map (fixed, no competing pan/zoom gestures against page scroll) — enough to glance at overall office status from a phone, with a clear prompt to open a larger screen for room editing or detailed interaction.

---

## 7. Decoration and company branding

**Decoration palette:** a curated, brand-consistent set of placeable objects per room type (desks, chairs, monitors, plants, shelving, break-room and kitchen fixtures, lounge seating and TVs), placed via a simple drag-or-tap-to-place palette, all built from the fixed-tile system in Section 2. Deliberately curated rather than an open sandbox with hundreds of items, so every office looks considered regardless of who's decorating it — this is a genuine differentiating feature (a person's office is theirs to arrange), but within a designed system, not a free-for-all.

**Placing employees:** any agent from the roster can be placed at any desk in any workroom — purely visual/organizational (e.g., grouping a "Marketing" team into one room), with zero effect on the agent's actual configuration, which stays defined in Agent Detail per `10_ui_ux_guide.md` Section 13b.

**Company banner:** a person can set their company's name and, optionally, their logo (using the lockups defined in `09_brand_identity.md` Section 6.5 if it's the same brand, or an uploaded image for a white-label/agency use case) as a banner displayed prominently in the entrance/lobby room — a wall-mounted sign or a banner graphic above the reception area, visible from Room View and still recognizable at Floor View. This is a direct, simple way to make the office feel like *this specific person's* company rather than a generic template, and something the reference project has no equivalent of at all.

---

## 8. Safe customization — UI-only scripting, never backend

Some users (or their coding agent, working on their behalf) will want to add small custom decorative touches beyond the curated palette — a fish tank with animated fish, a custom desk toy, a personalized wall poster. This is supported through a **deliberately constrained, sandboxed, UI-only scripting layer**:

- **Scope, stated as a hard boundary:** custom scripting can only affect *decorative, cosmetic* elements within the Orbit View canvas — placing a custom sprite, defining a simple looping animation, adding a small interactive decoration (something a person can click for a cosmetic reaction, like a fish tank that ripples when tapped). It has **no access whatsoever** to task execution, agent configuration, account data, other tenants' data, or any backend system — this is enforced structurally (a sandboxed execution context with no network access and no access to any API beyond a small, whitelisted decoration-rendering interface), not just discouraged by convention, since this is a place where a person might paste code generated by an external tool and it must be safe by construction regardless of what that code contains.
- **What it's for:** letting a technically-inclined person (or, more commonly, their own coding agent asked to "add a small fish tank to my office") extend the visual personality of their space without needing a product update — a genuine, unique differentiator, since the reference project offers no user-authored content at all.
- **What it's not for, stated plainly in the product's own UI wherever this feature is surfaced:** this is not a way to script agent behavior, task logic, or anything that touches real work — the copy here should be as clear and calm as the rest of the brand voice, e.g., "Add custom decorations to your office. This only affects what you see here — it can't change how your agents work."

---

## 9. Dark/light mode

Since the scene is illustrated art rather than plain UI chrome, it doesn't invert the same way a dashboard does. Define two coherent art passes — a light-office palette and a dark/evening-office palette, both built from the same brand tokens (`09_brand_identity.md` Section 6.7) — rather than algorithmically inverting one asset set, so the office always looks intentionally designed in either mode, never like a color filter was applied after the fact. The lounge TVs and any ambient lighting details (Section 5) can lean into the evening palette specifically — a dim, cozy lounge glow at "night" is a small, cheap way to reinforce that dark mode is a deliberate art pass, not an inversion.

---

## 10. Technical implementation notes

*(Written for whoever builds this feature — a technical companion to the descriptive spec above, not a page spec for a UI generator.)*

### 10.1 Suggested stack

A 2D canvas/WebGL game framework with a companion DOM layer for chrome, following the same split the reference project itself uses <cite index="68-1">— a Phaser-based game view alongside a React HUD, communicating over a websocket connection to a backend</cite>. Phaser (or an equivalent lightweight 2D framework) is a reasonable starting point specifically because it auto-selects between canvas and WebGL rendering depending on browser support, and handles batching/texture-atlas mechanics largely out of the box rather than requiring them hand-rolled. Surrounding chrome (top bar, chat dock, minimap, decoration palette, banner editor) stays ordinary React/DOM — only the room scene itself needs to be canvas-rendered.

### 10.2 Data model — spatial and behavioral state is metadata, not source of truth

Room layout, furniture placement, agent desk assignment, and company banner content are purely presentational/organizational metadata layered on top of the same task/agent state used by the Standard Dashboard (`10_ui_ux_guide.md` Sections 6-13) — never a parallel or divergent data model. Concretely: a `room_layout` table (room shapes, doorway connections, room type), a `placed_object` table (object type, room, position, rotation — covers both curated decor and any custom-scripted decorations from Section 8), an `agent_placement` mapping (agent ID → home desk position), and a `company_banner` record (name text, optional logo/image reference). All of this is purely additive to the existing agent/task schema (`04_database_design.md`); Orbit View reads live task/status data from the exact same source the dashboard does, so the two views can never show conflicting information about what an agent is actually doing.

**Idle-state simulation is ephemeral, not persisted.** An agent's current wander position and idle activity (Section 5) is client-side, tick-based simulation state, not something written to the database on every movement — only meaningful state (task status, desk assignment) needs to persist; where an idle agent happens to be standing at any given moment does not, which keeps the realtime/backend surface area small regardless of how lively the idle simulation is.

### 10.3 Realtime sync

The same live-update connection already specified for the dashboard's activity feed drives Orbit View's task-related agent state — a task moving to "running" updates both the dashboard's activity feed and the corresponding desk's status badge from the same event, not two separate polling mechanisms. This keeps the dashboard and Orbit View trustworthy as genuinely interchangeable windows onto the same system, never one being a stale mirror of the other. Idle-time wandering (Section 5) does not need realtime sync at all, since it's ephemeral, client-side, and cosmetic per Section 10.2.

### 10.4 Build sequencing note

Given the scope here, recommend building this in layers rather than attempting the full spec at once: (1) a single workroom with desks and the core delegation state machine from Section 4, using the same task data the dashboard already has — this alone validates the rendering approach and proves the core "watch work happen spatially" value; (2) idle-time wandering and one amenity room (Section 5), which is what first makes the office feel alive rather than mechanical; (3) the remaining room types, decoration palette, and company banner (Sections 3, 7); (4) zoom tiers and the overview map (Section 6); (5) the sandboxed scripting layer (Section 8), last, since it's the most novel and highest-risk piece to get safe.
