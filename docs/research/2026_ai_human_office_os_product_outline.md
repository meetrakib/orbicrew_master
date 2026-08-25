# AI + Human Office OS — Product Outline & Requirements

Status: fresh strategy/outline doc, evaluating a broader feature set against the direction set in `2026_ai_native_company_os_opportunity.md`. Not a replacement for `01_AI_Office_Platform_Requirements.md` — this is the next layer down, once the "AI-native company OS" direction (accepted in the companion doc) needs an actual feature list, a target-customer call, and a build/don't-build line.

Date: 2026-08-08

---

## 1. The core question this doc answers

You listed a wide feature set: agent building, MCP/skills/rules, client handling, PM with humans+agents, ERP (ledger/expenses/salaries/inventory), meetings/notetaking, chat, an agents+human office, secrets storage, content/social, personal and product branding/research, full product development (research → UI/UX → engineering), and eventually open source + marketplace.

This doc sorts that list into three buckets — **build natively, give agents the capability, or integrate with an existing tool** — makes the target-customer call explicit, and lays out the resulting product structure.

---

## 2. Who this is for — solopreneur only, or solo + agency?

**Both, as one continuous segment — not two products.** The reason to keep them unified rather than picking one:

- The dominant 2026 pattern documented in the companion doc is a founder growing from "just me" to "10 workers, 9 of them agents" *without ever formally hiring a traditional team* — that transition happens **inside** the same operator, not as a handoff to a different kind of company. A product that only serves the solopreneur stage loses its best customers exactly when they'd pay the most (as they scale); a product that only serves agencies has no funnel — nobody starts as a 10-person agency.
- Every feature in your list (agent roster, PM board, client handling, ledger) reads identically whether the "team" is one person wearing every hat or a 12-person studio with a mix of humans and agents — the data model (workers = human or agent, projects, tasks, clients) doesn't change shape as headcount grows, only the number of rows does.

**The boundary that *does* matter — and the one real narrowing this doc recommends** — is by **business type, not headcount**: target **digital/knowledge-work businesses** (software studios, digital marketing/growth agencies, design studios, content/creator businesses, consulting/research shops, and solo operators doing any mix of the above) and explicitly **not** physical-goods businesses (retail, e-commerce fulfillment, manufacturing). That line is what determines whether "inventory management" belongs in the product at all (it doesn't — see Section 4).

**Restated target:** solo operators and small teams (roughly 1-20 people, mixed human+agent), running any digital-first business — this covers "software company, digital marketing company, design studio, and so on" exactly as you described, as one coherent segment, because they all share the same underlying shape (project/client-based delivery, no physical inventory, knowledge work) even though the *work itself* differs. Enterprise stays out of scope for now, per the existing marketing plan and Section 6 below.

---

## 3. Sorting the feature list: build, agent-capability, or integrate

The single most important discipline for a product this ambitious is **not defaulting to "build" for everything on the list.** Three buckets, applied to every item:

- **Build natively** — this is genuinely differentiated, or it's cheap because it's just a new view over data the platform already owns (tasks, workers, projects).
- **Agent capability, not a subsystem** — the feature already exists the moment an agent has the right tool; it needs a tool integration, not a new UI or data model.
- **Integrate, don't rebuild** — a mature, commoditized, or regulated category where a best-in-class existing tool already solves it well; owning it ourselves adds liability or distraction without real differentiation (the same principle already applied to accounting/payroll/CMS in the companion doc).

| Your item | Bucket | Verdict |
|---|---|---|
| Train and build their own agents | **Build natively** | Core to the product — already the existing agent-config model (`Memory/Skills/Soul/Setting`), extend with feedback-based training as already planned |
| MCP, skills, rules | **Build natively (as the extensibility spine)** | Not previously in the plan explicitly — see Section 4.1, this is a genuinely important addition |
| Client handling | **Build natively, lightweight** | A "Client Hub," not a Salesforce competitor — see Section 4.2 |
| PM with clients, agents, humans | **Build natively** | Already recommended in the companion doc §13.2/13.6 — central, not new |
| ERP: ledger, expenses, salaries | **Build natively** (records only, never custody/tax-filing) | Per companion doc §13.1 |
| ERP: inventory management | **Cut** | Wrong customer — see Section 4.3 |
| Meetings, AI notetaker | **Build natively** | Leverages Whisper/STT already in the stack — see Section 4.4 |
| Chats | **Build narrowly (task/project comments), integrate for full messaging** | Don't rebuild Slack — see Section 4.5 |
| Agents + human office | **Build natively** | Orbit View extension, already recommended |
| Low cost but best output | Not a feature — existing architecture | The router/Auto-Manual model selection already does this |
| Secrets variable storage | **Build natively** | Per companion doc §13.7 — genuinely differentiating |
| Content and social media | **Agent capability + integrate publishing** | Don't build a Buffer competitor — give the Marketing Agent tool access to Buffer/Meta/LinkedIn APIs |
| Personal branding and research | **Agent capability, already covered** | Existing Research + Marketing/Brand agents — no new subsystem |
| Product branding and research | **Agent capability, already covered** | Same as above |
| Product dev: research, UI/UX, engineering | **Agent capability, already covered** | Existing Coding/Design/Research agent roster (`14_universal_task_coverage.md`) |
| Open source | **Build a thin slice, selectively** | See Section 4.6 |
| Marketplace | **Build later, Phase 4+** | Already recommended in companion doc §13.4 — confirmed, not accelerated |

---

## 4. What to add or reframe beyond your list

### 4.1 MCP, Skills, and Rules — the extensibility spine (new, and important)

This is the single highest-leverage addition not yet in any existing doc. **Adopt the Model Context Protocol (MCP) as a first-class client capability inside the agent runtime**, alongside — not instead of — the existing first-party tool framework:

- **Why it matters:** MCP is an open, growing standard for connecting agents to tools and data (Slack, GitHub, Google Drive, Notion, databases, and a fast-expanding server ecosystem, including ones vendors publish themselves). Supporting it as a client means an Orbicrew agent can reach a huge and growing integration surface **without Orbicrew first-party-building every one of those integrations**, which materially changes the "who builds the tools" math from the companion doc §13.10 — MCP servers become a fourth tier (built by the wider ecosystem, not Orbicrew and not the tenant) sitting alongside first-party tools, tenant-scoped custom tools, and the future curated marketplace.
- **Skills:** map onto the existing agent-config "Soul/Skills" concept (`03_system_design.md` §10) — a packaged bundle of instructions + reference material an agent can be equipped with, e.g., "Brand Voice Skill," "SOC2 Audit Prep Skill." Treat as a formalized, shareable version of what the config model already implies, not a new subsystem.
- **Rules:** name and expose what the existing action-whitelist/budget-cap/ask-vs-proceed settings already do (`01_AI_Office_Platform_Requirements.md` §9) as a plain-language **Rules Engine** in the UI — e.g., "never publish without approval," "always loop in a human for anything over $500," "CC me on client emails." This is mostly a UX/framing change over existing planned mechanics, not new architecture.

### 4.2 Client Hub (lightweight, not a CRM competitor)

A simple contacts/client-profile layer (client name, contacts, active projects, notes, linked invoices) sitting underneath the Client Layer (project invites, white-label, trust page). **Deliberately not a full CRM** (no lead scoring, no sales pipeline automation, no marketing-automation sequencing) — those are Salesforce/HubSpot territory and out of scope. Just enough structure to answer "who is this client, what are we doing for them, what have we billed them."

### 4.3 Cut: inventory management

Physical inventory/supply-chain management is the one item on your list that doesn't fit the target customer at all — it belongs to retail/e-commerce/manufacturing businesses, not software studios, marketing agencies, or design shops. Including it pulls the roadmap toward general SME ERP (NetSuite territory), a much larger and differently-shaped market. **Recommend cutting it entirely.** If there's a real underlying need, it's closer to "**software/subscription tracking**" (what SaaS tools and licenses is the agency itself paying for) — a much smaller, genuinely relevant feature, worth folding into the expense-tracking module (4.1 of the companion doc's ERP section) rather than standing up a real inventory system.

### 4.4 AI meeting notetaker

Build natively — this is a strong fit because it reuses infrastructure already planned rather than starting fresh: Whisper STT is already the planned transcription backbone for the voice pipeline (`01_AI_Office_Platform_Requirements.md` §7.3-7.4). Scope it precisely:

- A bot joins a call (via Zoom/Google Meet/Microsoft Teams bot APIs — integrate the *joining* mechanism, don't build a competing video conferencing product).
- Whisper transcribes; an LLM summarizes and extracts action items.
- Action items become real tasks on the PM board, assignable to a human or an agent — this is what makes it more than a transcription tool, it closes the loop into the same roster/task system as everything else.

### 4.5 Chat — scoped narrowly

Two different things were bundled under "chats" and they get different answers:

- **Task/project-level threaded comments** (like Linear or Asana comments, or Notion inline comments) — **build natively**, cheap, and high-value, since it's just a new view over tasks/projects that already exist.
- **A general team messaging app** (a Slack replacement) — **don't build.** This is a mature, commoditized category; instead, deepen the already-planned Slack integration (`01_AI_Office_Platform_Requirements.md` §9) so agents and tasks are reachable from a team's existing Slack, rather than asking them to adopt a new chat app.

### 4.6 Open source — a thin, deliberate slice

"Some open source" is worth doing, but scope it as a **trust/ecosystem play, not a licensing strategy for the core product**: open-source a thin SDK or the agent-config schema/format (how an agent's Memory/Skills/Soul/Setting is defined) and maybe an MCP client library — the kind of thing that builds developer goodwill and lets technical users inspect/port their own agent configs, the same pattern Supabase or Vercel use (open SDK, proprietary hosted platform). **Keep the orchestration engine, billing, multi-tenant infrastructure, and the router/cost-control logic closed** — that's the actual business, per the existing "this is our core IP" framing in `03_system_design.md` §12.

### 4.7 Two more worth adding, not on your list

- **Time tracking**, tied to the Worker Ledger — since agencies frequently bill by time and the ledger's whole value is accurate cost/revenue attribution, a lightweight timer/time-entry per task (for humans; automatic for agents, since their execution time and cost are already logged) closes a real gap the ledger would otherwise have.
- **Contracts/e-signature** — not built, but worth a named integration (DocuSign/PandaDoc API) reachable from the Client Hub and HR-lite modules, since client agreements and contractor onboarding both need it and it's a small, well-solved integration, not a build.

---

## 5. Product outline — the resulting structure

Eight pillars. Each is either a core native subsystem, or explicitly an integration/agent-capability layer — no pillar is "build everything under this heading from scratch."

```
A. Office Core         — human+agent roster, task/project engine, PM board, Orbit View, Digest
B. Agent Platform       — agent builder/training, MCP client, Skills, Rules engine, model router, secrets vault, tenant-scoped custom tools
C. Client Layer         — Client Hub, client-guest project access, trust page, white-label/custom domain
D. Business Ops (ERP-lite) — Worker Ledger, expenses (incl. subscription tracking), invoicing (Stripe Connect), payroll tracking + provider integration, HR-lite, time tracking
E. Comms & Meetings     — task/project comments, AI meeting notetaker, Digest delivery channels, Telegram/Discord/WhatsApp/Slack adapters
F. Work Capability       — the existing full agent roster (coding, design, marketing, research, writing, video, etc.) — no new subsystem, this is what B's agents actually do
G. Branding & Customization — UI theming, white-label, custom domain (shared surface with C)
H. Ecosystem             — thin open-source SDK/schema, curated marketplace (Phase 4+)
```

### 5.1 Functional requirements by pillar (outline level)

**A. Office Core**
- Every worker (human or agent) is a row in one roster; `assignee_type: human | agent` on every task.
- Task/project state machine already specified (`03_system_design.md` §4) extended with project-level client visibility (see C).
- PM board (Kanban/list) as the default view; Orbit View as the spatial alternative over the same data — never two data models.
- Digest mechanism (on-demand, per `03_system_design.md` §7) extended to summarize both human and agent activity in one feed.

**B. Agent Platform**
- Agent creation/edit/delete UI (existing `01_AI_Office_Platform_Requirements.md` §5.1), extended with an MCP-server connection panel (add a server by URL/config, scope which agents can use it).
- Skills as shareable, named bundles attached to an agent's Soul config.
- Rules engine surfaced as plain-language toggles mapped to the existing action-whitelist/budget/ask-vs-proceed settings.
- Secrets vault (companion doc §13.7): encrypted storage, reference-only resolution at tool-execution time, never exposed to any LLM request payload.
- Self-serve custom tools, tenant-scoped only (companion doc §13.10) — the middle tier between first-party tools and the future marketplace.

**C. Client Layer**
- Client Hub: client profile, contacts, linked projects, linked invoices.
- Client-guest role: project-scoped access to the PM board (who's doing what, human or agent), no billing/roster/ledger visibility.
- Opt-in shareable trust page (companion doc §9.4) as a lighter-weight alternative to a full invite, for clients who shouldn't get a login.
- White-label (logo, remove platform branding) + custom domain, gated to Growth/Studio.

**D. Business Ops (ERP-lite)**
- Worker Ledger: cost and (once invoicing is connected) revenue, per worker, per project, per period.
- Expense/subscription tracking: manual entry + optional read-only bank/card feed (Plaid); no money movement.
- Invoicing: document generation and status tracking natively; actual payment collection via Stripe Invoicing/Connect only.
- Payroll: tracking natively (what was paid, to whom, when); tax withholding/filing via a payroll provider integration (Gusto/Deel/Rippling), never built in-house.
- HR-lite: roster, roles, contract storage (via e-signature integration), onboarding task templates — record-keeping only, no labor-law-compliance automation.
- Time tracking: manual timer/entry for humans, automatic from execution logs for agents, feeding the Worker Ledger.

**E. Comms & Meetings**
- Task/project threaded comments.
- Meeting notetaker: bot joins via Zoom/Meet/Teams API, Whisper transcription, LLM summary, action items become real tasks.
- Digest delivery over whichever channel adapter exists (Telegram/Discord/WhatsApp/Slack), per the existing "one backend, many faces" principle.

**F. Work Capability**
- No new requirements beyond what's already specified in `14_universal_task_coverage.md` — this pillar is "what the agents in B actually do," not a separate build.

**G. Branding & Customization**
- Tenant-level theme (logo, color accent within the existing design system, not arbitrary CSS) — enough to feel "personally branded" without turning into a page-builder.
- Shares the white-label/custom-domain surface with C.

**H. Ecosystem**
- Thin open-source SDK/agent-config schema (Section 4.6).
- Curated marketplace, Phase 4+, Stripe Connect for third-party developer payouts (companion doc §13.4).

---

## 6. What stays explicitly out of scope

Carried forward and reaffirmed, not new decisions:

- Physical inventory/supply-chain management (Section 4.3).
- A CMS/website builder — agents get tool access to Webflow/WordPress/Vercel/Framer instead (companion doc §13.8).
- Owning payment custody, payroll tax calculation/filing, or claiming to be a tax-ready system of record (companion doc §13.1).
- A general-purpose chat app to replace Slack (Section 4.5).
- Owning compute/hosting as a cloud provider (companion doc §13.9) — integrate Railway/Fly.io instead, and only opportunistically.
- Enterprise as a go-to-market target for now (companion doc §13.12) — this scope quietly de-risks a future enterprise motion but doesn't change today's answer.
- Open-sourcing the orchestration engine, router, or billing logic (Section 4.6).

---

## 7. How to build it — architecture principle for this expanded scope

One rule carries almost the entire feature list above without the roadmap exploding: **every new item is either (1) a new view over the existing worker/task/project data model, (2) a new tool an existing agent can call, or (3) an integration with a best-in-class external system.** Very little of what you listed actually requires a new data model or a new subsystem — most of it is MCP/tool coverage, a new UI surface over data already being collected (Worker Ledger, PM board, Client Hub are all reads over tasks+cost+projects), or a connector to something like Stripe/Gusto/Zoom that already solves the regulated or commoditized part well. Treat any proposed feature that doesn't fit one of those three shapes as a signal to slow down and re-examine it before building.

---

## 8. Suggested sequencing (maps onto the existing phased plan)

- **Phase 2 (multi-tenancy & pricing, as already planned):** Client Hub (lightweight), task/project comments, Rules engine framing over existing settings.
- **Phase 3 (differentiation & scale, as already planned):** MCP client support, Skills as named bundles, secrets vault, tenant-scoped custom tools, PM board with human+agent assignment, client-guest invites, white-label/custom domain, Worker Ledger v1, meeting notetaker, HR-lite, time tracking, invoicing (Stripe Connect).
- **Phase 3-4:** payroll-provider integration, expense/subscription tracking with bank feeds, contracts/e-signature integration, thin open-source SDK.
- **Phase 4+ (unchanged):** curated marketplace.

---

## 9. Moat map — what's actually hard to copy, vs. what should just be a connector

A feature list this wide looks copyable unless it's explicit about *where* the defensibility actually lives. Not everything in Section 5 deserves the same investment — some of it should be built deep and native precisely because it's hard to replicate, and the rest should be built as thin, fast, well-designed connections to tools that already exist, both to move faster and because trying to out-build mature categories (accounting, CMS, video conferencing, social scheduling) is a losing, distracting fight regardless of engineering effort.

### 9.1 The three places real defensibility lives

| Moat | What it is | Why it's hard to copy |
|---|---|---|
| **Unified worker + ledger data model** | Humans and agents as one roster; the Worker Ledger attributing cost and revenue to each, with (opt-in, anonymized) cross-tenant benchmarking over time | Not a feature to screenshot and clone — it's a *data model* decision made from day one, plus a data asset that compounds the longer tenants use it. A competitor can copy the UI in a week; they can't copy two years of attribution data or retrofit "human and agent are the same kind of row" into a product that was built agent-only |
| **Orchestration depth** | The router/budget guard, hallucination-isolation architecture (`11_hallucination_isolation_and_scaling.md`), and the secrets vault's "never enters an LLM request payload" execution model | Easy to *claim*, hard to *get right*. Competitors can add a secrets field or a budget cap in a sprint; getting the reliability, isolation, and safety properties actually correct — and keeping them correct as scale and concurrency grow — is the multi-quarter engineering investment most competitors in this research visibly haven't made |
| **Orbit View (human+agent) + client lock-in** | The owned spatial UI extended to human avatars, and client-guest access to shared project boards | Orbit View is original IP requiring real game-engine build effort to replicate credibly, not a fork of anything. The client-guest board is a *business-model* moat, not technical — once an agency's own clients are used to checking status here, switching costs the agency their client relationship, not just a subscription |

Everything else on the feature list — MCP support itself, a meeting notetaker, task comments, invoicing documents, social scheduling, branding/theming — is **table stakes to build well, not a place to seek differentiation.** MCP support in particular is worth naming explicitly: adding an MCP client is *not* a moat (it's an open standard, every competitor can and likely will add one) — the moat is what Orbicrew's orchestration layer does *around* an MCP tool call (budget check, approval gate, audit log, secrets injection), which a bare MCP client elsewhere doesn't get for free.

### 9.2 "Connect your own tools" — the architecture that follows from this

Directly implementing your principle — don't force a migration off tools someone already uses and likes; be the hub they run everything *through*, not a forced replacement for everything they already have. Three connection tiers, matched to how core each integration is:

1. **Native, deep integrations** — reserved for the handful of connectors that are load-bearing for the core differentiated pillars: Stripe (Invoicing/Connect, underpins the Worker Ledger and the "we never touch client money" boundary), a payroll provider (Gusto/Deel), and video-call platforms for the meeting notetaker. These get first-party-built, maintained connectors because they're structurally central, not because they're hard to reach otherwise.
2. **MCP connections** — the default path for everything else with an existing MCP server (Slack, GitHub, Notion, Google Workspace, and a fast-growing list of others, including ones a competitor tool itself might publish). A tenant with an existing content-scheduling tool, CRM, or CMS they like connects it via MCP rather than being asked to switch. This is what makes "already have tools in the market? connect them" true in practice, at low ongoing engineering cost to Orbicrew.
3. **Generic/custom connectors** — the self-serve, tenant-scoped custom-tool capability from Section 4.1/companion doc §13.10, for anything with an API but no MCP server yet — a technical customer wires it up for their own tenant, contained blast radius, no marketplace review needed.

This is the same "no lock-in" principle the platform already applies to models (BYO-API, three patterns, all through one gateway) extended to tools generally — a tenant is never stuck because Orbicrew doesn't natively support their preferred invoicing tool, social scheduler, or CMS; they connect it. What Orbicrew actually sells is the roster, the ledger, the orchestration, and the trust layer that everything — native or connected — flows through, which is exactly where Section 9.1's real moat lives.

---

## 10. Positioning: what to call it

Not "fully autonomous" — it contradicts the product's own design (approval gates, Rules engine, client-guest boards, the human+agent roster all exist because humans stay in the loop) and is exactly the "AI handles everything" illusion the fluxio.dev piece names as a fatal pitfall. Not "AI-native office suite" as the headline either — "AI-native ERP" is already a crowded, named 2026 category (Campfire, DualEntry, Rillet, Flow ERP); leading with it blends in rather than standing apart, though it's a fine supporting architecture claim.

**Recommended category claim: "human + AI collaboration office suite."** It's the accurate description of the real moat (Section 9.1), and it rides language the market is already being educated on for free — Asana just paid ~$75M and branded StackAI explicitly as "an operating system for human-agent teams," and Gartner/Forrester both call 2026 the multi-agent breakout year.

**Phase this in rather than switching all at once:** Phase 0-2 keeps the existing pitch (`17_marketing_plan.md` §3.1, agent-centric — correct for a buyer who often has no other humans yet); Phase 3+, once Team Members/PM board/client invites/ERP-lite ship, introduce the wider "human and AI, working together" frame as the umbrella, with "autonomous overnight execution" demoted to one specific, still-flagship capability rather than the whole product's promise.

---

## Sources

Builds directly on `2026_ai_native_company_os_opportunity.md` (same folder) — see that doc for full source citations on the underlying market/trend research (fluxio.dev solo-company trend piece, YC Fall 2026 RFS, Asana/StackAI acquisition, AI-native ERP and AI-employee competitor research).
