# AI Employee Office Platform

**Project Outline & Requirements Analysis**

Prepared for: Senior Full-Stack + AI Engineer (Founder)

Document type: Working requirements & architecture reference

Status: Draft v2 — updated with researched cost/pricing data; companion technical documentation suite added

Date: August 2026

> **This is the top-level requirements document.** A full technical documentation set (system design, database design, API guide, security & scalability, phased development plan, DevOps & deployment, brand identity, UI/UX guide) lives alongside it in the `tech/` folder — start at `tech/00_INDEX.md` for the full map.

## 1. Executive Summary

This document consolidates our conversation into a single reference: what we're building, why, for whom, what open-source and paid components we plan to use, and the phased plan to get there. It is a living document — expect it to change as pilot customers give real feedback.

Core concept: a multi-agent "AI office" where a primary assistant agent (the "office manager") receives tasks from the founder/customer through any interface — a game-like animated office UI, a plain chat GUI, or messaging apps like Telegram/Discord/WhatsApp — and delegates work to specialized "employee" agents (coding, design, video, writing, marketing, research, etc.). Work can run unattended overnight. The whole thing is also a sellable multi-tenant product with two pricing models: managed platform (we host, we charge monthly with usage tiers) and BYO-API (customer brings their own model keys, we charge a smaller platform fee).

### 1.1 Guiding priorities, in order

- **Cost control first.** Even when using frontier models (Claude Opus/Fable tier, GPT-5.6 Terra, etc.), the system must keep actual spend low through routing, caching, and budgets — this is the #1 architectural constraint, not an afterthought.

- **Reliability over flash.** The avatar/office UI is a differentiator, not the product. Task completion correctness, memory, and approval gates are what make it sellable.

- **Multi-tenant from day one.** Because the end goal is to sell this, isolation between customers' data/agents/keys must be designed in from the start, not retrofitted.

- **Global, English-first.** The market, the brand, the marketing (see `17_marketing_plan.md`), and the default UI are all English-first, targeting agencies/freelancers/small teams anywhere English is a working business language (US, UK, Canada, Australia, Europe, and beyond) — not a Bangladesh-first or Bangla-first product. Multilingual chat — people can converse with their agents in any language they choose, Bangla included — remains a genuine, useful product capability, but it is a feature layered on a global-English product, not the market strategy or the brand identity. Full UI localization (translating the interface chrome itself into other languages) is an explicit future item, not a Phase 0-2 requirement.

- **Extensibility by the user.** Customers (and eventually the system itself) must be able to create, edit, and remove agents without engineering help.

## 2. Problem Statement & Goals

### 2.1 Personal goal (Phase 0 — you as first user)

Run your own freelance/agency work — branding, product ideas, marketing, client project delivery (including software builds for clients) — through a team of AI agents instead of doing everything yourself or hiring humans. Talk to it by voice or text, in whatever language is natural to you, hand off tasks, and get consolidated results, including work completed overnight while you sleep.

### 2.2 Commercial goal (Phase 2+ — sellable product)

Package the same system as a multi-tenant SaaS: customers get their own "office" of AI agents, can pick a visual style of interaction (animated office, plain chat, or chat apps like Telegram/Discord/WhatsApp), pay either a managed monthly subscription or a smaller platform fee if they bring their own model API keys, and can manage their own agent roster without needing a developer.

### 2.3 Target early customers

| **Segment**                                             | **Why they buy**                                                                      | **Priority**                                |
|---------------------------------------------------------|---------------------------------------------------------------------------------------|---------------------------------------------|
| Solo/small agencies & freelancers (your own peer group) — global, English-first | Multiply output without hiring; you already understand this buyer because you are one | High — first customers                      |
| Small agencies/creative-marketing/dev shops (2-15 people), same markets | Same core pain at slightly larger scale; buys later in the funnel but same channels/messaging | High — natural expansion of the same launch cohort |
| Solopreneurs / coaches / consultants                    | Want a virtual EA/marketing team without hiring                                       | Medium                                      |
| Non-technical founders                                  | Want to "manage a dev team" without knowing how to code                               | Medium — needs strong approval/guardrail UX |

See `17_marketing_plan.md` for the full breakdown of this segment's pain points, channels, positioning, and outreach playbook. Bangla-speaking SMEs are not a primary launch segment — multilingual chat remains a real product capability (see the "Global, English-first" priority above), but market strategy and campaign spend target global English-speaking markets, not a Bangladesh-first go-to-market.

## 3. System Overview

The system has five logical layers. Each is independently replaceable, which matters both for cost optimization and for avoiding lock-in to any single vendor or open-source project.

### 3.1 Layer map

| **Layer**                           | **Responsibility**                                                                                                               | **Primary tech candidates**                                                                 |
|-------------------------------------|----------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| 1. Interface layer                 | How the human talks to the system: animated office UI, plain chat GUI, voice, Telegram/Discord/WhatsApp bots                     | Next.js (App Router) + React + TypeScript, Phaser (office UI); Telegram/Discord bot SDKs; WhatsApp Business API |
| 2. Orchestration layer             | The "office manager" agent — receives tasks, plans, routes to specialist agents, tracks status, enforces budgets/approvals       | Python + FastAPI backend hosting LangGraph — supervisor + sub-agent graph. FastAPI is the confirmed backend framework: async-native, typed, WebSocket-capable for live task status, and the natural pairing for LangGraph, which is Python-native — bridging LangGraph from a non-Python backend would add latency/complexity for no benefit. |
| 3. Specialist agent layer          | Individually configured "employee" agents: coding, DevOps/deployment, system/DB design, UI/UX & graphic design, video editing, writing, marketing, research, proofreading, etc. — the agent-roster model generalizes to any task type; see `tech/14_universal_task_coverage.md` for the full mapping and for infrastructure/server-access agents specifically (a distinct, higher-risk category with its own safeguards) | LangGraph nodes / sub-agents, each with its own system prompt, tools, model tier            |
| 4. Model routing & execution layer | Chooses cheapest sufficient model per task, calls it through a self-hosted LiteLLM gateway (platform keys for managed tenants, any of three BYO patterns for BYO tenants), enforces caps/caching | Self-hosted LiteLLM (open source) + custom router service; see `tech/13_byo_provider_architecture.md` |
| 5. Data & memory layer             | Tasks, agent configs, conversation history, long-term memory/embeddings, files, billing/usage records                            | Postgres + pgvector, object storage (S3-compatible / R2), Redis for queues                  |

**Frontend/backend split of responsibility, confirmed:** Next.js (App Router) + React + TypeScript owns the UI, the auth session, and a thin proxy/BFF layer (API routes handling auth-token passthrough and webhook receipt); the Python + FastAPI backend owns orchestration, LangGraph execution, LiteLLM gateway calls, database access, and billing logic. The two communicate over a typed internal API (OpenAPI schema, with a generated TypeScript client on the frontend side) plus a WebSocket/SSE connection for live task-status streaming to the UI. Always use the latest stable release of Next.js, React, FastAPI, and every other library in this stack at time of build — don't pin to versions named in this document once real development starts, since these documents will age and package ecosystems move fast.

### 3.2 High-level flow

- User sends input (text, voice, image, video, live cam frame) via any connected interface.

- Interface layer normalizes input (STT for voice, frame sampling for video/cam) and forwards to the orchestration layer over a common internal protocol.

- Office-manager agent classifies the task, decides which specialist agent(s) are needed, and — critically — decides model tier per sub-task before any expensive call is made.

- Specialist agents execute (may use tools: code execution, web search/fetch, file generation, image/video generation APIs), reporting status through the task-state machine (queued → running → done/failed).

- For unattended/overnight runs, tasks execute inside budget and permission guardrails; anything outside the whitelist (e.g., sending an email, publishing a post, spending money) is queued for approval instead of executed.

- Office-manager agent consolidates results and delivers a single summary back through whichever interface the user is on — with links to real output artifacts (files, PRs, drafts), not just chat text.

## 4. Interface & UI Switcher Requirements

This is one of your explicit new requirements: the same backend must be reachable through multiple interchangeable interfaces, selectable per user preference, not locked to one UI.

### 4.1 Interface modes required

| **Mode**                | **Description**                                                                                      | **Notes**                                                                                                           |
|-------------------------|------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------|
| Animated office UI      | Phaser-style 2D pixel office; walk up to agents, assign tasks in-world, watch status bubbles         | Reference: Agent Town (open source, see Section 6). Fork or reimplement using same UI/Backend/Connector separation. |
| Standard agent/chat GUI | Conventional dashboard: agent list, task list/kanban, chat panel, file outputs, usage/cost dashboard | This should be the default, low-effort, professional mode — many B2B buyers will prefer this over a game UI.        |
| Telegram bot            | Chat + voice note support, task assignment and status via bot commands                               | Telegram Bot API is well-documented and free to use.                                                                |
| Discord bot             | Same capability inside a Discord server; good fit for teams already using Discord                    | Discord.js or discord.py; supports voice channel integration if wanted later.                                       |
| WhatsApp                | Widely used by SMEs and freelancers globally, not a single-region priority          | WhatsApp Business Platform (Cloud API) — has approval/verification steps to plan for.                               |
| Voice mode (any UI)     | Speak instead of typing, in English or Bangla, response spoken back                                  | Shared STT/TTS service used by all interfaces, not rebuilt per channel.                                             |

### 4.2 Key requirement: one backend, many faces

All interfaces must talk to the same orchestration/backend API — never duplicate agent logic per channel. This mirrors the Agent Town project's own roadmap direction (UI/backend/connector separation) and is the only way to keep multi-channel support maintainable as you add more channels later (e.g., Slack, email).

- **UI switcher:** user-level setting stored per account — "preferred interface" — but all channels remain simultaneously usable (e.g., someone can glance at the office UI on desktop and get a Telegram ping on mobile for the same task).

- **Session continuity:** a task started in one interface (e.g., typed in the chat GUI) must be visible/continuable from another (e.g., checked on Telegram) — this requires shared task/session state in the data layer, not per-channel local state.

## 5. Agent Management Requirements

### 5.1 User-created and user-removed agents

Non-negotiable for a sellable product: customers must be able to create, configure, and delete specialist agents themselves, without a developer.

- Agent creation UI: name, role/persona description, system prompt, model tier default, tool/permission whitelist, and — matching the reference UI you found — a "Memory / Skills / Soul / Setting" style config panel (long-term memory scope, tool access, personality, operating constraints).

- Agent templates library: pre-built starting points (coding agent, video editor agent, brand strategist agent, proofreader, etc.) that customers can clone and customize rather than starting blank.

- Agent lifecycle: enable/disable/delete, with task history retained even after an agent is removed (for audit/billing purposes).

- Per-agent budget and rate limit controls, set by the customer, enforced by the platform.

### 5.2 System-generated agents ("the system can build new agents")

This is an advanced, higher-risk feature — sequence it after the manual version is solid. Two viable approaches, not mutually exclusive:

- **Meta-agent (agent-builder agent):** a specialist agent whose job is to take a natural-language request ("I need someone who edits product photos and writes Instagram captions") and generate a new agent's config — system prompt, tool whitelist, suggested model tier — for the user to review and approve before activation. Human approval gate required at first; do not auto-activate system-generated agents without review.

- **Dynamic tool/skill composition:** rather than generating an entirely new freeform agent, the system assembles a new agent from existing vetted building blocks (approved tools + prompt templates) — safer, more predictable, easier to cost-control and support.

Recommendation: launch with the meta-agent producing a config that requires one-click human approval. Full autonomous agent creation without review is a support/safety risk not worth taking early.

## 6. Reference Implementation: Agent Town (Open Source)

You located the specific project behind the video you found. This section documents it precisely so we don't rebuild something that already exists.

| **Attribute**                       | **Detail**                                                                                                                                                                                                                        |
|-------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Project                             | Agent Town — geezerrrr/agent-town on GitHub                                                                                                                                                                                       |
| License                             | MIT (safe to fork/modify/commercialize with attribution per MIT terms)                                                                                                                                                            |
| Tech stack                          | Next.js 16, React 19, TypeScript, Phaser 3 (game engine), Tiled (map editor for tilesets/sprites)                                                                                                                                 |
| Agent runtime                       | OpenClaw, connected via a standalone connector process                                                                                                                                                                            |
| State management                    | React context + reducer + a typed event bus                                                                                                                                                                                       |
| Current architecture                | Game UI currently connects directly to an OpenClaw gateway via a WebSocket proxy                                                                                                                                                  |
| Target architecture (their roadmap) | Game UI -\> Agent Town Backend (WSS) \<- Connector (outbound WSS) \<- local OpenClaw Gateway. Connector keeps credentials local; backend can run in the cloud.                                                                    |
| Key UX pattern                      | Walk up to an agent "worker," press E, assign a task conversationally (no forms). Task moves through queued -\> returning -\> sending -\> running -\> done/failed, shown live via bubbles and a collapsible tool-call chat panel. |
| Capacity model                      | Seats: each seat is a Worker (tool slot executing tasks dispatched by a main agent) or an independent Agent (own workspace/model/memory). Seen in your video as "4/7 seat", "1/4 busy."                                           |
| Maturity                            | Early stage — 191 GitHub stars, 35 forks, v0.4.1 as of March 2026, explicitly described by the author as "today it's a local office" with a broader roadmap not yet complete.                                                     |
| **Voice support**                   | **Not confirmed in the public repo/README as of this writing.** The documented feature set is text-only. You've since stated you saw the video's creator speaking to the agent in both Bangla and English — that's a direct, credible observation this document can't independently verify without more information (e.g., the video's channel/creator name, a more recent commit, or a specific fork). Treat voice as **unconfirmed for this specific project** rather than confirmed-absent; if you can share the source, it's worth a direct follow-up search rather than resolving this from documentation alone. |
| **Multi-channel support**           | **Not present.** Task assignment happens only through the in-game UI, routed through OpenClaw. No Telegram/Discord/WhatsApp integration, no BYO-API support, no multi-tenant billing. |

**Bottom line: Agent Town gives you one thing — a working, appealing game-style office UI over text-based task assignment.** Voice, multi-channel support, cost routing, multi-tenant billing, project isolation, and hallucination/security safeguards are not part of it and remain entirely your own build regardless of whether you fork it. This directly informs the recommendation below: fork it for the visual layer specifically, and don't expect it to reduce scope on anything else in this document.

### 6.1 Decision: fork vs. reimplement

- **Recommended: fork as the visual/UI layer only.** Reuse the Phaser office scene, sprite/task-bubble UX, and their UI/backend/connector separation pattern. Replace "OpenClaw gateway" with our own orchestration backend (LangGraph-based) behind the same connector-style boundary. This gets you a working, appealing visual layer fast while keeping full control of the actual agent logic, cost routing, and multi-tenant billing — which the reference project does not attempt to solve.

- Do not depend on OpenClaw as our core agent runtime long-term — it's a third-party dependency for someone else's roadmap. Use it (or Agent Town's connector pattern) as inspiration for the protocol shape, not as our production agent engine.

- Build the "standard GUI" mode independently (plain dashboard) since it's lower effort and likely to be the default for B2B buyers who don't want a game.

## 7. Technical Architecture

### 7.1 Orchestration & agent framework

- **LangGraph** (Python, open source) — supervisor graph pattern: one "office manager" node routes to specialist agent nodes, with persistent state for checkpointing long-running/overnight tasks.

- Alternative considered: CrewAI or AutoGen/AG2 — simpler mental model, less fine-grained control over routing/state than LangGraph. LangGraph preferred given the complexity of your routing and budget-guard requirements.

### 7.2 Model routing & cost control (highest priority requirement)

This is the component that makes "use Opus/Fable/GPT-5.6-tier models but keep cost low" actually possible. It must be built as a first-class service, not an afterthought.

- **Classifier/router step:** every incoming task first passes through a cheap, fast model that decides task category, complexity, and the minimum model tier that can handle it. Most tasks should never reach the frontier tier.

- **Self-hosted LiteLLM gateway** (open source) sits between the Router and every model call, for both managed and BYO tenants — one OpenAI-compatible interface across 140+ providers, self-hosted so it doesn't trade one vendor lock-in for another. See `tech/13_byo_provider_architecture.md` for the full design.

- **BYO-API covers three distinct patterns**, not just "a provider key": (A) a direct platform key (Anthropic, OpenAI, Google, etc.), (B) a customer's own OpenRouter account, or (C) a self-hosted/custom-endpoint model (e.g., their own vLLM or Ollama deployment). All three route through the same LiteLLM gateway, so the platform doesn't need separate integration code per pattern. Full detail in `tech/13_byo_provider_architecture.md`.

- **Managed-tier tenants** use the platform's own pooled keys through the same LiteLLM gateway, with per-tenant virtual keys and budget caps enforced natively by LiteLLM as a second layer beneath our own Budget Guard.

- **Prompt caching:** reuse cached system prompts, brand-guideline documents, and repo/codebase context across calls instead of resending — large, direct cost saver on repeated-context tasks.

- **Context compression:** summarize long chat histories with a cheap model before they're passed to an expensive model.

- **Hard budget ceilings:** per-task and per-tenant token/dollar caps enforced by the router, not just monitored after the fact — essential for unattended overnight runs.

- **Batch APIs:** use provider batch endpoints (typically ~50% cheaper) for non-realtime bulk tasks (e.g., a queue of 50 proofreading jobs run overnight).

### 7.3 Multi-modal input handling

| **Input type**        | **Approach**                                                                                                                                           |
|-----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| Text (any language)   | Passed directly to the router/orchestrator — the person can type in whatever language they use day to day; no language-specific handling needed at this stage |
| Speech                | Whisper (open source, self-hosted or API) for STT — benchmark accuracy on whichever languages real customers actually speak; Deepgram/AssemblyAI as hosted alternatives to test against |
| Text-to-speech reply  | ElevenLabs (best quality, broad language coverage, paid) or Google Cloud TTS (cheaper, broad language coverage) — benchmark both on real customer languages before committing to a default |
| Image                 | Sent natively to vision-capable models (Claude, GPT-4o, Gemini all support image input)                                                                |
| Video / live cam      | Sample frames at an interval and treat as sequential image input, or use a model with native video understanding (e.g., Gemini) for richer cases       |

### 7.4 Realtime voice/avatar pipeline (for the office-avatar interface)

- **Pipecat** or

- **LiveKit Agents** (both open source) — purpose-built for mic-in -\> STT -\> LLM -\> TTS -\> speaker-out pipelines in-browser via WebRTC, with interruption handling.

- LiveKit Agents has an existing avatar plugin ecosystem (Tavus, Simli) for real-time lip-synced talking heads if a fully rendered face is wanted beyond the Live2D/pixel-sprite option.

- Live2D (via web SDK) or simple sprite-state swapping (idle/talking/listening) remains the cheapest option with no per-minute API cost, and is closer to the pixel-art aesthetic already validated by the Agent Town reference.

### 7.5 Data & memory layer

| **Store**       | **Purpose**                                                       | **Technology**                                                                                                                |
|-----------------|-------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| Relational data | Tenants, users, agents, tasks, sessions, billing/usage records    | Self-hosted Postgres — no managed-database vendor, per explicit no-vendor-lock-in requirement. See `tech/08_devops_and_deployment.md` Section 9 for the full setup. |
| Vector memory   | Long-term agent memory, semantic search over past work/context    | pgvector (co-located with self-hosted Postgres — simpler than a separate vector DB) or Qdrant if scale demands it             |
| File storage    | Generated documents, images, video outputs, uploads               | Self-hosted MinIO (S3-compatible, open source — note: AGPLv3, see `tech/16_open_source_licensing_compliance.md` Section 2.8 for the copyleft implications before committing) or Cloudflare R2 (zero egress fees, S3-API-compatible so not a hard lock-in)   |
| Task queue      | Async task distribution to specialist agents, overnight job queue | Self-hosted Redis + a queue library (e.g., BullMQ for Node, or Celery for Python), or Postgres-backed queue for simplicity at small scale |
| Audit log       | "What did my AI employee actually do today" — trust and debugging | Append-only table in Postgres; surfaced in UI as an activity feed                                                             |

### 7.6 Autonomous overnight execution — required safeguards

This is explicitly one of your goals: hand off a project and sleep while it's worked on. It's also the highest-risk feature from a cost and trust standpoint, so these are treated as requirements, not nice-to-haves.

- **Checkpointing:** long tasks persist progress (via LangGraph state persistence) so a crash/timeout doesn't lose hours of work.

- **Hard budget caps:** per-task and per-night ceilings; task pauses and flags for review if exceeded, rather than continuing to spend.

- **Action whitelist:** explicit allow-list per agent (e.g., "write code and run tests" = allowed unattended; "deploy to production" or "email a client" = requires morning approval).

- **Sandboxed execution:** any code the agents run executes in an isolated container, never against production systems or the user's real machine directly.

- **Retry-then-escalate:** a stuck cheap-tier agent retries once, then escalates to a stronger model rather than looping indefinitely and burning budget.

- **Morning summary:** one consolidated report on wake-up — what finished, what's blocked, what needs a decision — with links to real artifacts, not raw logs.

### 7.7 Tool architecture — how many tools, who builds them, marketplace or not

Two related but distinct questions: whether the system needs a separate visual automation/workflow builder (n8n/Zapier-style), and how the underlying tool catalog (web search, file access, code execution, SSH/infra access, and everything added later) should grow and who's allowed to build into it.

**On automation (n8n/LangGraph-style):** the platform already has this, structurally, and does not need a separate visual workflow-builder product bolted on top — LangGraph (Section 7.1) *is* the orchestration/automation engine, and the agent-roster model already gives customers automation outcomes (assign a task, an agent or chain of agents executes it, including multi-step work) without requiring them to hand-design a workflow DAG the way n8n requires. Building a full visual node-editor is out of scope through at least Phase 2 — it would be a multi-year distraction (n8n itself took years of dedicated, funded engineering) from the actual differentiators in Section 1.1's guiding priorities. If customers later want to define their own trigger → sequence-of-agent-actions flows, the right answer is a **constrained, templated** version (e.g., "when a new lead email arrives → research agent enriches it → writing agent drafts a reply → pause for approval") scoped as a Phase 3+ power-user feature, not a general-purpose automation builder.

**On tool count and ownership:** the realistic long-run tool catalog is large (30-80+ tools across coding, research, design, video, marketing, and infrastructure agents once `tech/14_universal_task_coverage.md`'s full scope is built out) but not unbounded, and the right approach is a **tool framework**, not a fixed list — adding tool #31 should be a routine engineering task, not an architecture change.

- **Phase 0-2: first-party only.** All tools — including the foundational, high-trust ones every agent depends on (web search/fetch, file read/write, code execution, basic CRUD integrations) — are built and controlled directly. No external tool authors in this window.
- **Phase 3+: curated/vetted third-party tools**, not an open marketplace — a reviewed, sign-off-required directory (similar in spirit to Anthropic's own MCP connector directory), which gets ecosystem leverage without an open free-for-all.
- **A fully open marketplace (any developer publishes, revenue share) is a Phase 4+/maybe-never decision**, gated on a mature per-tool capability-scoping/sandboxing model and a real legal/liability review, since this product is resold commercially (see `tech/16_open_source_licensing_compliance.md`).
- **Infrastructure/SSH/server-access tools specifically stay first-party-only indefinitely, marketplace or not.** This category is already called out in `tech/14_universal_task_coverage.md` as a distinct, higher-risk tier with its own safeguards (scoped credentials, dry-run-first, approval gates, rollback requirements) — that risk tier is exactly why this category is never opened to third-party tool authors, even once a curated marketplace exists for lower-risk tool types.

### 7.8 Building on open-source components we're not permitted to resell — clean-room rewrite option

A real, current question given the licensing constraints already documented in `tech/16_open_source_licensing_compliance.md` (e.g., LangGraph Platform's Elastic License, LiteLLM's Enterprise tier, MinIO's AGPL): where a specific open-source project's *code* can't legally be used/resold as-is, is it viable to study its plan/structure and build an independent implementation instead?

**Short answer: yes, but this is a real legal risk area, not a clean loophole, and needs case-by-case legal judgment, not a blanket policy.** A concrete, current (March 2026) example is directly relevant here: a widely-discussed dispute arose when an AI coding agent produced a from-scratch, differently-licensed rewrite of an existing LGPL-licensed Python library (`chardet`) — same public API, same package name, functionally a drop-in replacement — and the original maintainer disputed that this was a legitimate "clean room" implementation specifically *because* the rewriting agent had clear prior exposure to the original licensed code, which is the crux of clean-room legitimacy: a true clean-room implementation requires the people/systems doing the rewrite to have no exposure to the original source, working only from a behavioral/interface specification written by someone else who *did* have access. Reimplementing "from the same plan and structure" you got by reading the original code is a materially different, weaker legal position than a genuine clean-room process.

Practical guidance for this project:

- **Prefer avoidance over rewrite where a genuinely compatible alternative exists.** Most of this project's flagged licensing risks (per `tech/16_open_source_licensing_compliance.md`) already have a viable non-colliding alternative (e.g., self-hosted LiteLLM's open-source tier instead of the Enterprise tier, Cloudflare R2 instead of MinIO's AGPL where that copyleft is unacceptable) — that's a cleaner path than a rewrite in nearly every case already identified.
- **Where a rewrite is genuinely necessary,** structure it as an actual clean-room process: a written functional specification produced by someone without access to the original source, handed to a separate implementation effort (human or AI) that never reads the original code — not "read the code, then write something similar," which is the exact pattern currently under real legal dispute.
- **Get real legal review before shipping any rewrite of a specific existing project's functionality commercially** — this is already the existing recommendation in `tech/16_open_source_licensing_compliance.md`'s closing guidance, and it applies with extra weight to anything built via the rewrite path rather than the avoidance path, given how unsettled this specific question currently is.

## 8. Multi-Tenancy & Pricing Model

### 8.1 Two pricing tracks (as specified)

| **Track**        | **How it works**                                                           | **What we charge for**                                                                                               | **Key requirements**                                                                                                                           |
|------------------|----------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| Managed platform | We host and provide model access (via our own API keys/OpenRouter account) | Monthly subscription, tiered by usage/rate limits (e.g., number of tasks, tokens, agents, concurrent overnight jobs) | Usage metering per tenant; rate limiting per tier; our own cost-routing must protect our margin since we're paying the model bill              |
| BYO-API          | Customer supplies their own model access — a direct platform key, an OpenRouter account, or a self-hosted/custom endpoint (see `tech/13_byo_provider_architecture.md` for all three patterns) | Smaller flat platform fee for the orchestration/UI/infrastructure, not for model usage                               | Secure per-tenant key storage/isolation (e.g., encrypted at rest, never logged); customer's own spend is untouched by our routing margin logic; all three patterns route through a self-hosted LiteLLM gateway |

### 8.1.1 Concrete pricing (researched against real competitors)

Full research, competitor benchmarking, and cost derivation lives in `tech/02_cost_and_pricing.md`. Summary:

**Managed platform:**

| Tier | Price | Positioning |
|---|---|---|
| Starter | $19/mo | Undercuts Lindy Plus ($49.99) and Manus Pro ($20) while including more real usage via cost-routing |
| Growth | $59/mo | Undercuts Lindy Pro/Manus mid-tier |
| Studio | $149/mo | Undercuts Lindy Max ($199.99) |
| Enterprise | Custom (~$400–2,000+/mo) | Real negotiated infra/SLA cost, never a fixed self-serve number |

**BYO-API:**

| Tier | Price |
|---|---|
| BYO Starter | $9/mo |
| BYO Growth | $25/mo |
| BYO Studio | $59/mo |

Deliberately no fixed "$5,000/mo" self-serve tier — competitors offering that kind of number are typically doing custom enterprise deals, not shelf pricing, and shelf-pricing that high erodes trust with the SME/agency segment we're targeting first.

**Initial maintenance cost (Phase 0, you as sole user):** realistically **$20–120/mo all-in** (infra + LLM + voice), almost entirely LLM-usage-driven rather than infra-driven since infra alone can run under $10/mo (Hetzner CX22 ~$4.59/mo running self-hosted Postgres, Redis, and MinIO alongside the app — no managed-database vendor). See `tech/02_cost_and_pricing.md` Section 3 for the full breakdown by usage scenario.

### 8.1.2 Pay-as-you-go overage — a third billing primitive, not a third plan

A managed-tier subscriber who reaches their plan's included usage allowance should be able to keep working by paying for the overage, rather than hard-stopping mid-task or being forced into a full tier upgrade for a temporary spike. This is standard practice for usage-based products (API platforms, cloud hosting) and directly serves the target buyer, who has genuinely variable month-to-month workload as client demand fluctuates.

- **Opt-in by default, not automatic.** Overage billing is off until the customer explicitly enables it. When a plan's included allowance is reached with overage disabled, tasks pause with a clear in-product prompt to either enable metered overage or upgrade tiers — never a silent, unexpected charge. This matters specifically for the target buyer's trust in cost transparency, which is a core brand pillar (see `tech/09_brand_identity.md` Section 5) — a surprise bill would directly contradict the product's own positioning.
- **Metered, per-unit rate, shown plainly.** Overage is billed per task/per-token at a clear, published rate (not an opaque "credits" system, consistent with the cost-honest-language voice principle), visible on the Usage & Billing dashboard (`tech/10_ui_ux_guide.md` Section 11) in real time as it accrues.
- **BYO tenants don't need model-cost overage** (they're not paying us for model usage), but they still need the platform-resource limits described in 8.1.3 below, since orchestration/storage/compute cost us money regardless of billing track.

### 8.1.3 Resource limits — required on both tracks, for different reasons

Limits are not just a managed-tier cost-protection mechanism — they're also a platform-stability mechanism that applies regardless of who's paying for model usage.

| Limit type | Managed tier | BYO tier | Why it's needed either way |
|---|---|---|---|
| Concurrent/simultaneous jobs | Tiered (e.g., Starter: 2, Growth: 5, Studio: 15) | Tiered, generally higher ceilings since no model-cost risk to us | Protects platform stability (DB, queue, app-server load) regardless of who pays for tokens — a BYO tenant running hundreds of concurrent jobs still loads shared infrastructure |
| Token/dollar budget cap (per-task, per-tenant) | Hard cap, enforced by the router — protects our margin since we pay the model bill | Not applicable in the cost sense (their spend, their account) but a soft cap still recommended as a runaway-loop safety net for the customer's own protection | Managed: margin protection. BYO: customer protection against their own agent misconfiguration burning their own budget |
| Storage quota (generated files/artifacts) | Tiered, real infra cost | Tiered, real infra cost | Applies identically on both tracks — storage cost isn't a model-usage cost, it's ours either way |
| API/webhook rate limits | Tiered | Tiered | Abuse/DoS protection, applies identically on both tracks |
| Agent count per tenant | Tiered, mostly a UX/complexity ceiling | Tiered, same rationale | Keeps roster management usable and bounds per-tenant DB/memory footprint |
| Message/task-history retention window | Tiered | Tiered | Controls long-term Postgres/vector-DB growth on both tracks |

This confirms and extends the existing `03_system_design.md` Section 5 (Auto/Manual model selection) and `tech/11_hallucination_isolation_and_scaling.md` Part C (Office Manager scaling under concurrent jobs) — the limits above are the billing/product-facing surface of the same underlying concurrency and cost-control architecture already specified there.



- Full data isolation between customer accounts — separate rows/schemas keyed by tenant ID at minimum; consider schema-per-tenant if compliance needs grow.

- Per-tenant agent rosters, memory stores, and task queues — one customer's agents/context must never leak into another's.

- Per-tenant usage dashboard: cost so far, tasks run, rate-limit status — needed for both trust and support.

- Admin/ops view (your side): aggregate cost monitoring across all tenants, since managed-platform tenants' model spend is your direct cost. **Implementation:** dedicated `orbicrew-admin` app (separate Next.js deployable), calling privileged operator APIs on `orbicrew-api` — not an `/admin` route inside the tenant `orbicrew-web` dashboard.

## 9. Feature Checklist for a Sellable Product

Carried forward from our earlier discussion — these determine retention and trust, not just the demo appeal.

- **Reliability:** agents that reliably complete tasks with proper retry/error handling.

- **Persistent memory/context:** remembers client, brand guidelines, past decisions without re-explaining.

- **Real output artifacts:** files, deployed apps, rendered video, docs — not just chat replies.

- **Human-in-the-loop approval gates:** for anything client-facing, costly, or irreversible.

- **Multi-tenancy:** isolated per customer (see Section 8).

- **Usage-based cost control:** metering and routing to protect your margin on managed-platform tenants.

- **Integrations:** Gmail, WhatsApp Business, Google Drive, Slack, Trello/Notion, Facebook/Instagram — businesses pay for what plugs into their existing tools.

- **Audit trail/activity log:** "what did my AI employee actually do today."

- **UI switcher (new):** animated office, standard GUI, Telegram, Discord, WhatsApp — user's choice, one backend.

- **User-managed agent roster (new):** create/edit/delete agents without engineering help.

- **System-assisted agent creation (new, advanced):** meta-agent proposes new agent configs from a plain-language request, with human approval before activation.

- **Grounded, non-hallucinating agent output (new):** claims agents make must trace back to real retrieved data (project files, web search), not be fabricated — enforced via retrieval-before-generation, citation tagging, and groundedness checks. Full detail in `tech/11_hallucination_isolation_and_scaling.md`.

- **Per-client/per-project context isolation within one tenant (new):** two client projects run by the same user/tenant must never contaminate each other's data or context, even though both belong to the same tenant — requires a `project` scoping layer beneath tenant-level isolation. Full detail in `tech/11_hallucination_isolation_and_scaling.md`.

- **Adversarial security / prompt injection defense (new):** the system must resist hackers and bad actors attempting to hijack agents via direct, indirect, or stored prompt injection — layered defense (input screening, structural delimiting, privilege reduction, output verification, human approval) mapped to the OWASP Top 10 for Agentic Applications. Full detail in `tech/12_adversarial_security.md`.

- **Auto vs. Manual model selection, per agent (new):** every agent defaults to Auto mode — the router picks the cheapest sufficient model per task, live, task by task. Any agent can instead be pinned to a specific model in Manual mode, overriding Auto entirely for that agent. This is a per-agent setting, not a global switch, so different agents can run different modes simultaneously. Full detail in `tech/03_system_design.md` Section 5.

- **Coordinator (Office Manager) scaling under many simultaneous jobs (new):** the Office Manager must stay accurate and cost-efficient when juggling many (e.g., 100) concurrent jobs at once — achieved via a registry/focus-session pattern where it holds only compact status snapshots by default and pulls a job's full context only when actively working that specific job, structurally preventing one job's details from bleeding into another's even under heavy concurrent load. This also correctly handles resuming/extending a previously-assigned job when referenced later. Full detail in `tech/11_hallucination_isolation_and_scaling.md` Part C.

- **Fast, best-output, cheapest-cost execution at any scale (new):** from a tiny 30-second task to a full enterprise client project, the system must feel fast, produce the best output, and cost as little as possible — achieved through parallel execution, streaming status, right-sized model selection per task, scale-appropriate decomposition (small jobs never pay a tax for the platform's sophistication, large jobs don't get disproportionately expensive), and getting tasks correct on the first pass. Research tasks specifically must stay fast through parallel data gathering, never through skipping real data collection. Full detail in `tech/15_performance_training_and_settings.md` Section 1.

- **Ask-vs-proceed clarification behavior, per agent (new):** users can control whether an agent pauses to ask a question and wait for a reply when it hits an ambiguous decision, or proceeds using its best judgment and states its assumption. Three modes per agent (ask only when it matters, always proceed, always ask), with unattended overnight tasks automatically proceeding rather than stalling, and every assumption made overnight visible in the morning summary. Full detail in `tech/15_performance_training_and_settings.md` Section 5.

- **Agent training (new):** users can improve their agents over time — by default through fast, immediate prompt/memory/feedback-based refinement (thumbs up/down, corrections, "teach me" instructions), with real model-level fine-tuning available as an explicit opt-in advanced tier once there's real production usage data to train on. Full detail in `tech/15_performance_training_and_settings.md` Section 2.

- **Full user control over agent and system settings, with sensible defaults (new):** every setting — model selection mode, budget caps, action whitelists, tool access, verification level, infrastructure risk tier, interface preferences — is visible and editable by the user, ships with a recommended default, and can be set as a tenant-wide default profile that seeds new agents while remaining fully overridable per agent. Full detail in `tech/15_performance_training_and_settings.md` Section 4.

## 10. Key Risks & Mitigations

| **Risk**                                                                                                | **Mitigation**                                                                                                                                                              |
|---------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| LLM cost volatility / runaway agent loops                                                               | Hard per-task and per-tenant budget caps enforced at the router; retry-then-escalate instead of infinite retry; batch APIs for non-realtime work                            |
| Trust/liability from autonomous actions (bad email, bad post published under a client's name)           | Action whitelist + human approval gates for anything external-facing or irreversible, especially in unattended overnight mode                                               |
| Competitive crowding in the global agent-tooling market (Lindy, Relay, CrewAI-based startups, etc.)        | Lead with genuine differentiators: cost transparency, the office-UI experience, and vertical focus on the solo-agency/freelancer/small-team workflow rather than competing head-on generically — see `17_marketing_plan.md` Section 3 for positioning detail |
| Dependency on a third-party open-source project (Agent Town / OpenClaw) whose roadmap you don't control | Fork only the UI/visual layer; keep the actual agent runtime and orchestration on our own LangGraph-based backend behind a similar connector-style boundary                 |
| Multilingual STT/TTS quality gaps (some languages have weaker provider support than others)              | Benchmark Whisper vs. Deepgram/AssemblyAI for STT, and ElevenLabs vs. Google Cloud TTS for TTS, on whichever languages real customers actually use, before committing to a default provider |
| Multi-tenant data leakage                                                                               | Tenant ID isolation at the data layer from day one; encrypted per-tenant key storage for BYO customers; no shared memory/context across tenants                             |
| Agents fabricating facts (hallucination) in client-facing output                                        | Retrieval-before-generation, citation tagging, claim-level groundedness checks on client-facing/factual tasks, human approval gate as final backstop — see `tech/11_hallucination_isolation_and_scaling.md` |
| Two client projects under the same tenant contaminating each other's context (distinct from cross-tenant leakage) | New `project_id` scoping layer beneath tenant, structurally excluding one project's memory/context from another's agent execution — see `tech/11_hallucination_isolation_and_scaling.md` |
| Hackers, malicious users, or adversarial prompt injection (direct, indirect, or stored) exploiting agent tool access | Layered defense (delimiting, privilege reduction, goal-consistency checks, approval gates) mapped to the OWASP Top 10 for Agentic Applications — see `tech/12_adversarial_security.md` |
| Open-source license non-compliance when reselling commercially (e.g., accidentally depending on a paid-only component, or violating a "use vs. resell" distinction like ElevenLabs') | Full tool-by-tool license audit before launch, real legal review of the flagged edge cases (LangGraph Platform, LiteLLM Enterprise, MinIO's AGPL, ElevenLabs resale terms) — see `tech/16_open_source_licensing_compliance.md` |

## 11. Open Source & Tooling Notes (Reference List)

Consolidated list of every open-source project, framework, and paid service discussed in our conversation, for quick reference when we start building.

### 11.1 Orchestration & agents

| **Tool**                          | **Type**             | **Role in our system**                                                                        |
|-----------------------------------|----------------------|-----------------------------------------------------------------------------------------------|
| LangGraph                         | Open source (Python) | Core supervisor/agent orchestration graph — primary recommendation                            |
| CrewAI                            | Open source          | Alternative orchestration framework, considered, not selected as primary                      |
| AutoGen / AG2                     | Open source          | Alternative orchestration framework, considered, not selected as primary                      |
| Agent Town (geezerrrr/agent-town) | Open source, MIT     | Reference/fork candidate for the animated office UI layer only                                |
| OpenClaw                          | Open source          | Agent runtime used by Agent Town; evaluate but do not depend on long-term as our core runtime |
| OpenHands / OpenDevin             | Open source          | Candidate for autonomous coding-agent tasks (repo-level work)                                 |
| Claude Code                       | Anthropic product    | Candidate for coding-agent tasks the office can shell out to                                  |

### 11.2 Model access & routing

| **Tool**               | **Type**                      | **Role in our system**                                                                              |
|------------------------|-------------------------------|-----------------------------------------------------------------------------------------------------|
| LiteLLM                | Open source, self-hosted      | Unified gateway abstracting all model access (managed pooled keys + all three BYO patterns) behind one OpenAI-compatible interface; self-hosted to avoid vendor lock-in. Primary recommendation — see `tech/13_byo_provider_architecture.md`. |
| OpenRouter             | Paid aggregator (usage-based) | One of the providers LiteLLM can route to; also directly usable by BYO customers who bring their own OpenRouter account (Pattern B) |
| Anthropic API (direct) | Paid                          | Provider behind LiteLLM for managed-tier calls and BYO customers using Pattern A (direct platform key) |
| OpenAI API (direct)    | Paid                          | Provider behind LiteLLM for managed-tier calls and BYO customers using Pattern A (direct platform key) |

### 11.3 Voice, speech & avatar

| **Tool**              | **Type**                           | **Role in our system**                                                          |
|-----------------------|------------------------------------|---------------------------------------------------------------------------------|
| Whisper (large-v3)    | Open source                        | Primary STT candidate, broad multilingual coverage — benchmark on real customer languages before committing |
| Deepgram / AssemblyAI | Paid API                           | Hosted STT alternatives to benchmark against Whisper on real customer languages |
| ElevenLabs            | Paid (free tier to test)           | Primary TTS candidate for quality, broad language coverage, has streaming       |
| Google Cloud TTS      | Paid, usage-based                  | Cheaper TTS alternative with broad language/voice coverage                      |
| Coqui TTS / XTTS-v2   | Open source                        | Self-hosted TTS option, voice cloning capable — quality on any given language needs testing |
| Pipecat               | Open source                        | Realtime voice pipeline orchestration (mic -\> STT -\> LLM -\> TTS -\> speaker) |
| LiveKit Agents        | Open source                        | Alternative realtime pipeline orchestration; has avatar plugin ecosystem        |
| Simli / Tavus         | Paid, per-minute                   | Real-time lip-synced talking-head avatar APIs, plug into LiveKit Agents         |
| Live2D (web SDK)      | Commercial license (has free tier) | 2D animated character rendering with lip-sync, no per-minute cost               |

### 11.4 Data, infra & hosting

| **Tool**                            | **Type**                             | **Role in our system**                                                                                      |
|-------------------------------------|--------------------------------------|-------------------------------------------------------------------------------------------------------------|
| Postgres                            | Open source                          | Primary relational data store — self-hosted, no managed-database vendor, per explicit no-vendor-lock-in requirement |
| pgvector                            | Open source (Postgres extension)     | Vector/embedding storage for long-term agent memory, co-located with self-hosted Postgres                    |
| Qdrant                              | Open source (self-host)              | Alternative dedicated vector DB if scale requires separating from Postgres — self-hosted, not the paid cloud tier |
| WAL-G                               | Open source                          | Continuous Postgres backup to object storage — primary backup tool (see `tech/08_devops_and_deployment.md` Section 9) |
| MinIO                               | Open source                          | Self-hosted S3-compatible object storage, avoids dependency on a cloud storage vendor                        |
| Redis                               | Open source                          | Task queue backing store, caching — self-hosted                                                              |
| Cloudflare R2                       | Paid, usage-based (no egress fees)   | Optional alternative to self-hosted MinIO for object storage — S3-API-compatible, easy to migrate away from if preferred over full self-hosting |
| Hetzner                             | Paid VPS                             | Cost-efficient general compute hosting                                                                      |
| Railway / Render                    | Paid, free-tier available            | Early-stage app hosting before dedicated infra is justified                                                 |
| Replicate / Together.ai / Fireworks | Paid, pay-per-use                    | On-demand GPU inference for image/video generation and self-hosted models, avoids owning GPU hardware early |

### 11.5 Frontend & UI

| **Tool**                     | **Type**    | **Role in our system**                                                                                         |
|------------------------------|-------------|----------------------------------------------------------------------------------------------------------------|
| Next.js / React / TypeScript | Open source | Core web app framework — matches Agent Town's stack, good for shared codebase if forking their UI              |
| Phaser 3                     | Open source | 2D game engine for the animated office scene                                                                   |
| Tiled                        | Open source | Map/tileset editor for office scene assets                                                                     |
| Open WebUI                   | Open source | Alternative pre-built multi-model chat UI, candidate for the "standard GUI" mode instead of fully custom build |

### 11.6 Messaging channel integrations

| **Tool**                               | **Type**                     | **Role in our system**                                                                           |
|----------------------------------------|------------------------------|--------------------------------------------------------------------------------------------------|
| Telegram Bot API                       | Free                         | Telegram interface channel                                                                       |
| Discord.js / discord.py                | Open source                  | Discord interface channel                                                                        |
| WhatsApp Business Platform (Cloud API) | Free tier + paid usage tiers | WhatsApp interface channel — high priority for South Asia market, requires business verification |

### 11.7 Research & data tools

| **Tool**                            | **Type**          | **Role in our system**                                      |
|-------------------------------------|-------------------|-------------------------------------------------------------|
| Tavily / Brave Search API / SerpAPI | Paid, usage-based | Web search tool for the research agent                      |
| Custom fetch/scrape tool            | Custom build      | Retrieve and parse full page content for research synthesis |

## 12. Phased Build Plan

Phase 0 — Personal tool (validate for yourself first)

- Postgres + pgvector schema: tasks, agents, clients, memory.

- LangGraph supervisor + 2–3 working specialist agents (e.g., writing, research, coding) with OpenRouter-based model routing.

- Basic chat GUI (skip the game UI initially) with text input; add voice (Whisper + TTS) once core loop works.

- Manual approval step before any external action (sending, publishing).

- Use this yourself daily on real client work before selling anything.

Phase 1 — Overnight autonomy + multi-channel

- Task queue (Redis) + checkpointing for long-running/unattended tasks.

- Budget guard + action whitelist + morning summary report.

- Add Telegram and/or WhatsApp as a second interface, proving the "one backend, many faces" architecture.

- Add research agent with real web search/fetch tools.

Phase 2 — Multi-tenancy & pricing

- Tenant isolation in the data layer; per-tenant usage metering and rate limits.

- Managed-platform billing tiers + BYO-API key management (encrypted storage).

- User-facing agent creation/edit/delete UI (manual, template-based first).

- Pilot with 3–5 real customers from your target segments before broad launch.

Phase 3 — Differentiation & scale

- Animated office UI (fork Agent Town's Phaser layer, wire to our backend) as a premium/differentiated experience alongside the standard GUI.

- Discord bot; deeper integrations (Gmail, Drive, Slack, Notion, Facebook/Instagram).

- Meta-agent for system-assisted agent creation, with human approval gate.

- STT/TTS quality tuning across the languages real customers actually use, based on live usage data — not a Bangla-specific priority, a general "keep multilingual chat quality good in whatever languages people actually speak to their agents" priority.

## 13. Open Questions to Resolve Before Building

- Which specific specialist agents matter most for your own first month of use — this should drive which agents get built first in Phase 0, not a guessed full roster.

- Exact pricing numbers for both tracks — needs competitor benchmarking (Lindy, Relay, and similar) plus your real infra/model cost data from Phase 0 usage.

- WhatsApp Business API verification timeline — start this early since business verification can take time.

- Multilingual STT/TTS provider choice — needs a hands-on benchmark with real audio samples in whichever languages early customers actually use, not a decision from documentation alone, and not scoped to any single language as a priority.

- Final product name selection under the Leangine brand, and real verification — `tech/09_brand_identity.md` Section 0a covers how the product should sit under your existing Leangine company brand (own name, sub-brand, or "[Name] by Leangine"), with a researched product-name shortlist per option (English/global, Bangla-rooted, or Leangine-family) and individual web-search checks already run — but a real WHOIS/registrar `.com` check and a formal trademark search (US at minimum, UK/EU given the global target market) still need to happen for whichever product name is chosen before design work begins against it.

- How much of Agent Town's code is realistically forkable vs. how much is faster to reimplement once your backend protocol is defined — worth a hands-on spike before committing.
